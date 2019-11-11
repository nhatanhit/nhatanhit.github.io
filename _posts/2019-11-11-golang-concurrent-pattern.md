---
layout: post
title: Golang Concurrent Pattern
subtitle: Golang Concurrent Pattern
comments: true
---
# Reference
[Concurrency Patterns: Golang](https://medium.com/@thejasbabu/concurrency-patterns-golang-5c5e1bcd0833)

# Brief descriptions

## Generator
The golang pattern *generator* is used for resolving the concurrent  problem of accessing a sharing buffer between producer and  consumer

```go
func fib(n int) <-chan int {
    c := make(chan int) //accesing buffer, which is shared by producer consumer
    go func() {
		for i, j:= 0, 1; i < n ; i, j = i+j,i {
			fmt.Printf("sending %v\n",i)
			c <- i  //producer send value to buffer c
		}
    	close(c)
    }()
    //this statement does not actually return channel c, it creates a channel c , and let another go routine to access the buffer channel
    
    return c    
}
func main() {
    // fib returns the fibonacci numbers lesser than 1000
    for i := range fib(1000) {
    // Consumer which consumes the data produced by the generator, which further does some extra computations
        //Do other computations on value i at here
        //fmt.Println()
        fmt.Printf("Receiving %v\n",i)
    }
}

//output
/*
sending 0
sending 1
Receiving 0
Receiving 1
sending 1
sending 2
Receiving 1
Receiving 2
sending 3
sending 5
Receiving 3
Receiving 5
sending 8
sending 13
Receiving 8
Receiving 13
sending 21
sending 34
Receiving 21
Receiving 34
sending 55
sending 89
Receiving 55
Receiving 89
sending 144
sending 233
Receiving 144
Receiving 233
sending 377
sending 610
Receiving 377
sending 987
Receiving 610
Receiving 987
*/
```

## Futures
Future pattern is something simlar with callback model in Nodejs. You create one task, but don't want it blocks others tasks you have to do. You create a callback for this task and ask it to execute the callback once it's done. 


```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

type data struct {
	Body  []byte
	Error error
}

func futureData(url string) <-chan data {
	c := make(chan data, 1)
    //separate another go routine and process on it
	go func() {
		var body []byte
		var err error

		resp, err := http.Get(url)
		if (resp != nil) {
			defer resp.Body.Close()
			body, err = ioutil.ReadAll(resp.Body)
			//send data to channel
			c <- data{Body: body, Error: err}
		}
		

		
	}()

	return c
}

func main() {
    //calling a function
	future := futureData("https://mazii.net/")

	// do many other things
    fmt.Println("hello world")
    
    //this is section of callback
	body := <-future
	fmt.Printf("response: %#v", string(body.Body))
	fmt.Printf("error: %#v", body.Error)
}
```

## Fan-in Fan-out
Fan-in Fan-out is something like a model multiplex / de-multiplex
It's used in parallel computing, where 2 or more go-routines get data from a collection.  THey getting values from a collection (de-multiplex) without requiring the lock of accessing resource. After receiving , they   manipulates on the data they received to get a single identity, then they send their single identity for calculating the final output from other identiy from other channels (multiplex).

```go
func main() {
	randomNumbers := []int{13, 44, 56, 99, 9, 45, 67, 90, 78, 23}
    // generate the common channel with inputs
    //create a channel that holds  collection data
	inputChan := generatePipeline(randomNumbers)

	// Fan-out to 2 Go-routine (de-multiplex)
	c1 :=squareNumber(inputChan)
	c2 :=squareNumber(inputChan)

	// Fan-in the resulting squared numbers (multiplex)
	c := fanIn(c1, c2)
	sum := 0

	// Do the summation
	for i := 0; i < len(randomNumbers); i++ {
		sum += <-c
	}
	fmt.Printf("Total Sum of Squares: %d", sum)
}

func generatePipeline(numbers []int) chan int {
	out := make(chan int)
	go func() {
		for _, n := range numbers {
			out <- n
		}
		close(out)
	}()
	return out
}

func squareNumber(in chan int) chan int {
	out := make(chan int)
	go func() {
		for n := range in {
			fmt.Printf("%d is comming\n",n)
			out <- n
		}
		close(out)
	}()
	return out
}


func fanIn(input1, input2 chan int) chan int {
	c := make(chan int)
	go func() {
		for {
			select {
			case s := <-input1:  c <- s
			case s := <-input2:  c <- s
			}
		}
	}()
	return c
}
```

