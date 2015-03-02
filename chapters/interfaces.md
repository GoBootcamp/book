# Interfaces
\label{cha:interfaces}

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


## Interfaces are satisfied implicitly

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

[Package io](http://golang.org/pkg/io/) defines Reader and Writer so you don't have to.


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

[See in Playground](http://play.golang.org/p/z50pUnAe4q)


## Exercise: Errors
\label{sec:exercise_errors}

[Online assignment](https://tour.golang.org/methods/9)

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
