# errors
errors with stack, supports multi-cause error format as tree, 
inspired by [cockroachdb/errors](github.com/cockroachdb/errors)

> 博客：[造一个 Go 语言错误库的轮子](https://youthlin.com/?p=1868)

Example
```go
package main

import (
	stdErr "errors"
	"fmt"
	"testing"

	"code.gopub.tech/errors"
	cdbErr "github.com/cockroachdb/errors"
	pkgErr "github.com/pkg/errors"
)

func TestErrors(t *testing.T) {
	var (
		errStd  = stdErr.New("err-std\nnew\nline")
		errPkg  = pkgErr.New("err-pkg\nnew\nline")
		errCdb  = cdbErr.New("err-cdb\nnew\nline")
		errThis = errors.New("err-this\nnew\nline")
	)
	fmt.Printf("%+v\n", errors.Wrap(errors.Join(errStd, errPkg, errCdb, errThis), "prefix"))
}

```

Output
```
prefix: err-std
new
line
err-pkg
new
line
err-cdb
new
line
err-this
new
line
(1) attached stack trace
 │ -- stack trace:
 │ code.gopub.tech/example.TestErrors
 │ 	/go/src/code.gopub.tech/example/main_test.go:20
 │ [...repeated from below...]
Next: (2) prefix
Next: (3) attached stack trace
 │ -- stack trace:
 │ code.gopub.tech/example.TestErrors
 │ 	/go/src/code.gopub.tech/example/main_test.go:20
 │ [...repeated from below...]
Next: (4) err-std
 │ new
 │ line
 │ err-pkg
 │ new
 │ line
 │ err-cdb
 │ new
 │ line
 │ err-this
 │ new
 │ line
 ├─ Wraps: (5) err-std
 │  │  new
 │  └─ line
 ├─ Wraps: (6) err-pkg
 │  │  new
 │  │  line
 │  │  -- stack trace:
 │  │  code.gopub.tech/example.TestErrors
 │  │  	/go/src/code.gopub.tech/example/main_test.go:16
 │  └─ [...repeated from below...]
 ├─ Wraps: (7)
 │  │ -- stack trace:
 │  │ code.gopub.tech/example.TestErrors
 │  │ 	/go/src/code.gopub.tech/example/main_test.go:17
 │  │ [...repeated from below...]
 │ Next: (8) err-cdb
 │  │  new
 │  └─ line
 └─ Wraps: (9) attached stack trace
    │ -- stack trace:
    │ code.gopub.tech/example.TestErrors
    │ 	/go/src/code.gopub.tech/example/main_test.go:18
    │ testing.tRunner
    │ 	/sdk/go1.21.6/src/testing/testing.go:1595
    │ runtime.goexit
    │ 	/sdk/go1.21.6/src/runtime/asm_amd64.s:1650
   Next: (10) err-this
    │  new
    └─ line
Error types: (1) *errors.withStack (2) *errors.withPrefix (3) *errors.withStack (4) *errors.joinError (5) *errors.errorString (6) *errors.fundamental (7) *withstack.withStack (8) *errutil.leafError (9) *errors.withStack (10) *errors.errorString
```