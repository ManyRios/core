# RBOB Ledger

## 目的

本模块记录开源贡献 grant，并提供可审计的账本查询能力。

## 职责边界

- 存储贡献积分
- 用管理员密钥签名账本条目
- 校验账本条目
- 导出余额和排行榜数据

## 不负责

- 不参与推理路由
- 不替代生产见证记录
- 不管理超出账本需要范围的贡献者身份

## 接口定义

```ts
class RbobLedger {
  grant(
    params: GrantParams,
    adminSecretKey: Uint8Array,
    adminPublicKey: Uint8Array,
  ): LedgerEntry;
  verifyEntry(entry: LedgerEntry, adminPublicKey: Uint8Array): boolean;
  balance(contributor: string): BalanceResult;
  leaderboard(limit?: number): LeaderboardEntry[];
  exportJSON(): TokenClaimExport;
}
```

## 数据流

输入：贡献者身份、积分、理由、PR 引用。  
处理：校验 grant、签名消息、写入账本、查询汇总。  
输出：已签名的贡献记录和导出载荷。

## 状态管理

- 持久化：`rbob_ledger`
- 内存：查询结果和导出对象

## 错误处理

- 积分非正数
- contributor 缺失
- reason 缺失
- 签名校验无效

## 安全约束

- 管理员签名是账本信任锚
- 贡献账务必须与生产见证记录账务分离

## 测试要求

- grant、verify、balance、leaderboard、export
- 非法 grant 拒绝

## 依赖关系

- 调用：`crypto`
- 被调用：`cli`、维护者
