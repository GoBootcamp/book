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

You have 50 bitcoins to distribute to 10 users:
Matthew, Sarah, Augustus, Heidi, Emilie,
Peter, Giana, Adriano, Aaron, Elizabeth
The coins will be distributed based on the vowels contained in each name where:

a: 1 coin
e: 1 coin
i: 2 coins
o: 3 coins
u: 4 coins

and a user can't get more than 10 coins.
Print a map with each user's name and the amount of coins
distributed. After distributing all the coins, you should have 2 coins
left.


The output should look something like that:

```
map[Matthew:2 Peter:2 Giana:4 Adriano:7 Elizabeth:5 Sarah:2 Augustus:10 Heidi:5 Emilie:6 Aaron:5]
Coins left: 2
```

Note that Go doesn't keep the order of the keys in a map, so your
results might not look exactly the same but the key/value mapping should
be the same.

Here is some starting code:

```go
package main

import "fmt"

var (
	coins = 50
	users = []string{
		"Matthew", "Sarah", "Augustus", "Heidi", "Emilie",
		"Peter", "Giana", "Adriano", "Aaron", "Elizabeth",
	}
	distribution = make(map[string]int, len(users))
)

func main() {
	fmt.Println(distribution)
	fmt.Println("Coins left:", coins)
}
```

[See in Playground](http://play.golang.org/p/jaKZWoCHbD)

### Solution

```go
package main

import "fmt"

var (
	coins = 50
	users = []string{
		"Matthew", "Sarah", "Augustus", "Heidi", "Emilie",
		"Peter", "Giana", "Adriano", "Aaron", "Elizabeth",
	}
	distribution = make(map[string]int, len(users))
)

func main() {
	coinsForUser := func(name string) int {
		var total int
		for i := 0; i < len(name); i++ {
			switch string(name[i]) {
			case "a", "A":
				total++
			case "e", "E":
				total++
			case "i", "I":
				total = total + 2
			case "o", "O":
				total = total + 3
			case "u", "U":
				total = total + 4
			}
		}
		return total
	}

	for _, name := range users {
		v := coinsForUser(name)
		if v > 10 {
			v = 10
		}
		distribution[name] = v
		coins = coins - v
	}
	fmt.Println(distribution)
	fmt.Println("Coins left:", coins)
}
```

[See in Playground](http://play.golang.org/p/D0HfGeICyj)
