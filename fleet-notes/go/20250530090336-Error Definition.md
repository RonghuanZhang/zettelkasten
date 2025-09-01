---
"type:": fleet-note
"title:": 20250530090336-Error Definition
"id:": 20250530090341
"created:": 2025-05-30T09:03:41
url: 
tags:
  - fleet-note
  - golang
"processed:": false
"archived:": false
---
```go
package adapter

import (
	"net/http"
)

type BizError struct {
	code    int    // HTTP status code
	reason  string // Custom error reason.
	message string
	details string
}

func (b BizError) Error() string {
	//TODO implement me
	panic("implement me")
}

func NewBizError(code int, reason, message, details string) *BizError {
	return &BizError{
		code:    code,
		reason:  reason,
		message: message,
		details: details,
	}
}

var (
	ParamValidationFailedBizError = NewBizError(http.StatusBadRequest, "XXX-11", http.StatusText(http.StatusBadRequest), "name is null")
)

```

# Reference