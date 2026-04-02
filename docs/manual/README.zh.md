# 操作手册

## 目的

本节说明安装完成之后如何运行和使用 Veil。

本页描述的是当前以 CLI 为中心的运行方式。Veil 的目标运行模型仍然是由 Claw 托管接入、定价、发布和恢复流程，把人工步骤压到更低。

## 按角色使用

### Consumer

- 用 `veil init` 初始化钱包
- 将 OpenAI 兼容客户端指向 `http://localhost:9960/v1`
- 按需配置本地 API 认证

### Provider

- 用 `veil provide init` 配置凭证
- 用 `veil provide start` 启动 Provider
- 关注健康状态和上游失败
- 这代表当前的人工运行路径，不是最终的运维体验

### Relay

- 用 `veil relay start` 启动 Relay
- 观察 Provider 注册和请求转发
- 按需导出和检查见证记录数据

## 运行检查清单

### 启动前

- 钱包已经创建
- 配置文件存在
- 目标端口未被占用
- Provider 节点的上游凭证有效

### 运行中

- 检查 `health` 接口
- 观察日志中的 Relay 断连、Provider 错误或认证失败
- 确认见证记录数据仍在持续写入

### 排障时

- 运行 `npm test`
- 运行定向 Vitest 用例
- 检查 `VEIL_HOME` 或 `~/.veil` 下的本地运行文件

## 常用命令

```bash
veil init
veil provide init
veil provide start
veil relay start
npm test
npx vitest run --reporter=verbose
```

## 目标运维路径

目标运维模型会把以上手工步骤收敛成：

- 运维者表达意图
- 设置策略护栏
- 批准凭证使用
- 由 Claw 托管接入、卖量、暂停与恢复流程

## 下一步阅读

- [运维与运行时](../operations/README.zh.md)
- [协议设计](../technical-design/protocol/README.zh.md)
- [模块设计](../technical-design/modules/README.zh.md)
- [Claw Autopilot](../technical-design/modules/claw-autopilot/README.zh.md)
