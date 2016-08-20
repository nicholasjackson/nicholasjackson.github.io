---
layout: post
title:  "You call that optimisation? This is optimisation!"
tags: [microservices, go]
author: nic
image: /images/posts/not_optimisation/dundee.jpg
---
Whilst writing a chapter for my book "Building Microservices in Go" I was working on a section for JSON parsing in Go.  As I was explaining the various semantics of the *encoding/json* package I wrote a really simple example of using *json.Marshal* to convert a struct into it most simple string representation.  In this example (see listing 1) I decoded the struct into a byte array and then wrote it to the response using *fmt.FPrint*.  Not the code I would write in a handler but to explain the simple useage of the package before moving on to show how you can use the prefered method which are Encoders (see listing 2).


### listing 1

```go
func BenchmarkHelloHandlerVariable(b *testing.B) {                                                                                
        b.ResetTimer()                                                                                                            
                                                                                                                                  
        var writer = ioutil.Discard                                                                                               
        response := Response{Message: "Hello World"}                                                                              
                                                                                                                                  
        for i := 0; i < b.N; i++ {                                                                                                
                data, _ := json.Marshal(response)                                                                                 
                fmt.Fprint(writer, string(data))                                                                                  
        }                                                                                                                         
}                                                                                                                                 
                                                                                                                                  
```

### listing 2
```go
func BenchmarkHelloHandlerEncoder(b *testing.B) {                                                                                 
        b.ResetTimer()                                                                                                            
                                                                                                                                  
        var writer = ioutil.Discard                                                                                               
        response := Response{Message: "Hello World"}                                                                              
                                                                                                                                  
        for i := 0; i < b.N; i++ {                                                                                                
                encoder := json.NewEncoder(writer)                                                                                
                encoder.Encode(response)                                                                                         
        }                                                                                                                         
}          
```

After writing this code I decided that I would have a quick look and see the inefficiencies between the first method and the second.  It turns out there is a 40% speed improvement in the second example.  This is mainly due to the fact that the using the encoder.Encode rather than copying the output from Marshal to a byte array dramatically reduces the number of memory allocations that the method has to perform from 5 to 2.

If we optimise this further by passing a reference to the encoder rather than passing by value (see listing 3) we see further gains by 100 nanoseconds a 50% reduction over the first method, again this is because we are reducing the number of memory allocations performed.

### listing 3

```go
func BenchmarkHelloHandlerEncoderReference(b *testing.B) {                                                                                 
        b.ResetTimer()                                                                                                            
                                                                                                                                  
        var writer = ioutil.Discard                                                                                               
        response := Response{Message: "Hello World"}                                                                              
                                                                                                                                  
        for i := 0; i < b.N; i++ {                                                                                                
                encoder := json.NewEncoder(writer)                                                                                
                encoder.Encode(&response)                                                                                         
        }                                                                                                                         
}          
```
 
### benchmarks 

```bash
go test -v -run="none" -bench=. -benchtime="5s"  -benchmem
testing: warning: no tests to run
PASS
BenchmarkHelloHandlerVariable           10000000              1187 ns/op             248 B/op          5 allocs/op
BenchmarkHelloHandlerEncoder            10000000               731 ns/op              24 B/op          2 allocs/op
BenchmarkHelloHandlerEncoderReference   10000000               660 ns/op               8 B/op          1 allocs/op
ok      github.com/nicholasjackson/building-microservices-in-go/chapter1/bench  28.394s
```

Now these are tiny numbers but if this was the only operation that the server was performing then we could get away with half the hardware to satisfy the same load, that means saving money.

I hope this has shown you that allocating memory does not come for free and that with a little thought and knowledge you can dramatically increase the speed of your application.  I would never advocate premature optimisation and there is none of this here, this is simply understanding that memory allocation takes time and if we can avoid it then we should.  It is also about using the right options that you already have available to you and to do that you need to understand your core language and framework.