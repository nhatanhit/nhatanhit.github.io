---
layout: post
title: Golang structures, slice, pointer, map, closure
subtitle: Golang structures, slice, pointer, map, closure
comments: true
image: /img/gopher.gif
---
# Table of Contents
1. [Struct](#structure)
2. [Pointer](#pointer)
3. [Array](#array)
4. [Slices](#slices)
5. [Maps](#maps)
6. [Function](#function)
7. [More articles](#more-articles)
# Structure
Declare Struct
```go
type Vertex struct {
    X, Y int
}
type chart struct {
    X int
    val float64
}
//initialize struct
func main() {
    v := Vertex{1,2}
    fmt.Println(v)
    fmt.Println(Vertex{5,6})
    //Y default
    v1 := Vertex{X: 1}
    //X, Y default
    v2 := Vertex{}
    //access variable member
    v.X = 5
    fmt.Println(v.X)
}
```
# Pointer
The type *T is a pointer point to a T value
```go
var p *int
//amperand , getting address of variable
i := 42
p := &i
fmt.Println(*p)
//modify value of i through pointer p
*p = 43
//pointer to structure
v := Vertex{1,2}
p := &v
p2 := &Vertex{1,2}
p.X = 1e9
```

# Array
Array Declaration
```go
var a[2]string
var b[3]int
//initialize array
primes :=[6]int{2, 3, 5, 7, 11, 13}
fmt.Println(primes)
```

Notation [...]
```go
package main

import (
	_"fmt"
)

func main() {
	arr1 := [3][...]float32 { {0:1,2:0}, {1:1,2:0}, {2:1} }
	arr3 := [...][...]float32 { {0:1,2:0}, {1:1,2:0}, {2:1} }
	
}
//compiled error: use of [...] array outside of array literal
```
[Explanation](https://stackoverflow.com/questions/27116462/creating-composite-literal-of-array-of-arrays) 

# Slices
A slice is a reference to an underlying array, it's dynamically-sized, it provides a "view" to an array
A slice is formed by specifying two indices, a low and high bound, separated by a colon:
```text
a[low : high]
```
```go
//declare array
primes := [6]int{2, 3, 5, 7, 11, 13}
//declare an slice, to provide a view to array
var s []int =  primes[1 : 4]
fmt.Println(s); //output [3 5 7]
```
Changing the elements of a slice modifies the corresponding elements of its underlying array.
Other slices that share the same underlying array will see those changes.
```go
names := [4]string {
    "John",
	"Paul",
	"George",
	"Ringo"
}
a := names[0:2]
b := names[1:3]
//change first elemnt of slice b, means change value of second element of underlying array
b[0] = "XXX"
fmt.Println(names) //output [John XXX George Ringo]
```
### Slice literals
slice literal is like array literal without length
```go
//with go built-type
s := []int{2, 3, 5, 7, 11, 13}
//with struct
s :=[]struct{
    i int
	b bool
}{
    {2, true},
    {3, false},
    {5, true},
    {7, true},
    {11, false},
    {13, true},
}
//already defined the struct before
type Vertex struct {
    X int,
    Y int
}
s := []Vertex{
    {2, true},
    {3, false},
    {5, true},
    {7, true},
    {11, false},
    {13, true},
}
//nil type
var nSlice []int
fmt.Println(s, len(nSlice), cap(nSlice))
if nSlice == nil {
	fmt.Println("nil!")
}
```
### slice upper bounds and lower bounds
```go
s := []int{2, 3, 5, 7, 11, 13}
y := s[1:4] //output 3,5,7
fmt.Println(y)
z := s[:2] // output 2,3
fmt.Println(z)
t := s[:] //output full array
fmt.Println(t)
w := s[1:] //output 3, 5, 7, 11, 13

//operates on slice itself
s := s[1:4] //output 3,5,7
fmt.Println(s)

s := s[:2] // output 3,5
fmt.Println(s)

s := s[1:] //output 5
fmt.Println(s)
```
### length and capacity
The length of a slice is the number of elements it contains.
The capacity of a slice is the number of elements in the underlying array, counting from the first element in the slice.  / the possible number of elements a slice can contain
```go
func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlice(s) //len = 6, cap = 6, [2 3 5 7 11 13]

	// Slice the slice to give it zero length.
	s = s[:0]
	printSlice(s)   //len = 0, cap = 6 , []

	// Extend its length.
	s = s[:4]   
	printSlice(s)   //len = 4, cap = 6 [2 3 5 7]

    // Drop its first two values, as we drop elements, it's possible for capacity of slice s shrink to 4
    //even it's shrunk to 4, it is able to contain 2 elements
	s = s[2:]       
	printSlice(s)   //len = 2, cap = 4 [5,7]
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```

### Creating slices with make
```go
func main() {
	a := make([]int, 5)
	printSlice("a", a)  // len=5, cap = 5 {0,0,0,0,0}

	b := make([]int, 0, 5)  //len 0, cap = 5 {}
	printSlice("b", b)

	c := b[:2]
	printSlice("c", c)  // len = 2, cap = 5 {0,0}

	d := c[2:5]
    printSlice("d", d)  //len = 3, cap = 3 
    
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}
```
### Array of slices 
```go
//array of array slice 
board := [][]string{
    []string{"_", "_", "_"},
    []string{"_", "_", "_"},
    []string{"_", "_", "_"},
}
```
### Append to a slice

```text
func append(s []T, vs ...T) []T
```
The first parameter s of append is a slice of type T, and the rest are T values to append to the slice.
The resulting value of append is a slice containing all the elements of the original slice plus the provided values.
If the backing array of s is too small to fit all the given values a bigger array will be allocated. The returned slice will point to the newly allocated array.

```go
func main() {
	var s []int
	printSlice(s)   //len = 0, cap = []

	// append works on nil slices.
	s = append(s, 0)    
	printSlice(s)   //len = 1 , cap = 2 [0]

	// The slice grows as needed.
	s = append(s, 1)    
	printSlice(s)   //len = 2, cap = 2 [0 1]

	// We can add more than one element at a time.
	s = append(s, 2, 3, 4) //len = 5 , cap = 2 ^ 3 > 5 = 8
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```
### Range
The range form of the for loop iterates over a slice or map.
When ranging over a slice, two values are returned for each iteration. The first is the index, and the second is a copy of the element at that index.

```go
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
    }
    //skip the index , replacing it by _
    for _, value := range pow {
		fmt.Printf("%d\n", value)
	}
}
```
# Maps
Similar with other languages
```go
type Vertex struct {
	Lat, Long float64
}


func main() {
    //create a map with built-in function make
    var m map[string]Vertex
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
    }
    fmt.Println(m["Bell Labs"])

    //initialize a map with a struct
    var initMap = map[string]Vertex{
	    "Bell Labs": Vertex{
		    40.68433, -74.39967,
	    },
	    "Google": Vertex{
		    37.42202, -122.08408,
	    },
    }
    //or
    var initMapAgain = map[string]Vertex{
	    "Bell Labs": {40.68433, -74.39967},
	    "Google":    {37.42202, -122.08408},
    }

}
```

### Map operations
We can edit / delete values in map , similar with other languages, or test if the key in map existed
```go
m := make(map[string]int)

m["Answer"] = 42
fmt.Println("The value:", m["Answer"])

m["Answer"] = 48
fmt.Println("The value:", m["Answer"])

delete(m, "Answer")
fmt.Println("The value:", m["Answer"])

v, ok := m["Answer"]
fmt.Println("The value:", v, "Present?", ok)
```

# Function 
Functions are values too. They can be passed around just like other values.
Function values may be used as function arguments and return values.
```go
//pass function as argument, with parameter , along with return value
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```

### Function closure
 A closure is a function value that references variables from outside its body. The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables

```go
func adder() func(int) int {
	sum := 0    //value sum is still kept even though the function adder is returned
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos := adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			
		)
	}
}
//output 0 , 1, 3, 6, 10 , 15 , 21, 28 , 36 , 45
```

# More articles
[Golang tips: why pointers to slices are useful and how ignoring them can lead to tricky bugs](https://medium.com/swlh/golang-tips-why-pointers-to-slices-are-useful-and-how-ignoring-them-can-lead-to-tricky-bugs-cac90f72e77b)