+++
title = "A Tour of Go - Exercise: Fibonacci closure"
description = "Go 指南练习题"
date = 2020-11-12
categories = ["Go"]
tags = ["Go", "A Tour of Go", "Exercise"]
draft = false
+++

### [Exercise: Fibonacci closure](https://tour.golang.org/moretypes/26)

Let's have some fun with functions.

Implement a `fibonacci` function that returns a function (a closure) that returns successive [fibonacci numbers](https://en.wikipedia.org/wiki/Fibonacci_number) (0, 1, 1, 2, 3, 5, ...).

<!--more-->

---

### 1. 解题

在解这道题之前先简单回忆一下什么是闭包，在学习《流畅的 Python》 一书时基本理解了闭包的概念，我先用 Python 的类和闭包来解下这道题。

Python 类实现：

```python
class Fibonacci():

    def __init__(self):
        self.a = 1
        self.b = 0

    def __call__(self):
        self.a, self.b = self.b, self.a + self.b
        return self.a


if __name__ == "__main__":
    f = Fibonacci()
    for i in range(10):
        print(f())
```

Python 闭包实现：

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

这两个实现有共通之处：调用 `Fibonacci()` 或 `fibonacci()` 得到一个可调用对象 f，它会更新历史值，然后计算当前值。 
 
`Fibonacci()` 类的实例 f 在哪里存储历史值很明显：self.a 和 self.b 实例属性。但是闭包实现的 f 函数在哪里寻找 a 和 b 的历史值呢？

在 `func` 函数中，a 和 b 为自由变量（free variable），指未在本地作用域中绑定的变量，审查返回的 func 对象，可以发现 Python 在 `f.__code__.co_freevars` 属性中保存自由变量的名称；在 `f.__closure__` 中的各个元素对应 `f.__code__.co_freevars` 中的一个名称，这些元素是 cell 对象，有个 cell_contents 属性，保存其值。

```python
print(f.__code__.co_freevars) # ('a', 'b')
print(f.__closure__[0].cell_contents, f.__closure__[1].cell_contents) # 34 55
```

所以，闭包是一种函数，它会保留定义函数时存在的自由变量的绑定，这样调用函数时，虽然定义作用域不可用了，但是仍能使用那些绑定。

Go 语言也一样，在[上一节](https://tour.golang.org/moretypes/25)的学习中，介绍了闭包是一个函数值，它引用了其函数体之外的变量。该函数可以访问并赋予其引用变量的值，换句话说该函数被这些变量「绑定」在一起。

Go 语言实现：

```go
package main

import "fmt"

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
	a, b := 1, 0
	return func() int {
		a, b = b, a+b
		return a
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```

### 2. 思考

**Q：为什么 Python 闭包中的 func 中比 Go 闭包中多了一个 `nolocal` 变量声明？**

在上次的 [exercise-maps](../exercise-maps/) 中已经有了解答：

Python 中的数字、字符串、元组等类型是不可变的，只能读取，不能更新。如果尝试绑定，比如 a = a + 1，其实会隐式创建局部变量 a。这样 a 就不是自由变量了，因此不会保存在比包中。

所以 Python3 引入了 nonlocal 声明，它的作用是把变量标记为自由变量，即使在函数中为变量赋予新值了，也会变成自由变量。如果为 nonlocal 声明的变量赋予新值，闭包中保存的绑定会更新。

而 Go 中的数字类型是可变的，就不存在这个问题。

### 3. 参考

A Tour of Go 的 Github 仓库中的[答案](https://github.com/golang/tour/blob/master/solutions/fib.go)。

这次也翻了之前看的 《流程的 Python》，现在看也感觉还是很有收获。

