# 运维与运行时

## 目的

本节说明 Veil 作为运行中系统时的行为，包括端口、状态、持久化、可观测性和发布门槛。

## 适用场景

- 你在运行 Consumer、Provider、Relay 或 Bootstrap
- 你需要区分哪些状态会持久化，哪些只存在于内存
- 你要判断某个运行时改动是否已经具备发布条件

## 运行模式

- Consumer 网关运行在 `9960`
- Provider 健康检查端口为 `9962`
- Relay 运行在 `8080`
- Bootstrap 作为 HTTP 注册服务

## 主要持久化数据

- Consumer：`wallet.json`、`config.json`
- Provider：`wallet.json`、`provider.json`
- Relay：`relay.db`、`witness.db`
- Bootstrap：Relay 注册表数据库
- Governance：RBOB 账本数据库

## 运行时状态

- Consumer：待处理请求、Provider 列表、Relay 连接
- Provider：活跃请求计数、指标、Relay 连接
- Relay：Provider 映射、请求映射、限流器
- Bootstrap：Relay 缓存和注册状态

## 可观测性

- `health` 接口
- 结构化日志
- 见证记录统计与导出
- Provider 指标

## 限制

- 消息大小有上限
- 请求年龄有上限
- Provider 并发有上限
- Consumer 预算和时长有上限

## 发布门槛

新的运行时能力只有在满足以下条件后才应进入主线：

- 有清晰的模块负责人
- 有明确的失败语义
- 有测试
- 有运行时可见性

## 经济运行就绪检查

当改动涉及市场或结算能力时，进入主线前应至少满足：

- 对抽样完成请求，见证记录导出结果可重放、可复核
- 需要确定性结算重放的见证记录，仍可关联有效的定价快照引用
- 运行时输出和导出中，报价预算字段与结算资产字段保持显式分离
