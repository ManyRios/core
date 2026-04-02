# 模型

## Gateway 对外暴露的模型 ID

当前网关对客户端暴露的模型 id：

- `claude-sonnet-4-20250514`
- `claude-haiku-3-5-20241022`

## 事实来源

网关对外暴露的模型列表来自 `src/config/bootstrap.ts` 中的运行时默认值。

客户端应把 `GET /v1/models` 当作运行时事实来源。
