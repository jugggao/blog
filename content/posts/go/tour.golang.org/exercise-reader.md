+++
title = "A Tour of Go - Exercise: Reader"
description = "Go 指南练习题"
date = 2020-11-26
categories = ["Go"]
tags = ["Go", "A Tour of Go", "Exercise"]
draft = false
+++

### [Exercise: Readers](https://tour.golang.org/methods/22)

Implement a `Reader` type that emits an infinite stream of the ASCII character `'A'`.

<!--more-->

### 1. 解题

Go 语言实现：

思路：

1. 参考代码中 [`reader.Validate`](https://github.com/golang/tour/blob/master/reader/validate.go) 方法很简单：

    ```go
    func Validate(r io.Reader) {
        b := make([]byte, 1024, 2048)
        i, o := 0, 0
        for ; i < 1<<20 && o < 1<<20; i++ { // test 1mb
            n, err := r.Read(b)
            for i, v := range b[:n] {
                if v != 'A' {
                    fmt.Fprintf(os.Stderr, "got byte %x at offset %v, want 'A'\n", v, o+i)
                    return
                }
            }
            o += n
            if err != nil {
                fmt.Fprintf(os.Stderr, "read error: %v\n", err)
                return
            }
        }
        if o == 0 {
            fmt.Fprintf(os.Stderr, "read zero bytes after %d Read calls\n", i)
            return
        }
        fmt.Println("OK!")
    }
    ```

2. 将 `A` 把 `b[:n]` 填满即可。

```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: Add a Read([]byte) (int, error) method to MyReader.
func (r MyReader) Read(b []byte) (n int, err error) {
    for i := range b {
        b[i] = 'A'
    }
    return len(b), nil
}

func main() {
    reader.Validate(MyReader{})
}
```

### 2. 参考

A Tour of Go 的 Github 仓库中的[答案](https://github.com/golang/tour/blob/master/solutions/readers.go)







