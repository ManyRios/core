# Wallet 与身份

## 目的

本模块拥有本地身份材料、钱包加密和凭证保护职责。

## 职责边界

- 生成签名密钥对和加密密钥对
- 对钱包文件进行落盘加密
- 把本地密钥加载到运行时内存
- 加密 Provider 侧 API 凭证

## 不负责

- 不负责请求路由
- 不执行上游推理
- 不存储见证记录或贡献账本记录

## 接口定义

```ts
interface Wallet {
  signingPublicKey: Uint8Array;
  signingSecretKey: Uint8Array;
  encryptionPublicKey: Uint8Array;
  encryptionSecretKey: Uint8Array;
}

function createWallet(password: string, veilHome?: string): Promise<WalletPublicInfo>;
function loadWallet(password: string, veilHome?: string): Promise<Wallet>;
function encryptApiKey(apiKey: string, password: string): EncryptedSecret;
```

## 数据流

输入：密码、钱包文件、API key。  
处理：生成密钥、加密与解密、读写本地配置。  
输出：运行时钱包对象和加密后的本地文件。

## 状态管理

- 持久化：`wallet.json`、`config.json`、加密凭证
- 内存：解锁后的密钥材料

## 错误处理

- 钱包不存在
- 密码错误
- 钱包文件损坏

## 安全约束

- 不记录私钥日志
- 钱包文件权限必须收紧
- 钱包秘密与 Provider 凭证分属不同保护域

## 测试要求

- 创建、加载、导出、改密
- 错误密码
- 文件损坏处理

## 依赖关系

- 调用：`crypto`
- 被调用：`cli`、`consumer`、`provider`、`relay`
