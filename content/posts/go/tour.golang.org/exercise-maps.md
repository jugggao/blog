+++
title = "A Tour of Go - Exercise: Maps"
description = "Go 指南练习题"
date = 2020-11-11
categories = ["Go"]
tags = ["Go", "A Tour of Go", "Exercise"]
draft = false
+++

### [Exercise: Maps](https://tour.golang.org/moretypes/23)

Implement `WordCount`. It should return a map of the counts of each “word” in the string `s`. The `wc.Test` function runs a test suite against the provided function and prints success or failure.

You might find [strings.Fields](https://golang.org/pkg/strings/#Fields) helpful. 

<!--more-->

---

### 1. 解题

单词统计是一个很经典的小程序了，使用 Python 解法如下：

```python
from collections import Counter
import re

def word_count(s):
    words = [i.lower() for i in re.findall("\w+", s)]
    return Counter(words)
```

使用 Go 解法如下：

```go
package main

import (
	"strings"
	"golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
	strSlice := strings.Fields(s)
	wordCount := make(map[string]int)
	for _, i := range strSlice {
		wordCount[i]++
	}
	return wordCount
}

func main() {
	wc.Test(WordCount)
}
```

### 2. 思考

**Q：Python 代码量比 Go 要少，得益于 Python 标准库的丰富。那么如果不使用 `Counter` 类，Python 代码量如何？**

```python
import re

def word_count(s):
    words = [i.lower() for i in re.findall("\w+", s)]
    count = {}
    for i in words:
        if i in count.keys():
            count[i] += 1
        else:
            count[i] = 1
    return count
```

Python 比 Go 要多一个步骤，首先判断 key 是否存在，因为 python 是动态类型编程语言，如果读取一个不存在的 key，会报错。
但是 Go 是静态类型语言，实现已经定义了 Map 里的元素类型是 Int，读取不存在的 key 为其零值 0，可以直接计算。
这也是静态类型语言的优点。

**Q：Python 在自增时使用 += 运算符，为什么不能像 C 或 Go 语言用 ++ 运算符来进行自增呢？**

`+=` 代表改变了变量，而 `++` 代表改变对象本身。
Python 的模型规定，数值对象是不可变的，所以 `++` 就没有意义了，而在使用 `+=` 运算符的过程实际是：将 a + 1 存储到另一个内存区域中，然后再将 a 赋值至新的内存区域当中。

```python
>>> a = 1
>>> id(a)
2306726521136
>>> a += 1
>>> id(a)
2306726521168
```

Python 中的变量赋值可以用「指针」的概念去理解，更像是贴标签的操作，多个变量名可以指向同一内存区域表示的值。

```python
>>> a = 1
>>> b = 1
>>> c = 1
>>> id(a)
2306726521136
>>> id(b)
2306726521136
>>> id(c)
2306726521136
```

所以，我们也不想看到改变 a 自身的值把 b 也改变的情况出现吧。  
而 Go 中的变量已变量名为基准，就很容易理解了，就是将 a 原本的内存区域表示的值替换为原本的值 +1。

这次我更倾向于 Python 的语法，可读性高，并且少一些选择，少一些烦恼。

**Q: Go 语言中的 strings.Fields 方法和 Python 中的正则和列表推导相比孰优孰劣？**

看起来 Go 更好一些，我一向是不喜欢使用正则的，效率低。  
也不知道 Go 底层是怎么处理的，等以后有读源码的能力在解答吧。

### 3. 其他

另外，这道题提供了一个[测试函数](https://github.com/golang/tour/blob/master/wc/wc.go) `wc.Test`，可以看一看，可以初步了解 Go 语言程序测试的写法。