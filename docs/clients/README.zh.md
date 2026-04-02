# 客户端接入

## 目的

本节说明调用方如何通过本地网关集成 Veil。

当前 Veil 对客户端暴露的入口很窄：

- `GET /health`
- `GET /v1/models`
- `POST /v1/chat/completions`

## 阅读顺序

1. [openai-compatible.md](./openai-compatible.zh.md)
2. [curl.md](./curl.zh.md)
3. [javascript.md](./javascript.zh.md)
4. [ide.md](./ide.zh.md)
5. [agents.md](./agents.zh.md)
6. [models.md](./models.zh.md)

## 主要规则

客户端应该把 Veil 当作本地 OpenAI 兼容网关，而不是直接对接 Relay 或 Provider API。
这个本地网关只是 Relay/Provider 市场网络的接入面，不是独立的代理型产品定义。

## 网络上下文

- 本地网关是接入入口，市场行为由 Relay 准入、Provider 执行和见证记录链路共同定义。
- 客户端使用的报价预算属于路由阶段控制，不替代结算接口语义。
- 角色边界与请求归属请参考 [系统模型](../technical-design/system-model/README.zh.md)。
- 经济与结算边界请参考 [治理与经济边界](../product-design/governance-and-economics/README.zh.md)。
