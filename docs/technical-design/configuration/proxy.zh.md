# Proxy 配置

## 目的

本地 proxy 是一个可选进程，用来把上游 API key 隔离在 Veil 主运行时之外。

它的信任边界很窄：

- 只绑定 `127.0.0.1`
- 每个请求都要求共享密钥
- 上游 API key 只保留在 proxy 进程内存里

## 运行时输入

- `ANTHROPIC_API_KEY`：必需的上游凭证
- `PROXY_SECRET`：通过 `x-proxy-secret` 校验的共享密钥
- `PROXY_PORT`：本地监听端口，默认 `4000`
- `UPSTREAM_URL`：上游基础 URL，默认 `https://api.anthropic.com`

## 交互方式

1. 在本机启动 proxy
2. 捕获或预先配置 `PROXY_SECRET`
3. 让 Provider 指向 `PROXY_URL`
4. 通过 proxy 转发 `/v1/*` 流量，而不是把 API key 直接交给 Provider 进程

## 范围

proxy 是一个本地隔离边界，不是公开 API，不应该暴露到外部网络接口。
