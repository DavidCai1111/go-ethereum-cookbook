---
description: 定义了 ABI 结构体 ，以及最顶层的编码/解码函数
---

# abi.go

定义了 ABI 结构体，用于保存一个合约中所有具体与 ABI 生成相关的状态。

```go
type ABI struct {
	Constructor Method
	Methods     map[string]Method
	Events      map[string]Event
	Errors      map[string]Error

	Fallback Method
	Receive  Method
}
```

