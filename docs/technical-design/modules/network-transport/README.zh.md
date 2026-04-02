# Network Transport

## 目的

本模块提供 Consumer、Relay 和 Provider 共享的连接层。
尽管本模块只负责传输层，它仍直接支撑 `Access` 与 `Automation` 结果：保障网关连通性与低人工运维场景下的稳定运行。

## 职责边界

- 创建 WebSocket 客户端和服务端
- 序列化并发送协议消息
- 用 ping 和 pong 检测存活
- 按策略执行重连

## 不负责

- 不定义业务路由策略
- 不校验应用层签名
- 不存储持久化运行记录

## 接口定义

```ts
interface Connection {
  send(msg: WsMessage): void;
  close(): void;
  readonly readyState: 'connecting' | 'open' | 'closing' | 'closed';
}

function connect(options: ConnectionOptions): Promise<Connection>;
function createServer(
  options: { port: number; onConnection: (...) => void },
): ServerHandle;
```

## 数据流

输入：`WsMessage` 对象。  
处理：JSON 序列化、socket 收发、重连、心跳。  
输出：消息回调与连接回调。

## 状态管理

- 内存：WebSocket 对象、重连计数、ping 定时器、pong 超时
- 持久化：无

## 错误处理

- 首次连接失败
- 非法 JSON 帧
- pong 超时
- 已关闭 socket 上的发送

## 安全约束

- 限制消息大小
- 快速剔除失活连接
- 不把关闭连接上的发送当作成功

## 测试要求

- 连接与重连
- 心跳超时
- 非法帧处理
- 最大载荷行为

## 依赖关系

- 调用：`config`、`logger`
- 被调用：`consumer`、`provider`、`relay`
