---
description: https://github.com/ethereum/go-ethereum/tree/master/accounts/abi
---

# abi

该目录下的代码，根据[以太坊智能合约的 ABI 编码定义](https://docs.soliditylang.org/en/latest/abi-spec.html)，完整实现了从 Go 语言结构体与具体 ABI 输出之间的编码与解码。其内部的 bind 子目录，提供了工具（可通过 `./cmd/abigen` 使用）根据 ABI 文件或 Solidity 代码文件生成 Golang/Java/OC 桩代码（类似于 Grpc 里的 Stub 文件），方便强类型语言客户端进行调用/部署，具体用法可参阅[这篇文档](https://geth.ethereum.org/docs/dapp/native-bindings)。

```
accounts/abi
├── abi.go // 定义了 ABI struct ，以及最顶层的编码/解码函数
├── abi_test.go
├── argument.go // 定义函数与日志相关参数的具体编码/解码函数
├── bind // 包含了工具函数来根据 ABI 文件或 Solidity 代码生成 Golang/Java/OC 桩代码
│   ├── auth.go // 为客户端发送交易提供了身份认证工具函数
│   ├── backend.go // 定义了一系列后端链代码需要实现的接口
│   ├── backends
│   │   ├── simulated.go // 提供一个用于测试目的的模拟链，该链可通过函数调用直接出块
│   │   └── simulated_test.go
│   ├── base.go // 定义桩代码与合约交互的读写函数的基本接口和基础实现
│   ├── base_test.go
│   ├── bind.go // 定义入口函数，供 ./cmd/abigen 使用
│   ├── bind_test.go
│   ├── template.go // 桩代码的生产模板
│   ├── util.go // 相关工具函数
│   └── util_test.go
├── doc.go // 只有注释，仅用于文档生成
├── error.go // 从 ABI JSON 中读取的错误（error）相关状态
├── error_handling.go // 定义了 ABI 编码/解码中相关错误检查函数
├── event.go // 从 ABI JSON 中读取的日志相关状态
├── event_test.go
├── method.go // 从 ABI JSON 中读取的函数相关状态
├── method_test.go
├── pack.go // 定义了各类型数据的 ABI 具体编码逻辑
├── packing_test.go
├── pack_test.go
├── reflect.go // 定义了各数据类型从 ABI 解码后反射到 Golang 原生类型的逻辑
├── reflect_test.go
├── topics.go // 从 ABI JSON 中解析日志 topic 相关信息
├── topics_test.go
├── type.go // 各种数据类型的上层编码/解码入口函数
├── type_test.go
├── unpack.go // 定义了各数据类型的 ABI 具体解码逻辑
└── unpack_test.go
```
