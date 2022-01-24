---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/hd.go
---

# hd.go

根据 BIP 规范，定义了一系列分层确定性钱包（hierarchical deterministic wallet）相关常量与逻辑。

根据 [BIP-32 规范](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)，定义的派生路径格式为：

```txt
m / purpose' / coin_type' / account' / change / address_index
```

根据 [BIP-44 规范](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)，加密货币的 purpose 值为 44'，根据 [SLIP-44 文档](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) 定义，以太坊的 coin_type 为 60'。

所以以太坊的派生根目录为 `m/44'/60'/0'/0` ：

```go
var DefaultRootDerivationPath = DerivationPath{0x80000000 + 44, 0x80000000 + 60, 0x80000000 + 0, 0}
```

那么第一个账户就是 `m/44'/60'/0'/0/0`：

```go
var DefaultBaseDerivationPath = DerivationPath{0x80000000 + 44, 0x80000000 + 60, 0x80000000 + 0, 0, 0}
```
