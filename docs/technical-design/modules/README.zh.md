# 模块设计

## 目的

本节按实现边界组织 Veil。每个模块页都遵循同一套合同模板，便于维护者和贡献者在不同模块之间切换时不用重新适应格式。

## 适用场景

- 你要修改某个具体模块
- 你想先看模块合同再读代码
- 你需要理解模块之间的依赖关系

## 模块模板

每个模块页都按同样的顺序组织：

- 目的
- 职责边界
- 不负责
- 接口定义
- 数据流
- 状态管理
- 错误处理
- 安全约束
- 测试要求
- 依赖关系

## 模块列表

标记为“规划中”的模块仍然属于产品主路径，并不是可有可无的附属件。Veil 要走到目标市场和自动化模型，这些模块必须回到正式实现面。

- [wallet-identity/](./wallet-identity/README.zh.md)
- [consumer-gateway/](./consumer-gateway/README.zh.md)
- [network-transport/](./network-transport/README.zh.md)
- [relay/](./relay/README.zh.md)
- [provider-engine/](./provider-engine/README.zh.md)
- [metering-witness/](./metering-witness/README.zh.md)
- [bootstrap-discovery/](./bootstrap-discovery/README.zh.md)
- [claw-autopilot/](./claw-autopilot/README.zh.md)（规划中）
- [pricing-risk-policy/](./pricing-risk-policy/README.zh.md)（规划中）
- [settlement-payout/](./settlement-payout/README.zh.md)（规划中）
- [cli/](./cli/README.zh.md)
- [rbob-ledger/](./rbob-ledger/README.zh.md)
