+++
title = "A Tour of Go - Exercise: Slices"
description = "Go 指南练习题"
date = 2020-11-10
categories = ["Go"]
tags = ["Go", "A Tour of Go", "Exercise"]
draft = false
+++

### [Exercise: Slices](https://tour.golang.org/moretypes/18)

Implement Pic. It should return a slice of length `dy`, each element of which is a slice of `dx` 8-bit unsigned integers. When you run the program, it will display your picture, interpreting the integers as grayscale (well, bluescale) values.

The choice of image is up to you. Interesting functions include `(x+y)/2`, `x*y`, and `x^y`.

(You need to use a loop to allocate each `[]uint8` inside the `[][]uint8`.)

(Use `uint8(intValue)` to convert between types.)

<!--more-->

---

### 1. 解题

这道题官网提供了一个[画图函数](https://github.com/golang/tour/blob/4bb7d986198d/pic/pic.go) `pic.Show`，可以生成图片。

`pic.Show` [语法](https://pkg.go.dev/golang.org/x/tour/pic#Show)为：

```go
func Show(f func(dx, dy int) [][]uint8)
```

它接受一个函数作为参数，要求的正是编写这个函数。

图片处理一点不懂 :sweat:，所以只能按照官网给的提示来完成。 
 
思路：
1. 定义一个外层 Slice 长度为 `dy`，子元素也为  Slice，且元素长度为 `dx`。  
2. 然后使用两层 For 循环嵌套来计算里层 Slice 的值，里层 Slice 的值根据 `(x+y)/2`、`x*y` 或 `x^y` 来计算。

```go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
	pixel := make([][]uint8, dy)

	for i := range pixel {
        data := make([]uint8, dx)
		for j := range data {
			data[j] = uint8((i + j) / 2)
		}
		pixel[i] = data
	}
	return pixel
}

func main() {
	pic.Show(Pic)
}
```

得出一个 Base64 字符串，可添加至浏览器进行查看：

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAQAAAAEACAIAAADTED8xAAACaUlEQVR42uzVMRGAAAzAwLSHf8tgAAf95QVkyVNvNRN50FWBl10V6ABa0AFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIB6ADqEAHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdAA6gBZ0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIBSAcgHYB0ANIB6AAq0AFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgHQA0gFIByAdgA6gAh2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADSAUgHIB2AdADyxy8AAP//YSoDD5pLB7MAAAAASUVORK5CYII=
```

![](/images/go/a-tour-of-go-exercise-slices.png)

### 2. 参考

参考 A Tour of Go 的 Github 仓库中的[答案](https://github.com/golang/tour/blob/master/solutions/slices.go)，更符合我的实现思路。

```go
func Pic(dx, dy int) [][]uint8 {
	p := make([][]uint8, dy)
	for i := range p {
		p[i] = make([]uint8, dx)
	}

	for y, row := range p {
		for x := range row {
			row[x] = uint8(x * y)
		}
	}

	return p
}
```

先定义好里外层 Slice 结构，再进行计算。