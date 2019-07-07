---
layout: post
title: Golang notes
subtitle: Some interesting basic stuffs  of Golang
comments: true
---
#Basic 
## Variables Syntax style
Variable name first, then variable type
Ex: 

```go
i int
f float64
```
Declare variable
```go
var b int
```
Initialize variable
```go
//explicit initialization
var b,c int = 1,2
//implicity initialization
d := 4.0
```

## Type Conversion
```go
var d = 5
var e = float64(d)
```
## Functions
### Function declaration
```go
func A(a, b int) int {

}
func B(a int , b float) int {

}
```
### Function return multiple results
```go
func swap(a string, b string) (string, string) {
    return y, x
}
```
### naked return
```go
func split(sum int) (x, y int) {
    x = sum * 4 / 9
	y = sum - x
	return
}
```
## Constant
Constants are declared like variables, but with the const keyword.
Constants cannot be declared using the := syntax.
```go
const Pi = 3.14
const (
    Big = 1 << 100
    Small = Big >> 99
)
```
#Flow control and statements
## For
Syntax of For also carries on the declaration of while syntax
```go
//for statement
sum := 0
for i := 0; i < 10; i++ {
	sum += i
}
//remove initial and post statements
sum := 0
for ; sum < 1000; {
    sum += sum
}
##
```
## If
Like for, if can start with short statement before executing the condition
```go
import (
    "math"
    "fmt"
)
func pow(x, n, lim float64) float64 {
    //initialize v with short statement first
	if v := math.Pow(x, n); v < lim {
        //v can be understood in if scope when v is declared / assigned before executing the conditon
		return v
	}
	return lim
}
``` 
## Switch
Go only run the select cased, no need to be use break statement as Go auto provide it
```go
switch os:=runtime.GOOS; os {
    case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}
```
The switch valuation order is top -> down, stopping when the case succeed
```go
switch time.Saturday {
	case time.Now().Weekday() + 0:
		fmt.Println("Today.")
	case time.Now().Weekday() + 1:
		fmt.Println("Tomorrow.")
	case time.Now().Weekday() + 2:
		fmt.Println("In two days.")
	default:
		fmt.Println("Too far away.")
	}
```
Switch with no condition
```go
var t := time.Now()
switch {
    case t.Hour() < 12:
        fmt.Println("Good morning")
    case t.Hour() < 17:
        fmt.Println("Good afternoon")
    default:
        fmt.Prinln("Good evening")
}

```
##defer
A defer statement defers the execution of a function until the surrounding function returns.
The deferred call's arguments are evaluated immediately, but the function call is not executed until the surrounding function returns.

```go
func main() {
    defer fmt.Println("world")
    fmt.Println("Hello")
}
```
Stacking defer, first in , last out
```go
func main() {
    fmt.Println("counting")
    for (var i:= 0; i < 10; i++) {
        defer fmt.Println(i)
    }
    fmt.Println("done")
}
```







