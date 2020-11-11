+++
title = "A Tour of Go - Exercise: Fibonacci closure"
description = "Go 指南练习题"
date = 2020-11-12
categories = ["Go"]
tags = ["Go", "A Tour of Go", "Exercise"]
draft = true
+++

### [Exercise: Fibonacci closure](https://tour.golang.org/moretypes/26)

Let's have some fun with functions.

Implement a `fibonacci` function that returns a function (a closure) that returns successive [fibonacci numbers](https://en.wikipedia.org/wiki/Fibonacci_number) (0, 1, 1, 2, 3, 5, ...).

<!--more-->

### 1. 解题

这道题的关键信息有两点：
- 闭包函数会引用函数体以外的值，可以对其修改。
- 

```go

```

```python
def fibonacci():
    a, b = 1, 0
    def func():
        nonlocal a, b
        a, b = b, a+b
        return a
    return func

if __name__ == "__main__":
    f = fibonacci()
    for i in range(10):
        print(f())
```