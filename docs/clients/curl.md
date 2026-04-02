# cURL Examples

## Health Check

```bash
curl http://localhost:9960/health
```

## List Models

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

## Streaming

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
