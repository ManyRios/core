# Relay

## 目的

本模块是 Consumer 与 Provider 之间的控制平面中介节点。

## 职责边界

- 接收 Provider 注册
- 校验 Consumer 请求
- 应用限流
- 把请求路由到在线 Provider
- 建立 request id 到 Consumer 的映射
- 在请求完成时记录见证记录

Relay 同时承担路由控制节点与市场见证角色，其记录必须持续满足“报价到结算”语义分离约束。

## 不负责

- 不解密密封业务载荷
- 不作为公开客户端 API 网关
- 不拥有贡献账务

## 接口定义

```ts
interface RelayOptions {
  port: number;
  wallet: Wallet;
  dbPath: string;
  bootstrapUrl?: string;
  witnessDbPath?: string;
}

function startRelay(options: RelayOptions): Promise<{ close(): Promise<void> }>;
function verifyRequest(...): boolean;
function createWitness(...): RelayWitness;
```

## 数据流

输入：Provider 的 `provider_hello`、Consumer 的 `request`、Provider 的 `response` 和 `stream_end`。  
处理：验签、限流、查找 Provider、转发、持久化见证记录。  
输出：被转发的协议消息和见证记录。

## 状态管理

- 持久化：`provider_state`、`usage_log`、`witness`、独立 `witness.db`
- 内存：在线 Provider 映射、请求元信息映射、Consumer 连接映射、限流器

## 错误处理

- Provider 签名错误
- Consumer 签名错误
- 限流拒绝
- Provider 缺失
- 重复见证记录插入

## 安全约束

- Relay 不应解密业务载荷
- 必须检查请求年龄
- 见证记录必须可签名、可导出

## 测试要求

- Provider 注册
- Consumer 验签
- 限流
- 见证记录与校验

## 依赖关系

- 调用：`network`、`crypto`、`db`、`witness`、`rate_limiter`
- 被调用：`consumer`、`provider`
