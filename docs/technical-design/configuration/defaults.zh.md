# 默认值

## 核心默认值

以下默认值来自 `src/config/bootstrap.ts` 和相关运行时模块：

| 键 | 默认值 |
|-----|---------|
| 当前官方 Relay 默认 URL | `wss://relay-jp.runveil.io` |
| Bootstrap URL | `https://bootstrap.runveil.io` |
| 网关端口 | `9960` |
| Relay 端口 | `8080` |
| Provider 健康检查端口 | `9962` |
| 请求最大年龄 | `300000 ms` |
| Ping 间隔 | `30000 ms` |
| Pong 超时 | `10000 ms` |
| 最大 WebSocket 载荷 | `10 MB` |
| 发现缓存 TTL | `60000 ms` |
| 发现最大 Relay 数 | `20` |
| 默认请求报价预算 | `1.0 USDC` |
| 默认请求时长 | `120000 ms` |
| Relay 限流默认值 | `60` |

## 解释说明

- 当前官方 Relay URL 属于运行时默认值和上手便利项，不代表长期单运维者架构假设
- 默认请求报价预算用于预算保护和价格比较，表达的是当前报价语言，不等于最终结算资产

## 重试默认值

Provider 重试参数：

| 键 | 默认值 |
|-----|---------|
| maxRetries | `3` |
| baseDelayMs | `1000` |
| maxDelayMs | `30000` |
| jitterFactor | `0.5` |

## 内置模型

当前网关对外暴露的模型 id：

- `claude-sonnet-4-20250514`
- `claude-haiku-3-5-20241022`
