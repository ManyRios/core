# Wallet & Identity

## Purpose

This module owns local identity material, wallet encryption, and credential protection.

## Responsibility Boundary

- generate the signing and encryption keypairs
- encrypt the wallet on disk
- load local key material into runtime memory
- encrypt Provider-side API credentials

## Out Of Scope

- does not route requests
- does not perform upstream inference
- does not store witness or contribution ledger records

## Interface

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

## Data Flow

Input: passwords, wallet files, API keys.  
Process: generate keys, encrypt and decrypt, read and write local config.  
Output: runtime wallet objects and encrypted files.

## State

- persistent: `wallet.json`, `config.json`, encrypted credential blobs
- memory: unlocked key material

## Errors

- wallet missing
- wrong password
- corrupted wallet file

## Security Constraints

- never log private keys
- keep wallet files permission-restricted
- keep wallet secrets and Provider credentials in separate protection domains

## Test Requirements

- wallet create, load, export, change password
- invalid password
- damaged file handling

## Dependencies

- calls: `crypto`
- called by: `cli`, `consumer`, `provider`, `relay`
