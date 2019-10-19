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

## `collections/OrderedDict`

Unfortunately Go doesn't include an ordered map type in its standard library.
[iancoleman/orderedmap](https://github.com/iancoleman/orderedmap) appears to be
a suitable candidate; explicitly stating it's equivalence to `OrderedDict`.

## `configparser`

[go-ini/ini](https://github.com/go-ini/ini) appears to be a suitable
replacement.

```go
package main

import (
	"fmt"

	"gopkg.in/ini.v1"
)

func main() {
	cfg, err := ini.Load([]byte(`
[user]
	name = My Name
	email = my_email@provider.com
	username = myusername
[core]
	editor = sublime
[web]
	browser = google-chrome
[difftool]
	prompt = false
`))

	if err != nil {
		fmt.Println(err)
	}

	// simple get key
	name := cfg.Section("user").Key("name").String()
	fmt.Println(name)

	// get key with validation and default value
	editor := cfg.Section("core").Key("editor").In("emacs", []string{"vim", "emacs", "nano"})
	fmt.Println(editor)

	// get key with auto type conversion and default value
	prompt := cfg.Section("difftool").Key("prompt").MustBool(false)
	fmt.Println(prompt)
}
```

`ini` has support for validation in both value and type. Both support defining a
default value if none were supplied. Keys can be retrieved as well as set,
saving changes back to disk.
