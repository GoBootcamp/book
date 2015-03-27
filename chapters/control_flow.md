#Control flow
\label{cha:control_flow}


## If statement
\label{sec:if-statement}

The `if` statement looks as it does in C or Java, except that the `( )`
are gone and the `{ }` are required. Like `for`, the `if` statement can start with a short statement to execute before the condition.
Variables declared by the statement are only in scope until the end of
the `if`.
Variables declared inside an if short statement are also available inside any of the else blocks.

* [`If` statement example](https://tour.golang.org/flowcontrol/)


```go
if answer != 42 {
    return "Wrong answer"
}
```


* [`If` with a short statement](https://tour.golang.org/flowcontrol/6)


```go
if err := foo(); err != nil {
    panic(err)
}
```

* [`If` and `else` example](https://tour.golang.org/flowcontrol/7)


## For Loop
\label{sec:for-loop}

Go has only one looping construct, the for loop.
The basic for loop looks as it does in C or Java, except that the ( ) are gone (they are not even optional) and the { } are required.
As in C or Java, you can leave the pre and post statements empty.


* [For loop example](https://tour.golang.org/flowcontrol/1)

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

* [For loop without pre/post statements](https://tour.golang.org/flowcontrol/2)

```go
sum := 1
for ; sum < 1000; {
    sum += sum
}
```

* [For loop as a `while` loop](https://tour.golang.org/flowcontrol/3)

```go
sum := 1
for sum < 1000 {
    sum += sum
}
```

* [Infinite loop](https://tour.golang.org/flowcontrol/4)

```go
for {
  // do something in a loop forever
}
```

## Switch case statement
\label{sec:switch_case_statement}

Most programming languages have some sort of switch case statement
to allow developers to avoid doing complex and ugly series of `if else` 
statements.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	now := time.Now().Unix()
	mins := now % 2
	switch mins {
	case 0:
		fmt.Println("even")
	case 1:
		fmt.Println("odd")
	}
}
```

* [See in Playground](http://play.golang.org/p/1_T5_w8yJm)

There are a few interesting things to know about this statement in Go:

* You can only compare values of the same type.

* You can set an optional default statement to be executed if all the
  others fail.

* You can use an expression in the case statement, for instance you can
  calculate a value to use in the case:

```go
package main

import "fmt"

func main() {
	num := 3
	v := num % 2
	switch v {
	case 0:
		fmt.Println("even")
	case 3 - 2:
		fmt.Println("odd")
	}
}
```

* [See in Playground](http://play.golang.org/p/Gu1Ey1M8uI)

* You can have multiple values in a case statement:

```go
package main

import "fmt"

func main() {
	score := 7
	switch score {
	case 0, 1, 3:
		fmt.Println("Terrible")
	case 4, 5:
		fmt.Println("Mediocre")
	case 6, 7:
		fmt.Println("Not bad")
	case 8, 9:
		fmt.Println("Almost perfect")
	case 10:
		fmt.Println("hmm did you cheat?")
	default:
		fmt.Println(score, " off the chart")
	}
}
```

* [See in Playground](http://play.golang.org/p/KHjUOUtWgv)


* You can execute all the following statements after a match using the
  `fallthrough` statement:

```go
package main

import "fmt"

func main() {
	n := 4
	switch n {
	case 0:
		fmt.Println("is zero")
		fallthrough
	case 1:
		fmt.Println("is <= 1")
		fallthrough
	case 2:
		fmt.Println("is <= 2")
		fallthrough
	case 3:
		fmt.Println("is <= 3")
		fallthrough
	case 4:
		fmt.Println("is <= 4")
		fallthrough
	case 5:
		fmt.Println("is <= 5")
		fallthrough
	case 6:
		fmt.Println("is <= 6")
		fallthrough
	case 7:
		fmt.Println("is <= 7")
		fallthrough
	case 8:
		fmt.Println("is <= 8")
		fallthrough
	default:
		fmt.Println("Try again!")
	}
}
```

```
is <= 4
is <= 5
is <= 6
is <= 7
is <= 8
Try again!
```

* [See in Playground](http://play.golang.org/p/Se9GbB1QCr)

You can use a `break` statement inside your matched statement to exit
the switch processing:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	n := 1
	switch n {
	case 0:
		fmt.Println("is zero")
		fallthrough
	case 1:
		fmt.Println("<= 1")
		fallthrough
	case 2:
		fmt.Println("<= 2")
		fallthrough
	case 3:
		fmt.Println("<= 3")
		if time.Now().Unix()%2 == 0 {
			fmt.Println("un pasito pa lante maria")
			break
		}
		fallthrough
	case 4:
		fmt.Println("<= 4")
		fallthrough
	case 5:
		fmt.Println("<= 5")
	}
}
```

* [See in Playground](http://play.golang.org/p/rFalZllNn1)

```
<= 1
<= 2
<= 3
un pasito pa lante maria
```

## Exercise
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

* [See in Playground](http://play.golang.org/p/jaKZWoCHbD)

## Solution

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

* [See in Playground](http://play.golang.org/p/D0HfGeICyj)
