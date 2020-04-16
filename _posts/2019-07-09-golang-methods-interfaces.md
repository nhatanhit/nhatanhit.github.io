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
You can only declare a method with a receiver whose type is defined in the same package as the method. You cannot declare a method with a receiver whose type is defined in another package 

A value receiver of a method can accept both value and pointer to pass into the method. 
See method Abs in below

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
	
	v2 := &Vertex{3, 4}
	fmt.Println(v2.Abs())	//calling method through pointer

    f := MyFloat(-math.Sqrt2)   //initialize MyFloat
    fmt.Println(f.Abs())
}
```


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
func (v Vertex) swapCoordinate() {
	t := v.X
	v.X = v.Y
	v.Y = t
}


func (i *int) Scale(i float64) {    //run time error, "cannot define new methods on non-local type int
    fmt.Println("I modified int pointer")
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10) //pass by reference to change the object itself
	fmt.Println(v.Abs()) //even though the calling object is object (not pointer) while Abs() using pointer receiver, this statement still works, output 50

	v.swapCoordinate()
	//because receiver of  swapCoordinate is value, so the value of v doesn't change,
	fmt.Println(v.X)	 //output  3
	fmt.Println(v.Y)	//output 4
}
```

Now we change the receiver in swapCoordinate as a pointer receiver 

```go
func (v *Vertex) swapCoordinate() {
	t := v.X
	v.X = v.Y
	v.Y = t
}

func main() {
	v := Vertex{3, 4}
	

	v.swapCoordinate()
	//because receiver of  swapCoordinate is pointer, so the value of v is changed,
	fmt.Println(v.X)	 //output  4
	fmt.Println(v.Y)	//output 3
}
```
## When we  should use pointer receiver or value receiver ?
When the decision is made that a struct type value should not be mutated when something needs to be added or removed from the value, then we should use value receiver.
When we want the value of the object should be shared with the rest of the program that needs to manipulate on , we should use pointer receiver.
The decision to use a value or pointer receiver should not be based on whether the method is mutating the receiving value. The decision should be based on the nature of the type. 

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

## Type embedding
Go allows you to take existing types and both extend and change their behavior. This capability is important for code reuse and for changing the behavior of an existing type to suit a new need. This is accomplished through type embedding. It works by taking an existing type and declaring that type within the declaration of a new struct type. The type that is embedded is then called an inner type of the new outer type.

Through inner type promotion, identifiers from the inner type are promoted up to the outer type. These promoted identifiers become part of the outer type as if they were declared explicitly by the type itself. The outer type is then composed of everything the inner type contains, and new fields and methods from inner type can be added. The outer type can also declare the same identifiers as the inner type and override any fields or methods it needs to. This is how an existing type can be both extended and changed.

```go
// Sample program to show how to embed a type into another type and
// the relationship between the inner and outer type.
package main

 import (
     "fmt"
 )

 // user defines a user in the program.
 type user struct {
     name  string
     email string
 }

 // notify implements a method that can be called via
 // a value of type user.
 func (u *user) notify() {
     fmt.Printf("Sending user email to %s<%s>\n",
     u.name,
     u.email)
 }

 // admin represents an admin user with privileges.
 type admin struct {
     user  // Embedded Type
     level string
 }

 // main is the entry point for the application.
 func main() {
     // Create an admin user.
     ad := admin{
         user: user{
             name:  "john smith",
             email: "john@yahoo.com",
         },
         level: "super",
     }

     // We can access the inner type's method directly.
     ad.user.notify()

     //IMPORTANT The inner type's method is promoted.
	 ad.notify()
	// // Send the admin user a notification.
	// The embedded inner type's implementation of the
	// interface is "promoted" to the outer type.
	sendNotification(&ad)

 }
func sendNotification(n notifier) {
	n.notify()
}

```

