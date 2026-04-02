# Agents 与自动化

## 目的

自动化工具应该像普通 OpenAI 兼容客户端一样调用本地 Veil 网关。
网关只是自动化接入面，产品行为本质上仍由其后的 Relay/Provider 市场角色驱动。
自动化请求中的预算字段属于报价阶段控制，应与结算资产接口语义保持分离。

## 基本规则

把 Veil 当作本地 HTTP 入口：

```text
http://localhost:9960/v1
```

不要让自动化脚本直接连接 Relay 或 Provider 进程。

## 典型接入方式

1. 启动本地网关
2. 把 agent 或脚本的 base URL 指向 `http://localhost:9960/v1`
3. 如果启用了本地 API 鉴权，提供本地 bearer token
4. 从 `GET /v1/models` 返回的模型列表中选择 model id

## 推荐模式

- 业务逻辑保留在客户端侧
- 让 Veil 负责路由、预算和执行
- 通过本地网关读取健康状态和模型可用性

## 不要这样做

- 在自动化里硬编码 Relay 地址
- 假设所有 OpenAI API 面都已经实现
- 在共享环境里绕过本地鉴权

自动化侧需要遵守的市场与结算边界请参考 [治理与经济边界](../product-design/governance-and-economics/README.zh.md)。
