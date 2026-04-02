# Consumer 配置

## 本地文件

Consumer 运行时读取 `config.json`。

经过校验的字段：

```json
{
  "relay_url": "wss://relay-jp.runveil.io",
  "relay_discovery": "bootstrap",
  "bootstrap_url": "https://bootstrap.runveil.io",
  "gateway_port": 9960,
  "consumer_pubkey": "<64 hex chars>",
  "encryption_pubkey": "<64 hex chars>"
}
```

## 字段语义

- `relay_url`：静态 Relay 兜底地址
- `relay_discovery`：优先使用的 Relay 获取模式
- `bootstrap_url`：当 `relay_discovery` 为 `bootstrap` 时使用的 Bootstrap 地址
- `gateway_port`：必填，本地 HTTP 端口
- `consumer_pubkey`：必填，签名公钥
- `encryption_pubkey`：必填，加密公钥

## 这些配置为何存在

- `relay_discovery`、`relay_url`、`bootstrap_url` 用于在无账户接入前提下保持可路由能力，同时避免把客户端绑定到单一固定运维者。
- 身份相关密钥配置用于维持签名与加密边界，确保路由元数据与提示词明文在角色间保持分离。
- 报价相关字段只用于接入阶段预算控制，必须与结算资产语义保持分离。

## 额外运行时字段

CLI 和网关在运行时也会读取某些可选值：

- `default_quote_budget`
- `quote_unit`

这些值用于细化运行时行为和报价预算，不能被理解成最终结算输出。

## 运行时输入

- `VEIL_HOME` 控制 `config.json` 的加载路径
- `VEIL_PASSWORD` 可绕过交互式密码输入

## API 认证

网关可以通过 `apiKey` 运行时参数启用本地 API key 认证。开启后，调用方必须发送：

```http
Authorization: Bearer <token>
```
