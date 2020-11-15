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

