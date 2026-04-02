# Provider 配置

## 本地文件

Provider 运行时读取 `provider.json`。

经过校验的字段：

```json
{
  "version": 1,
  "models": ["claude-sonnet-4-20250514"],
  "api_keys": [
    {
      "provider": "anthropic",
      "salt": "...",
      "iv": "...",
      "ciphertext": "...",
      "tag": "..."
    }
  ],
  "max_concurrent": 5
}
```

## 字段语义

- `version`：当前必须为 `1`
- `models`：非空的模型 id 列表
- `api_keys`：加密后的上游凭证
- `max_concurrent`：可选，正整数

## 这些配置为何存在

- Provider 配置把卖方意图转化为可执行的市场供给，而不是后台隐性能力。
- 加密凭证与并发限制用于把执行行为保持在明确的安全与风险边界内。
- 发现与 Relay 端点相关配置用于维持市场可达性，并为自动化路径上的策略化发布保留空间。

## 运行时输入

Provider 路径还会读取：

- `relay_mode`：节点使用 Bootstrap 发现、静态 Relay 端点或运维者提供的 Relay 列表
- `bootstrap_url`：启用发现时使用的 Bootstrap 地址
- `relay_url` 或 `relay_urls`：静态兜底 Relay 端点
- `ANTHROPIC_API_KEY`：当本地解密凭证不可用时的兜底凭证
- `PROXY_URL`：可选的上游代理 URL
- `PROXY_SECRET`：可选的代理共享密钥
- `VEIL_PROVIDER_HEALTH_PORT`：覆盖默认健康检查端口

## 模型映射

Provider 在调用上游 AI API 之前，会通过 `MODEL_MAP` 把对外模型 id 映射成上游模型名。
