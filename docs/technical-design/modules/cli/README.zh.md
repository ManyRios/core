# CLI

## 目的

本模块提供面向运维者的命令入口，用于本地初始化、运行控制和 RBOB 查询。

当前 CLI 仍然是主要运行入口。随着产品主线推进，它也应成为受支持的 Claw 自动化入口，而不是把所有运维动作长期固定在手工子命令上。
CLI 同时是走向 `Automation` 的过渡界面：手工命令应逐步映射为受支持的 Claw 工作流（接入、发布、暂停与恢复）。

## 职责边界

- 初始化钱包
- 配置并启动 Consumer、Provider、Relay
- 渲染运行时状态
- 暴露 RBOB 账本命令

## 不负责

- 不拥有协议核心逻辑
- 不独立保存长期运行状态
- 不替代运行时健康检查或见证记录存储模块

## 接口定义

```bash
veil init
veil provide init
veil provide start
veil relay start
veil status
veil rbob balance
veil rbob leaderboard
```

目标扩展：

```bash
veil autopilot init
veil autopilot join
veil autopilot show
```

## 数据流

输入：命令行参数、密码、本地配置。  
处理：调用运行时模块、校验前置条件、格式化输出。  
输出：可读状态和可执行的错误提示。

## 状态管理

- 内存：当前命令状态、spinner、格式化表格
- 持久化：通过钱包文件、Provider 配置、Relay DB、RBOB DB 间接持久化

## 错误处理

- 钱包缺失
- 密码错误
- Provider 配置非法
- 上游验证失败

## 安全约束

- 密码不能通过 argv 传入
- 输出不能暴露秘密信息

## 测试要求

- 初始化流程
- provide init 流程
- 状态输出
- RBOB 查询

## 依赖关系

- 调用：`wallet`、`consumer`、`provider`、`relay`、`rbob`
- 被调用：运维者与贡献者
