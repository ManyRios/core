# Provider Engine

## 目的

本模块拥有执行边界，在这里密封请求会被转换成真实的上游 AI 调用。
Provider 执行边界属于 `Market` 主路径：只有可按策略执行的容量才是真实供给，并且必须保持可用于后续 `Settlement` 的证据连续性。

## 职责边界

- 保持一个或多个 Relay 连接
- 解密密封请求
- 调用上游 AI 服务
- 返回普通或流式响应
- 执行本地并发与凭证边界控制

## 不负责

- 不决定全局路由策略
- 不持久化 Relay 见证记录
- 不暴露面向客户端的公开 API

## 接口定义

```ts
type RelayMode = 'bootstrap' | 'static' | 'manual';

interface ProviderOptions {
  wallet: Wallet;
  relayMode?: RelayMode;
  relayUrls?: string[];
  bootstrapUrl?: string;
  apiKeys: Array<{ provider: 'anthropic'; key: string }>;
  maxConcurrent: number;
}

function startProvider(options: ProviderOptions): Promise<{ close(): Promise<void> }>;
function handleRequest(...): Promise<HandleRequestResult>;
```

## 数据流

输入：Relay 转发的 `request`。  
处理：通过发现结果或显式配置维持一个或多个 Relay 连接，解密内层载荷、构造上游请求、调用模型 API、流式或聚合输出。  
输出：`response`、`stream_start`、`stream_chunk`、`stream_end` 或 `error`。

## 状态管理

- 持久化：`provider.json`、加密后的 API key
- 内存：活跃请求数、Relay 连接、指标存储

## 错误处理

- 容量达到上限
- 上游认证失败
- 上游 429 和 5xx
- 流式 body 非法
- 在选定模式下没有可达的 Relay

## 安全约束

- 上游凭证必须加密落盘
- Provider 不应接收不必要的 Consumer 身份信息
- 明文必须限制在执行边界内

## 测试要求

- 非流式请求
- 流式请求
- 重试行为
- 最大并发
- 多 Relay 连接

## 依赖关系

- 调用：`network`、`crypto`、`metrics`
- 被调用：`relay`
