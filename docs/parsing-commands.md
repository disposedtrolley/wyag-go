# Parsing Commands

The Python implementation supports a variety of common Git commands, all with a
varying number of positional and named arguments. `init` for instance accepts a
single optional positional argument, defaulting to the current directory (`.`),
while `tag` accepts zero, one, or two arguments.

One of the outstanding design decisions is how we want to delegate the task of
parsing arguments for these commands. We can design all of the entrypoint
functions to share a uniform interface, i.e.:

```go
type Command interface {
    Execute(args []string) error
}

type InitCommand struct {
    dir string
}

func (c *InitCommand) Execute(args []string) error {
    // parse args
}
``` 

or create types to encapsulate each command's unique set of arguments, like so:

```go
type InitCommandArgs struct {
    Dir string
}

func Init(args InitCommandArgs) {
    // initialise repo at args.Dir
}
```

or combine the command's arguments and execution logic in a single struct, where
arguments are provided via a constructor function:

```go
type Command interface {
    func Execute() error
}

type InitCommand struct {
    Dir string
}

func (c *InitCommand) Execute() error {
    // initialise repo at c.Dir
}

func NewInitCommand(dir string) *InitCommand {
    return &InitCommand{
        Dir: dir,
    }
}
```

I think I prefer the last approach, since we can still define a common interface
for the execution of each command
