# 定价与风险策略

## 目的

本模块定义 Provider 容量在市场中如何定价、发布、暂停或受限。

## 职责边界

- 定义 Provider 侧定价规则和最低报价边界
- 把运维者策略转换成卖方限制
- 决定容量是否应发布、调价、限流或暂停
- 为见证记录和结算消费方提供确定性的定价快照

## 不负责

- 不执行推理请求
- 不持久化见证记录
- 不负责资金转移或资产托管
- 不替代 Relay 的准入控制

## 接口定义

```ts
type QuoteUnit = 'usd_estimate';

interface CapacityOffer {
  model: string;
  capacityUnits: number;
  askQuoteUnitsPerUnit: number;
  quoteUnit: QuoteUnit;
  settlementAssetHint?: string;
}

interface RiskEnvelope {
  maxDailyLossQuote: number;
  maxConcurrent: number;
  pauseOn429Burst: boolean;
}

interface OfferDecision {
  publish: boolean;
  offers: CapacityOffer[];
  reason?: string;
}

function evaluateOffer(
  inputs: PricingInputs,
  risk: RiskEnvelope,
): OfferDecision;
```

## 数据流

输入：Provider 容量、上游成本信号、健康数据、运维者意图、见证记录历史。  
处理：评估利润空间，应用风险护栏，推导有效报价，并生成定价条款快照。  
输出：市场报价、暂停决定，以及携带报价单位和结算资产提示的确定性定价快照。

## 状态管理

- 持久化：策略配置、定价快照、报价历史
- 内存：当前健康信号、最近的限流事件、冷却状态

## 错误处理

- 暴露模型缺少定价输入
- 策略会导致低于最低边界出售
- 健康数据过期导致无法调价
- 报价快照与见证记录版本不一致
- 已发布报价缺少报价单位或结算资产提示

## 安全约束

- 被结算消费的定价快照必须可确定、可版本化
- 遇到未知健康或成本输入时，风险策略必须默认关闭暴露面
- 报价发布不能绕过运维者设置的护栏
- 报价单位必须与最终结算资产保持显式区分

## 测试要求

- 相同输入下的确定性定价
- 健康退化时的暂停行为
- 成本变化后的重新定价
- 缺少必要定价输入时拒绝发布
- 与见证记录消费方的快照兼容性
- 报价与结算元数据一致性

## 依赖关系

- 调用：`metering-witness`、`provider-engine`、`claw-autopilot`
- 被调用：`provider-engine`、`claw-autopilot`、`settlement-payout`
