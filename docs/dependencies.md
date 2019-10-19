# Dependencies

The official implementation of WYAG requires a [few
dependencies](https://wyag.thb.lt/#getting-started) from the Python standard
library. In this document we'll aim to find substitues in Go.

## `argparse`

Go's alternative to `argparse` is probably the
[flag](https://golang.org/pkg/flag/) package. Usage is as follows:

```go
package main

import (
	"flag"
	"fmt"
)

func main() {
	aString := flag.String("stringArg", "foo", "a string")

	flag.Parse()
	unnamedArgs := flag.Args()

	fmt.Printf("aString: %s\n", *aString)
	fmt.Printf("others: %v\n", unnamedArgs)
}

```

The `flag` package supports types and validation for named arguments, such as
`stringArg` in the above example. Positional arguments can be accessed through
`flag.Args()` after `flag.Parse()` is called, and returns an array of arguments
which remain after named arguments are parsed.

```
// aString: foo  others: []
go run main.go

// aString: hello  others: [add]
go run main.go add -stringArg=hello
```
