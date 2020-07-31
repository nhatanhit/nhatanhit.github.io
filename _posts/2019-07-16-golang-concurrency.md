---
layout: post
title: Golang concurrency
subtitle: Golang concurrency
comments: true
image: /img/gopher.gif
tags: [golang]
---
# Table of Contents
1. [Goroutines](#goroutines)
2. [Channels](#channels)
3. [Range And Close](#range-and-close)
4. [Select](#select)
5. [Mutex](#mutex)

# Goroutines
A goroutine is a lightweight thread managed by the Go runtime.
The evaluation of f, x, y, and z happens in the current goroutine and the execution of f happens in the new goroutine.
Goroutines run in the same address space, so access to shared memory must be synchronized. 
```go
import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}
/*
the output is 
world
hello
hello
world
world
hello
hello
world
world
hello
*/

//but if you disable the time sleep in function say, output is
/*hello
hello
hello
hello
hello*/
```

# Channels
Channels are a typed conduit through which you can send and receive values with the channel operator, <-.
```go
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and assign value to v
```
Like maps and slices, channels must be created before use:
```go
ch := make(chan int)
```
By default, sends and receives block until the other side is ready. This allows goroutines to synchronize without explicit locks or condition variables.

```go
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
	
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)  //output -5, 17, 12. The thread of sum(s[len(s)/2:], c) will run first. and send value to channel
}
```

## Buffered Channel
Channel can be buffer. Providing the buffer length as the second argument to make to initialize a buffered channel

```go
ch := make(chan int, 100)
```

{: .box-note}
**Note:**  Sends to a buffered channel block only when the buffer is full. Receives block when the buffer is empty.

```go
func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}

```
If number of elements is added to buffer is greater than buffer length, error is raised. 

```go
func main() {
	ch := make(chan int, 2)
	ch <- 1
    ch <- 2
    ch <- 3 //add 3 to channel
	fmt.Println(<-ch)
	fmt.Println(<-ch)
    /*
    output
    fatal error: all goroutines are asleep - deadlock!
    */
}
```
For fixing above code, wait for the elements in channel release, then we can more element to channel 

```go
func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	fmt.Println(<-ch) // release 1, so buffer length is 1
	ch <- 3 //add 3 to channel
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

## Empty Buffered Channel

In below example, as ch is unbuffered channel, when executing sending data to channel, there is no receiver ready for receiving data from channel ch. So deadlock happens

```go
package main
func main() {
	ch := make(chan int)
	ch <- 1	 	//output all goroutines are asleep - deadlock!
}
```


{: .box-note}
**Note:**  Communication on an unbuffered channel does not proceed until both a sending and receiving goroutine are ready for channel operations


Fix by separating the sender into another routine

```go
package main
import "fmt"

func main() {
	ch := make(chan int)
	go func() {
		ch <- 1
	}()
	
	fmt.Printf("%v\n",<-ch)
	
}
```


# Range And Close
A sender can close a channel to indicate that no more values will be sent. Receivers can test whether a channel has been closed by assigning a second parameter to the receive expression: after
```go
v, ok := <-ch
```
ok is false if there are no more values to receive and the channel is closed.
The loop for i := range c receives values from the channel repeatedly until it is closed.

{: .box-note}
**Note:**  Only the sender should close a channel, never the receiver. Sending on a closed channel will cause a panic.

{: .box-note}
**Note:**  Channels aren't like files; you don't usually need to close them. Closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a range loop.

```go
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
        fmt.Printf("i am sending value %v\n", x)
		c <- x
		x, y = y, x+y
    }
    fmt.Println("finish to send\n")
	close(c)    
}

func main() {
	c := make(chan int, 10)
    go fibonacci(cap(c), c)
    //wait for the signal close from channel c, then print the elements in the channel
	for i := range c {
        fmt.Printf("length of channel: %v\n", len(c)) 
		fmt.Println(i)
	}
}
/*output
i am sending value 0
i am sending value 1
i am sending value 1
i am sending value 2
i am sending value 3
i am sending value 5
i am sending value 8
i am sending value 13
i am sending value 21
i am sending value 34
finish to send

length of channel: 9
0
length of channel: 8
1
length of channel: 7
1
length of channel: 6
2
length of channel: 5
3
length of channel: 4
5
length of channel: 3
8
length of channel: 2
13
length of channel: 1
21
length of channel: 0
34

*/
```
# Select
The select statement lets a goroutine wait on multiple communication operations.
A select blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.



```go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		//by using select, we let fibonaci wait for incomming element on channels c 		//and quit
		select {
		case c <- x:	//sending x to channel c
			fmt.Println("x comming")
			x, y = y, x+y
		case <-quit: //receive value from channel quit
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	//create two channels
	c := make(chan int)
	quit := make(chan int)
	
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```
## default selection
The default case in a select is run if no other case is ready.
Use a default case to try a send or receive without blocking:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	tick := time.Tick(100 * time.Millisecond)
	boom := time.After(500 * time.Millisecond)
	
	for {
		select {
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}
```

## Points need to clarify 
1. [When x send to c](https://stackoverflow.com/questions/34931059/go-tutorial-select-statement)
2. [Pipeline](https://blog.golang.org/pipelines)

# Mutex
Mutex help to ensure that one go routine can access a shared variable at a time to avoid conflicts
Go's standard library provides mutual exclusion with sync.Mutex and its two methods: Lock & Unlock
We can define a block of code to be executed in mutual exclusion by surrounding it with a call to Lock and Unlock

We can also use defer to ensure the mutex will be unlocked

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mux.Unlock()	//after done updating, release the lock
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()	// Lock so only one goroutine at a time can access the map c.v.
	
	defer c.mux.Unlock()	//unlock the mutex when finish readind value from shared variable
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```


