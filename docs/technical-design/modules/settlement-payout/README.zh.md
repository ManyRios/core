# 结算与收益分发

## 目的

本模块定义如何把基于见证记录的用量证据和确定性报价快照转换成可结算的账务记录。

## 职责边界

- 消费见证记录和定价快照
- 推导确定性的结算条目
- 生成 Provider 和 Relay 角色可支付的余额
- 提供可导出的收益分发指令，同时不与贡献账务混账

## 不负责

- 不负责流量路由
- 不在没有见证记录证据时擅自生成用量
- 不拥有贡献 grant
- 不强制绑定某一种支付轨实现

## 接口定义

```ts
type QuoteUnit = 'usd_estimate';

interface SettlementEntry {
  requestId: string;
  pricingVersion: string;
  quoteUnit: QuoteUnit;
  providerQuoteAmount: number;
  relayQuoteAmount: number;
  feesQuoteAmount: number;
  settlementAsset?: string;
  evidenceHash: string;
  dedupeKey: string;
  status: 'quoted' | 'ready' | 'exported';
}

interface PayoutInstruction {
  recipient: string;
  quoteAmount: number;
  settlementAsset: string;
  reason: 'provider' | 'relay';
}

function settleWitness(
  witness: WitnessRecord,
  pricing: PricingSnapshot,
): SettlementEntry;

function buildPayouts(
  entries: SettlementEntry[],
  settlementAsset: string,
): PayoutInstruction[];
```

## 数据流

输入：见证记录、定价快照、角色身份、收益分发窗口、结算资产映射。  
处理：校验证据，计算结算条目，聚合可支付余额，解析结算资产，导出收益分发指令。  
输出：结算账本条目、余额、可直接用于收益分发的导出结果。

## 状态管理

- 持久化：结算账本、收益分发批次、导出历史
- 内存：当前聚合窗口、校验缓存

## 错误处理

- 请求的结算缺少见证记录
- 缺少定价快照版本
- 见证记录签名无效
- 重复的结算条目
- 导出时缺少结算资产映射
- 收益分发导出不一致

## 安全约束

- 结算绝不能信任不透明的旁路用量数据
- 收益分发指令必须能从见证记录和定价输入中重现
- 贡献账务与市场结算必须保持独立账本
- 报价金额与结算资产必须保持显式区分

## 测试要求

- 确定性的结算条目生成
- 重复保护
- 无效见证记录拒绝
- 按角色聚合收益分发
- 导出结果可重现
- 报价到结算导出的一致性

## 依赖关系

- 调用：`metering-witness`、`pricing-risk-policy`
- 被调用：`cli`、`claw-autopilot`
