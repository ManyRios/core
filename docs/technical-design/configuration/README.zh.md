# 配置

## 目的

本节说明 Veil 的运行时配置入口。

当前 Veil 的配置主要来自四个地方：

- 本地文件，例如 `config.json` 和 `provider.json`
- 钱包初始化时生成的默认值
- 环境变量
- 少量运行时 CLI 参数

## 阅读顺序

1. [defaults.md](./defaults.zh.md)
2. [consumer.md](./consumer.zh.md)
3. [provider.md](./provider.zh.md)
4. [proxy.md](./proxy.zh.md)
5. [relay.md](./relay.zh.md)
6. [bootstrap.md](./bootstrap.zh.md)
7. [environment.md](./environment.zh.md)

## 主要配置对象

- Consumer 配置：本地 `config.json`
- Provider 配置：本地 `provider.json`
- Proxy 配置：本地回环端口、上游 URL 和共享密钥
- Relay 配置：端口、数据库路径、限流和发现相关环境变量
- Bootstrap 配置：bootstrap URL、缓存 TTL、Relay 选择上限

## 范围

本节只记录当前代码实际读取和校验的配置项。

## 愿景护栏

配置项演进应持续满足产品愿景约束：

- 保持 Relay 的显式市场角色，而不退化为纯传输跳点
- 保留 Claw 或等价 agent 自动化的主线接入能力
- 持续区分报价预算语义与最终结算资产语义
