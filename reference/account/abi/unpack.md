---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/unpack.go
---

# unpack.go

该文件中定义的是各数据类型的解码。

### 整数，无符号整数

```go
func ReadInteger(typ Type, b []byte) interface{} {
	if typ.T == UintTy {
    // 无符号整数，直接取最后对应长度的字节
		switch typ.Size {
		case 8:
			return b[len(b)-1]
		case 16:
			return binary.BigEndian.Uint16(b[len(b)-2:])
		case 32:
			return binary.BigEndian.Uint32(b[len(b)-4:])
		case 64:
			return binary.BigEndian.Uint64(b[len(b)-8:])
		default:
      // binary.BigEndian 最多只支持到 64 位，这里使用 big.Int
			// uint256：直接取完 32 字节
			return new(big.Int).SetBytes(b)
		}
	}
	switch typ.Size {
  // 整数，除 int256 外与无符号整数一样操作，取完后进行类型转换
	case 8:
		return int8(b[len(b)-1])
	case 16:
		return int16(binary.BigEndian.Uint16(b[len(b)-2:]))
	case 32:
		return int32(binary.BigEndian.Uint32(b[len(b)-4:]))
	case 64:
		return int64(binary.BigEndian.Uint64(b[len(b)-8:]))
	default:
		ret := new(big.Int).SetBytes(b)
		if ret.Bit(255) == 1 {
			ret.Add(MaxUint256, new(big.Int).Neg(ret))
			ret.Add(ret, common.Big1)
			ret.Neg(ret)
		}
		return ret
	}
}
```

int256 解码有点特殊，由于 golang 的 `new(big.Int).SetBytes(b)` 只会认为传入的大端序字节是正整数。不过，在 EVM 中，若返回值已经超过了 int256 正数的最大值，那么就表示这是一个负值。具体表现为第 255 位有值（`ret.Bit(255) == 1`）。若判断返回是的负数，那么就会先对其正值取负，然后加上 `MaxUint256` ，将其结果 +1 后再取负，得到最终值。

### 布尔值

```go
// unpack.go
func readBool(word []byte) (bool, error) {
  // 除最后一个字节外，其他应全为 0
	for _, b := range word[:31] {
		if b != 0 {
			return false, errBadBool
		}
	}

  // 根据最后一位进行判断
	switch word[31] {
	case 0:
		return false, nil
	case 1:
		return true, nil
	default:
		return false, errBadBool
	}
}
```

### 固定长度字节

```go
func ReadFixedBytes(t Type, word []byte) (interface{}, error) {
	// ...
	array := reflect.New(t.GetType()).Elem()

	reflect.Copy(array, reflect.ValueOf(word[0:t.Size]))
	return array.Interface(), nil
}
```

即直接将输出值的数组类型设定好后，将输入的字节直接拷贝进去。

### 字符串，字节数组

```go
func toGoType(index int, t Type, output []byte) (interface{}, error) {
  // ...
  	case StringTy: // variable arrays are written at the end of the return bytes
		  return string(output[begin : begin+length]), nil
    case BytesTy:
		  return output[begin : begin+length], nil
  // ...
}
```

都是直接取对应的输入字节，若字节是字符串 unicode 表示则再进行一次 Golang 类型转换。

账户地址（addrss），哈希，也都是直接将输入的字节做类型转换，就不再赘述了。

### 数组

```go
func forEachUnpack(t Type, output []byte, start, size int) (interface{}, error) {
  // ...
	// 此数 start, size 已通过最前端的信息和 ABI 描述里提供的元信息被解析出来
  // 所以后面不再考虑是否是固定长度/可变长度元素
	var refSlice reflect.Value

  // 按指定类型可返回 Slice 和 Array 两种反射类型
	if t.T == SliceTy {
		refSlice = reflect.MakeSlice(t.GetType(), size, size)
	} else if t.T == ArrayTy {
		refSlice = reflect.New(t.GetType()).Elem()
	} else {
		return nil, fmt.Errorf("abi: invalid type in array/slice unpacking stage")
	}

	// 获取具体元素大小
	elemSize := getTypeSize(*t.Elem)

	for i, j := start, 0; j < size; i, j = i+elemSize, j+1 {
		inter, err := toGoType(i, *t.Elem, output)
		if err != nil {
			return nil, err
		}

		// 推入解析出来的元素
		refSlice.Index(j).Set(reflect.ValueOf(inter))
	}

	return refSlice.Interface(), nil
}
```

### 元组

```go
func forTupleUnpack(t Type, output []byte) (interface{}, error) {
	retval := reflect.New(t.GetType()).Elem()
	virtualArgs := 0
	for index, elem := range t.TupleElems {
		marshalledValue, err := toGoType((index+virtualArgs)*32, *elem, output)
		if elem.T == ArrayTy && !isDynamicType(*elem) {
			// If we have a static array, like [3]uint256, these are coded as
			// just like uint256,uint256,uint256.
			// This means that we need to add two 'virtual' arguments when
			// we count the index from now on.
			//
			// Array values nested multiple levels deep are also encoded inline:
			// [2][3]uint256: uint256,uint256,uint256,uint256,uint256,uint256
			//
			// Calculate the full array size to get the correct offset for the next argument.
			// Decrement it by 1, as the normal index increment is still applied.
			virtualArgs += getTypeSize(*elem)/32 - 1
		} else if elem.T == TupleTy && !isDynamicType(*elem) {
			// If we have a static tuple, like (uint256, bool, uint256), these are
			// coded as just like uint256,bool,uint256
			virtualArgs += getTypeSize(*elem)/32 - 1
		}
		if err != nil {
			return nil, err
		}
		retval.Field(index).Set(reflect.ValueOf(marshalledValue))
	}
	return retval.Interface(), nil
}
```
