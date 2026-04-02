# Metering 与 Witness

## 目的

本模块把上游用量转换成标准化报价估值和可用于结算的见证记录。

## 职责边界

- 归一化不同上游的用量
- 计算基于模型的成本估算
- 签名并存储 Relay 见证记录
- 支持见证记录查询、统计、清理和导出

## 不负责

- 不负责请求路由
- 不负责 Provider 选择
- 不拥有贡献账务

## 接口定义

```ts
type QuoteUnit = 'usd_estimate';

interface NormalizedUsage {
  input: number;
  output: number;
  cacheRead?: number;
  cacheWrite?: number;
}

interface PricingSnapshot {
  version: string;
  model: string;
  quoteUnit: QuoteUnit;
}

interface QuoteEstimate {
  total: number;
  quoteUnit: QuoteUnit;
}

interface WitnessRecord {
  requestId: string;
  providerId: string;
  relayId: string;
  model: string;
  usage: NormalizedUsage;
  pricingVersion: string;
  quoteUnit: QuoteUnit;
  completionStatus: 'success' | 'error' | 'aborted';
  evidenceHash: string;
  dedupeKey: string;
  relaySignature: string;
}

function normalizeUsage(usage: AnyUsage): NormalizedUsage;
function buildQuoteEstimate(
  usage: NormalizedUsage,
  pricing: PricingSnapshot,
): QuoteEstimate;

class WitnessStore {
  record(...): WitnessRecord;
  verify(...): boolean;
  export(...): WitnessRecord[];
}
```

## 数据流

输入：上游用量对象、Relay 请求元数据、确定性的定价快照。  
处理：归一化用量、生成报价估值、把 pricing version 和 evidence hash 一起签入见证记录载荷，并持久化记录。  
输出：报价估值和可验证的见证记录。

## 状态管理

- 持久化：见证记录数据库记录、定价快照引用
- 内存：单请求用量汇总和报价估值

## 错误处理

- 模型价格未知
- 缺少定价快照版本
- 用量载荷非法
- 见证记录签名无效
- 证据哈希不匹配
- request id 重复

## 安全约束

- 见证记录签名必须覆盖确定性的载荷
- 见证记录签名必须覆盖 pricing version、quote unit 和 evidence hash
- 导出的见证记录必须保持可验证性
- 未知价格下的报价估算必须保守

## 测试要求

- 多供应商用量归一化
- 报价估值生成
- 见证记录的记录、查询、统计、清理、导出
- 签名校验
- pricing version 不匹配时拒绝消费

## 依赖关系

- 调用：`crypto`
- 被调用：`consumer`、`relay`、`operations`、`settlement-payout`
