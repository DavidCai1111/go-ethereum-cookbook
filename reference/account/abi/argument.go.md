---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/argument.go
---

# argument.go

定义了 Argument 结构体，是函数/日志/错误的入参出参的抽象。

结构体定义为：

```go
type Argument struct {
	Name    string
	Type    Type
	Indexed bool // 仅会在日志的入参中用到
}

type Arguments []Argument

type ArgumentMarshaling struct {
	Name         string
	Type         string
	InternalType string
	Components   []ArgumentMarshaling // 用于元组类型，描述子类型
	Indexed      bool
}
```

并且实现了参数列表的编码与解码，具体编码/解码函数请参阅 [pack.go](pack.go.md) ，[unpack.go](unpack.go.md) 和 [type.go](type.go.md) 。
