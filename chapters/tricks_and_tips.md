# Tips and Tricks
\label{cha:tricks_and_tips}

This section will grow over time but the main goal is to share some
tricks experienced developers discovered over time.
Hopefully this tips will get new users more productive faster.


## 140 char tips

* leave your object oriented brain at home. Embrace the interface. [@mikegehard](https://twitter.com/mikegehard)
* Learn to do things the Go way, don't try to force your language idioms into Go. [@DrNic](https://twitter.com/drnic)
* It's better to over do it with using interfaces than use too few of them. [@evanphx](https://twitter.com/evanphx)
* Embrace the language: simplicity, concurrency, and composition.
  [@francesc](https://twitter.com/francesc)
* read all the awesome docs that they have on [golang.org](http://golang.org). [@vbatts](https://twitter.com/vbatts)
* always use `gofmt`. [@darkhelmetlive](https://twitter.com/darkhelmetlive)
* read a lot of source code. [@DrNic](https://twitter.com/drnic)

## Organization

Go is a pretty easy programming language to learn but the hardest
thing that developers at first is how to organize their code. Rails
became popular for many reasons and scaffolding was one of them. It gave
new developers clear directions and places to put their code and idioms
to follow.

To some extent, Go does the same thing by providing developers with
great tools like [`go fmt`](http://blog.golang.org/go-fmt-your-code)
and by having a strict compiler that won't compile unused variables or
unused import statements.

However, the challenge most new developers face is the question of data
organization. Coming from an [OOP](http://en.wikipedia.org/wiki/Object-oriented_programming) background a lot of us are used to inheritance, something that isn't supported by Go.
Instead you have to think in terms of composition and interfaces.

The go team wrote a [short but good segment](http://golang.org/doc/effective_go.html#embedding) on this topic.

Composition of something known by OOP progammers and Go supports it,
here is an example:

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
	// job := &Job{Command: "demo", Logger: log.New(os.Stderr, "Job: ", log.Ldate)}
	job.Logger.Print("test")
}
```

[See in playground](http://play.golang.org/p/3yYJadlmHS)


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

Note that you still need to set the logger and that's often a good
reason to use a constructor (custom constructor are used when you need
to set a structure before using a value, see (Section~\ref{sec:custom_constructors}) ).
What is really nice with the implicit composition is that it allows to
easily and cheaply make your structs implement interfaces.
Imagine that you have a function that takes variables implementing an
interface with the `Print` method. My adding `*log.Logger` to your
struct (and initializing it properly), your struct is now implementing
the interface without you writing any custom methods.

### Custom Constructors
\label{sec:custom_constructors}


A question I often hear is when should I use custom constructors like
`NewJob`. My answer is that in most cases you don't need to. However,
whenever you need to set your value at initialization time and you have
some sort of default values, it's a good candidate for a constructor.
In the above example, adding a constructor makes a lot of sense so we
can set a default logger.

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

func NewJob(command string) *Job {
	return &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
}

func main() {
	NewJob("demo").Print("starting now...")
}
```

### Breaking down code in packages

TODO


## Dependency management
\label{sec:dependency_management}


TODO:

[gpm](https://github.com/pote/gpm)

Set `GOPATH`

## Using errors
\label{sec:using_errors}

TODO:

errors as a pattern  errors vs exceptions

## go imports

TODO

## Expvar

TODO [package](http://golang.org/pkg/expvar/)


