+++
title = "A Tour of Go - Exercise: Errors"
description = "Go 指南练习题"
date = 2020-11-15
categories = ["Go"]
tags = ["Go", "A Tour of Go", "Exercise"]
draft = false
+++

### [Exercise: Errors](https://tour.golang.org/methods/20)

Copy your Sqrt function from the [earlier exercise](https://tour.golang.org/flowcontrol/8) and modify it to return an `error` value.

`Sqrt` should return a non-nil error value when given a negative number, as it doesn't support complex numbers.

Create a new type
```
type ErrNegativeSqrt float64
```

and make it an error by giving it a
```
func (e ErrNegativeSqrt) Error() string
```

method such that `ErrNegativeSqrt(-2).Error()` returns `"cannot Sqrt negative number: -2"`.

Note: A call to `fmt.Sprint(e)` inside the `Error` method will send the program into an infinite loop. You can avoid this by converting e first: `fmt.Sprint(float64(e))`. Why?

Change your `Sqrt` function to return an `ErrNegativeSqrt` value when given a negative number.

<!--more-->

---

### 1. 解题

思路：
1. 为内建接口 `error` 新添一个 `func(e ErrNegativeSqrt) Error() string` 方法。
2. 在函数体中添加判断小于 0 则返回这个方法值。

```
package main

import (
	"fmt"
	"math"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprintf("cannot Sqrt negative number: %v", float64(e))
}

const delta = 1e-6

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, ErrNegativeSqrt(x)
	}
	
	z := x
	n := 0.0

	for math.Abs(n-z) > delta {
		n, z = z, z-(z*z-x)/(2*z)
	}
	return z, nil
}
```

Go 语言中争议最多的错误处理，在之后的实践中再亲自感受在发表自己的看法吧。

### 2. 参考

A Tour of Go 的 Github 仓库中的[答案](https://github.com/golang/tour/blob/master/solutions/errors.go)。

