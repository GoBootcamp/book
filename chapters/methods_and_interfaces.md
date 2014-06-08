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

func (u User) Greeting() string {
	return fmt.Sprintf("Dear %s %s", u.FirstName, u.LastName)
}

func main() {
	u := User{"Matt", "Aimonetti"}
	fmt.Println(u.Greeting())
}
```

[See in playground](http://play.golang.org/p/ITVfJkCiwk)


Note how methods are defined outside of the struct, if you have
been writing Object Oriented code for a while, you might find that a
bit odd at first. The method on the `User` type could de defined
anywhere in the package.

[Go tour page](http://tour.golang.org/#52)

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
// try to keep the lowest amount of structs per file when possible.
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
\label{sec:method_receivers}

Methods can be associated with a named type (`User` for instance) or a pointer to a named type (`*User`).
In the two type aliasing examples above, methods were defined on the
value types (`MyStr` and `MyFloat`).

There are two reasons to use a pointer receiver.
First, to avoid copying the value on each method call (more efficient if the value type is a large struct).
The above example would have been better written as follows:

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

[See in playground](http://play.golang.org/p/tEVN-vhyAi)

Remember that Go passes everything by value, meaning that when
`Greeting()` is defined on the value type, every time you call
`Greeting()`, you are copying the `User` struct. Instead when using a
pointer, only the pointer is copied (cheap).


The other reason why you might want to use a pointer is so that the method can modify the value that its receiver points to.

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := &Vertex{3, 4}
	v.Scale(5)
	fmt.Println(v, v.Abs())
}
```

[See in playground](http://play.golang.org/p/F-PI1fj5AZ)

In the example above, `Abs()` could be defined on the value type or the
pointer since the method doesn't modify the receiver value (the vertex).
However `Scale()` has to be defined on a pointer since it does modify the receiver.
`Scale()` resets the values of the `X` and `Y` fields.


[Go tour page](http://tour.golang.org/#54)

## Interfaces
\label{sec:interfaces}

An interface type is defined by a set of methods.
A value of interface type can hold any value that implements those methods.

Here is a refactored version of our earlier example.
This time we made the greeting feature more generic by defining a
function called `Greet` which takes a param of interface type `Namer`.
`Namer` is a new interface we defined which only defines one method:
`Name()`. So `Greet()` will accept as param any value which has a
`Name()` method defined.

To make our `User` struct implement the interface, we defined
a `Name()` method. We can now call `Greet` and pass our pointer to `User` type.

```go
package main

import (
	"fmt"
)

type User struct {
	FirstName, LastName string
}

func (u *User) Name() string {
	return fmt.Sprintf("%s %s", u.FirstName, u.LastName)
}

type Namer interface {
	Name() string
}

func Greet(n Namer) string {
	return fmt.Sprintf("Dear %s", n.Name())
}

func main() {
	u := &User{"Matt", "Aimonetti"}
	fmt.Println(Greet(u))
}
```

[See in playground](http://play.golang.org/p/aXNaPqMbpV)


We could now define a new type that would implement the same interface
and our `Greet` function would still work.

```go
package main

import (
	"fmt"
)

type User struct {
	FirstName, LastName string
}

func (u *User) Name() string {
	return fmt.Sprintf("%s %s", u.FirstName, u.LastName)
}

type Customer struct {
	Id       int
	FullName string
}

func (c *Customer) Name() string {
	return c.FullName
}

type Namer interface {
	Name() string
}

func Greet(n Namer) string {
	return fmt.Sprintf("Dear %s", n.Name())
}

func main() {
	u := &User{"Matt", "Aimonetti"}
	fmt.Println(Greet(u))
	c := &Customer{42, "Francesc"}
	fmt.Println(Greet(c))
}
```

[See in playground](http://play.golang.org/p/16TYeeXHp5)

[Go tour page](http://tour.golang.org/#55)

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

[See in playground](http://play.golang.org/p/vEmswt3Urz)

[Package io](http://golang.org/pkg/io/) defines Reader and Writer; you don't have to.

[Go tour page](http://tour.golang.org/#56)


## Errors
\label{sec:errors}

An error is anything that can describe itself as an error string. The idea is captured by the predefined, built-in interface type, error, with its single method, `Error`, returning a string:

```go
type error interface {
    Error() string
}
```

The `fmt` package's various print routines automatically know to call the method when asked to print an error.

```go
package main

import (
    "fmt"
    "time"
)

type MyError struct {
    When time.Time
    What string
}

func (e *MyError) Error() string {
    return fmt.Sprintf("at %v, %s",
        e.When, e.What)
}

func run() error {
    return &MyError{
        time.Now(),
        "it didn't work",
    }
}

func main() {
    if err := run(); err != nil {
        fmt.Println(err)
    }
}
```

[Go tour page](http://tour.golang.org/#57)


## Exercise: Errors
\label{sec:exercise_errors}

[Online assignment](http://tour.golang.org/#58)

Copy your `Sqrt` function from the earlier exercises (Section~\ref{sec:exercise_loops_and_funcs}) and modify it to return an `error` value.


`Sqrt` should return a non-nil error value when given a negative number, as it doesn't support complex numbers.

Create a new type

```go
type ErrNegativeSqrt float64
```

and make it an error by giving it a

```go
func (e ErrNegativeSqrt) Error() string
```

method such that `ErrNegativeSqrt(-2).Error()` returns

`cannot Sqrt negative number: -2`.

Note: a call to `fmt.Print(e)` inside the Error method will send the program into an infinite loop. You can avoid this by converting e first: `fmt.Print(float64(e))`. Why?

Change your `Sqrt` function to return an `ErrNegativeSqrt` value when given a negative number.


### Solution

```go
package main

import (
	"fmt"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprintf("cannot Sqrt negative number: %g", float64(e))
}

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, ErrNegativeSqrt(x)
	}

	z := 1.0
	for i := 0; i < 10; i++ {
		z = z - ((z*z)-x)/(2*z)
	}
	return z, nil
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}
```

[See in playground](http://play.golang.org/p/gY0vYSRvSL)

**Tip**: When doing an inferred declaration of a float, you can omit the
decimal value and do the following:

```go
z := 1.
// same as
// z := 1.0
```
