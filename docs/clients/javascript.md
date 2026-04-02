# JavaScript

## Using `fetch`

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

## OpenAI SDK Style

If your client supports overriding the base URL, point it at:

```text
http://localhost:9960/v1
```

Use your local Veil token as the API key when local auth is enabled.
