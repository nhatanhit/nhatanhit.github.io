---
layout: post
title: Golang structures, slice, pointer, map, closure
subtitle: Golang structures, slice, pointer, map, closure
comments: true
---
## Structure
Declare Struct
```go
type Vertex struct {
    X, Y int
}
//initialize struct
func main() {
    v := Vertex{1,2}
    fmt.Println(v)
    fmt.Println(Vertex{5,6})
    //access variable member
    v.X = 5
    fmt.Println(v.X)
}
```
## Pointer
The type *T is a pointer point to a T value
```go
var p *int
//amperand , getting address of variable
i := 42
p = &i
fmt.Println(*p)
```
