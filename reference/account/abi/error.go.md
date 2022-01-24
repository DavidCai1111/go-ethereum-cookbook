---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/error.go
---

# error.go

定义了 Error 结构体，是 [abi.go](abi.go.md) ABI 结构体的子结构体。用于记录合约内定义的相关错误，例如：

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract TestToken {
    error InsufficientBalance(uint256 available, uint256 required);
    function transfer(address /*to*/, uint amount) public pure {
        revert InsufficientBalance(0, amount);
    }
}
```

其中的 `InsufficientBalance(uint256 available, uint256 required)` 便是定义的错误。

具体定义如下：

```go
type Error struct {
  // 名称
	Name   string
  // 入参
	Inputs Arguments
  // 该结构体的整体字符串表示，用于日志打印输出
	str    string
  // 签名只会用到名称，和参数类型
  // 忽略参数名称
	Sig string
	// ID 为签名的 Keccak256 哈希值
	ID common.Hash
}
```
