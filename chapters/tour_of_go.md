#Tour of Go 
\label{cha:tour_of_go}

Go is often referred as a "simple" programming language, a language that can be
learned in a few hours if you already know another language.
Go was designed to feel familiar and to stay as simple as possible,
[the entire language specification](http://golang.org/ref/spec) fits
in just a few pages.

Here are the concepts we are going to explore before writing our
first application. To walk through these various concepts, we are
going to use Go's official [Tour of Go](https://tour.golang.org/) web
application. 

## Basic Concepts
\label{sec:basic_concepts}

### Packages

Every Go program is made up of packages.
Programs start running in package main.
By convention, the package name is the same as the last element of the import path. For instance, the "math/rand" package comprises files that begin with the statement package rand.

[Package example](https://tour.golang.org/basics/1)

### Imports

* [Imports example](https://tour.golang.org/basics/2)

### Exported names

After importing a package, you can refer to the names it exports.
In Go, a name is exported if it begins with a capital letter.
`Foo` is an exported name, as is `FOO`. The name `foo` is not exported.

* [Exported names example](https://tour.golang.org/basics/3)

### Functions, signature, return values, named results

A function can take zero or more typed arguments.
The type comes after the variable name. 
Functions can be defined to return any number of values. 
Return values are always typed.


* [Function example](https://tour.golang.org/basics/4)
* [Function with arguments sharing the same type](https://tour.golang.org/basics/5)
* [Function with multiple results](https://tour.golang.org/basics/6)
* [Function with named results](https://tour.golang.org/basics/7)

#### Resources

* [Go's declaration
Syntax](http://blog.golang.org/gos-declaration-syntax)

### Variables / inferred typing, short assignment

The var statement declares a list of variables; as in function argument lists, the type is last.
A var declaration can include initializers, one per variable.
If an initializer is present, the type can be omitted; the variable will take the type of the initializer.
Inside a function, the `:=` short assignment statement can be used in place of a var declaration with implicit type.
Outside a function, every construct begins with a keyword (`var`,
`func`, and so on) and the `:=` construct is not available.

* [Variables example](https://tour.golang.org/basics/8)
* [Initialization of variables](https://tour.golang.org/basics/9)
* [Short variable declaration: `:=`](https://tour.golang.org/basics/10)


### Basic types

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

* [Basic types example](https://tour.golang.org/basics/11)

### Type conversion

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

* [Type conversion example](https://tour.golang.org/basics/13)


### Constants

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

* [Constants example](https://tour.golang.org/basics/15)
* [Numeric Constants](https://tour.golang.org/basics/16)

### For Loop

Go has only one looping construct, the for loop.
The basic for loop looks as it does in C or Java, except that the ( ) are gone (they are not even optional) and the { } are required.
As in C or Java, you can leave the pre and post statements empty.

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

```go
sum := 1
for ; sum < 1000; {
    sum += sum
}
```

```go
sum := 1
for sum < 1000 {
    sum += sum
}
```

```go
for {
  // do something in a loop forever  
}
```

* [For loop example](https://tour.golang.org/flowcontrol/1)
* [For loop without pre/post statements](https://tour.golang.org/flowcontrol/2)
* [For loop as a `while` loop](https://tour.golang.org/flowcontrol/3)
* [Infinite loop](https://tour.golang.org/flowcontrol/4)

### If statement

The `if` statement looks as it does in C or Java, except that the `( )`
are gone and the `{ }` are required. Like `for`, the `if` statement can start with a short statement to execute before the condition.
Variables declared by the statement are only in scope until the end of
the `if`.
Variables declared inside an if short statement are also available inside any of the else blocks.

```go
if answer != 42 {
    return "Wrong answer"
}
```

```go
if err := foo(); err != nil {
    panic(err)
}
```

* [`If` statement](https://tour.golang.org/flowcontrol/5)
* [`If` with a short statement](https://tour.golang.org/flowcontrol/6)
* [`If` and `else` example](https://tour.golang.org/flowcontrol/7)


###Exercises

[Loops and Functions exercise](https://tour.golang.org/flowcontrol/8)

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


### Structs

### Pointers

### Initializing

### Arrays

### Slices

### Range

*Exercise*

### Maps

*Exercise*

### First class functions, closures

*Exercise*

### Switch statement

*Exercise*


## Methods and interfaces
\label{sec:methods_and_interfaces}

### Methods

### Interfaces

### Errors

*Exercise*

## Concurrency

### Goroutines

### Channels

### Select

*Exercise*


<!---

This is a section. You can refer to it using the \LaTeX\ cross-reference syntax, like so: Section~\ref{sec:a_section}.


```ruby
def hello
  puts "hello, world!"
end
```

The last of these can be combined with \PolyTeX's `codelisting` environment to make code listings with linked cross-references (Listing~\ref{code:hello}).

\begin{codelisting}
\codecaption{Hello, world.}
\label{code:hello}
```ruby
def hello
  puts "hello, world!"
end
```
\end{codelisting}


--->
