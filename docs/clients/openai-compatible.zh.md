# OpenAI 兼容接口

## 已支持的端点

当前本地网关暴露：

- `GET /health`
- `GET /v1/models`
- `POST /v1/chat/completions`

## 请求结构

当前网关支持常见的 chat-completions 字段：

- `model`
- `messages`
- `stream`
- `temperature`
- `max_tokens`
- `top_p`
- `stop`

运行时还支持一个 Veil 扩展字段：

```json
{
  "budget": {
    "max_cost_usdc": 1.0,
    "max_duration_ms": 120000
  }
}
```

`max_cost_usdc` 当前用于路由与比较阶段的报价预算，不等同于最终结算资产语义。

## 认证

当启用本地 API 认证时，调用方必须发送：

```http
Authorization: Bearer <token>
```

## 错误语义

当前网关错误结构使用 OpenAI 风格的 envelope：

```json
{
  "error": {
    "message": "No providers available",
    "type": "api_error",
    "code": "no_providers"
  }
}
```

## 当前未支持的能力

当前网关还不暴露：

- embeddings
- responses API
- assistants
- images
- file APIs
