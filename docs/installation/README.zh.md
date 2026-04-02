# 安装

## 目的

本节说明如何从源码安装 Veil，并准备首次运行所需的本地目录。

## 适用场景

- 你第一次搭建 Veil
- 你需要明确运行前提条件
- 你想在启动任何角色前先了解默认本地文件布局

## 环境要求

- Node.js `22+`
- `npm`
- 当你运行 Provider 时，需要具备访问上游模型 API 的网络能力
- 可写的本地目录，或自定义 `VEIL_HOME`

## 从源码安装

```bash
git clone https://github.com/runveil-io/core.git
cd core
npm install
npm test
```

## 本地运行目录

默认情况下，Veil 会把本地运行文件放在 `~/.veil` 下。

常见文件包括：

- `wallet.json`
- `config.json`
- `provider.json`
- `data/*.db`

你也可以通过下面的环境变量覆盖默认目录：

```bash
export VEIL_HOME=/path/to/veil-home
```

## 首次初始化

初始化钱包：

```bash
veil init
```

这一步会创建：

- 签名密钥对
- 加密密钥对
- 本地网关和 Relay 连接配置

## 作为 Consumer 运行

把本地网关接给 OpenAI 兼容客户端：

```bash
veil init
# 将你的客户端指向 http://localhost:9960/v1
```

默认网关端口：`9960`

## 作为 Provider 运行

配置并启动 Provider：

```bash
veil provide init
veil provide start
```

默认 Provider 健康检查端口：`9962`

## 作为 Relay 运行

启动 Relay 节点：

```bash
veil relay start
```

默认 Relay 端口：`8080`

## 验证

运行测试集：

```bash
npm test
```

更多运行方式请继续阅读 [操作手册](../manual/README.zh.md)。
