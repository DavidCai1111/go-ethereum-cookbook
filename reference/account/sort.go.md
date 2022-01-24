---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/sort.go
---

# sort.go

这个文件定义了账户和钱包的排序逻辑，都是根据 URL 排序，具体 URL 的排序逻辑参阅 [url.go](url.go.md) 。

```go
type AccountsByURL []Account

func (a AccountsByURL) Len() int           { return len(a) }
func (a AccountsByURL) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a AccountsByURL) Less(i, j int) bool { return a[i].URL.Cmp(a[j].URL) < 0 }

type WalletsByURL []Wallet

func (w WalletsByURL) Len() int           { return len(w) }
func (w WalletsByURL) Swap(i, j int)      { w[i], w[j] = w[j], w[i] }
func (w WalletsByURL) Less(i, j int) bool { return w[i].URL().Cmp(w[j].URL()) < 0 }
```
