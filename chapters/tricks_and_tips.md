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

## goimports

[Goimports](http://godoc.org/code.google.com/p/go.tools/cmd/goimports) is a tool that updates your Go import lines, adding missing ones and
removing unreferenced ones.

It acts the same as gofmt (drop-in replacement) but in addition to code
formatting, also fixes imports.

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


## Dependency package management
\label{sec:dependency_management}

Unfortunately, Go doesn't ship with its own dependency package management system.
Probably due to its roots in the C culture, packages aren't
versioned and explicit version dependencies aren't addressed.

The challenge is that if you have multiple developers on your
project, you want all of them to be on the same version of your
dependencies. Your dependencies might also have their own dependencies
and you want to make sure everything is in a good state.
It gets even tricker when you have multiple projects using different
versions of the same dependency. This is typically the case in a [CI](http://en.wikipedia.org/wiki/Continuous_integration)
environment.

The Go community came up with a lot of different solutions for these
problems. But for me, none are really great so at
[Splice](https://splice.com) we went for the simplest working solution we found: [gpm](https://github.com/pote/gpm)

Gpm is a simple bash script, we end up [modifying it a little](https://gist.github.com/mattetti/9334318) so we could
drop the script in each repo. The bash script uses a custom file called
`Godeps` which lists the packages to install.

When switching to a different project, we run the project `gpm` script
to pull down or set the right revision of each package.

In our CI environment, we set `GOPATH` to a project specific folder
before running the test suite so packages aren't shared between
projects.

## Using errors
\label{sec:using_errors}

Errors are very important pattern in Go and at first, new developers are
surprised by the amount of functions returning a value and an error.

Go doesn't have a concept of an exception like you might have seen in
other programming languages. Go does have something called `panic` but
as its name suggests they are really exceptional and shouldn't be
rescued (that said, they can be).

The error handling in Go seems cumbersome and repetitive at first, but
quickly becomes part of the way we think. Instead of creating exceptions
that bubble up and might or might not be handled or passed higher,
errors are part of the response and designed to be handled by the
caller. Whenever a function might generate an error, its response should
contain an error param.

[Andrew Gerrand](https://twitter.com/enneff) from the Go team wrote a great [blog post on errors](http://blog.golang.org/error-handling-and-go) I strongly recommend you read it.


[Effective Go section on errors](http://golang.org/doc/effective_go.html#errors)


## Return statements and conditions

TODO



## Expvar

TODO [package](http://golang.org/pkg/expvar/)


## Set the build id using git's SHA
\label{sec:git_sha_in_go_binary}

It's often very useful to burn a build id in your binaries.
I personally like to use the SHA1 of the git commit I'm committing.
You can get the short version of the sha1 of your latest commit by
running the following `git` command from your repo:

```bash
git rev-parse --short HEAD
```

The next step is to set an exported variable that you will set at
compilation time using the `-ldflags` flag.

```go
package main

import "fmt"

// compile passing -ldflags "-X main.Build <build sha1>"
var Build string

func main() {
	fmt.Printf("Using build: %s\n", Build)
}
```

[See in playground](http://play.golang.org/p/8wbsQ53ZV5)

Save the above code in a file called `example.go`.
If you run the above code, `Build` won't be set, for that you need to
set it using `go build` and the `-ldflags`.

```bash
$ go build -ldflags "-X main.Build a1064bc" example.go
```

Now run it to make sure:

```bash
$ ./example
Using build: a1064bc
```

Now, hook that into your deployment compilation process, I personally like [Rake](http://en.wikipedia.org/wiki/Rake_(software)) to
do that, and this way, every time I compile, I think of [Jim Weirich](http://en.wikipedia.org/wiki/Jim_Weirich).


## How to see what packages my app imports
\label{sec:list_imported_go_packages}

It's often practical to see what packages your app is importing.
Unfortunately there isn't a simple way to do that, however it is doable
via the `go list` tool and using templates.

Go to your app and run the following.

```bash
$ go list -f '{{join .Deps "\n"}}' |  
  xargs go list -f '{{if not .Standard}}{{.ImportPath}}{{end}}'
```

Here is an example with the clirescue refactoring example:

```bash
$ cd $GOPATH/src/github.com/GoBootcamp/clirescue
$ go list -f '{{join .Deps "\n"}}' | 
  xargs go list -f '{{if not .Standard}}{{.ImportPath}}{{end}}'
github.com/GoBootcamp/clirescue/cmdutil
github.com/GoBootcamp/clirescue/trackerapi
github.com/GoBootcamp/clirescue/user
github.com/codegangsta/cli
```

If you want the list to also contain standard packages, edit the
template and use:

```bash
$ go list -f '{{join .Deps "\n"}}' |  xargs go list -f '{{.ImportPath}}'
```

## Web resources

* [Dave Cheney](https://twitter.com/davecheney) maintains a [list of resources](http://dave.cheney.net/resources-for-new-go-programmers) for new Go developers.
