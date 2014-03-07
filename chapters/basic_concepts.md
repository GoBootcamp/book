#Basic concepts
\label{cha:basic_concepts}

Go is often referred as a "simple" programming language, a language that can be
learned in a few hours if you already know another language.
Go was designed to feel familiar and to stay as simple as possible,
[the entire language specification](http://golang.org/ref/spec) fits
in just a few pages.

Here are the concepts we are going to explore before writing our
first application. To walk through these various concepts, we are
going to use Go's official [Tour of Go](http://tour.golang.org/) web
application. 

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

If you writing an executable code (versus a library), then you need
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

Usually, non standard lib packages are namespaced using a web
url, for instance I ported to Go some Rails logic, including the cryptography code used in
Rails 4. I hosted the source code containing a few packages on github, in the following repository [http://github.com/mattetti/goRailsYourself](http://github.com/mattetti/goRailsYourself)

To import the crypto package, I would need to use the following import
statement:

```
import "github.com/mattetti/goRailsYourself/crypto"
```

We will see later on how to use the `go get` command to fetch the remote code before compiling our code.

* [Go tour page](http://tour.golang.org/#4)
* [Go tour page 2](http://tour.golang.org/#5)

## Exported names
\label{sec:exported_names}

After importing a package, you can refer to the names it exports.
In Go, a name is exported if it begins with a capital letter.
`Foo` is an exported name, as is `FOO`. The name `foo` is not exported.

See the difference bewteen:

```go
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

`Pi` is exported and can be accessed from outside the page, while `pi`
isn't available.


```
cannot refer to unexported name math.pi
```

* [Exported names example](http://tour.golang.org/#6)

## Functions, signature, return values, named results
\label{sec:functions}

A function can take zero or more typed arguments.
The type comes after the variable name. 
Functions can be defined to return any number of values. 
Return values are always typed.


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

* [Function example](http://tour.golang.org/#7)

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


* [Function with arguments sharing the same type](http://tour.golang.org/#8)

In this example, the `swap` function return two string values.

```go
package main

import "fmt"

func swap(x, y string) (string, string) {
    return y, x
}

func main() {
    a, b := swap("hello", "world")
    fmt.Println(a, b)
}
```


* [Function with multiple results](http://tour.golang.org/#9)

Functions take parameters. In Go, functions can return multiple "result parameters", not just a single value. They can be named and act just like variables.

If the result parameters are named, a return statement without arguments returns the current values of the results.

```go
package main

import "fmt"

func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return
}

func main() {
    fmt.Println(split(17))
}
```

I personally recommend against using named return parameters because
they often cause more confusion than they save time or help clarify your
code.

* [Function with named results](http://tour.golang.org/#10)

### Resources

* [Go's declaration
Syntax](http://blog.golang.org/gos-declaration-syntax)

## Variables / inferred typing, short assignment
\label{sec:variables}

The var statement declares a list of variables; as in function argument lists, the type is last.
A var declaration can include initializers, one per variable.
If an initializer is present, the type can be omitted; the variable will take the type of the initializer.
Inside a function, the `:=` short assignment statement can be used in place of a var declaration with implicit type.
Outside a function, every construct begins with a keyword (`var`,
`func`, and so on) and the `:=` construct is not available.

* [Variables example](http://tour.golang.org/#11)
* [Initialization of variables](http://tour.golang.org/#12)
* [Short variable declaration: `:=`](http://tour.golang.org/#13)


## Basic types
\label{sec:basic_types}

```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

* [Basic types example](http://tour.golang.org/#14)

## Type conversion
\label{sec:type-conversion}

The expression T(v) converts the value v to the type T.
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
Go assignment between items of different type requires an explicit conversion. 

* [Type conversion example](http://tour.golang.org/#15)


## Constants
\label{sec:constants}

Constants are declared like variables, but with the `const` keyword.
Constants can be character, string, boolean, or numeric values.
Constants cannot be declared using the `:=` syntax.
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

* [Constants example](http://tour.golang.org/#16)
* [Numeric Constants](http://tour.golang.org/#17)

## For Loop
\label{sec:for-loop}

Go has only one looping construct, the for loop.
The basic for loop looks as it does in C or Java, except that the ( ) are gone (they are not even optional) and the { } are required.
As in C or Java, you can leave the pre and post statements empty.


* [For loop example](http://tour.golang.org/#18)

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

* [For loop without pre/post statements](http://tour.golang.org/#19)

```go
sum := 1
for ; sum < 1000; {
    sum += sum
}
```

* [For loop as a `while` loop](http://tour.golang.org/#20)

```go
sum := 1
for sum < 1000 {
    sum += sum
}
```

* [Infinite loop](http://tour.golang.org/#21)

```go
for {
  // do something in a loop forever
}
```


## If statement
\label{sec:if-statement}

The `if` statement looks as it does in C or Java, except that the `( )`
are gone and the `{ }` are required. Like `for`, the `if` statement can start with a short statement to execute before the condition.
Variables declared by the statement are only in scope until the end of
the `if`.
Variables declared inside an if short statement are also available inside any of the else blocks.

* [`If` statement example](http://tour.golang.org/#22)


```go
if answer != 42 {
    return "Wrong answer"
}
```


* [`If` with a short statement](http://tour.golang.org/#23)


```go
if err := foo(); err != nil {
    panic(err)
}
```

* [`If` and `else` example](http://tour.golang.org/#24)


## Exercise: Loops and Functions

[Assignment](http://tour.golang.org/#25)

As a simple way to play with functions and loops, implement the square root function using Newton's method.

In this case, Newton's method is to approximate `Sqrt(x`) by picking a starting point `z` and then repeating:

\( z = z - \frac{z2 - x}{2z} \)

To begin with, just repeat that calculation 10 times and see how close you get to the answer for various values (1, 2, 3, ...).

Next, change the loop condition to stop once the value has stopped changing (or only changes by a very small delta). See if that's more or fewer iterations. How close are you to the math.Sqrt?

Hint: to declare and initialize a floating point value, give it floating point syntax or use a conversion:

```go
z := float64(1)
z := 1.0
```

Work on the exercises and check the provided solution below.

**Solutions**:

Repeat that calculation 10 times and see how close you get to the answer for various values:

```go
package main

import (
	"fmt"
	"math"
)

func Sqrt(x float64) float64 {
	// nested function to generate an approximation
	approximation := func(z, x float64) float64 {
		return z - ((z*z)-x)/(2*z)
	}

	z := 1.0
	for i := 0; i < 10; i++ {
		z = approximation(z, x)
	}
	return z
}

func main() {
	for i := 1.0; i < 11.0; i++ {
		fmt.Printf("%d: %f vs %f\n", int(i), Sqrt(i), math.Sqrt(i))
	}
}
```
[Go Playground](http://play.golang.org/p/me3tBkzd5S)

Stop once the value has stopped changing (or only changes by a very small delta). See if that's more or fewer iterations.


```go
package main

import (
	"fmt"
	"math"
)

func Sqrt(x float64) (float64, int) {
	approximation := func(z, x float64) float64 {
		return z - ((z*z)-x)/(2*z)
	}
	i := 0
	z := approximation(1.0, x)
	for math.Abs(approximation(z, x)-z) > 0.0000001 {
		z = approximation(z, x)
		i++
	}
	return z, i
}

func main() {
	for i := 1.0; i < 11.0; i++ {
		ours, iterations := Sqrt(i)
		fmt.Printf("%d: in %d iterations %f vs %f\n",
			int(i),
			iterations, ours,
			math.Sqrt(i))
	}
}
```
[Go Playground](http://play.golang.org/p/VBvPwQmDLI)


## Structs
\label{sec:structs}

A struct is a collection of fields/properties.
If you are coming from an object-oriented background, you can
consider a struct to be a light class that doesn't support
inheritance. Methods (Section~\ref{sec:methods}) and interfaces (Section~\ref{sec:interfaces}) are discussed at length in Chapter~\ref{cha:methods_and_interfaces}

You don't need to define getters and setters on struct fields, they can
be accessed automatically. However, note that only exported fields can
be accessed from outside of the package.

A struct literal denotes a newly allocated struct value by listing the values of its fields.
You can list just a subset of fields by using the "Name:" syntax. (And the order of named fields is irrelevant.)
The special prefix & constructs a pointer to a newly allocated struct.

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
	fmt.Println(Bootcamp{Lat: 34.012836,
		Lon:  -118.495338,
		Date: time.Now()})
}
```

Declaration of struct literals:

```go
package main

import "fmt"

type Vertex struct {
    X, Y int
}

var (
    p = Vertex{1, 2}  // has type Vertex
    q = &Vertex{1, 2} // has type *Vertex
    r = Vertex{X: 1}  // Y:0 is implicit
    s = Vertex{}      // X:0 and Y:0
)

func main() {
    fmt.Println(p, q, r, s)
}
```

[Go playground](http://play.golang.org/p/Fx--Pr6PTN)

* [Struct example](http://tour.golang.org/#26)
* [Struct fields](http://tour.golang.org/#27)
* [Struct Literals](http://tour.golang.org/#29)

## Pointers
\label{sec:pointers}

Go has pointers, but no pointer arithmetic.
Struct fields can be accessed through a struct pointer. The indirection through the pointer is transparent.

Note that by default Go passes arguments by values (copying the
arguments), if you want to pass the arguments by reference, you need
to pass pointers (or use a structure using reference values like
slices (Section~\ref{sec:slices}) and maps (Section~\ref{sec:maps}).

To get the pointer of a value, use the `&` symbol in front of the
value, to dereference a
pointer, use the `*` symbol.

Methods are often defined on pointers and not values (although they can
be defined on both), so you will often store a pointer in a variable as
in the example below:

```go
client := &http.Client{}
resp, err := client.Get("http://gobootcamp.com")
```


* [Pointers example](http://tour.golang.org/#28)

## Initializing
\label{sec:initializing_values}

Go supports the `new` expression to allocate a
zeroed value of the requested type and to return a pointer to it.

```go
x := new(int)
```

As seen in (Section~\ref{sec:structs}) a common way to "initialize" a
variable containing struct or a reference to one, is to create a struct literal. Another option is to create a constructor.
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

Note that slices (Section~\ref{sec:slices}), maps (Section~\ref{sec:maps}) and channels (Section~\ref{sec:channels}) are usually allocated using `make`
so the data structure these types are built upon can be initialized.

### Resources

* [Allocation with `new` - effective Go](http://golang.org/doc/effective_go.html#allocation_new)
* [Composite Literals - effective Go](http://golang.org/doc/effective_go.html#composite_literals)
* [Allocation with `make` - effective Go](http://golang.org/doc/effective_go.html#allocation_make)

## Arrays
\label{sec:arrays}

The type `[n]T` is an array of `n` values of type `T`.

The expression:

```go
var a [10]int
```

declares a variable `a` as an array of ten integers.

An array's length is part of its type, so arrays cannot be resized.
This seems limiting, but don't worry; Go provides a convenient way of working with arrays.

```go
package main

import "fmt"

func main() {
    var a [2]string
    a[0] = "Hello"
    a[1] = "World"
    fmt.Println(a[0], a[1])
    fmt.Println(a)
}
```

[Golang tour page](http://tour.golang.org/#31)


You can also set the array entries as you declare the array:

```go
package main

import "fmt"

func main() {
    a  := [2]string{"hello", "world!"}
    fmt.Printf("%q", a)
}
```

[Go Playground](http://play.golang.org/p/87fdeul4H7)

Finally, you can use an ellipsis to use an implicit length when you
pass the values:

```go
package main

import "fmt"

func main() {
	a := [...]string{"hello", "world!"}
	fmt.Printf("%q", a)
}
```

[Go Playground](http://play.golang.org/p/lxVUhtyJJP)

### Printing arrays

Note how we used the [`fmt` package](http://golang.org/pkg/fmt/) using `Printf` and used the `%q` "verb" to print each element quoted.

If we had used `Println` or the `%s` verb, we would have add a different result:

```go
package main

import "fmt"

func main() {
	a := [2]string{"hello", "world!"}
	fmt.Println(a)
	// [hello world!]
	fmt.Printf("%s\n", a)
	// [hello world!]
	fmt.Printf("%q\n", a)
	// ["hello" "world!"]
}
```

[Go Playground](http://play.golang.org/p/jsvGaXW6uH)


### Multi-dimensional arrays
\label{sec:multi-dimensional-arrays}

You can also create multi-dimensional arrays:

```go
package main

import "fmt"

func main() {
	var a [2][3]string
	for i := 0; i < 2; i++ {
		for j := 0; j < 3; j++ {
			a[i][j] = fmt.Sprintf("row %d - column %d", i+1, j+1)
		}
	}
	fmt.Printf("%q", a)
	// [["row 1 - column 1" "row 1 - column 2" "row 1 - column 3"]
	//  ["row 2 - column 1" "row 2 - column 2" "row 2 - column 3"]]
}
```

[Go Playground](http://play.golang.org/p/L6faG4RMPx)


Trying to access or set a value at an index that doesn't exist will
prevent your program from compiling, for instance, try to compile the
following code:

```go
package main

func main() {
    var a [2]string
    a[3] = "Hello"
}
```

You will see that the compiler will report the following error:

```
Invalid array index 3 (out of bounds for 2-element array)
```

That's because our array is of length 2 meaning that the only 2
available indexes are 0 and 1. Trying to access index 3 results in an
error that tells us that we are trying to access an index that is of
bounds since our array only contains 2 elements and we are trying to
access the 4th element of the array.

Slices, the type that we are going to see next is more often used, due
  to the fact that we don't always know in advance the length of the array we need.


## Slices
\label{sec:slices}

Slices wrap arrays to give a more general, powerful, and convenient interface to sequences of data. Except for items with explicit dimension such as transformation matrices, most array programming in Go is done with slices rather than simple arrays.

Slices hold references to an underlying array, and if you assign one slice to another, both refer to the same array. If a function takes a slice argument, changes it makes to the elements of the slice will be visible to the caller, analogous to passing a pointer to the underlying array.

A slice points to an array of values and also includes a length.
Slices can be resized since they are just a wrapper on top of another
data structure.

`[]T` is a slice with elements of type `T`.

```go
package main

import "fmt"

func main() {
    p := []int{2, 3, 5, 7, 11, 13}
    fmt.Println(p)
    // [2 3 5 7 11 13]
}
```

[Go tour page](http://tour.golang.org/#33)

### Slicing a slice

Slices can be re-sliced, creating a new slice value that points to the same array.

The expression

```go
s[lo:hi]
```

evaluates to a slice of the elements from lo through hi-1, inclusive. Thus

```go
s[lo:lo]
```

is empty and

```go
s[lo:lo+1]
```
has one element.

Note: lo and hi would be integers representing indexes.


```go
package main

import "fmt"

func main() {
	mySlice := []int{2, 3, 5, 7, 11, 13}
	fmt.Println(mySlice)
	// [2 3 5 7 11 13]

	fmt.Println(mySlice[1:4])
	// [3 5 7]

	// missing low index implies 0
	fmt.Println(mySlice[:3])
	// [2 3 5]

	// missing high index implies len(s)
	fmt.Println(mySlice[4:])
	// [11 13]
}
```

[Go Playground](http://play.golang.org/p/2z_6hNt_Vg)

[Go tour page](http://tour.golang.org/#33)


### Making slices

Besides creating slices by passing the values right away (array literal), you can also use `make`.
You create an empty slice of a specific length and then populate each
entry:

```go
package main

import "fmt"

func main() {
	cities := make([]string, 3)
	cities[0] = "Santa Monica"
	cities[1] = "Venice"
	cities[2] = "Los Angeles"
	fmt.Printf("%q", cities)
	// ["Santa Monica" "Venice" "Los Angeles"]
}
```

It works by allocating a zeroed array and returning a slice that refers to that array.

[Go tour](http://tour.golang.org/#34)


### Appending to a slice

Note however, that you would get a runtime error if you were to do that:

```go
cities := []string{}
cities[0] = "Santa Monica"
```

As explained above, a slice is seating on top of an array, in this case,
the array is empty and the slice can't set a value in the referred array.
There is a way to do that tho, and that is by using the `append`
function:

```go
package main

import "fmt"

func main() {
	cities := []string{}
	cities = append(cities, "San Diego")
	fmt.Println(cities)
	// [San Diego]
}
```

[Go playground](http://play.golang.org/p/rNRt6jvlFl)


You can append more than one entry to a slice:

```go
package main

import "fmt"

func main() {
	cities := []string{}
	cities = append(cities, "San Diego", "Mountain View")
	fmt.Printf("%q", cities)
	// ["San Diego" "Mountain View"]
}
```

[Go Playground](http://play.golang.org/p/fdh3daTttz)

And you can also append a slice to another using an ellipsis:

```go
package main

import "fmt"

func main() {
	cities := []string{"San Diego", "Mountain View"}
	otherCities := []string{"Santa Monica", "Venice"}
	cities = append(cities, otherCities...)
	fmt.Printf("%q", cities)
	// ["San Diego" "Mountain View"]
}
```

[Go Playground](http://play.golang.org/p/CjR88q7CIo)

Note that the ellipsis is a built-in feature of the language that means
that the element is a collection.
We can't append an element of type slice of strings (`[]string`) to
a slice of strings, only strings can be appended. However, using the
ellipsis (`...`) after our slice, we indicate that we want to append
each element of our slice. Because we are appending strings from another
slice, the compiler will accept the operation since the types are
matching.

You obviously can't append a slice of type `[]int` to another slice of
type `[]string`.


### Length

At any time, you can check the length of a slice by using `len`:

```go
package main

import "fmt"

func main() {
	cities := []string{
		"Santa Monica",
		"San Diego",
		"San Francisco",
	}
	fmt.Println(len(cities))
	// 3
	countries := make([]string, 42)
	fmt.Println(len(countries))
	// 42
}
```

[Go Playground](http://play.golang.org/p/EvvVrOpPII)


### Nil slices

The zero value of a slice is nil.
A nil slice has a length and capacity of 0.

```go
package main

import "fmt"

func main() {
    var z []int
    fmt.Println(z, len(z), cap(z))
    // [] 0 0
    if z == nil {
        fmt.Println("nil!")
    }
    // nil!
}
```

[Go tour page](http://tour.golang.org/#35)

### Resources

For more details about slices:

* [Go slices, usage and internals](http://golang.org/doc/articles/slices_usage_and_internals.html)
* [Effective Go - slices](http://golang.org/doc/effective_go.html#slices)
* [Append function documentation](http://golang.org/pkg/builtin/#append)
* [Slice tricks](https://code.google.com/p/go-wiki/wiki/SliceTricks)
* [Effective Go - slices](http://golang.org/doc/effective_go.html#slices)
* [Effective Go - two-dimensional slices](http://golang.org/doc/effective_go.html#two_dimensional_slices)
* [Go by example - slices](https://gobyexample.com/slices)


## Range
\label{sec:range}

The `range` form of the for loop iterates over a `slice` (Section~\ref{sec:slices}) or a `map` (Section~\ref{sec:maps}).
Being able to iterate over all the elements of of a data structure is
very useful and `range` simplifies the iteration.

```go
package main

import "fmt"

var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
    for i, v := range pow {
        fmt.Printf("2**%d = %d\n", i, v)
    }
}
```

Which will print:

```
2**0 = 1
2**1 = 2
2**2 = 4
2**3 = 8
2**4 = 16
2**5 = 32
2**6 = 64
2**7 = 128
```

[Go tour page](http://tour.golang.org/#36)


You can skip the index or value by assigning to `_`.
If you only want the index, drop the ", value" entirely.

```go
package main

import "fmt"

func main() {
    pow := make([]int, 10)
    for i := range pow {
        pow[i] = 1 << uint(i)
    }
    for _, value := range pow {
        fmt.Printf("%d\n", value)
    }
}
```

[Go tour page](http://tour.golang.org/#37)

### Break & continue

As if you were using a normal for loop, you can stop the iteration
anytime by using `beak`:

```go
package main

import "fmt"

func main() {
	pow := make([]int, 10)
	for i := range pow {
		pow[i] = 1 << uint(i)
		if pow[i] > 16 {
			break
		}
	}
	fmt.Println(pow)
	// [1 2 4 8 16 0 0 0 0 0]
}
```

[Go Playground](http://play.golang.org/p/T65dcE8fZ7)


You can also skip an iteration by using `continue`:

```go
package main

import "fmt"

func main() {
	pow := make([]int, 10)
	for i := range pow {
		if i%2 == 0 {
			continue
		}
		pow[i] = 1 << uint(i)
	}
	fmt.Println(pow)
	// [0 2 0 8 0 32 0 128 0 512]
}
```

[Go Playground](http://play.golang.org/p/TT1vfYKpOy)


### Range and maps

Range can also be used on maps (Section~\ref{sec:maps}), in that case,
the first parameter isn't an incremental integer but the map key:

```go
package main

import "fmt"

func main() {
	cities := map[string]int{
		"New York":    8336697,
		"Los Angeles": 3857799,
		"Chicago":     2714856,
	}
	for key, value := range cities {
		fmt.Printf("%s has %d inhabitants\n", key, value)
	}
}
```

Which will output:

```
New York has 8336697 inhabitants
Los Angeles has 3857799 inhabitants
Chicago has 2714856 inhabitants
```

[Go Playground](http://play.golang.org/p/rg5sc_Nl-P)

## Exercise: Slice

[Assignement](http://tour.golang.org/#38) (can be done online)

Implement `Pic`.
It should return a slice of length dy, each element of which is a slice of dx 8-bit unsigned integers. 
When you run the program, it will display your picture, interpreting the integers as grayscale (well, bluescale) values.

The choice of image is up to you. Interesting functions include `x^y`, `(x+y)/2`, and `x*y`.

(You need to use a loop to allocate each `[]uint8` inside the `[][]uint8`.)
(Use `uint8(intValue)` to convert between types.)

Starting code base:

```go
package main

import "code.google.com/p/go-tour/pic"

func Pic(dx, dy int) [][]uint8 {
}

func main() {
    pic.Show(Pic)
}
```

Start by pulling down the `pic` package:

```bash
$ go get code.google.com/p/go-tour/pic
```

Here is a preview of the package code:

```go
package pic

import (
	"bytes"
	"encoding/base64"
	"fmt"
	"image"
	"image/png"
)

func Show(f func(int, int) [][]uint8) {
	const (
		dx = 256
		dy = 256
	)
	data := f(dx, dy)
	m := image.NewNRGBA(image.Rect(0, 0, dx, dy))
	for y := 0; y < dy; y++ {
		for x := 0; x < dx; x++ {
			v := data[y][x]
			i := y*m.Stride + x*4
			m.Pix[i] = v
			m.Pix[i+1] = v
			m.Pix[i+2] = 255
			m.Pix[i+3] = 255
		}
	}
	ShowImage(m)
}

func ShowImage(m image.Image) {
	var buf bytes.Buffer
	err := png.Encode(&buf, m)
	if err != nil {
		panic(err)
	}
	enc := base64.StdEncoding.EncodeToString(buf.Bytes())
	fmt.Println("IMAGE:" + enc)
}
```

### Solution

```go
package main

import "code.google.com/p/go-tour/pic"

func Pic(dx, dy int) [][]uint8 {
	img := make([][]uint8, dy)
	for y := range img {
		img[y] = make([]uint8, dx)
		for x := 0; x < dx; x++ {
			img[y][x] = uint8(x * y)
		}
	}
	return img
}

func main() {
	pic.Show(Pic)
}
```

[Go Playground](http://play.golang.org/p/iTzF6Y972V)


## Maps
\label{sec:maps}

Maps are somewhat similar to what other languages call "dictionaries" or "hashes".

A map maps keys to values. Here we are mapping string keys (actor names)
to an integer value (age).

```go
package main

import "fmt"

func main() {
	celebs := map[string]int{
		"Nicolas Cage":       50,
		"Selena Gomez":       21,
		"Jude Law":           41,
		"Scarlett Johansson": 29,
	}

	fmt.Printf("%#v", celebs)
}
```

[Go Playground](http://play.golang.org/p/ttJ-3xgzuk)


When not using map literals like above, maps must be created with make (not new) before use.
The nil map is empty and cannot be assigned to.

Assignments follow the Go convention and can be observed in the example
below.

```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{40.68433, -74.39967}
	fmt.Println(m["Bell Labs"])
}
```

When using map literals, if the top-level type is just a type name, you can omit it from the elements of the literal.


```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	// same as "Bell Labs": Vertex{40.68433, -74.39967}
	"Google": {37.42202, -122.08408},
}

func main() {
	fmt.Println(m)
}
```


[Go tour page](http://tour.golang.org/#39)
[Go tour page 2](http://tour.golang.org/#40)
[Go tour page 3](http://tour.golang.org/#41)


### Mutating maps

Insert or update an element in map m:

```go
m[key] = elem
```

Retrieve an element:

```go
elem = m[key]
```

Delete an element:

```go
delete(m, key)
```

Test that a key is present with a two-value assignment:

```go
elem, ok = m[key]
```

If key is in m, ok is true. If not, ok is false and elem is the zero value for the map's element type.
Similarly, when reading from a map if the key is not present the result is the zero value for the map's element type.


[Go tour page](http://tour.golang.org/#42)


### Resources

* [Go team blog post on maps](http://blog.golang.org/go-maps-in-action)
* [Effective Go - maps](http://golang.org/doc/effective_go.html#maps)

## Exercise: Map

[Online assignment](http://tour.golang.org/#43)

Implement WordCount. 

It should return a map of the counts of each "word" in the string `s`. 
The `wc.Test` function runs a test suite against the provided function and prints success or failure.

You might find [strings.Fields](http://golang.org/pkg/strings/#Fields) helpful.


### Solution

```go
package main

import (
	"code.google.com/p/go-tour/wc"
	"strings"
)

func WordCount(s string) map[string]int {
	words := strings.Fields(s)
	count := make(map[string]int)
	for _, word := range words {
		count[word]++
	}
	return count
}

func main() {
	wc.Test(WordCount)
}
```

[Go Playground](http://play.golang.org/p/M0bb5rWa7t)
