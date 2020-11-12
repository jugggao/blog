+++
title = "Jenkins Pipeline - Groovy 知识点"
description = "Go 指南练习题"
date = 2020-11-12
categories = ["Jenkins"]
tags = ["Jenkins", "Jenkins Pipeline", "Groovy"]
draft = false
+++

### Groovy 知识点

如果想深入学习 Jenkins，并编写 Jenkins Pipeline 共享库，Groovy 是必须学习并了的。  
Groovy 是动态语言，语法和 Java 基本一致，实际上就是 Java，但是又有一些特性。  
这里就记录自己需要注意的特性。

<!--more-->

### 1. 数据类型

- Groovy 同时支持静态类型和动态类型。

    ```groovy
    def dynamicDate  = new Date()
    Date staticDate = new Date()
    // 可以用 var.class 获取变量类型
    println dynamicDate.class == staticDate.class
    ```
    
    如果声明了静态类型，就不能再改变该变量的类型。
      
### 2. 作用域

在编写 Jenkins Pipeline 共享库的时候需要注意，跨 Stage 传参的时候就需要注意其参数的作用域。

- Groovy 类的作用域和 Java 相同。

- **Groovy 脚本中的作用域分为绑定域和本地域。**
    - 绑定域：脚本内的全局作用域，相当于该脚本对象的成员变量。如果未定义变量，仅仅初始化但未声明，其作用域就是绑定域（与 JavaScript 中行为相同）。
    - 本地域：脚本内的代码块。如果定义过变量，其作用域就是本地域。
    ```groovy
    String hello = "hello"	// 定义变量，作用域是本地域
    def world = "world"		//定义变量，作用域是本地域

    helloworld = "hello world"	// 全局变量，作用域是绑定域
    
    void check() {
        println hello // 找不到变量
        println world // 找不到变量
        println helloworld // 可以正常打印
    }
    check()  
    ```
  
- Groovy 通过 new 建立的脚本对象可以绑定指定的值到该脚本的绑定域中。
    ```groovy
    // File: test.groovy
    helloworld = "hello world" // 绑定域
    
    def hello() { println(helloworld) }

    def s = new test()
    s.binding.goodbye = "good bye"	//绑定不存在的变量不会报错
    s.binding.helloworld = "hello groovy" // 修改绑定域中存在的变量的值
    s.hello()	// hello groovy
    ```

- 使用 [@Filed](https://docs.groovy-lang.org/2.5.8/html/gapi/index.html?groovy/transform/Field.html) 标注也可以将变量的作用域变为绑定域中（和 Python 中的 global 声明变量类似）。
    ```groovy
    import groovy.transform.Field
    @Field List awe = [1, 2, 3]
    def awesum() { awe.sum() }
    assert awesum() == 6
    ```