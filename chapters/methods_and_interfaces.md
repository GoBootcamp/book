#Methods and interfaces
\label{cha:methods_and_interfaces}

While technically Go isn't an [Object Oriented Programming
language](http://en.wikipedia.org/wiki/Object-oriented_programming),
types and methods allow for an object-oriented style of programming.
The big difference is that Go does not support type inheritance but
instead has a concept of interface.

In this chapter, we will focus on Go's use of methods and interfaces.

## Methods
\label{sec:methods}

*Note*: A frequently asked question is "what is the difference between
a function and a method". A method is a function that has a defined
receiver, in OOP terms, a method is a function on an instance of an
object.

Go does not have classes. However, you can define methods on struct types.

The *method receiver* appears in its own argument list between the `func` keyword and the method name. Here is an example with a `User` struct containing two fields: `FirstName` and `LastName` of string type.


```go
package main

import (
	"fmt"
)

type User struct {
	FirstName, LastName string
}

func (u *User) Greeting() string {
	return fmt.Sprintf("Dear %s %s", u.FirstName, u.LastName)
}

func main() {
	u := &User{"Matt", "Aimonetti"}
	fmt.Println(u.Greeting())
}
```

Note how methods are defined outside of the struct, if you have
been writing Object Oriented code for a while, you might find that a
bit odd at first.

### Code organization

Methods can be defined on any file in the package, but my
recommendation is to organize the code as shown below:


```go
package models

// list of packages to import
import (
	"fmt"
)

// list of constants
const (
	ConstExample = "const before vars"
)

// list of variables
var (
	ExportedVar    = 42
	nonExportedVar = "so say we all"
)

// Main type(s) for the file,
// try to keep to keep the lowest amount of structs per file when possible.
type User struct {
	FirstName, LastName string
	Location            *UserLocation
}

type UserLocation struct {
	City    string
	Country string
}

// List of functions
func NewUser(firstName, lastName string) *User {
	return &User{FirstName: firstName,
		LastName: lastName,
		Location: &UserLocation{
			City:    "Santa Monica",
			Country: "USA",
		},
	}
}

// List of methods
func (u *User) Greeting() string {
	return fmt.Sprintf("Dear %s %s", u.FirstName, u.LastName)
}
```
[Exercise](http://tour.golang.org/#52)


In fact, you can define a method on `any` type you define in your package, not just structs.
You cannot define a method on a type from another package, or on a basic type.

### Type aliasing

To define methods on a type you don't "own", you need to define an alias
for the type you want to extend:

```go
package main

import (
	"fmt"
	"strings"
)

type MyStr string

func (s MyStr) Uppercase() string {
	return strings.ToUpper(string(s))
}

func main() {
	fmt.Println(MyStr("test").Uppercase())
}
```

[Playground Example](http://play.golang.org/p/PIa4nYfDMm)

```go
package main

import (
    "fmt"
    "math"
)

type MyFloat float64

func (f MyFloat) Abs() float64 {
    if f < 0 {
        return float64(-f)
    }
    return float64(f)
}

func main() {
    f := MyFloat(-math.Sqrt2)
    fmt.Println(f.Abs())
}
```

[Playground Example](http://tour.golang.org/#53)


### Method receivers

Methods can be associated with a named type or a pointer to a named type.
In the two type aliasing examples above, methods were defined on the
value types (`MyStr` and `MyFloat`) but in the earlier example the
method was defined on the `*User` pointer type.

There are two reasons to use a pointer receiver.
First, to avoid copying the value on each method call (more efficient if the value type is a large struct).
Second, so that the method can modify the value that its receiver points to.

[Example](http://tour.golang.org/#54)

## Interfaces
\label{sec:interfaces}

An interface type is defined by a set of methods.

A value of interface type can hold any value that implements those methods.

[Example](http://tour.golang.org/#55)

### Interfaces are satisfied implicitly

A type implements an interface by implementing the methods.

*There is no explicit declaration of intent.*

Implicit interfaces decouple implementation packages from the packages that define the interfaces: neither depends on the other.

It also encourages the definition of precise interfaces, because you don't have to find every implementation and tag it with the new interface name.

```go
package main

import (
    "fmt"
    "os"
)

type Reader interface {
    Read(b []byte) (n int, err error)
}

type Writer interface {
    Write(b []byte) (n int, err error)
}

type ReadWriter interface {
    Reader
    Writer
}

func main() {
    var w Writer

    // os.Stdout implements Writer
    w = os.Stdout

    fmt.Fprintf(w, "hello, writer\n")
}
```

[Package io](http://golang.org/pkg/io/) defines Reader and Writer; you don't have to.

[Example](http://tour.golang.org/#56)

## Errors
\label{sec:errors}

## Exercises
\label{sec:exercises}

