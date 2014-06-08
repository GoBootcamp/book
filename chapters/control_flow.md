#Control flow
\label{cha:control_flow}


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


## Exercise: Loops and Functions
\label{sec:exercise_loops_and_funcs}

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
[See in playground](http://play.golang.org/p/me3tBkzd5S)

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
[See in playground](http://play.golang.org/p/VBvPwQmDLI)

