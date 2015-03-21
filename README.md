# gost [![Build Status](https://drone.io/github.com/gfjalar/gost/status.png)](https://drone.io/github.com/gfjalar/gost/latest) [![Coverage Status](https://coveralls.io/repos/gfjalar/gost/badge.svg?branch=master)](https://coveralls.io/r/gfjalar/gost?branch=master) [![GoDoc](https://godoc.org/github.com/gfjalar/gost?status.png)](http://godoc.org/github.com/piotrgalar/gost)

Gost provides a simple function chaining mechanism. It's main purpose is to make function composition in Go more readable.

### How does it work?
Gost uses named struct Chain which holds an array of interface{} objects, links.

New() function creates a new Chain object where links array is initialised to nil and returns its address.

Pointer method Compose takes variadic number of interface{} objects and appends them to the links array of 
the chain on which it was called.g

Pointer method Then takes variadic number of interface{} objects and prepends them to the links array of
the chain on which it was called. The interfaces{} are put in the reverse order ie. calling Then(a, b)
on an empty chain c will result return a chain with links in the following order [b, a].

Pointer method MergeCompose takes variadic number of pointers to Chain objects. Calling chain.MergeCompose(a, b)
would be equivalent to calling chain.Compose(a.links..., b.links...).

Pointer method MergeThen takes variadic number of pointers to Chain objects. Calling chain.MergeThen(a, b)
would be equivalent to calling chain.Then(b.links..., a.links...)

All the above return a pointer to a new Chain object which ensures chain immutability.

Calling Build on a chain objects executes the chain and returns []interface{} as a result of the chain.
It uses a stack to remember the intermediate results. It analyses the chain links in the reverse order.
Now, if the currently processed link is not a function it puts it on the stack. Otherwise, it takes
the necessary number of arguments from the stack(reverses them to ensure correctness), calls the function
and puts the results on the stack in the reverse order. Once all the links were analysed, it returns the stack
in the reverse order.

Variadic functions are treated a bit differently. Firstly, it's checked if the item on top of the stack is
of type VarArgs which indicates how many arguments should be passed to the variadic function. If it's not there,
as many arguments of the correct type as possible will be taken.

Eg. 
```
stack := [0, 1, 2, "dog", 3, 4] // top -> bottom
function := func(ns ...int) int { return sum ns }
execute(function, stack) // newStack := [3, "dog", 3, 4]
```

Note, that if there are not enough arguments on the stack for the specific function or if the arguments on the stack
aren't of the correct type, the Build will panic.

### Usage

To import:
```go
import "github.com/gfjalar/gost"
```

To start a new chain:
```go
chain := gost.New()
```

To create an expression of form f(g):
```go
chain.Compose(f, g)
```
or
```go
chain.Compose(g).Then(f)
```

To reuse a previously created chains chain1 and chain2:
```go
chain.MergeCompose(chain1, chain2)
```
or
```go
chain.MergeThen(chain1, chain2)
```

To execute the chain:
```go
result := chain.Build()
```