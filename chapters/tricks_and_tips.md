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
* Learn and become familiar with tools and utilities, or create your own! They are as vital to your success as knowing the language. [@coreyprak](https://twitter.com/coreyprak)


## Alternate Ways to Import Packages

There are a few other ways of importing packages. We'll use the `fmt` package in the following examples:

- `import format "fmt"` - Creates an alias of `fmt`. Preceed all `fmt` package content with `format.` instead of `fmt.`.

- `import . "fmt"` - Allows content of the package to be accessed directly, without the need for it to be preceded with `fmt`.

- `import _ "fmt"` - Suppresses compiler warnings related to `fmt` if it is not being used, and executes initialization functions if there are any. The remainder of `fmt` is inaccessible.

See [this](http://learngowith.me/alternate-ways-of-importing-packages/) blog post for more detailed information.


## goimports

[Goimports](http://godoc.org/code.google.com/p/go.tools/cmd/goimports) is a tool that updates your Go import lines, adding missing ones and
removing unreferenced ones.

It acts the same as gofmt (drop-in replacement) but in addition to code
formatting, also fixes imports.

## Organization

Go is a pretty easy programming language to learn but the hardest
thing for developers at first is how to organize their code. Rails
became popular for many reasons and scaffolding was one of them. It gave
new developers clear directions and places to put their code and idioms
to follow.

To some extent, Go does the same thing by providing developers with
great tools like [`go fmt`](http://blog.golang.org/go-fmt-your-code)
and by having a strict compiler that won't compile unused variables or
unused import statements.

## Custom Constructors
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

## Breaking down code in packages

See this blog post on [refactoring Go code](http://matt.aimonetti.net/posts/2014/04/28/refactoring-go-code/), the first part talks about package organization.


## Sets

You might want to find a way to extract unique value from a collection.
In other languages, you often have a set data structure not allowing
duplicates. Go doesn't have that built in, however it's not too hard to
implement (due to a lack of generics, you do need to do that for most
types, which can be cumbersome).

```go
// UniqStr returns a copy of the passed slice with only unique string results.
func UniqStr(col []string) []string {
	m := map[string]struct{}{}
	for _, v := range col {
		if _, ok := m[v]; !ok {
			m[v] = struct{}{}
		}
	}
	list := make([]string, len(m))

	i := 0
	for v := range m {
		list[i] = v
		i++
	}
	return list
}
```

[See in playground](http://play.golang.org/p/AtG9pTe8yt)

I used a few interesting tricks that are interesting to know.
First, the map of empty structs:

```go
  m := map[string]struct{}{}
```

We create a map with the keys being the values we want to be unique, the associated value doesn't really matter much so it could be anything. For instance:

```go
  m := map[string]bool{}
```

However I chose an empty structure because it will be as fast as a
boolean but doesn't allocate as much memory.

The second trick can been seen a bit further:

```go
if _, ok := m[v]; !ok {
  m[v] = struct{}{}
}
```

What we are doing here, is simply check if there is a value associated
with the key `v` in the map `m`, we don't care about the value itself,
but if we know that we don't have a value, then we add one.

Once we have a map with unique keys, we can extract them into a new
slice of strings and return the result.


Here is the test for this function, as you can see, I used a table test,
which is the idiomatic Go way to run unit tests:

```go
func TestUniqStr(t *testing.T) {

	data := []struct{ in, out []string }{
		{[]string{}, []string{}},
		{[]string{"", "", ""}, []string{""}},
		{[]string{"a", "a"}, []string{"a"}},
		{[]string{"a", "b", "a"}, []string{"a", "b"}},
		{[]string{"a", "b", "a", "b"}, []string{"a", "b"}},
		{[]string{"a", "b", "b", "a", "b"}, []string{"a", "b"}},
		{[]string{"a", "a", "b", "b", "a", "b"}, []string{"a", "b"}},
		{[]string{"a", "b", "c", "a", "b", "c"}, []string{"a", "b", "c"}},
	}

	for _, exp := range data {
		res := UniqStr(exp.in)
		if !reflect.DeepEqual(res, exp.out) {
			t.Fatalf("%q didn't match %q\n", res, exp.out)
		}
	}

}
```

[See in the playground](http://play.golang.org/p/elRIpSKGjD)


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


## Quick look at some compiler's optimizations
\label{sec:compiler_optimizations}

You can pass specific compiler flags to see what optimizations
are being applied as well as how some aspects of memory management.
This is an advanced feature, mainly for people who want to understand
some of the compiler optimizations in place.

Let's take the following code example from an earlier chapter:

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

func NewUser(id int, name, location string) *User {
	id++
	return &User{id, name, location}
}

func main() {
	u := NewUser(42, "Matt", "LA")
	fmt.Println(u.Greetings())
}
```

* [See in Playground](http://play.golang.org/p/7y0-u_FiKD)

Build your file (here called `t.go`) passing some `gcflags`:

```
$ go build -gcflags=-m t.go
# command-line-arguments
./t.go:15: can inline NewUser
./t.go:21: inlining call to NewUser
./t.go:10: leaking param: u
./t.go:10: leaking param: u
./t.go:12: (*User).Greetings ... argument does not escape
./t.go:15: leaking param: name
./t.go:15: leaking param: location
./t.go:17: &User literal escapes to heap
./t.go:15: leaking param: name
./t.go:15: leaking param: location
./t.go:21: &User literal escapes to heap
./t.go:22: main ... argument does not escape
```

The compiler notices that it can inline the `NewUser` function defined on line
15 and inline it on line 21.
[Dave Cheney](http://dave.cheney.net/) has a [great post](http://dave.cheney.net/2014/06/07/five-things-that-make-go-fast) about why Go's inlining is helping your
programs run faster.

Basically, the compiler moves the body of the `NewUser` function (L15) to where
it's being called (L21) and therefore avoiding the overhead of a function call but
increasing the binary size.

The compiler creates the equivalent of:

```go
func main() {
	id := 42 + 1
	u := &User{id, "Matt", "LA"}
	fmt.Println(u.Greetings())
}
```
On a few lines, you see the potentially alarming `leaking param`
message. It doesn't mean that there is a memory leak but that the param
is kept alive even after returning.
The "leaked params" are:

* On the `Greetings`'s method: `u` (receiver)
* On the `NewUser`'s functon: `name`, `location`

The reason why `u` "leaks" in the `Greetings` method is because it's
being used in the `fmt.Sprintf` function call as an argument.
`name` and `location` are also "leaked" because they are used in the
`User`'s literal value. Note that `id` doesn't leak because it's a
value, only references and pointers can leak.

X `argument does not escape` means that the argument doesn't "escape"
the function, meaning that it's not used outside of the function so it's
safe to store it on the stack.

On the other hand, you can see that `&User literal escapes to heap`.
What it means is that the address of a literal value is used outside of the function
and therefore can't be stored on the stack. The value _could_ be stored on the stack, 
except a pointer to the value escapes the function, so the value has to be moved to the heap to prevent the pointer referring to incorrect memory once the function returns.
 This is always the case when calling a method on a value and the method uses one or more fields.

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

## Iota: Elegant Constants
\label{sec:iota_elegant_constants}

Some concepts have names, and sometimes we care about those names, even (or
especially) in our code.

```go
const (
    CCVisa            = "Visa"
    CCMasterCard      = "MasterCard"
    CCAmericanExpress = "American Express"
)
```

At other times, we only care to distinguish one thing from the other. There
are times when there’s no inherently meaningful value for a thing. For
example, if we’re storing products in a database table we probably don’t want
to store their category as a string. We don’t care how the categories are
named, and besides, marketing changes the names all the time.

We care only that they’re distinct from each other.

```go
const (
    CategoryBooks    = 0
    CategoryHealth   = 1
    CategoryClothing = 2
)
```

Instead of 0, 1, and 2 we could have chosen 17, 43, and 61. The values are
arbitrary.

Constants are important but they can be hard to reason about and difficult to
maintain. In some languages like Ruby developers often just avoid them. In Go,
constants have many interesting subtleties that, when used well, can make the
code both elegant and maintainable.

### Auto Increment

A handy idiom for this in golang is to use the iota identifier, which
simplifies constant definitions that use incrementing numbers, giving the
categories exactly the same values as above.

```go
const (
    CategoryBooks = iota // 0
    CategoryHealth       // 1
    CategoryClothing     // 2
)
```

### Custom Types

Auto-incrementing constants are often combined with a custom type, allowing
you to lean on the compiler.

```go
type Stereotype int

const (
    TypicalNoob Stereotype = iota // 0
    TypicalHipster                // 1
    TypicalUnixWizard             // 2
    TypicalStartupFounder         // 3
)
```

If a function is defined to take an int as an argument rather than a
Stereotype, it will blow up at compile-time if you pass it a Stereotype:

```go
func CountAllTheThings(i int) string {
                return fmt.Sprintf("there are %d things", i)
}

func main() {
    n := TypicalHipster
    fmt.Println(CountAllTheThings(n))
}

// output:
// cannot use TypicalHipster (type Stereotype) as type int in argument to CountAllTheThings
```

The inverse is also true. Given a function that takes a Stereotype as an
argument, you can’t pass it an int:

```go
func SoSayethThe(character Stereotype) string {
    var s string
    switch character {
    case TypicalNoob:
        s = "I'm a confused ninja rockstar."
    case TypicalHipster:
        s = "Everything was better we programmed uphill and barefoot in the snow on the SUTX 5918"
    case TypicalUnixWizard:
        s = "sudo grep awk sed %!#?!!1!"
    case TypicalStartupFounder:
        s = "exploit compelling convergence to syndicate geo-targeted solutions"
    }
    return s
}

func main() {
    i := 2
    fmt.Println(SoSayethThe(i))
}

// output:
// cannot use i (type int) as type Stereotype in argument to SoSayethThe
```

There’s a dramatic twist, however. You could pass a number constant, and it would work:

```go
func main() {
    fmt.Println(SoSayethThe(0))
}

// output:
// I'm a confused ninja rockstar.
```

This is because constants in Go are loosely typed until they are used in a
strict context.

### Skipping Values

Imagine that you’re dealing with consumer audio output. The audio might not
have any output whatsoever, or it could be mono, stereo, or surround.

There’s some underlying logic to defining no output as 0, mono as 1, and
stereo as 2, where the value is the number of channels provided.

So what value do you give Dolby 5.1 surround?

On the one hand, it’s 6 channel output, but on the other hand, only 5 of those
channels are full bandwidth channels (hence the 5.1 designation – with the .1
referring to the low-frequency effects channel).

Either way, we don’t want to simply increment to 3.

We can use underscores to skip past the unwanted values.

```go
type AudioOutput int

const (
    OutMute AudioOutput = iota // 0
    OutMono                    // 1
    OutStereo                  // 2
    _
    _
    OutSurround                // 5
)
```

### Expressions

The iota can do more than just increment. Or rather, iota always increments,
but it can be used in expressions, storing the resulting value in the
constant.

Here we’re creating constants to be used as a bitmask.

```go
type Allergen int

const (
    IgEggs Allergen = 1 << iota // 1 << 0 which is 00000001
    IgChocolate                 // 1 << 1 which is 00000010
    IgNuts                      // 1 << 2 which is 00000100
    IgStrawberries              // 1 << 3 which is 00001000
    IgShellfish                 // 1 << 4 which is 00010000
)
```

This works because when you have only an identifier on a line in a const
group, it will take the previous expression and reapply it, with the
incremented iota. In the language of [the
spec](http://golang.org/ref/spec#Iota), this is called _implicit
repetition of the last non-empty expression list_.

If you’re allergic to eggs, chocolate, and shellfish, and flip those bits to
the “on” position (mapping the bits right to left), then you get a bit value
of 00010011, which corresponds to 19 in decimal.

```go
fmt.Println(IgEggs | IgChocolate | IgShellfish)

// output:
// 19
```

There’s a great example in [Effective
Go](https://golang.org/doc/effective_go.html#constants) for defining orders of
magnitude:

```go
type ByteSize float64

const (
    _           = iota              // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)  // 1 << (10*1)
    MB                              // 1 << (10*2)
    GB                              // 1 << (10*3)
    TB                              // 1 << (10*4)
    PB                              // 1 << (10*5)
    EB                              // 1 << (10*6)
    ZB                              // 1 << (10*7)
    YB                              // 1 << (10*8)
)
```

Today I learned that after zettabyte we get yottabyte. #TIL

### But Wait, There's More

What happens if you define two constants on the same line?
What is the value of Banana? 2 or 3? And what about Durian?

```go
const (
    Apple, Banana = iota + 1, iota + 2
    Cherimoya, Durian
    Elderberry, Fig
)
```

The iota increments on the next line, rather than as soon as it gets referenced.

```go
// Apple: 1
// Banana: 2
// Cherimoya: 2
// Durian: 3
// Elderberry: 3
// Fig: 4
```

Which is messed up, because now you have constants with the same value.

### So, Yeah

There's a lot more to be said about constants in Go, and you should probably
read Rob Pike's [blog post on the subject](http://blog.golang.org/constants)
over on the golang blog.

_This was first published by Katrina Owen on the [Splice
blog](https://splice.com/blog/iota-elegant-constants-golang/)._

## Web resources

* [Dave Cheney](https://twitter.com/davecheney) maintains a [list of resources](http://dave.cheney.net/resources-for-new-go-programmers) for new Go developers.
