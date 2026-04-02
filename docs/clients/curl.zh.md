# cURL 示例

## 健康检查

```bash
curl http://localhost:9960/health
```

## 模型列表

```bash
curl http://localhost:9960/v1/models \
  -H "Authorization: Bearer YOUR_LOCAL_TOKEN"
```

## Chat Completion

```bash
curl http://localhost:9960/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_LOCAL_TOKEN" \
  -d '{
    "model": "claude-sonnet-4-20250514",
    "messages": [
      { "role": "user", "content": "Say hello." }
    ]
  }'
```

## 流式请求

```bash
curl http://localhost:9960/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_LOCAL_TOKEN" \
  -d '{
    "model": "claude-sonnet-4-20250514",
    "stream": true,
    "messages": [
      { "role": "user", "content": "Stream a short answer." }
    ]
  }'
```
