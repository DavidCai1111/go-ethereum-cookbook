---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/event.go
---

# event.go

定义了 Event 结构体，是 [abi.go](abi.go.md) ABI 结构体的子结构体。用于记录合约内定义的相关日志，例如：

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract TestToken {
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
}
```

具体结构如下：

```go
type Event struct {
  // 日志名称，由于日志是支持重载的，所以如果重名的话，例如：
  // * foo(int,int)
	// * foo(uint,uint)
  // 那么一个日志的 Name 会是 foo ，第二个日志的 Name 会是 foo0
	Name string
	// 原名称，未经 Name 字段这样的处理，可能会重名
	RawName   string
  // 是否是匿名日志
	Anonymous bool
  // 入参
	Inputs    Arguments
  // 该结构体的整体字符串表示，用于日志打印输出
	str       string
	// 签名只会用到名称，和参数类型
  // 忽略参数名称
	Sig string
	// ID 为签名的 Keccak256 哈希值
	ID common.Hash
}
```
