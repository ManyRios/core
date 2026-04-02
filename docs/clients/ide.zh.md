# IDE 与工具接入

## Base URL

对 OpenAI 兼容的 IDE 和工具，请使用：

```text
http://localhost:9960/v1
```

## 典型流程

1. 启动本地网关
2. 在 IDE 中配置 base URL
3. 如果启用了本地认证，填入本地 token
4. 选择 Veil 暴露出的模型 id

## 兼容目标

客户端侧的目标是让 Veil 看起来像一个本地 AI provider，而不是一个自定义协议端点。
这个兼容接入面连接的是 Veil 的市场网络，底层仍由 Relay 与 Provider 角色共同定义路由和供给行为。

如需查看这层本地接入面背后的角色边界，请参考 [系统模型](../technical-design/system-model/README.zh.md)。
