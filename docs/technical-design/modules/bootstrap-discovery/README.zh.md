# Bootstrap 与 Discovery

## 目的

本模块定义 Relay 节点如何被注册、刷新、列出和选择。
它通过把中介可用性与选择过程显式化，保障 Relay 作为市场角色而非隐形基础设施。

## 职责边界

- 注册 Relay 节点
- 接收 Relay 心跳
- 发布 Relay 列表
- 在客户端侧对 Relay 进行打分和选择

## 不负责

- 不转发推理流量
- 不解密业务载荷
- 不存储 Consumer 提示词明文

## 接口定义

```ts
function createBootstrapApp(db: Database): Hono;

class RelayDiscoveryClient {
  fetchRelays(region?: string): Promise<RelayInfo[]>;
  selectRelay(exclude?: string[]): Promise<RelayScore | null>;
  refreshCache(): Promise<RelayInfo[]>;
}
```

## 数据流

输入：Relay 注册与心跳请求、Consumer 和 Provider 的列表拉取。  
处理：校验签名、更新注册表状态、缓存 Relay 列表、计算打分。  
输出：活跃 Relay 清单和最佳 Relay 选择结果。

## 状态管理

- 持久化：`relay_registry`
- 内存：缓存的 Relay 列表、时延测量结果

## 错误处理

- 签名错误
- endpoint 重复
- Relay 状态陈旧
- bootstrap 拉取失败

## 安全约束

- Bootstrap 只处理 Relay 元数据
- Relay 注册必须带签名
- 选择输入必须有边界且可缓存

## 测试要求

- 注册、心跳、离线清理
- 列表缓存行为
- 打分计算与选择

## 依赖关系

- 调用：`crypto`、`logger`
- 被调用：`consumer`、`provider`、Relay 运维
