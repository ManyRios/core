# Consumer Gateway

## 目的

本模块把 Veil 暴露为本地 OpenAI 兼容网关，供终端用户、工具和 Agent 使用。

## 职责边界

- 提供 OpenAI 兼容 HTTP API
- 认证本地调用方
- 从 Relay 发布的容量中选择 Provider
- 对外发请求进行签名和加密
- 把 Relay 返回结果转换成 JSON 或 SSE
- 执行单请求预算和超时控制

## 不负责

- 不直接执行上游推理
- 不持久化见证记录或贡献账本数据
- 不单独定义底层传输协议语义

## 接口定义

```ts
type RelayMode = 'bootstrap' | 'static' | 'manual';
type QuoteUnit = 'usd_estimate';

interface GatewayOptions {
  port: number;
  wallet: Wallet;
  relayMode?: RelayMode;
  relayUrl?: string;
  bootstrapUrl?: string;
  apiKey?: string;
  defaultQuoteBudget?: number;
  quoteUnit?: QuoteUnit;
}

function startGateway(
  options: GatewayOptions,
): Promise<{ close(): Promise<void>; port: number }>;
```

## 数据流

输入：HTTP `chat/completions` 请求。  
处理：认证、模型校验、Relay 发现或选择、Provider 选择、请求签名、载荷加密、Relay 往返、响应解码。  
输出：OpenAI 兼容响应。

## 状态管理

- 内存：Relay 连接、Provider 列表、pending requests、预算跟踪器
- 持久化：除钱包和本地配置外无新增核心持久化

## 错误处理

- 请求体非法
- 模型不存在
- Relay 不可用
- Provider 不可用
- 流式响应中断
- 在选定模式下没有发现可用 Relay

## 安全约束

- 必须支持本地 API 认证
- 请求签名必须绑定元数据和密文摘要
- 预算检查必须默认收紧而不是放过
- 报价预算必须与最终结算语义分离

## 测试要求

- OpenAI 兼容请求与响应
- 流式链路
- 预算超限与超时
- 无 Provider 路径

## 依赖关系

- 调用：`network`、`crypto`、`wallet`、`budget`
- 被调用：本地 AI 客户端、IDE、Agent
