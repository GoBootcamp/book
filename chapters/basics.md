#The Basics
\label{cha:basics}

Go is often referred to as a "simple" programming language, a language that can be
learned in a few hours if you already know another language.
Go was designed to feel familiar and to stay as simple as possible,
[the entire language specification](http://golang.org/ref/spec) fits
in just a few pages.

There are a few concepts we are going to explore before writing our
first application.

## Variables & inferred typing
\label{sec:variables}

The var statement declares a list of variables with the type declared last.

```go
var (
	name     string
	age      int
	location string
)
```

Or even

```go
var (
	name, location  string
	age             int
)
```


Variables can also be declared one by one:

```go
var name     string
var age      int
var location string
```

A var declaration can include initializers, one per variable.

```go
var (
	name     string = "Prince Oberyn"
	age      int    =  32
	location string = "Dorne"
)
```

If an initializer is present, the type can be omitted, the variable will take the type of the initializer (inferred typing).

```go
var (
	name     = "Prince Oberyn"
	age      =  32
	location = "Dorne"
)
```

You can also initialize variables on the same line:

```go
var (
	name, location, age = "Prince Oberyn", "Dorne", 32
)
```

Inside a function, the `:=`  short assignment statement can be used in place of a var declaration with implicit type.

```go
func main() {
	name, location := "Prince Oberyn", "Dorne"
	age := 32
	fmt.Printf("%s (%d) of %s", name, age, location)
}
```

[See in Playground](http://play.golang.org/p/TenQy6FMQS)

A variable can contain any type, including functions:

```go
func main() {
	action := func() {
		//doing something
	}
	action()
}
```

[See in Playground](http://play.golang.org/p/S0Gq-tSESX)

Outside a function, every construct begins with a keyword (`var`,
`func`, and so on) and the `:=` construct is not available.


* [Go's declaration
Syntax](http://blog.golang.org/gos-declaration-syntax)


## Constants
\label{sec:constants}

Constants are declared like variables, but with the `const` keyword.

Constants can only be character, string, boolean, or numeric values
and cannot be declared using the `:=` syntax.
An untyped constant takes the type needed by its context.

```go
const Pi = 3.14
const (
        StatusOK                   = 200
        StatusCreated              = 201
        StatusAccepted             = 202
        StatusNonAuthoritativeInfo = 203
        StatusNoContent            = 204
        StatusResetContent         = 205
        StatusPartialContent       = 206
)
```

```go
package main

import "fmt"

const (
	Pi    = 3.14
	Truth = false
	Big   = 1 << 62
	Small = Big >> 61
)

func main() {
	const Greeting = "ハローワールド"
	fmt.Println(Greeting)
	fmt.Println(Pi)
	fmt.Println(Truth)
}
```

[See in Playground](http://play.golang.org/p/fPlqsffS-J)


## Printing Constants and Variables

While you can print the value of a variable or constant using the built-in
`print` and `println` functions, the more idiomatic and flexible way is to use the
[`fmt` package](http://golang.org/pkg/fmt/)

```go
func main() {
	cylonModel := 6
	fmt.Println(cylonModel)
}
```

`fmt.Println` prints the passed in variables' values and appends a newline.
`fmt.Printf` is used when you want to print one or multiple values
using a defined format specifier.

```go
func main() {
	name := "Caprica-Six"
	aka := fmt.Sprintf("Number %d", 6)
	fmt.Printf("%s is also known as %s",
		name, aka)
}
```

[See in Playground](http://play.golang.org/p/sTZgAH7TWn)

Read the [`fmt` package documentation](http://golang.org/pkg/fmt/#hdr-Printing) to see the available flags to create a format specifier.


## Packages and imports
\label{sec:packages}

Every Go program is made up of packages.
Programs start running in package `main`.

```go
package main

func main() {
	print("Hello, World!\n")
}
```

If you are writing an executable code (versus a library), then you need
to define a `main` package and a `main()` function which will be
the entry point to your software.

By convention, the package name is the same as the last element of the import path. For instance, the "math/rand" package comprises files that begin with the statement package rand.

Import statement examples:

```go
import "fmt"
import "math/rand"
```

Or grouped:

```go
import (
    "fmt"
    "math/rand"
)
```

* [Go Tour: Packages](https://tour.golang.org/basics/1)
* [Go Tour: Imports](https://tour.golang.org/basics/2)


Usually, non standard lib packages are namespaced using a web
url. For instance, I ported to Go some Rails logic, including the cryptography code used in
Rails 4. I hosted the source code containing a few packages on github, in the following repository [http://github.com/mattetti/goRailsYourself](http://github.com/mattetti/goRailsYourself)

To import the crypto package, I would need to use the following import
statement:

```
import "github.com/mattetti/goRailsYourself/crypto"
```

## Code location
\label{sec:code_location}

The snippet above basically tells the compiler to import the crypto
package available at the `github.com/mattetti/goRailsYourself/crypto`
path. It doesn't mean that the compiler will automatically pull down the
repository, so where does it find the code?


You need to pull down the code yourself. The easiest way is to use the
`go get` command provided by Go.

```bash
$ go get github.com/mattetti/goRailsYourself/crypto
```

This command will pull down the code and put it in your Go path.
When installing Go, we set the `GOPATH` environment variable and that is
what's used to store binaries and libraries. That's also where you
should store your code (your workspace).

```bash
$ ls $GOPATH
bin	pkg	src
```

The `bin` folder will contain the Go compiled binaries. You
should probably add the bin path to your system path.

The `pkg` folder contains the compiled versions of the available libraries
so the compiler can link against them without recompiling them.

Finally the `src` folder contains all the Go source code organized by
import path:

```bash
$ ls $GOPATH/src
bitbucket.org	code.google.com	github.com	launchpad.net
```

```bash
$ ls $GOPATH/src/github.com/mattetti
goblin			goRailsYourself		jet
```

When starting a new program or library, it is recommended to do so
inside the `src` folder, using a fully qualified path (for instance:
`github.com/<your username>/<project name>`)


## Exported names
\label{sec:exported_names}

After importing a package, you can refer to the names it exports
(meaning variables, methods and functions that are available from
outside of the package).
In Go, a name is exported if it begins with a capital letter.
`Foo` is an exported name, as is `FOO`. The name `foo` is not exported.

See the difference between:

```go
import (
    "fmt"
    "math"
)

func main() {
    fmt.Println(math.pi)
}
```

and

```go
func main() {
    fmt.Println(math.Pi)
}
```

`Pi` is exported and can be accessed from outside the package, while `pi`
isn't available.


```
cannot refer to unexported name math.pi
```

* [See in Playground](http://play.golang.org/p/Fsanbxo-A2)

Use the provided Go [documentation](http://golang.org/pkg/) or
[godoc.org](http://godoc.org/) to find exported names.

* [Exported names example](http://play.golang.org/p/5y_evW6jiS)


## Functions, signature, return values, named results
\label{sec:functions}

A function can take zero or more typed arguments.
The type comes after the variable name.
Functions can be defined to return any number of values that are always typed.


```go
package main

import "fmt"

func add(x int, y int) int {
    return x + y
}

func main() {
    fmt.Println(add(42, 13))
}
```

* [Go Tour Functions example](https://tour.golang.org/basics/4)

In the following example, instead of declaring the type of each parameter,
we only declare one type that applies to both.


```go
package main

import "fmt"

func add(x, y int) int {
    return x + y
}

func main() {
    fmt.Println(add(42, 13))
}
```


* [See in Playground](http://play.golang.org/p/AqCI22fz91)

In the following example, the `location` function returns two string values.

```go
func location(city string) (string, string) {
	var region string
	var continent string

	switch city {
	case "Los Angeles", "LA", "Santa Monica":
		region, continent = "California", "North America"
	case "New York", "NYC":
		region, continent = "New York", "North America"
	default:
		region, continent = "Unknown", "Unknown"
	}
	return region, continent
}

func main() {
	region, continent := location("Santa Monica")
	fmt.Printf("Matt lives in %s, %s", region, continent)
}
```


* [See in playground](http://play.golang.org/p/e265ckMmSC)

Functions take parameters. In Go, functions can return multiple "result parameters", not just a single value. They can be named and act just like variables.

If the result parameters are named, a return statement without arguments returns the current values of the results.


```go
func location(city string) (region, continent string) {
	switch city {
	case "Los Angeles", "LA", "Santa Monica":
		region, continent = "California", "North America"
	case "New York", "NYC":
		region, continent = "New York", "North America"
	default:
		region, continent = "Unknown", "Unknown"
	}
	return
}

func main() {
	region, continent := location("Santa Monica")
	fmt.Printf("Matt lives in %s, %s", region, continent)
}
```

* [See in Playground](http://play.golang.org/p/aX92mw7USF)

I personally recommend against using named return parameters because
they often cause more confusion than they save time or help clarify your
code.


## Pointers
\label{sec:pointers}

Go has pointers, but no pointer arithmetic.
Struct fields can be accessed through a struct pointer. The indirection through the pointer is transparent (you can directly call fields and methods on a pointer).

Note that by default Go passes arguments by value (copying the
arguments), if you want to pass the arguments by reference, you need
to pass pointers (or use a structure using reference values like
slices (Section~\ref{sec:slices}) and maps (Section~\ref{sec:maps}).

To get the pointer of a value, use the `&` symbol in front of the
value; to dereference a
pointer, use the `*` symbol.

Methods are often defined on pointers and not values (although they can
be defined on both), so you will often store a pointer in a variable as
in the example below:

```go
client := &http.Client{}
resp, err := client.Get("http://gobootcamp.com")
```


## Mutability
\label{sec:mutability}

In Go, only constants are immutable. However because arguments are
passed by value, a function receiving a value argument and mutating it, won't
mutate the original value.

```go
package main

import "fmt"

type Artist struct {
	Name, Genre string
	Songs       int
}

func newRelease(a Artist) int {
	a.Songs++
	return a.Songs
}

func main() {
	me := Artist{Name: "Matt", Genre: "Electro", Songs: 42}
	fmt.Printf("%s released their %dth song\n", me.Name, newRelease(me))
	fmt.Printf("%s has a total of %d songs", me.Name, me.Songs)
}
```

```
Matt released their 43th song
Matt has a total of 42 songs
```

[See in Playground](http://play.golang.org/p/_hcMO1tI8C)

As you can see the total amount of songs on the `me` variable's value
wasn't changed.
To mutate the passed value, we need to pass it by reference, using a
pointer.

```go
package main

import "fmt"

type Artist struct {
	Name, Genre string
	Songs       int
}

func newRelease(a *Artist) int {
	a.Songs++
	return a.Songs
}

func main() {
	me := &Artist{Name: "Matt", Genre: "Electro", Songs: 42}
	fmt.Printf("%s released their %dth song\n", me.Name, newRelease(me))
	fmt.Printf("%s has a total of %d songs", me.Name, me.Songs)
}
```

[See in Playground](http://play.golang.org/p/FaWFYCZmfh)

The only change between the two versions is that `newRelease` takes a
pointer to an `Artist` value and when I initialize our `me` variable, I
used the `&` symbol to get a pointer to the value.

Another place where you need to be careful is when calling methods on
values as explained a bit later (Section~\ref{sec:method_receivers})
