---
"type:": fleet-note
"title:": 20250603101314-K8s Resource Input Validation
id:: 20250603101337  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-06-03T10:13:37  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---

```go
/Users/ryan/go/pkg/mod/k8s.io/apimachinery@v0.33.1/pkg/api/resource/quantity.go:138

// MustParse turns the given string into a quantity or panics; for tests// or other cases where you know the string is valid.  
func MustParse(str string) Quantity {  
    q, err := ParseQuantity(str)  
    if err != nil {  
       panic(fmt.Errorf("cannot parse '%v': %v", str, err))  
    }  
    return q  
}
```

# Reference