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

Subcommands can be managed via the
[`FlagSet`](https://golang.org/pkg/flag/#FlagSet) type. API documentation is a
bit sparse for this one, but this [blog
post](https://blog.rapid7.com/2016/08/04/build-a-simple-cli-tool-with-golang/)
covers it quite well. `FlagSet` exposes the same methods as `Flag`, but you can
declare many of them in your program and choose which one to use based on the
command. If we consider:

  go main.go add myfile.txt
  go main.go status
  
we can use [`os.Args`](https://golang.org/pkg/os/#pkg-variables) to retrieve the
subcommand (`os.Args[1]`), and then switch on its value to determine the correct
`FlagSet` to use. We then call `<FlagSet>.Parse(os.Args[2:])` to parse the
remainder of the arguments. Example:

```go
package main

import (
	"flag"
	"log"
	"os"
)

func main() {
	addCommand := flag.NewFlagSet("add", flag.ExitOnError)
	statusCommand := flag.NewFlagSet("status", flag.ExitOnError)

	// foo := addCommand.String(...)
	// bar := statusCommand.Int(...)

	if len(os.Args) < 2 {
		log.Fatal("subcommand is required")
	}

	switch os.Args[1] {
	case "add":
		// do addCommand stuff
		addCommand.Parse(os.Args[2:])
	case "status":
		// do statusCommand stuff
		statusCommand.Parse(os.Args[2:])
	default:
		log.Fatal("invalid subcommand provided")
	}
}
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

## `hashlib`

The `crypto` package in the standard library supports [SHA1](https://golang.org/pkg/crypto/sha1/)

```go
package main

import (
	"bytes"
	"crypto/sha1"
	"fmt"
	"io"
	"log"
)

func main() {
	contents := []byte(`
contents of my committed file
`)
	r := bytes.NewReader(contents)
	h := sha1.New()
	if _, err := io.Copy(h, r); err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%x", h.Sum(nil))
}
```

## `os`

Go has an aptly named [`os`](https://golang.org/pkg/os/) package too!

## `re`

Go's [`regexp`](https://golang.org/pkg/regexp/) is equivalent.

## `sys`

The Python implementation uses this to access unnamed command line arguments. We
can use [`os.Args`](https://golang.org/pkg/os/#pkg-variables) in the `os`
package to parse arguments like the subcommand (i.e. `add`, `status`, `init`)

## `zlib`

Go's [`zlib`](https://golang.org/pkg/compress/zlib/) package in the standard
library is equivalent.
