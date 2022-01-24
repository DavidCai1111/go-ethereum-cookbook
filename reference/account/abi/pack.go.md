---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/pack.go
---

# pack.go

该文件主要定义了除数组与元组外的其他类型数据的具体编码函数。

我们从 [type.go](type.go.md) 中的数组/元组编码函数的最后可以看到，若数据类型不是数组/元组，则会在本文件的 `packElement` 函数中进行编码：

```go
func packBytesSlice(bytes []byte, l int) []byte {
	// 字符串和字节 bytes 类型进行编码时会用到
	// 返回：长度 + 右侧补 0 编码值
	len := packNum(reflect.ValueOf(l))
	return append(len, common.RightPadBytes(bytes, (l+31)/32*32)...)
}

func packElement(t Type, reflectValue reflect.Value) ([]byte, error) {
	switch t.T {
	case IntTy, UintTy:
    		// 整数以及无符号整数
		return packNum(reflectValue), nil
	case StringTy:
    		// 字符串
		return packBytesSlice([]byte(reflectValue.String()), reflectValue.Len()), nil
	case AddressTy:
    		// 地址类型，等价于 uint160
		// ...
		return common.LeftPadBytes(reflectValue.Bytes(), 32), nil
	case BoolTy:
    		// 布尔值
		if reflectValue.Bool() {
			return math.PaddedBigBytes(common.Big1, 32), nil
		}
		return math.PaddedBigBytes(common.Big0, 32), nil
	case BytesTy:
    		// 字节
    		// ...
		return packBytesSlice(reflectValue.Bytes(), reflectValue.Len()), nil
	case FixedBytesTy, FunctionTy:
    		// 固定长度字节以及函数
    		// ...
		return common.RightPadBytes(reflectValue.Bytes(), 32), nil
	default:
		return []byte{}, fmt.Errorf("Could not pack element, unknown type: %v", t.T)
	}
}
```

### **整数，无符号整数**

首先是整数以及无符号整数，调用 `packNum()`：

```go
func packNum(value reflect.Value) []byte {
	switch kind := value.Kind(); kind {
	case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64:
		return math.U256Bytes(new(big.Int).SetUint64(value.Uint()))
	case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
		return math.U256Bytes(big.NewInt(value.Int()))
	case reflect.Ptr:
		return math.U256Bytes(new(big.Int).Set(value.Interface().(*big.Int)))
	default:
		panic("abi: fatal error")
	}
}

// common/math/big.go
func U256Bytes(n *big.Int) []byte {
	return PaddedBigBytes(U256(n), 32)
}
```

从源码可以看到，这两种类型编码都为在其大端序表示的左侧补足 0 至 32 字节（256 位）。

例如 uint32 = 69 编码后为：

```
0x0000000000000000000000000000000000000000000000000000000000000045
```

### **字符串与字节数组**

字符串先会通过 `[]byte(reflectValue.String())` 类型转换，转换成 unicode 的字节数组。然后和字节数组一样，先是长度值，再加上具体值在右侧补 0 至 32 字节（`common.RightPadBytes(reflectValue.Bytes(), 32)`）。

例如 string = "Hello World" 编码后为：

```
0x000000000000000000000000000000000000000000000000000000000000000b
0x48656c6c6f20576f726c64000000000000000000000000000000000000000000
```

### **布尔值**

```go
// common/big.go
var (
	Big1   = big.NewInt(1)
	Big0   = big.NewInt(0)
)
```

布尔值的编码逻辑比较简单，若是 true 则与 int64 = 1 相同，若是 false 则与 int64 = 0 相同，同样在左侧补 0 至 32 字节。

例如 bool = false 编码后为：

```
0x0000000000000000000000000000000000000000000000000000000000000000
```

### **固定长度字节以及函数**

固定长度字节与字节的编码相比，省去了前面的长度信息，值也是在右侧补 0 至 32 字节。

此处的函数指的函数的 `function_selector(f)` ，为函数签名的 Keccak-256 哈希的前 4 字节，即在 [abi.go](abi.go.md) 中所看到的 method.ID ，所以也是固定长度的。
