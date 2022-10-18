
# Golang Coding Styles

## Table of contents
- [Introduction](#introduction)
- [Go Doc Comments] (#go-doc-Comments)
- [Add context to errors](#add-context-to-errors)
- [Consistent error and log messages](#consistent-error-and-log-messages)
- [Dependency management](#dependency-management)
	- [Use modules](#use-modules)
	- [Use Semantic Versioning](#use-semantic-versioning)
- [Structured logging](#structured-logging)
- [Avoid global variables](#avoid-global-variables)
- [Keep the happy path left](#keep-the-happy-path-left)
- [Testing](#testing)
	- [Use an assert library](#use-an-assert-libary)
	- [Use subtests to structure functional tests](#use-sub-tests-to-structure-functional-tests)
	- [Use table-driven tests](#use-table-driven-tests)
	- [Avoid mocks](#avoid-mocks)
	- [Avoid DeepEqual](#avoid-deepequal)
	- [Avoid testing unexported funcs](#avoid-testing-unexported-funcs)
	- [Add examples to your test files to demonstrate usage](#add-examples-to-your-test-files-to-demonstrate-usage)
- [Use linters](#use-linters)
- [Use goimports](#use-goimports)
- [Use meaningful variable names](#use-meaningful-variable-names)
- [Avoid side effects](#avoid-side-effects)
- [Favour pure functions](#favour-pure-functions)
- [Don't over-interface](#dont-over-interface)
- [Don't under-package](#dont-under-package)
- [Handle signals](#handle-signals)
- [Divide imports](#divide-imports)
- [Avoid unadorned return](#avoid-unadorned-return)
- [Use canonical import path](#use-canonical-import-path)
- [Avoid empty interface](#avoid-empty-interface)
- [Main first](#main-first)
- [Use internal packages](#use-internal-packages)
- [Avoid helper/util](#avoid-helperutil)
- [Embed binary data](#embed-binary-data)
- [Use io.WriteString](#use-iowritestring)
- [Use functional options](#use-functional-options)
- [Structs](#structs)
	- [Use named structs](#use-named-structs)
	- [Avoid new keyword](#avoid-new-keyword)
- [Consistent header naming](#consistent-header-naming)

## Introduction

Styles are the conventions that govern code. The term style is a bit of a
misnomer, since these conventions cover far more than just source file
formatting—gofmt handles that for us.

The goal of this guide is to manage this complexity by describing in detail various guidelines and best practices for Golang projects under f5ingress

These guidelines will help achieving following outcomes:

- Enable high velocity of feature development by adopting to consistent code layout and format across different services and libraries
- Investing in tools and processes to catch problems early in dev-test-deploy cycle
- Making code reviews efficient and faster

This serves as a supplement to

1. [How to Write Go Code](https://go.dev/doc/code)
2. [Effective Go](https://golang.org/doc/effective_go.html)
3. [Go Common Mistakes](https://github.com/golang/go/wiki/CommonMistakes)
4. [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)

All code should be error-free when run through `golint` and `go vet`. We
recommend setting up your editor to:

- Run `goimports` on save
- Run `golint` and `go vet` to check for errors
  
# Go Doc Comments

“Doc comments” are comments that appear immediately before top-level package, const, func, type, and var declarations with no intervening newlines. Every exported (capitalized) name should have a doc comment.
