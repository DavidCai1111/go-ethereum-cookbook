---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/error_handling.go
---

# error_handling.go

该文件内定义了在 ABI 编码/解码过程中的一些类型错误检查函数。主要是 `sliceTypeCheck` 和 `typeCheck` 。

### sliceTypeCheck

```go
func sliceTypeCheck(t Type, val reflect.Value) error {
  // 非 slice 和 array ，报错
	if val.Kind() != reflect.Slice && val.Kind() != reflect.Array {
		return typeErr(formatSliceString(t.GetType().Kind(), t.Size), val.Type())
	}

  // 两个待检查参数长度不匹配，报错
	if t.T == ArrayTy && val.Len() != t.Size {
		return typeErr(formatSliceString(t.Elem.GetType().Kind(), t.Size), formatSliceString(val.Type().Elem().Kind(), val.Len()))
	}

  // 递归检查
	if t.Elem.T == SliceTy || t.Elem.T == ArrayTy {
		if val.Len() > 0 {
			return sliceTypeCheck(*t.Elem, val.Index(0))
		}
	}

  // 元素类型不匹配，报错
	if val.Type().Elem().Kind() != t.Elem.GetType().Kind() {
		return typeErr(formatSliceString(t.Elem.GetType().Kind(), t.Size), val.Type())
	}
	return nil
}
```

### typeCheck

```go
func typeCheck(t Type, value reflect.Value) error {
  // 跳转至数组检查
	if t.T == SliceTy || t.T == ArrayTy {
		return sliceTypeCheck(t, value)
	}

  // 类型不匹配，报错
	if t.GetType().Kind() != value.Kind() {
		return typeErr(t.GetType().Kind(), value.Kind())
	} else if t.T == FixedBytesTy && t.Size != value.Len() {
    // 固定长度字节的情况下，长度不匹配，报错
		return typeErr(t.GetType(), value.Type())
	} else {
		return nil
	}
}
```
