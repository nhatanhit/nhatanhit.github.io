---
layout: post
title: Golang methods , interfaces
subtitle: Golang methods , interfaces
comments: true
image: /img/gopher.gif
---
# Table of Contents
1. [Method](#methods)
2. [Interfaces](#interfaces)    

# Methods
Go does not provide classes.  However you can provide the methods for one  struct, or any user defined type by binding it to a function as receiver
## Binding struct/defined type as a receiver
```go
type Vertex struct {
	X, Y float64
}
type MyFloat float64
//binding struct Vertex to  method Abs as a receiver 
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
//binding new defined type MyFloat to method Abs as  a receiver
func (f float64) Abs() float64 {
    if f < 0 {
		return float64(-f)
	}
	return float64(f)
}
func main() {
	v := Vertex{3, 4}
    fmt.Println(v.Abs())    //calling an method through object
    f := MyFloat(-math.Sqrt2)   //initialize MyFloat
    fmt.Println(f.Abs())
}
```

You can only declare a method with a receiver whose type is defined in the same package as the method. You cannot declare a method with a receiver whose type is defined in another package 
## Pointer receiver
You can declare methods with pointer receivers . The receiver type has the literal syntax *T for some type T. (Also, T cannot itself be a pointer such as *int.)
With a value receiver, it's similar with other languages (C++). It pass argument by value, copy value of arguement to the function then mainpulate on it. When function return, the value of of argument still not changed. 
Thus, when you want to change the object through its methods , you have to pass the object as  a reference 
When passing as reference/ pointer. By declaring the receiver as pointer,  Golang intepreter is allow both   value / reference argument is passed into the method and can change them correspondingly  

```go

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {    //using pointer receiver
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) { //pass by reference to change the object itself
	v.X = v.X * f
	v.Y = v.Y * f
}
func (i *int) Scale(i float64) {    //run time error, "cannot define new methods on non-local type int
    fmt.Println("I modified int pointer")
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10) //pass by reference to change the object itself
	fmt.Println(v.Abs()) //even though the calling object is object (not pointer) while Abs() using pointer receiver, this statement still works, output 50
}
```
## Pointers and function
Unlike methods,  The normal function require the passed argument must be correct type (reference or value)

```go
type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func Scale(v Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	Scale(&v, 10)   //error cannot use &v (type *Vertex) as type Vertex in argument to Scale
	fmt.Println(Abs(v))
}

```
To fix above issue, you must change the parameter in function Scale to reference type

```go
type Vertex struct {
	X, Y float64
}
func Scale(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	Scale(&v, 10)   
	
}
```
## Choosing a value or pointer receiver
There are two reasons to use a pointer receiver.

The first is so that the method can modify the value that its receiver points to.

The second is to avoid copying the value on each method call. This can be more efficient if the receiver is a large struct, for example.

# Interfaces
An interface type is defined as a set of method signatures.
A value of interface type can hold any value that implements those methods.
```go
type Abser interface {
	Abs() float64
}
type MyFloat float64
type Vertex struct {
	X, Y float64
}
func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}
func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
func main() {
	var a Abser
	f := MyFloat(-math.Sqrt2)
	v := Vertex{3, 4}

	a = f  // a MyFloat implements Abser
	a = &v // a *Vertex implements Abser

	// In the following line, v is a Vertex (not *Vertex)
	// and does NOT implement Abser.
	a = v

	fmt.Println(a.Abs())
}
```

{: .box-note}
**Note:** if the method is not defined in interface type, when declaring the receiver as pointer,  Golang intepreter is allow both   value / reference argument is passed into the method and can change them correspondingly. But if method is defined in interface type, the statement isn't correct 




