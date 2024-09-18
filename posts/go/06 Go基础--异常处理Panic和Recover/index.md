---
title: Go基础--异常处理Panic和Recover
date: 2021-03-03 10:24:00
slug: go-panic-and-recover
tags:
  - Go
categories:
  - Go
---

## 概述

在Go语言中，`panic`和`recover`是用于处理异常情况的机制，类似于其他编程语言中的异常和异常处理机制。

## Panic

- panic用于表示程序发生了严重错误，类似于其他语言中的抛出异常。当程序遇到无法继续执行的致命错误时，可以调用panic函数来引发panic。
- 当程序发生panic时，当前goroutine的执行会被中止，但是deferred函数会继续执行，直到goroutine返回。然后程序会崩溃并打印panic信息，包括panic值和调用栈信息。

## Recover

- recover用于在deferred函数中捕获和处理panic，防止panic继续向上传播导致程序崩溃。
- 当在deferred函数中调用recover时，如果当前goroutine发生了panic，recover会返回传递给panic函数的值，并且停止panic继续传播。这样可以恢复程序的控制流，使程序能够继续执行而不崩溃。
- 需要注意的是recover只有在defer才能生效，并且只能处理自己goroutine中的panic。在其他作用域中调用不会发挥作用。

## 总结

一般来说，panic和recover应该被谨慎使用，因为过度使用它们会导致代码难以理解和维护。通常情况下，应该尽量避免使用panic，而是采取适当的错误处理策略。recover通常用于在必要时恢复程序的稳定性，确保程序在遇到意外情况时能够优雅地处理并继续执行。

```go
package main

import "fmt"

func recoverTest() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered from panic:", r)
		}
	}()

	// 引发panic
	panic("Oops! Panic occurs!")
}

func main() {
	fmt.Println("Start")
	recoverTest()
	fmt.Println("End")
}

```