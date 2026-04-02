# Bootstrap 配置

## 目的

Bootstrap 是 Relay 发现服务。

## 默认值

- bootstrap URL：`https://bootstrap.runveil.io`
- 发现缓存 TTL：`60000 ms`
- 发现最大 Relay 数：`20`

## Discovery Client 输入

`RelayDiscoveryClient` 接收：

- `bootstrapUrl`
- `cacheTtlMs`
- `maxRelays`

## 选择输入

当前 Relay 选择打分使用：

- latency
- fee
- uptime
- reputation
- exploration bonus

## API 面

Bootstrap 暴露：

- `POST /v1/relays/register`
- `POST /v1/relays/heartbeat`
- `GET /v1/relays`
- `DELETE /v1/relays/:pubkey`
- `GET /health`
