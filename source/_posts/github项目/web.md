---
title: Zed-web框架
date: 2023-3-8 15:45:48
tags:
  - Zed系列
categories:
  - Zed系列
---



# 一、简介

在使用过beego、gin、kratos等常用框架后，他们的设计理念和提供的功能有很多共性，为了更好提高自己的一些理解，决定简单的写一个web框架。



# 二、编写

## 1. 路由

参考gin框架，我们先写一个engine，将路由和对应的执行方法利用map的方式绑定起来，并且把engine的生成方法收束起来由框架控制。

```go
package zed

import (
	"fmt"
	"net/http"
)

// HandlerFunc defines the request handler used by zed
type HandlerFunc func(http.ResponseWriter, *http.Request)

// Engine implement the interface of ServeHTTP
type Engine struct {
	router map[string]HandlerFunc
}

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	key := req.Method + "-" + req.URL.Path
	if handler, ok := engine.router[key]; ok {
		handler(w, req)
	} else {
		fmt.Fprintf(w, "404 not found:%s\n", req.URL)
	}
}

func (engine *Engine) Run(addr string) (err error) {
	return http.ListenAndServe(addr, engine)
}

func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) {
	key := method + "-" + pattern
	engine.router[key] = handler
}

// GET defines the method to add GET request.
func (engine *Engine) GET(pattern string, handler HandlerFunc) {
	engine.addRoute("GET", pattern, handler)
}

// POST defines the method to add POST request.
func (engine *Engine) POST(pattern string, handler HandlerFunc) {
	engine.addRoute("POST", pattern, handler)
}

// New is the constructor of zed.Engine
func New() *Engine {
	return &Engine{router: map[string]HandlerFunc{}}
}

```

这样一个简单的路由方法就写好了，这样有什么问题呢，首先，在高并发的场景下，可能有大量的请求不会马上返回，这时候我们就需要一个超时控制，其次，在原生的net/http中每个请求都需要对它的header进行设置，代码繁杂，初次之外，对于框架来说，可能还要支撑一些其他功能，例如中间件，那么中间件的消息怎么传递呢，这时候我们就需要context。

## 2. context 

新增了一个Context的结构体，把请求参数和返回参数都放到里面，另外再封装一些常用信息和方法，这样可以给用户一些简洁的写法。

```go
package zed

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type H map[string]interface{}

type Context struct {
	//origin objects
	Writer http.ResponseWriter
	Req    *http.Request

	Method string
	Path   string

	StatusCode int
}

func newContext(w http.ResponseWriter, req *http.Request) *Context {
	return &Context{
		Writer: w,
		Req:    req,
		Method: req.Method,
		Path:   req.URL.Path,
	}
}

func (c *Context) PostForm(key string) string {
	return c.Req.FormValue(key)
}

func (c *Context) Query(key string) string {
	return c.Req.URL.Query().Get(key)
}

func (c *Context) Status(code int) {
	c.StatusCode = code
	c.Writer.WriteHeader(code)
}

func (c *Context) SetHeader(key string, value string) {
	c.Writer.Header().Set(key, value)
}

func (c *Context) String(code int, format string, value ...interface{}) {
	c.SetHeader("Content-Type", "text/plain")
	c.Status(code)
	c.Writer.Write([]byte(fmt.Sprintf(format, value...)))
}

func (c *Context) JSON(code int, obj interface{}) {
	c.SetHeader("Content-Type", "application/json")
	c.Status(code)
	encoder := json.NewEncoder(c.Writer)
	err := encoder.Encode(obj)
	if err != nil {
		http.Error(c.Writer, err.Error(), 500)
	}
}

func (c *Context) Data(code int, data []byte) {
	c.Status(code)
	c.Writer.Write(data)
}

func (c *Context) HTML(code int, html string) {
	c.SetHeader("Content-Type", "text/html")
	c.Status(code)
	c.Writer.Write([]byte(html))
}
```

同时重新封装一下路由，为后面路由的扩展做准备

```go
package zed

import (
	"log"
	"net/http"
)

type router struct {
	handlers map[string]HandlerFunc
}

func newRouter() *router {
	return &router{handlers: make(map[string]HandlerFunc)}
}

func (r *router) addRoute(method string, pattern string, handler HandlerFunc) {
	log.Printf("Route %4s - %s", method, pattern)
	key := method + "-" + pattern
	r.handlers[key] = handler
}

func (r *router) handle(c *Context) {
	key := c.Method + "-" + c.Path
	if handler, ok := r.handlers[key]; ok {
		handler(c)
	} else {
		c.String(http.StatusNotFound, "404 NOT FOUND: %s\n", c.Path)
	}
}
```

