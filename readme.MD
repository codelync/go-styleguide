# Coding Guide

This document describes various guidelines and best practices for Golang projects under f5ingress 

These guidelines have been developed to achieve following outcomes:

* Enable high velocity of feature development by adopting to consitent code layout and format across different services and
  libraries
* Investing in tools and processes to catch problems early in dev-test-deploy cycle
* Making code reviews efficient and faster 



## Golang guidelines

We want to follow recommendations from
[How to Write Go Code](https://golang.org/doc/code.html) and
[Effective Go]( https://golang.org/doc/effective_go.html). In addition,

* Document the public usage of a library or executable using `doc.go` for e.g.
  see [this](https://github.com/coreos/etcd/blob/master/embed/doc.go)
* Specify [interfaces](http://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go)
  exported by a library using `interface.go`
  ([example](https://github.com/appcelerator/amp/tree/2dd6ddfef77ba0efe0563f34e0805d1111ed104a/data/storage))  
  * The implementation for the interfaces SHOULD be in file(s) in a
    sub-directory (and therefore separate package,
    ([example](https://github.com/appcelerator/amp/blob/2dd6ddfef77ba0efe0563f34e0805d1111ed104a/data/storage/etcd/store.go))
  * Prefer to expose library abstractions via interface type instead of
    struct type. This allows for enhanced versions of structs over time
    without needing to modify consumers that don’t need it.
  * When possible implement and/or extend golang stdlib [interfaces](http://sweetohm.net/article/go-interfaces.en.html)
    (e.g [net.Conn](https://github.com/golang/go/blob/go1.9.2/src/net/net.go#L112))
    instead of new interface type. 
* Use NewXXX for XXX struct type, hiding complex constructions of struct w.r.t.
  set of fields to initialize exposed by simpler parameters in NewXXX, aka
  factory pattern, for e.g. see
  [this](http://matthewbrown.io/2016/01/23/factory-pattern-in-golang/)

