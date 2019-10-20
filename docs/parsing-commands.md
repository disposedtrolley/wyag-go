# Parsing Commands

The Python implementation supports a variety of common Git commands, all with a
varying number of positional and named arguments. `init` for instance accepts a
single optional positional argument, defaulting to the current directory (`.`),
while `tag` accepts zero, one, or two arguments.

One of the outstanding design decisions is how we want to delegate the task of
parsing arguments for these commands. The design should allow for commands to be
easily added in the future, with minimal change to the existing parsing logic.

We can structure all of the entrypoint functions to share a uniform interface,
i.e.:

```go
func Init(args []string) error {
    // parse args and init repo
}

func main() {
   switch os.Args[1] {
       case "init":
           err := Init(os.Args[2:])
       ...
   } 
}
``` 

Responsibility of parsing the subset of arguments related to the function is
delegated to the function itself. This means when we want to add a command in
the future, the only bit of existing code which needs to be altered is the
`switch` statement.

I played around with the idea of implementing an common `interface` across
commands, but it resulted in overengineered code:

```go
type Command interface {
    Execute(args []string) error
}

type Init struct {}

func (c *Init) Execute(args []string) error {
    // parse args and init repo
}

func main() {
    switch os.Args[1] {
        case "init":
            command := &Init{}
            err := command.Execute(os.Args[2:])
    }
}
```

Most importantly, the idea of an `interface` doesn't help here, as commands are
mutually exclusive and more than one command cannot be run in the same
invocation of the program.
