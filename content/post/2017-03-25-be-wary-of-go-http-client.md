---
date: 2017-03-25T09:30:00.000Z
lastmod: 2017-07-30T09:32:10.000Z
title: Be wary of  http/client.go
draft: false
slug: be-wary-of-go-http-client
tags: ["HTTP","Go"]

description: 
---

Recently, I found out an interesting problem in Go. The problem can be reduced to a simple client request to a HTTP server.

Suppose we have a HTTP server, which serves only one rooted path `/foo/`.

    package main
    
    import (
    	"io"
    	"log"
    	"net/http"
    	"net/http/httputil"
    )
    
    func handleFoo(w http.ResponseWriter, req *http.Request) {
    	// request details
    	dump, _ := httputil.DumpRequest(req, true)
    	log.Println(string(dump))
    
    	if auth := req.Header.Get("Authorization"); auth != "Bearer GoodToken" {
    		http.Error(w, "401 Unauthorized", http.StatusUnauthorized)
    		return
    	}
    
    	io.WriteString(w, "Hello World!")
    }
    
    func main() {
    	http.HandleFunc("/foo/", handleFoo)
    	log.Fatal(http.ListenAndServe(":12345", nil))
    }
    

`handleFoo` simplily verifies that correct token is sent in the `Authorization` header. Otherwise, it returns `401 Unauthorized`.

The client sends request with correct token to the server.

    package main
    
    import (
    	"io"
    	"log"
    	"net/http"
    	"os"
    )
    
    func main() {
    	req, _ := http.NewRequest(http.MethodGet, "http://localhost:12345/foo", nil)
    
    	req.Header.Set("Authorization", "Bearer GoodToken")
    
    	resp, err := http.DefaultClient.Do(req)
    	if err != nil {
    		log.Fatal(err)
    	}
    
    	defer resp.Body.Close()
    	if _, err := io.Copy(os.Stdout, resp.Body); err != nil {
    		log.Fatal(err)
    	}
    }
    

What do you think the response is? `401 Unauthorized` or `Hello World!`?

The answer is *it depends*. It depends on the version of Go that the **client** code is running. If the client code is running on Go **<1.8.0**, the response is `401 Unauthorized`. Otherwise, it's `Hello World!`.

But why?

It's not trivial to see at first glance. Let me list down what happens step by step.

1. 
Client sends

         GET /foo HTTP/1.1
         Host: localhost:12345
         Authorization: Bearer GoodToken
         ...
    

2. 
Server receives the request. `ServeMux` determines that the requested path `/foo` match the registered rooted path `/foo/`. `ServeMux` decides to send redirect ([doc](https://github.com/golang/go/commit/aaa0bc1043883390e052ec6f6775cbf0395dceb1)).

3. 
Server responds with header

         HTTP/1.1 301 Moved Permanently
         Location: /foo/
         ...
    

4. 
Client receives response and follows redirect by sending another request to server.

5. 
Server receives the 2nd request and let `handleFoo` handle it.

Both `1.8.0` and versions <`1.8.0` follow the same steps when processing the reqeust. The difference lies in Step #4.

Prior to `1.8.0`, following redirect in Go will **NOT** copy the original headers even if it's for the same domain. This wasn't changed util [this commit](https://github.com/golang/go/commit/6e87082d41f0267b39e6a1854d655b1d1c2f7541). What happened in Go <`1.8.0` is that the `Authorization` header wasn't copied unpon redirect. Therefore, `handleFoo` returns `401 Unauthorized`.

Initially, I was quite surprised by the behaviour. After reading through [this issue](https://github.com/golang/go/issues/4800) and [HTTP spec](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3), I realized that `http/client.go` didn't actually do anything wrong because the HTTP spec didn't specify  following redirect should copy original headers. It's just that the well known HTTP clients, e.g. [curl](https://linux.die.net/man/1/curl), [httpie](https://github.com/jakubroztocil/httpie), [rest-client](https://github.com/rest-client/rest-client), etc., have established the convention.

Be wary of `http/client.go`.
