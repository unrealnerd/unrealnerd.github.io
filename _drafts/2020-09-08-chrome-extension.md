---
layout: post
title: quick notes go middleware
description: quick notes on go authentication middleware
author: bitsmonkey
tags: golang, middleware, programming 
---
![golang middleware](/img/golang-server-cover.jpg){:.coverimage}

Middleware to handle something for every request in go web applciation. In this example I will be creating middleware to handle authentication. Plugging in this authentication on the request pipe keep my code clean and makes sure its authenticates every incoming request.

Before doing authentication let me show a simpler middleware using [gorilla logging handler](https://pkg.go.dev/github.com/gorilla/handlers@v1.4.2?tab=doc#LoggingHandler).

All we have to do is get a handler func wrap the main handler with `logginghandler`. Create a [mux](https://github.com/gorilla/mux)
```go
//func main()
loggingHandler := newLoggingHandler(logFile)

mux.Handle("/query", loggingHandler(mainhandler))
``` 
You get the gist of it. So now create our own handler to handle authentication using jwt token.





#### Further Reading for details on the go packages used
****
- [golang http package - used to create the web server](https://golang.org/pkg/net/http/)
- [template - used for passing data from the server](https://golang.org/pkg/html/template/)
