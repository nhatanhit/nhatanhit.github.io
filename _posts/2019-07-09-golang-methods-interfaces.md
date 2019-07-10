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
When passing as reference/ pointer. 

{: .box-note}
**Note:** By declaring the receiver as pointer,  Golang intepreter  allows both   value / reference argument is passed into the method and modify them  even though the passing type is pass by value

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

## Interfaces values
Under the hood, interface values can be thought of as a tuple of a value and a concrete type:

```text
(value, type)
```
An interface value holds a value of a specific underlying concrete type.
Calling a method on an interface value executes the method of the same name on its underlying type.

```go
type I interface {
	M()
}

type T struct {
	S string
}
type F float64
func (t *T) M() {
	fmt.Println(t.S)
}
func (f F) M() {
	fmt.Prinln(f)
}
func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}
func main() {
	var i I

	i = &T{"Hello"}
	describe(i)
	i.M()	//output (&{Hello}, *main.T)

	i = F(math.Pi)
	describe(i)	//output (3.141592653589793, main.F)
	i.M()
}
```

## Interface values with nil underlying values
If the concrete value inside the interface itself is nil, the method will be called with a nil receiver.

{: .box-note}
**Note:** An interface value that holds a nil concrete value is itself non-nil.

```go
type I interface {
	M()
}

type T struct {
	S string
}
func (t *T) M() {
	if (t == nil) {
		fmt.Println("<nil")
		return
	}
	fmt.Println(t.S)
}
func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}

func main() {
	var i I
	// nil interface
	var i2 I	
	var t T
	var pt *T
	i = &t	//get address of null object, but address is not null
	describe(i)	//output (&{}, *main.T)
	i.M()
	i = pt	//null address
	describe(i)	//(<nil>, *main.T)
	i.M()
	describe(i2) //(<nil>, <nil>)
}
```

## Empty interface
The empty interface contains no method
A way to init empty interface with concrete type and concrete value is assignment 

```go

type EI interface{}
func describe(i interface{}) {
	fmt.Printf("(%v, %T)\n", i, i)
}
func describeEI(i EI) {
	fmt.Printf("(%v, %T)\n", i, i)
}
func main() {
	var i interface{}
	describe(i)	//(<nil>, <nil>)

	i = 42
	describe(i)	//(42, int)

	i = "hello"
	describe(i)	//(hello, string)
	
	var ei EI
	describe(ei)	//(<nil>, <nil>)

	ei = "Hello world"
	describe(ei)	//(hello world, string)

	var anotherInterface interface{} = "init from assignement"
	describe(anotherInterface)	// ("init from assignment" , string)
}
```
{: .box-note}
**Note:** The nil interface contains the functions, but the concrete type and concrete value is nil. The empty interface doesn't contains any functions, but it can receive any concrete type / value through assigment and execute methods of those concrete types from which it is assigned.

## Assertion: A way to verify if we can access the concrete value underlying the interface
A type assertion provides access to an interface value's underlying concrete value.
```text
t := i.(T)
```
This statement asserts that the interface value i holds the concrete type T and assigns the underlying T value to the variable t.

If i does not hold a T, the statement will trigger a panic.

A way to verify if i hold a T 
```text
t, ok := i.(T)
if (ok == false) {
	//panic, i currently not hold concrete type T
}
```

Example
```go
func describe(i interface{}) {
	fmt.Printf("(%v, %T)\n", i, i)
}
func main() {
	var i interface{} = "hello"
	describe(i) //{hello , string}
	s := i.(string)
	fmt.Println(s)	//hello

	s, ok := i.(string)
	fmt.Println(s, ok)  //hello, true

	f, ok := i.(float64)
	fmt.Println(f, ok) //0, false

	f = i.(float64) // panic
	fmt.Println(f)
}
```
## The type switch

A type switch is like a regular switch statement, but the cases in a type switch specify types (not values), and those values are compared against the type of the value held by the given interface value.


{: .box-note}
**Note:** Because i.(type) return the value , not type. And the type is only identified through case of switch so getting the concrete type of one interface must be executed inside the type switch.

Example
```go
func do(i interface{}) {
	
	switch v:= i.(type)  {	//v is value, 
	case int:	//switch by type
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:	
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	do(21)
	do("hello")
	do(true)
}
```