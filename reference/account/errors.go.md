---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/errors.go
---

# errors.go

定义了一系列账户和钱包的函数交互中可能会抛出的错误。

```go
// 未找到账户
var ErrUnknownAccount = errors.New("unknown account")

// 未找到钱包
var ErrUnknownWallet = errors.New("unknown wallet")

// 请求了后端不支持的操作
var ErrNotSupported = errors.New("not supported")

// 密码不正确
var ErrInvalidPassphrase = errors.New("invalid password")

// 钱包已打开
var ErrWalletAlreadyOpen = errors.New("wallet already open")

// 钱包已关闭
var ErrWalletClosed = errors.New("wallet closed")

// 需要验证身份，通常需要再次提供密码或者 PIN code （冷钱包）
type AuthNeededError struct {
	Needed string // 具体需要提供的验证
}

// 创建 AuthNeededError
func NewAuthNeededError(needed string) error {
	return &AuthNeededError{
		Needed: needed,
	}
}

// 为 AuthNeededError 实现 Error 接口
func (err *AuthNeededError) Error() string {
	return fmt.Sprintf("authentication needed: %s", err.Needed)
}
```
