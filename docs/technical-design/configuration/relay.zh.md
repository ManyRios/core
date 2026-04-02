# Relay 配置

## 主要运行时输入

当前 Relay 配置由运行时参数和环境变量共同组成。

主要运行时参数：

- `port`
- `wallet`
- `dbPath`
- `bootstrapUrl`
- `witnessDbPath`

## 这些配置为何存在

- Relay 运行时参数用于保持 Relay 的显式市场中介与见证角色，而不是退化为纯传输跳点。
- `dbPath` 与 `witnessDbPath` 用于保留可重放、可复核的结算证据基础。
- 限流与 region 控制在维持准入边界的同时，支持可发现的市场运行行为。

## 环境变量

- `VEIL_RELAY_PORT`：覆盖默认 Relay 端口
- `VEIL_RELAY_RATE_LIMIT`：请求限流默认值
- `VEIL_RELAY_REGION`：发布到 Bootstrap 的 region 值

## 持久化状态

Relay 会写入：

- `relay.db`
- `witness.db`

## Bootstrap 注册

当配置了 `bootstrapUrl` 时，Relay 可以发布：

- endpoint
- 支持模型
- fee
- region
- 容量
- 活跃请求状态

## 运维说明

Relay 的配置当前仍有一部分由代码决定，因此本节需要与 `startRelay(...)` 以及为其供参的 CLI 行为保持同步。
