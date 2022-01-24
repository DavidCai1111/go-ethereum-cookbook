---
description: https://github.com/ethereum/go-ethereum/blob/master/accounts/url.go
---

# url.go

URL 用于标识一个账户（account）资源在其后端存储的位置。由 `Scheme` （用于标识存储方式）和 `Path` （用于标识存储路径）组成。

```go
type URL struct {
	Scheme string
	Path   string
}
```

URL 的 `Scheme` 和 `Path` 之间由 `://` 分割：

```go
func parseURL(url string) (URL, error) {
	parts := strings.Split(url, "://")
	if len(parts) != 2 || parts[0] == "" {
		return URL{}, errors.New("protocol scheme missing")
	}
	return URL{
		Scheme: parts[0],
		Path:   parts[1],
	}, nil
}
```

并且在比较时，若存在 `Scheme` 则会优先比较 `Scheme`：

```go
func (u URL) Cmp(url URL) int {
	if u.Scheme == url.Scheme {
		return strings.Compare(u.Path, url.Path)
	}
	return strings.Compare(u.Scheme, url.Scheme)
}
```
