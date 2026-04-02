# Claw Autopilot

## 目的

本模块定义目标自动化边界，用来把运维者意图转换成低人工干预的节点运营流程。

## 职责边界

- 接收接入网络和出售容量等运维意图
- 在节点暴露到网络前校验本地前置条件
- 编排钱包、发现、Provider 和 Relay 的启动流程
- 协调定价策略、健康状态、暂停和恢复行为
- 持久化运维者意图和自动化状态

## 不负责

- 不直接代理推理流量
- 不解密请求载荷
- 不替代 Relay 的准入逻辑
- 不在没有见证记录输入时擅自生成结算结果

## 接口定义

```ts
interface OperatorIntent {
  mode: 'join-network' | 'sell-capacity' | 'pause-market';
  providerMode: 'api-capacity' | 'self-hosted';
  relayMode: 'bootstrap' | 'static' | 'manual';
}

interface AutopilotPolicy {
  maxDailyLossQuote: number;
  minMarginPct: number;
  autoPauseOnHealthDegrade: boolean;
}

interface AutopilotRuntime {
  close(): Promise<void>;
  reconcile(): Promise<void>;
}

function startAutopilot(
  intent: OperatorIntent,
  policy: AutopilotPolicy,
): Promise<AutopilotRuntime>;
```

## 数据流

输入：运维者意图、本地凭证、发现结果、Provider 健康状态、见证记录和定价信号。  
处理：校验输入，规划期望运行状态，启动或重配置节点，监控偏移并持续协调。  
输出：运行中节点状态、策略动作、暂停或恢复决定、自动化日志。

## 状态管理

- 持久化：运维者意图、策略快照、最近一次 Relay 选择、最近一次 Provider 报价状态
- 内存：协调循环状态、健康快照、待执行动作、冷却计时器

## 错误处理

- 选定 Provider 模式缺少必要凭证
- 没有可达的 Relay 候选
- 策略冲突导致容量无法安全暴露
- 启动或协调动作失败
- 反复健康退化导致被强制暂停

## 安全约束

- 运维者凭证绝不能越过本地秘密边界
- 当策略或健康信号不明确时，autopilot 必须默认关闭暴露面
- 自动化层不能绕过 Relay、Provider 或结算系统的安全检查

## 测试要求

- 满足前置条件时的接入流程
- 应用策略后的卖量流程
- 健康退化触发的暂停与恢复
- Relay 丢失后的恢复
- 凭证或策略条件缺失时拒绝启动

## 依赖关系

- 调用：`wallet-identity`、`bootstrap-discovery`、`provider-engine`、`relay`、`pricing-risk-policy`、`settlement-payout`
- 被调用：`cli`
