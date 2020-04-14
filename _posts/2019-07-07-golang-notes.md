---
layout: post
title: Golang variables, control statements
subtitle: Golang variables, control statements
comments: true
tags: [golang, basic]
---
# Table of Contents
1. [Variable Syntax styles](#variable-syntax-styles)
2. [Type Conversion](#type-conversion)
3. [Functions](#functions)
4. [Constants](#constant)
5. [For](#for)
6. [If](#if)
7. [Switch](#switch)
8. [Defer](#defer)
9. [Block And Scope](#block-scope)
# Variable Syntax styles
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

# Type Conversion
```go
var d = 5
var e = float64(d)
```
# Functions
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
# Constant
Constants are declared like variables, but with the const keyword.
Constants cannot be declared using the := syntax.
```go
const Pi = 3.14
const (
    Big = 1 << 100
    Small = Big >> 99
)
```

# For
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
# If
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
# Switch
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
# defer
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

# Block Scope

```go
import "fmt"

var i = 1

func main() {
  fmt.Println("(1) i=",i) // [1]    output 1
  if true {
    fmt.Println("(2) i=",i) // [2]  output 1
    i, err := 3, true
    fmt.Println("(3) i=",i,err) // [3]  output 3
    reassign()  //output 2
    fmt.Println("(4) i=",i,err) // [4]  output 3
  }
  fmt.Println("(5) i=",i) // [5]    output 1 as i in outer scope of if
}

func reassign() {
    i, err := 2, true
    fmt.Println("(r) i=",i, err) // [r]
}

```
Read more at [Scope](https://medium.com/golangspec/scopes-in-go-a6042bb4298c) and [Block](https://medium.com/golangspec/blocks-in-go-2f68768868f6)
