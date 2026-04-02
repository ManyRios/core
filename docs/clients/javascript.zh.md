# JavaScript

## 使用 `fetch`

```js
const res = await fetch('http://localhost:9960/v1/chat/completions', {
  method: 'POST',
  headers: {
    'content-type': 'application/json',
    'authorization': 'Bearer YOUR_LOCAL_TOKEN',
  },
  body: JSON.stringify({
    model: 'claude-sonnet-4-20250514',
    messages: [{ role: 'user', content: 'Hello from JavaScript.' }],
  }),
});

const data = await res.json();
console.log(data);
```

## OpenAI SDK 风格

如果你的客户端支持覆盖 base URL，请将其指向：

```text
http://localhost:9960/v1
```

当启用了本地认证时，请把本地 Veil token 当作 API key 使用。
