---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/accounts.go
---

# accounts.go

主要定义了 `Account` 结构体，以及 `Wallet` 接口，供各类钱包进行实现。

### Account

```go
type Account struct {
	Address common.Address `json:"address"` // 地址
	URL     URL            `json:"url"`     // 位于其后端的 URL 地址
}
```

`Account` 结构体的组成就两个字段，首先是其它的账户地址，用于在以太坊中与其他账户交互，其次是它的 URL ，用于表示数据在后端中所在的具体位置。

### Wallet

`Wallet` 钱包用于同一管理其中的账户，主要功能有打开/关闭，返回状态，派生新账户，以及使用其中的账户对数据进行签名。

```go
type Wallet interface {
  // URL 表示钱包在其后端的具体位置信息
	URL() URL

  // 返回钱包状态
	Status() (string, error)

  // 打开钱包，若不提供密码，则可以只建立连接
	Open(passphrase string) error

  // 关闭钱包
	Close() error

  // 列出钱包内的所有账户，若是分层确定性钱包，则不会穷举所有可能的钱包，
  // 而是仅返回明确派生使用的钱包
	Accounts() []Account

  // 检查钱包中是否包含某账户
	Contains(account Account) bool

  // 从分层确定性钱包的指定路径下派生一个新账户。
	Derive(path DerivationPath, pin bool) (Account, error)

  // 从一个基础地址开始，寻找非零的派生账户。
	SelfDerive(bases []DerivationPath, chain ethereum.ChainStateReader)

  // 对指定数据进行签名，可能会返回 AuthNeededError 并进一步索要密码
	SignData(account Account, mimeType string, data []byte) ([]byte, error)

  // 在 SignData 的基础上，需要额外提供密码
	SignDataWithPassphrase(account Account, passphrase, mimeType string, data []byte) ([]byte, error)

  // 对指定一段数据进行签名（通常以 0x 为前缀），可能会返回 AuthNeededError 并进一步索要密码
	SignText(account Account, text []byte) ([]byte, error)

  // 在 SignText 的基础上，需要额外提供密码
	SignTextWithPassphrase(account Account, passphrase string, hash []byte) ([]byte, error)

  // 对指定交易进行签名，可能会返回 AuthNeededError 并进一步索要密码
	SignTx(account Account, tx *types.Transaction, chainID *big.Int) (*types.Transaction, error)

  // 在 SignTx 的基础上，需要额外提供密码
	SignTxWithPassphrase(account Account, passphrase string, tx *types.Transaction, chainID *big.Int) (*types.Transaction, error)
}
```

### Backend

按文档定义，`Backend` 是一个钱包提供者（wallet provider）。

```go
type Backend interface {
  // 当前所包含的钱包，这些钱包并没有被打开/解码
	Wallets() []Wallet

  // 监听已有钱包的状态改动，新钱包的进入和旧钱包的退出
	Subscribe(sink chan<- WalletEvent) event.Subscription
}
```
