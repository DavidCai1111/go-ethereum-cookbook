---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/method.go
---

# method.go

定义了 Method 结构体，是 [abi.go](abi.go.md) ABI 结构体的子结构体。用于记录合约内定义的相关函数。

支持的函数类型有：

```go
const (
	// 构造函数
	Constructor FunctionType = iota
	// Fallback 函数
	Fallback
	// Receive 函数
	Receive
	// 普通函数
	Function
)
```

具体结构如下：

```go
type Method struct {
  // 函数名称，由于函数是支持重载的，所以如果重名的话，例如：
  // * foo(int,int)
	// * foo(uint,uint)
  // 那么一个函数的 Name 会是 foo ，第二个函数的 Name 会是 foo0
	Name    string
  // 原名称，未经 Name 字段这样的处理，可能会重名
	RawName string

  // 函数类型，FunctionType 的具体枚举值上文已阐述
	Type FunctionType

  // 是否会改变合约状态的标识，默认值为 nonpayable
	StateMutability string

	// Solidity 0.6.0 之前才会编译出的 ABI JSON 字段
  Constant bool
	Payable  bool

  // 输入输出参数
	Inputs  Arguments
	Outputs Arguments
	str     string
	// 函数签名只会用到函数名称，和参数类型。
  // 忽略参数名称和返回值名称与类型。
	Sig string
	// ID 为签名的 Keccak-256 哈希的前 4 字节
	ID []byte
}
```
