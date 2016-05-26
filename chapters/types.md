#Types
\label{cha:types}

## Basic types
\label{sec:basic_types}

```
bool
string

Numeric types:

uint        either 32 or 64 bits
int         same size as uint
uintptr     an unsigned integer large enough to store the uninterpreted bits of
            a pointer value
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers
            (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts
complex128  the set of all complex numbers with float64 real and imaginary parts

byte        alias for uint8
rune        alias for int32 (represents a Unicode code point)
```

Example of some of the built-in types:

```go
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	goIsFun bool       = true
	maxInt  uint64     = 1<<64 - 1
	complex complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	const f = "%T(%v)\n"
	fmt.Printf(f, goIsFun, goIsFun)
	fmt.Printf(f, maxInt, maxInt)
	fmt.Printf(f, complex, complex)
}
```

```
bool(true)
uint64(18446744073709551615)
complex128((2+3i))
```

* [See in Playground](http://play.golang.org/p/FFLi1IwIis)

## Type conversion
\label{sec:type-conversion}

The expression `T(v)` converts the value `v` to the type `T`.
Some numeric conversions:

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```
Or, put more simply:

```go
i := 42
f := float64(i)
u := uint(f)
```
Go assignment between items of different type requires an explicit conversion which means that you manually need to convert types if you are passing a variable to a function expecting another type.


## Type assertion
\label{sec:type-assertion}

If you have a value and want to convert it
to another or a specific type (in case of `interface{}`), you can use type assertion.
A type assertion takes a value and tries to create another version in
the specified explicit type.

In the example below, the `timeMap` function takes a value
and if it can be asserted as a map of `interfaces{}` keyed by strings,
then it injects a new entry called "updated_at" with the current time
value.

```go
package main

import (
	"fmt"
	"time"
)

func timeMap(y interface{}) {
	z, ok := y.(map[string]interface{})
	if ok {
		z["updated_at"] = time.Now()
	}
}

func main() {
	foo := map[string]interface{}{
		"Matt": 42,
	}
	timeMap(foo)
	fmt.Println(foo)
}
```

[See in playground](http://play.golang.org/p/jNrIZLQ4s8)

The type assertion doesn't have to be done on an empty interface.
It's often used when you have a function taking a param
of a specific interface but the function inner code
behaves differently based on the actual object type.
Here is an example:


```go
package main

import "fmt"

type Stringer interface {
	String() string
}

type fakeString struct {
	content string
}

// function used to implement the Stringer interface
func (s *fakeString) String() string {
	return s.content
}

func printString(value interface{}) {
	switch str := value.(type) {
	case string:
		fmt.Println(str)
	case Stringer:
		fmt.Println(str.String())
	}
}

func main() {
	s := &fakeString{"Ceci n'est pas un string"}
	printString(s)
	printString("Hello, Gophers")

}
```

* [See in Playground](http://play.golang.org/p/69I8PAuoAV)

Another example is when checking if an error is of a certain type:

```go
if err != nil {
  if msqlerr, ok := err.(*mysql.MySQLError); ok && msqlerr.Number == 1062 {
    log.Println("We got a MySQL duplicate :(")
  } else {
    return err
  }
}
```


* [Read more in the Effective Go guide](http://golang.org/doc/effective_go.html#interface_conversions)

## Structs
\label{sec:structs}

A struct is a collection of fields/properties. You can define new
types as structs or interfaces (Chapter~\ref{cha:interfaces}).
If you are coming from an object-oriented background, you can
think of a struct to be a light class that supports composition but
not inheritance. Methods are discussed at length in Chapter~\ref{cha:methods}

You don't need to define getters and setters on struct fields, they can
be accessed automatically. However, note that only exported fields
(capitalized) can
be accessed from outside of a package.

A struct literal sets a newly allocated struct value by listing the values of its fields.
You can list just a subset of fields by using the `"Name:" syntax` (the order of named fields is irrelevant when using this syntax).
The special prefix `&` constructs a pointer to a newly allocated struct.

```go
package main

import (
	"fmt"
	"time"
)

type Bootcamp struct {
	// Latitude of the event
	Lat float64
	// Longitude of the event
	Lon float64
	// Date of the event
	Date time.Time
}

func main() {
	fmt.Println(Bootcamp{
		Lat:  34.012836,
		Lon:  -118.495338,
		Date: time.Now(),
	})
}
```

* [See in Playground](http://play.golang.org/p/qQceF9kimQ)

Declaration of struct literals:

```go
package main

import "fmt"

type Point struct {
	X, Y int
}

var (
	p = Point{1, 2}  // has type Point
	q = &Point{1, 2} // has type *Point
	r = Point{X: 1}  // Y:0 is implicit
	s = Point{}      // X:0 and Y:0
)

func main() {
	fmt.Println(p, q, r, s)
}
```

* [See in playground](http://play.golang.org/p/DOlEpcRSBQ)

Accessing fields using the dot notation:

```go
package main

import (
	"fmt"
	"time"
)

type Bootcamp struct {
	Lat, Lon float64
	Date     time.Time
}

func main() {
	event := Bootcamp{
		Lat: 34.012836,
		Lon: -118.495338,
	}
	event.Date = time.Now()
	fmt.Printf("Event on %s, location (%f, %f)",
		event.Date, event.Lat, event.Lon)

}
```

* [See in Playground](http://play.golang.org/p/QKwafh0TkQ)


## Initializing
\label{sec:initializing_values}

Go supports the `new` expression to allocate a
zeroed value of the requested type and to return a pointer to it.

```go
x := new(int)
```

As seen in (Section~\ref{sec:structs}) a common way to "initialize" a
variable containing a struct or a reference to one, is to create a struct literal. Another option is to create a constructor.
This is usually done when the zero value isn't good enough and you need
to set some default field values for instance.

Note that following expressions using `new` and an empty struct literal are equivalent and result in the same
kind of allocation/initialization:

```go
package main

import (
	"fmt"
)

type Bootcamp struct {
	Lat float64
	Lon float64
}

func main() {
	x := new(Bootcamp)
	y := &Bootcamp{}
	fmt.Println(*x == *y)
}
```

* [See in playground](http://play.golang.org/p/XgECtFpCw6)


Note that slices (Section~\ref{sec:slices}), maps (Section~\ref{sec:maps}) and channels (Section~\ref{sec:channels}) are usually allocated using `make`
so the data structure these types are built upon can be initialized.

Resources:

* [Allocation with `new` - effective Go](http://golang.org/doc/effective_go.html#allocation_new)
* [Composite Literals - effective Go](http://golang.org/doc/effective_go.html#composite_literals)
* [Allocation with `make` - effective Go](http://golang.org/doc/effective_go.html#allocation_make)



## Composition vs inheritance
\label{sec:struct_composition}

Coming from an [OOP](http://en.wikipedia.org/wiki/Object-oriented_programming) background a lot of us are used to inheritance, something that isn't supported by Go.
Instead you have to think in terms of composition and interfaces.

The Go team wrote a [short but good segment](http://golang.org/doc/effective_go.html#embedding) on this topic.

Composition (or embedding) is a well understood concept for most OOP programmers and Go supports it,
here is an example of the problem it's solving:

```go
package main

import "fmt"

type User struct {
	Id       int
	Name     string
	Location string
}

type Player struct {
	Id       int
	Name     string
	Location string
	GameId	 int
}

func main() {
	p := Player{}
	p.Id = 42
	p.Name = "Matt"
	p.Location = "LA"
	p.GameId = 90404
	fmt.Printf("%+v", p)
}
```

* [See in Playground](https://play.golang.org/p/Mqtjn5PPsW)


The above example demonstrates a classic OOP challenge,
our `Player` struct has the same fields as the `User` struct
but it also has a `GameId` field. Having to duplicate the field names
isn't a big deal, but it can be simplified by composing our struct.

```go
type User struct {
	Id             int
	Name, Location string
}

type Player struct {
	User
	GameId int
}
```

We can initialize a new variable of type `Player` two different ways.

Using the dot notation to set the fields:

```go
package main

import "fmt"

type User struct {
	Id             int
	Name, Location string
}

type Player struct {
	User
	GameId int
}

func main() {
	p := Player{}
	p.Id = 42
	p.Name = "Matt"
	p.Location = "LA"
	p.GameId = 90404
	fmt.Printf("%+v", p)
}
```

* [See in Playground](http://play.golang.org/p/kR-Cue8816)

The other option is to use a struct literal:

```go
package main

import "fmt"

type User struct {
	Id             int
	Name, Location string
}

type Player struct {
	User
	GameId int
}

func main() {
	p := Player{
		User{Id: 42, Name: "Matt", Location: "LA"},
		90404,
	}
	fmt.Printf(
		"Id: %d, Name: %s, Location: %s, Game id: %d\n",
		p.Id, p.Name, p.Location, p.GameId)
	// Directly set a field defined on the Player struct
	p.Id = 11
	fmt.Printf("%+v", p)
}
```

* [See in Playground](http://play.golang.org/p/wscd8inj9t)

When using a struct literal with an implicit composition, we can't just pass the composed fields.
We instead need to pass the types composing the struct.
Once set, the fields are directly available.

Because our struct is composed of another struct, the methods on the
`User` struct is also available to the `Player`.
Let's define a method to show that behavior:

```go
package main

import "fmt"

type User struct {
	Id             int
	Name, Location string
}

func (u *User) Greetings() string {
	return fmt.Sprintf("Hi %s from %s",
		u.Name, u.Location)
}

type Player struct {
	User
	GameId int
}

func main() {
	p := Player{}
	p.Id = 42
	p.Name = "Matt"
	p.Location = "LA"
	fmt.Println(p.Greetings())
}
```

* [See in Playground](http://play.golang.org/p/q_KFeTLwVX)

As you can see this is a very powerful way to build data structures but
it's even more interesting when thinking about it in the context of
interfaces. By composing one of your structure with one implementing a
given interface, your structure automatically implements the interface.

Here is another example, this time we will look at implementing a `Job`
struct that can also behave as a [logger](http://golang.org/pkg/log/#Logger).

Here is the explicit way:

```go
package main

import (
	"log"
	"os"
)

type Job struct {
	Command string
	Logger  *log.Logger
}

func main() {
	job := &Job{"demo", log.New(os.Stderr, "Job: ", log.Ldate)}
	// same as
	// job := &Job{Command: "demo",
	//            Logger: log.New(os.Stderr, "Job: ", log.Ldate)}
	job.Logger.Print("test")
}
```

* [See in playground](http://play.golang.org/p/3yYJadlmHS)


Our `Job` struct has a field called `Logger` which is a pointer to
another type ([log.Logger](http://golang.org/pkg/log/#Logger))

When we initialize our value, we set the logger so we can then call its
`Print` function by chaining the calls: `job.Logger.Print()`


But Go lets you go even further and use implicit composition.
We can skip defining the field for our logger and now all the
methods available on a pointer to `log.Logger` are available from our
struct:

```go
package main

import (
	"log"
	"os"
)

type Job struct {
	Command string
	*log.Logger
}

func main() {
	job := &Job{"demo", log.New(os.Stderr, "Job: ", log.Ldate)}
	job.Print("starting now...")
}
```

* [See in Playground](http://play.golang.org/p/mq3r9H9szz)

Note that you still need to set the logger and that's often a good
reason to use a constructor (custom constructors are used when you need
to set a structure before using a value, see (Section~\ref{sec:custom_constructors}) ).
What is really nice with the implicit composition is that it allows to
easily and cheaply make your structs implement interfaces.
Imagine that you have a function that takes variables implementing an
interface with the `Print` method. By adding `*log.Logger` to your
struct (and initializing it properly), your struct is now implementing
the interface without you writing any custom methods.


## Exercise

Looking at the `User` / `Player` example, you might have noticed that we
composed `Player` using `User` but it might be better to compose it with
a pointer to a `User` struct.
The reason why a pointer might be better is because in Go, arguments are
passed by value and not reference. If you have a small struct that is
inexpensive to copy, that is fine, but more than likely, in real life,
our `User` struct will be bigger and should not be copied. Instead we
would want to pass by reference (using a pointer).
(Section~\ref{sec:mutability} & Section~\ref{sec:method_receivers} discuss more in depth how calling
a method on a type value vs a pointer affects mutability and memory
allocation)

Modify the code to use a pointer but still
be able to initialize without using the dot notation.

```go
package main

import "fmt"

type User struct {
	Id             int
	Name, Location string
}

func (u *User) Greetings() string {
	return fmt.Sprintf("Hi %s from %s",
		u.Name, u.Location)
}

type Player struct {
	*User
	GameId int
}

func main() {
	// insert code
}
```

* [See in Playground](http://play.golang.org/p/Ia3-RiZ_ac)

**Question:** We defined the `Greetings` method on a pointer to a `User`
type. How come we were able to call it directly on the value?


### Solution

```go
package main

import "fmt"

type User struct {
	Id             int
	Name, Location string
}

func (u *User) Greetings() string {
	return fmt.Sprintf("Hi %s from %s",
		u.Name, u.Location)
}

type Player struct {
	*User
	GameId int
}

func NewPlayer(id int, name, location string, gameId int) *Player {
	return &Player{
		User:   &User{id, name, location},
		GameId: gameId,
	}
}

func main() {
	p := NewPlayer(42, "Matt", "LA", 90404)
	fmt.Println(p.Greetings())
}
```

* [See in Playground](http://play.golang.org/p/tpNVNC8hnX)

**Answer:** That is because methods defined on a pointer are also
automatically available on the value itself. The example didn't use a
pointer on purpose, so the dot notation was working right away. In the
pointer solution, a zero value player is composed of a nil pointer of
type `User` and therefore, we can't call a field on a nil pointer.
