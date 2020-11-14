+++
title = "A Tour of Go - Exercise: Stringers"
description = "Go 指南练习题"
date = 2020-11-15
categories = ["Go"]
tags = ["Go", "A Tour of Go", "Exercise"]
draft = false
+++

### [Exercise: Stringers](https://tour.golang.org/methods/18)

Make the `IPAddr` type implement `fmt.Stringer` to print the address as a dotted quad.

For instance, `IPAddr{1, 2, 3, 4}` should print as `"1.2.3.4"`.

<!--more-->

---

### 1. 解题

思路：
1. 首先需要了解 [`Stringer`](https://tour.go-zh.org/methods/17) 这个接口：
    ```
    type Stringer interface {
        String() string
    }
    ```
    `Stringer` 是一个可以用字符串描述自己的类型。`fmt` 包（还有很多包）都通过此接口来打印值。

2. 我们要做的就是为 `IPAddr` 类型实现 `Stringer` 接口的 `String() string` 方法。

Go 语言实现：

```go
package main

import (
	"fmt"
	"strconv"
	"strings"
)

type IPAddr [4]byte

// TODO: Add a "String() string" method to IPAddr.
func (ip IPAddr) String() string {
	return fmt.Sprintf("%v", ip.Convert())
}

func (ip IPAddr) Convert() string {
	s := make([]string, len(ip))
	for i := range ip {
		s[i] = strconv.Itoa(int(ip[i]))
	}
	return strings.Join(s, ".")
}

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}
```

解析：
1. 首先给 `IPAddr` 实现一个 `Convert() string` 的方法，将数组 `[4]byte` 转换为 `string`。

2. 然后给 `IPAddr` 添加一个 `String() string` 的方法，其中调用了 `Convert() string` 方法返回转换后的输出。

3. 这样调用 `fmt.Printf` 时，会对 `IPAddr [4]byte` 类型调用 `Stringer` 接口下定义的 `String()` 方法来输出。

Python 实现（使用类来实现修改 print 调用的接口）：

```python
class IPAddr:
    def __init__(self, ip):
        self.ip = ip

    def __repr__(self):
        return "{}".format('.'.join(self.convert()))

    def convert(self):
        return [str(i) for i in self.ip]


if __name__ == "__main__":
    hosts = {
        "loopback": IPAddr([127, 0, 0, 1]),
        "googleDNS": IPAddr([8, 8, 8, 8]),
    }
    for name, ip in hosts.items():
        print(name, ip)
```

### 2. 思考

Q：Go 语言的接口和 Python 语言的接口有什么区别？

Python 没有 `interface` 关键字，而且除了抽象基类，每个类都有接口：类实现或继承的公开属性、方法或数据属性，包括特殊方法（如 `__getitem__`、`__add__` 等等）。  
在 Python 中，接口是实现特定角色的方法集合。一个类可能会实现多个接口，从而让实例扮演多个角色。  
Python 接口定义如下：

```python
from abc import ABCMeta, abstractmethod

# 定义抽象基类
# 抽象类的目的就是让别的类继承它并实现特定的抽象方法
# 抽象基类的一个主要用途是在代码中检查某些类是否为特定类型，实现了特定接口
class IStream(metaclass=ABCMeta):
    @abstractmethod
    def read(self, maxbytes=-1):
        pass

    @abstractmethod
    def write(self, data):
        pass


# 在子类中必须实现特定接口，如果没有实现接口或者类型错误时，在创建对象时会报错
class SocketStream(IStream):
    def read(self, maxbytes=-1):
        pass

    def write(self, data):
        pass
```

而 Go 语言中的接口和 Python 中的接口采用的方式完全不同。  
Go 不支持继承。我们可以定义接口，但是无需明确地指出某个类型实现了某个接口，编译器能自动判断。  
因此，考虑到接口在编译时检查，但是真正重要的是实现了什么类型，Go 语言可以说是具有「静态鸭子类型」。

与 Python 相比，对 Go 来说就好像每个抽象基类都实现了 `__subclasshook__` 方法，它会检查方法的名称和签名，而我们自己从不需要继承或注册抽象基类。

### 3. 参考

A Tour of Go 的 Github 仓库中的[答案](https://github.com/golang/tour/blob/master/solutions/stringers.go)，比我实现的要干净利落：

```go
package main

import "fmt"

type IPAddr [4]byte

func (ip IPAddr) String() string {
	return fmt.Sprintf("%d.%d.%d.%d", ip[0], ip[1], ip[2], ip[3])
}

func main() {
	addrs := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for n, a := range addrs {
		fmt.Printf("%v: %v\n", n, a)
	}
}
```