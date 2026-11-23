# 烟韵四海 yanyunsihai
烟台文旅特产DApp商城｜Pi Network 黑客松项目

## 项目简介
烟韵四海是面向Pi生态的文旅特产商城DApp，主打烟台本地特产、文旅礼品，支持Pi支付。
本项目规划未来接入Pi测试网、主网。
当前阶段：Pi官方合约对接规范暂未开放，先采用链下方案完成沙盒全流程测试。

## 当前可实现能力
- 商品浏览、搜索、分类展示、多图卡片
- 购物车增减、总价计算
- Pi SDK沙盒支付
- 订单管理：待支付 / 已支付 / 已发货 / 已退款
- 链下库存管理、链下退款流程
- 多语言适配，兼容Pi Browser

## 后续预留（等待Pi官方合约接口开放后接入）
- 订单状态上链
- 库存锁定链上逻辑
- 链上退款
- 订单超时自动释放库存

## 技术栈
前端：React + TypeScript
状态管理：Jotai
支付：Pi SDK 2.0（沙盒模式）
后端：自研服务（链下订单、库存、退款）
静态资源：IPFS（商品图片、评价内容）

## 开发说明
现阶段不部署自定义智能合约，所有订单、退款、库存逻辑在后端链下实现，全部预留接口，后续平滑迁移至链上。
# 烟韵四海 - 技术方案与边界说明
> 阶段：沙盒演示阶段，目标：跑通完整业务闭环，等待Pi官方合约规范

### 核心功能清单
1. 商品管理：上架、下架、设置库存、Pi定价、多图展示
2. 购物车：添加商品、修改数量、实时总价计算
3. Pi支付：调用Pi SDK沙盒支付，接收支付回调
4. 订单追踪：待支付、已支付、已发货、已退款
5. 退款：链下退款流程（当前无合约端口，暂不链上退款）
6. 附加：限时活动、库存预警、多语言切换

### 技术边界（重点）
本应用长期规划接入Pi测试网、主网。
现阶段Pi官方暂未开放配套合约规范与合约对接端口，因此：
✅ 可用：Pi SDK支付沙盒能力
✅ 实现：后端链下完成订单、库存、退款业务
✅ 预留：订单上链、链上退款、链上库存锁定接口点位
❌ 暂不开发：智能合约代码

沙盒阶段闭环路径：
浏览商品 → 加入购物车 → 下单 → Pi SDK支付 → 回调更新订单 → 订单管理/发货/链下退款

### 安全预留点
未来接入Pi合约时，支付确认接口仅接受Pi官方回调签名校验，不对普通用户开放调用。
# 烟韵四海 DApp UI结构
├── 主页 Home
│   ├── 搜索栏（关键词、商品分类）
│   ├── 商品瀑布流卡片
│   │   ├── 商品图、名称、Pi价格
│   │   └── 加入购物车按钮
│   └── 轮播促销横幅
├── 购物车 Cart
│   ├── 商品列表（图片、数量、单价）
│   ├── 总价展示
│   ├── 增减数量、删除商品
│   └── Pi支付按钮
├── 订单页 Orders
│   ├── 状态筛选：全部 / 待支付 / 已支付 / 已发货 / 已退款
│   ├── 订单卡片：订单号、商品缩略图、状态
│   └── 确认收货
└── 个人中心 Profile
    ├── Pi账户信息
    ├── 语言切换
    ├── 客服入口
// Pi SDK 沙盒支付示例 【链下版本，无合约】
import { Pi } from '@pinetwork-js/sdk';

// 初始化SDK，沙盒模式
Pi.init({
  version: '2.0',
  sandbox: true
});

/**
 * 发起Pi支付
 * @param {string} orderId 后端生成的订单ID
 * @param {number} amount 支付金额
 */
async function handlePiPayment(orderId, amount) {
  try {
    const payment = await Pi.createPayment({
      amount: amount,
      memo: `烟韵四海订单 #${orderId}`,
      metadata: { orderId: orderId }
    });

    // 提交支付给Pi网络
    await payment.submit();
    console.log("支付已提交，等待Pi回调");

    // 等待支付状态更新
    await payment.waitForStatusUpdate((status) => {
      if (status.txid) {
        // 支付成功，通知后端更新订单状态
        notifyBackendOrderPaid(orderId, status.txid);
        return true;
      }
    });
  } catch (err) {
    console.error("支付失败：", err);
  }
}

// 通知后端：订单支付成功
function notifyBackendOrderPaid(orderId, txid) {
  fetch("/api/order/pay-success", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ orderId, txid })
  });
}

export default handlePiPayment;
