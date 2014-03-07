# Concurrency
\label{cha:concurrency}

Concurrent programming is a large topic but it's also one of the most
interesting aspects of the Go language.

Concurrent programming in many environments is made difficult by the subtleties required to implement correct access to shared variables. Go encourages a different approach in which shared values are passed around on channels and, in fact, never actively shared by separate threads of execution. Only one goroutine has access to the value at any given time. Data races cannot occur, by design. To encourage this way of thinking we have reduced it to a slogan:

Do not communicate by sharing memory; instead, share memory by communicating.

This approach can be taken too far. Reference counts may be best done by putting a mutex around an integer variable, for instance. But as a high-level approach, using channels to control access makes it easier to write clear, correct programs.

Although Go's approach to concurrency originates in [Hoare's Communicating Sequential Processes (CSP)](http://en.wikipedia.org/wiki/Communicating_sequential_processes), it can also be seen as a type-safe generalization of Unix pipes.

* [Rob Pike's concurrency slides (IO 2012)](http://talks.golang.org/2012/concurrency.slide#1)
* [Video of Rob Pike at IO 2012](http://www.youtube.com/watch?v=f6kdp27TYZs)
* [Video of Concurrency is not parallelism (Rob Pike)](http://vimeo.com/49718712)

## Goroutines
\label{sec:goroutines}

A goroutine is a lightweight thread managed by the Go runtime.

```go
go f(x, y, z)
```

starts a new goroutine running:

```go
f(x, y, z)
```

The evaluation of `f`, `x`, `y`, and `z` happens in the current goroutine and the execution of `f` happens in the new goroutine.

Goroutines run in the same address space, so access to shared memory must be synchronized. The [sync](http://golang.org/pkg/sync/) package provides useful primitives, although you won't need them much in Go as there are other primitives.

```go
package main

import (
    "fmt"
    "time"
)

func say(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}

func main() {
    go say("world")
    say("hello")
}
```

[Go tour page](http://tour.golang.org/#65)


## Channels
\label{sec:channels}

Channels are a typed conduit through which you can send and receive values with the channel operator, `<-`.

```go
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and
           // assign value to v.
```

(The data flows in the direction of the arrow.)

Like maps (Section~\ref{sec:maps}) and slices (Section~\ref{sec:slices}), channels must be created before use:

```go
ch := make(chan int)
```

By default, sends and receives block until the other side is ready. This allows goroutines to synchronize without explicit locks or condition variables.

```go
package main

import "fmt"

func sum(a []int, c chan int) {
    sum := 0
    for _, v := range a {
        sum += v
    }
    c <- sum // send sum to c
}

func main() {
    a := []int{7, 2, 8, -9, 4, 0}

    c := make(chan int)
    go sum(a[:len(a)/2], c)
    go sum(a[len(a)/2:], c)
    x, y := <-c, <-c // receive from c

    fmt.Println(x, y, x+y)
}
```

[Go tour page](http://tour.golang.org/#66)

### Buffered channels

Channels can be _buffered_. Provide the buffer length as the second argument to `make` to initialize a buffered channel:

```go
ch := make(chan int, 100)
```

Sends to a buffered channel block only when the buffer is full. Receives block when the buffer is empty.

```go
package main

import "fmt"

func main() {
    c := make(chan int, 2)
    c <- 1
    c <- 2
    fmt.Println(<-c)
    fmt.Println(<-c)
}
```

But if you do:

```go
package main

import "fmt"

func main() {
	c := make(chan int, 2)
	c <- 1
	c <- 2
	c <- 3
	fmt.Println(<-c)
	fmt.Println(<-c)
	fmt.Println(<-c)
}
```

You are getting a deadlock:

```
fatal error: all goroutines are asleep - deadlock!
```

That's because we overfilled the buffer without letting the code a
chance to read/remove a value from the channel.

However, this version using a goroutine would work fine:

```go
package main

import "fmt"

func main() {
	c := make(chan int, 2)
	c <- 1
	c <- 2
	c3 := func() { c <- 3 }
	go c3()
	fmt.Println(<-c)
	fmt.Println(<-c)
    fmt.Println(<-c)
}
```

[Playground example](http://play.golang.org/p/YKg79QtX2q)

The reason is that we are adding an extra value from inside a go
routine, so our code doesn't block the main thread. The goroutine
is being called before the channel is being emptied, but that is fine,
the goroutine will wait until the channel is available.
We then read a first value from the channel, which frees a spot and
out goroutine can push its value to the channel.

[Go tour page](http://tour.golang.org/#67)

## Range and close
\label{sec:range_close}

A sender can close a channel to indicate that no more values will be sent. Receivers can test whether a channel has been closed by assigning a second parameter to the receive expression: after

```go
v, ok := <-ch
```

`ok` is false if there are no more values to receive and the channel is closed.

The loop `for i := range c` receives values from the channel repeatedly until it is closed.

**Note:** Only the sender should close a channel, never the receiver. Sending on a closed channel will cause a panic.

**Another note:** Channels aren't like files; you don't usually need to close them. Closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a range loop.

```go
package main

import (
    "fmt"
)

func fibonacci(n int, c chan int) {
    x, y := 0, 1
    for i := 0; i < n; i++ {
        c <- x
        x, y = y, x+y
    }
    close(c)
}

func main() {
    c := make(chan int, 10)
    go fibonacci(cap(c), c)
    for i := range c {
        fmt.Println(i)
    }
}
```

[Go tour page](http://tour.golang.org/#68)

## Select
\label{sec:select}

The `select` statement lets a goroutine wait on multiple communication operations.

A `select` blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.

```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
    x, y := 0, 1
    for {
        select {
        case c <- x:
            x, y = y, x+y
        case <-quit:
            fmt.Println("quit")
            return
        }
    }
}

func main() {
    c := make(chan int)
    quit := make(chan int)
    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(<-c)
        }
        quit <- 0
    }()
    fibonacci(c, quit)
}
```

### Default case

The default case in a `select` is run if no other case is ready.

Use a `default` case to try a send or receive without blocking:

```go
select {
case i := <-c:
    // use i
default:
    // receiving from c would block
}
```

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    tick := time.Tick(100 * time.Millisecond)
    boom := time.After(500 * time.Millisecond)
    for {
        select {
        case <-tick:
            fmt.Println("tick.")
        case <-boom:
            fmt.Println("BOOM!")
            return
        default:
            fmt.Println("    .")
            time.Sleep(50 * time.Millisecond)
        }
    }
}
```

[Go tour page](http://tour.golang.org/#70)

## Exercise: Equivalent Binary Trees
\label{sec:exercise_equiv_bin_trees}


[Go tour page](http://tour.golang.org/#71)
