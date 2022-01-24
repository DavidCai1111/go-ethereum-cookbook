---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/abi.go
---

# abi.go

首先该文件中定义了 ABI 结构体，用于保存一个合约中所有具体与 ABI 编码/解码相关的状态：

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

其中的成员 `Errors` 可能有些小伙伴比较陌生，它是自 `Solidity` 0.8.0 后[引入 ABI 定义](https://docs.soliditylang.org/en/latest/abi-spec.html#errors)中。

该结构体支持直接从编译后的 ABI JSON 解析，但是反过来从 struct 到 ABI JSON 的 Marshal 并没有实现：

```go
func (abi *ABI) UnmarshalJSON(data []byte) error {
  	// 我们编译合约后，在 ABI JSON 中看到的内容，就是 []fields ，字段一一对应
	var fields []struct {
		Type    string
		Name    string
		Inputs  []Argument
		Outputs []Argument

		StateMutability string

		Constant bool
		Payable  bool

		Anonymous bool
	}
	if err := json.Unmarshal(data, &fields); err != nil {
		return err
	}
	abi.Methods = make(map[string]Method)
	abi.Events = make(map[string]Event)
	abi.Errors = make(map[string]Error)

  	// 根据 ABI JSON 数组中每一项的类型（type 字段），逐个
  	// 调用对应类型的 New 函数进行初始化，然后存入结构体中
	for _, field := range fields {
		switch field.Type {
		case "constructor":
			abi.Constructor = NewMethod("", "", Constructor, field.StateMutability, field.Constant, field.Payable, field.Inputs, nil)
		case "function":
			// ...
			abi.Methods[name] = NewMethod(name, field.Name, Function, field.StateMutability, field.Constant, field.Payable, field.Inputs, field.Outputs)
		case "fallback":
			// ...
			abi.Fallback = NewMethod("", "", Fallback, field.StateMutability, field.Constant, field.Payable, nil, nil)
		case "receive":
			// ...
			abi.Receive = NewMethod("", "", Receive, field.StateMutability, field.Constant, field.Payable, nil, nil)
		case "event":
			// ...
			abi.Events[name] = NewEvent(name, field.Name, field.Anonymous, field.Inputs)
		case "error":
			abi.Errors[field.Name] = NewError(field.Name, field.Inputs)
		default:
			return fmt.Errorf("abi: could not recognize type %v of field %v", field.Type, field.Name)
		}
	}
	return nil
}
```

对该合约函数请求的编码，也是结构体的一个方法，根据[说明文档](https://docs.soliditylang.org/en/latest/abi-spec.html)，该编码的组成为 `function_selector(f) enc((a_1, ..., a_n))`：

```go
// abi.go
func (abi ABI) Pack(name string, args ...interface{}) ([]byte, error) {
	// ...
  	// 各个调用参数，根据各自类型，一一编码
	arguments, err := method.Inputs.Pack(args...)
	if err != nil {
		return nil, err
	}
	// 将所有参数的编码结果拼接在 method.ID 之后
	return append(method.ID, arguments...), nil
}
```

显而易见，此处的 `method.ID` 即是 `function_selector(f)` ，根据文档定义，为函数签名的 Keccak-256 哈希的前 4 字节：

```go
// method.go
func NewMethod(name string, rawName string, funType FunctionType, mutability string, isConst, isPayable bool, inputs Arguments, outputs Arguments) Method {
	// ...
	var (
		sig string
		id  []byte
	)
	if funType == Function {
    		// 函数签名只会用到函数名称，和参数类型。
    		// 忽略参数名称和返回值名称与类型。
		sig = fmt.Sprintf("%v(%v)", rawName, strings.Join(types, ","))
		id = crypto.Keccak256([]byte(sig))[:4]
	}

  	// ...
}
```

既然支持合约请求进行 ABI 编码，自然也支持对合约返回 ABI 编码后的结果进行解码：

```go
func (abi ABI) Unpack(name string, data []byte) ([]interface{}, error) {
	args, err := abi.getArguments(name, data)
	if err != nil {
		return nil, err
	}

  	// 函数支持多返回值，会为每一个返回值执行 unpack
	return args.Unpack(data)
}
```

该具体数据类型的编码/解码，可参阅 [type.go](type.go.md) 。