Example of overriding method of inner type for outer type
```go
// Sample program to show what happens when the outer and inner
// types implement the same interface.
 package main

 import (
     "fmt"
 )

 // notifier is an interface that defined notification
 // type behavior.
 type notifier interface {
     notify()
 }

 // user defines a user in the program.
 type user struct {
     name  string
     email string
 }

 // notify implements a method that can be called via
 // a value of type user.
 func (u *user) notify() {
     fmt.Printf("Sending user email to %s<%s>\n",
         u.name,
         u.email)
 }

 // admin represents an admin user with privileges.
 type admin struct {
     user
     level string
 }

 // notify implements a method that can be called via
 // a value of type admin.
 func (a *admin) notify() {
     fmt.Printf("Sending admin email to %s<%s>\n",
         a.name,
         a.email)
 }

 // main is the entry point for the application.
 func main() {
     // Create an admin user.

     ad := admin{
        name:  "john smith",
        email: "john@yahoo.com", 
		level: "super",
     }	//throw error: cannot use promoted field user.name in struct literal of type admin , cannot use promoted field user.email in struct literal of type admin

     // Send the admin user a notification.
     // The embedded inner type's implementation of the
     // interface is NOT "promoted" to the outer type.
     sendNotification(&ad)

     // We can access the inner type's method directly.
     ad.user.notify()

     // The inner type's method is NOT promoted.
     ad.notify()
 }

 // sendNotification accepts values that implement the notifier
 // interface and sends notifications.
 func sendNotification(n notifier) {
     n.notify()
 }
``` 

{: .box-note}
**Note:**Promotion of inner type fields makes them visible but does not allow them to be used in type literal initializations

See below example


# Interfaces
## Simple conceptual view of interface after assignements

An interface type is defined as a set of method signatures.
A value of interface type can hold any value that implements those methods.

Interface values are two-word data structures. The first word contains a pointer to an internal table called an iTable, which contains type information about the stored value. The iTable contains the type of value that has been stored and a list of methods associated with the value. The second word is a pointer to the stored value. The combination of type information and pointer binds the relationship between the two values.
![A simple view of an interface value after concrete type value assignment](interface_assignment_value.png)

When a pointer is assigned to an interface value. In this case, the type information will reflect that a pointer of the assigned type has been stored, and the address being assigned is stored in the second word of the interface value.
![A simple view of an interface value after concrete type pointer assignment](interface_pointer_assignment.png)


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
	a = v	//output the error : cannot use v (type Vertex) as type Abser in assignment: Vertex does not implement Abser (Abs method has pointer receiver)
	fmt.Println(a.Abs()) //doesn't work
	a = &f //assign address of f to interface
	a.Abs() // it works even though value of a is adderss of f 

}
```



{: .box-note}
**Note:** When using the pointer receiver in any method declarations in a interface,then we  can use a value or pointer of the receiver type to make the method call, the compiler will reference or dereference the value if necessary to support the call.
Unlike when you call methods directly from values and pointers, when you call a method via an interface type value, the rules are different. Methods declared with pointer receivers can only be called by interface type values that contain pointers. Methods declared with value receivers can be called by interface type values that contain both values and pointers.

```go
// Method declared with a pointer receiver of type defaultMatcher
func (m *defaultMatcher) Search(feed *Feed, searchTerm string)

// Call the method via an interface type value
var dm defaultMatcher
var matcher Matcher = dm     // Assign value to interface type
matcher.Search(feed, "test") // Call interface method with value

> go build
//ERROR : cannot use dm (type defaultMatcher) as type Matcher in assignment

// Method declared with a value receiver of type defaultMatcher
func (m defaultMatcher) Search(feed *Feed, searchTerm string)

// Call the method via an interface type value
var dm defaultMatcher
var matcher Matcher = &dm    // Assign pointer to interface type
matcher.Search(feed, "test") // Call interface method with pointer
```


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
## Interface References links: 
[Application of Empty Interfaces](https://blog.golang.org/json-and-go)