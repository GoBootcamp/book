#Collection Types
\label{cha:collection_types}


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

* [Golang tour page](https://tour.golang.org/moretypes/6)

You can also set the array entries as you declare the array:

```go
package main

import "fmt"

func main() {
    a  := [2]string{"hello", "world!"}
    fmt.Printf("%q", a)
}
```

* [See in Playground](http://play.golang.org/p/87fdeul4H7)

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

* [See in Playground](http://play.golang.org/p/lxVUhtyJJP)


### Printing arrays

Note how we used the [`fmt` package](http://golang.org/pkg/fmt/) using `Printf` and used the `%q` "verb" to print each element quoted.

If we had used `Println` or the `%s` verb, we would have had a different result:

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

* [See in Playground](http://play.golang.org/p/jsvGaXW6uH)


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

* [See in Playground](http://play.golang.org/p/L6faG4RMPx)

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
error that tells us that we are trying to access an index that is out of
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

* [Go tour page](https://tour.golang.org/moretypes/7)


### Slicing a slice

Slices can be re-sliced, creating a new slice value that points to the same array.

The expression

```go
s[lo:hi]
```

evaluates to a slice of the elements from `lo` through `hi-1`, inclusive. Thus

```go
s[lo:lo]
```

is empty and

```go
s[lo:lo+1]
```
has one element.

Note: `lo` and `hi` would be integers representing indexes.

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

* [See in Playground](http://play.golang.org/p/2z_6hNt_Vg)
* [Go tour page](https://tour.golang.org/moretypes/8)


### Making slices

Besides creating slices by passing the values right away (slice literal), you can also use `make`.
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

* [See in Playground](http://play.golang.org/p/CX5z79KYsK)

It works by allocating a zeroed array and returning a slice that refers to that array.


### Appending to a slice

Note however, that you would get a runtime error if you were to do that:

```go
cities := []string{}
cities[0] = "Santa Monica"
```

As explained above, a slice is seating on top of an array, in this case,
the array is empty and the slice can't set a value in the referred array.
There is a way to do that though, and that is by using the `append`
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

* [See in Playground](http://play.golang.org/p/rNRt6jvlFl)

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

* [See in Playground](http://play.golang.org/p/fdh3daTttz)

And you can also append a slice to another using an ellipsis:

```go
package main

import "fmt"

func main() {
	cities := []string{"San Diego", "Mountain View"}
	otherCities := []string{"Santa Monica", "Venice"}
	cities = append(cities, otherCities...)
	fmt.Printf("%q", cities)
	// ["San Diego" "Mountain View" "Santa Monica" "Venice"]
}
```

* [See in Playground](http://play.golang.org/p/CjR88q7CIo)

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

* [See in Playground](http://play.golang.org/p/EvvVrOpPII)


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

* [See in Playground](http://play.golang.org/p/inw1CunExE)
* [Go tour page](https://tour.golang.org/moretypes/10)


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
Being able to iterate over all the elements of a data structure is
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

* [Go tour page](https://tour.golang.org/moretypes/12)

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

* [Go tour page](https://tour.golang.org/moretypes/13)


### Break & continue

As if you were using a normal for loop, you can stop the iteration
anytime by using `break`:

```go
package main

import "fmt"

func main() {
	pow := make([]int, 10)
	for i := range pow {
		pow[i] = 1 << uint(i)
		if pow[i] >= 16 {
			break
		}
	}
	fmt.Println(pow)
	// [1 2 4 8 16 0 0 0 0 0]
}
```

* [See in Playground](http://play.golang.org/p/1S4ApCLxaD)

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

* [See in Playground](http://play.golang.org/p/TT1vfYKpOy)


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

* [See in Playground](http://play.golang.org/p/rg5sc_Nl-P)


### Exercise

Given a list of names, you need to organize each name
within a slice based on its length.

```go
package main

var names = []string{"Katrina", "Evan", "Neil", "Adam", "Martin", "Matt",
	"Emma", "Isabella", "Emily", "Madison",
	"Ava", "Olivia", "Sophia", "Abigail",
	"Elizabeth", "Chloe", "Samantha",
	"Addison", "Natalie", "Mia", "Alexis"}

func main() {
	// insert your code here
}
```

* [See in Playground](http://play.golang.org/p/o1YicfGXCx)

After you implement your solution, you should get the following output
(slice of slice of strings):

```go
[[] [] [Ava Mia] [Evan Neil Adam Matt Emma] [Emily Chloe]
[Martin Olivia Sophia Alexis] [Katrina Madison Abigail Addison Natalie]
[Isabella Samantha] [Elizabeth]]
```


### Solution

```go
package main

import "fmt"

var names = []string{"Katrina", "Evan", "Neil", "Adam", "Martin", "Matt",
	"Emma", "Isabella", "Emily", "Madison",
	"Ava", "Olivia", "Sophia", "Abigail",
	"Elizabeth", "Chloe", "Samantha",
	"Addison", "Natalie", "Mia", "Alexis"}

func main() {
	var maxLen int
	for _, name := range names {
		if l := len(name); l > maxLen {
			maxLen = l
		}
	}
	output := make([][]string, maxLen)
	for _, name := range names {
		output[len(name)-1] = append(output[len(name)-1], name)
	}

	fmt.Printf("%v", output)
}
```

* [See in Playground](http://play.golang.org/p/gMOouTnYvC)

There are a few interesting things to note.
To avoid an out of bounds insert, we need our
`output` slice to be big enough. But we don't want it to
be too big. That's why we need to do a first pass through all the names
and find the longest.
We use the longest name length to set the length of the `output` slice
length.
Slices are zero indexed, so when inserting the names, we need to get the
length of the name minus one.


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

```
map[string]int{"Nicolas Cage":50, "Selena Gomez":21, "Jude Law":41,
    "Scarlett Johansson":29}
```

* [See in Playground](http://play.golang.org/p/ttJ-3xgzuk)

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

* [See in Playground](http://play.golang.org/p/NW1ODtLARA)

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

* [See in Playground](http://play.golang.org/p/nvGq-9gQ5z)


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

* [See in Playground](http://play.golang.org/p/pvfk9maSsh)

If `key` is in `m`, `ok` is true. If not, `ok` is false and `elem` is the zero value for the map's element type.
Similarly, when reading from a map if the key is not present the result is the zero value for the map's element type.


### Resources

* [Go team blog post on maps](http://blog.golang.org/go-maps-in-action)
* [Effective Go - maps](http://golang.org/doc/effective_go.html#maps)


### Exercise

Implement WordCount.

```go
package main

import (
    "golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
    return map[string]int{"x": 1}
}

func main() {
    wc.Test(WordCount)
}
```

* [See in Playground](http://play.golang.org/p/-7aN1ASYYx)
* [Online assignment](https://tour.golang.org/moretypes/19)

It should return a map of the counts of each "word" in the string `s`.
The `wc.Test` function runs a test suite against the provided function and prints success or failure.

You might find [strings.Fields](http://golang.org/pkg/strings/#Fields) helpful.


### Solution

```go
package main

import (
	"golang.org/x/tour/wc"
	"strings"
)

func WordCount(s string) map[string]int {
	words := strings.Fields(s)
	count := map[string]int{}
	for _, word := range words {
		count[word]++
	}
	return count
}

func main() {
	wc.Test(WordCount)
}
```

* [See in Playground](http://play.golang.org/p/M0bb5rWa7t)
