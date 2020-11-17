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
    
- Groovy 定义方法时，如果参数没有类型，可以省略 def。
    ```groovy
    void doSomething(param1, param2) { }
    ```
      
- Groovy 定义构造器时，避免使用 def 关键字。
    ```
    builder.students {
       student {
          studentname 'Joe'
          studentid '1'
    		
          Marks(
             Subject1: 10,
             Subject2: 20,
             Subject3: 30,
          )
       } 
    }
    ```

- Groovy 中，单引号引起来的字符串是java字符串，不能使用占位符来替换变量，双引号引起的字符串则是 Java 字符串或者 Groovy 字符串。 
 
- Groovy 中的 GString 提供了强大的格式化能力，可以避免使用 Java 的 `String.format()` 来格式化输出。
    ```groovy
    def version = 2.4
    
    // Java String：单引号中的是普通字符串
    def javaString = 'Java'
    def javaString2 = 'Java $version'
    
    // Groovy String：双引号中且有未被转义的 $ 的是 GString, 允许使用占位符
    def groovyString = "Groovy v$version"
    // Java String：双引号中、有被转义的 $ 是普通字符串
    def nonGroovyString = "Groovy v\$version"
    ```
    还有一种常用用法：
    ```groovy
    def world() {
        "world"
    }
    
    println "Hello ${world()}"
    ```
- Groovy 中字符串分行可以使用 `\` 或多行字符串 `"""` 来实现。
    ```groovy
    def version = 2.4
    def verionInfo = "Groovy version \
    is v${version}"
    def verionInfo2 = """Groovy
        version
        is
        v${version}"""
    println verionInfo
    println verionInfo2
    ```
  
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
  
### 3. 集合

- Groovy 中 List 和 Map 提供 `each()` 方法用于便捷访问。
    ```groovy
    def list = [1, 2, 3]
    list.each { num -> println num }
    // 使用 it 变量进行迭代
    list.each { println it }
    
    def map = [a: 1, b: 2, c: 3]
    map.each { k, v -> println "$k $v" }
    // 使用 it 变量进行迭代
    map.each { println "$it.key $it.value" }
    ```  

- Groovy Map 集合的 Key 默认是字符串类型，可以省略双引号。

- Groovy 可以用 () 将 Keys 包起来插入一个变量或者类作为 Key。
    ```groovy
    def language = "Groovy"
    def map = [(language): 2.4 ]
    ```

### 4. 运算符

- Groovy 中三目运算符通常来给定变量默认值。
    ```groovy
    def result = name != null ? name : "Unknown"
    ```
    可以进行简写：
    ```groovy
    def result = name ?: "Unknown"
    ```

### 4. 方法

- Groov 方法中最后一句表达式可作为返回值返回，而不需要 return 关键字。

- Groovy 允许在顶级表达式中省略圆括号，比如 println "Hello World"。
    ```
    def echo(a) {
        a
    }
    echo 1
  
    def add(a, b) {
        a + b
    }
    add 1, 2
    ```

- Groovy 支持命名参数。但是和 Python 中的命名参数有很大区别。Groovy 命名参数并不是真正意义上的命名参数，实际上，Groovy 仅仅是让方法的 Map 参数的 Key 可以在方法中直接使用而已（以 Map.Key 这种方式来访问参数）。
    ```groovy
    def add(map) {
        // 为所有参数设置默认值
        ['a', 'b', 'c'].each { map.get(it, 0)}
        return map.a + map.b
    }
    
    println (add b:2, a:1)
    println (add a:1)
    ```
  
- Groovy 支持默认参数。
    ```groovy
    def echo(num = 1){
        num
    }
    println echo()
    ```
  
- Groovy 的 Switch 方法更具实用性，可以接受更多的类型。
    ```groovy
    def x = 1.23
    def result = ""
    switch (x) {
        case "foo": result = "found foo"
        // lets fall through
        case "bar": result += "bar"
        case [4, 5, 6, 'inList']: result = "list"
        break
        case 12..30: result = "range"
        break
        case Integer: result = "integer"
        break
        case Number: result = "number"
        break
        case { it > 3 }: result = "number > 3"
        break
        default: result = "default"
    }
    assert result == "number"
    ```
  
- Groovy 检查方法传入的参数是否为空，可以使用 assert 检查。
    ```groovy
    def check(String name) {
        // name non-null and non-empty according to Groovy Truth
        assert name
        // safe navigation + Groovy Truth to check
        assert name?.size() > 3
    }
    ```
  
### 5. 闭包

闭包是 Groovy 最强大的特性。

- 闭包可以直接当成函数调用。
    ```groovy
    // 定义闭包
    def codeBlock = {print "hello closure"}
    codeBlock()
    ```

- 可以将闭包看作一个参数传递给另一个方法。
    ```groovy
    def codeBlock = {print "hello closure"}
    
    // 定义 pipeline 函数，接受一个闭包参数
    def pipeline(closure) {
        closure()
    }
    // 调用 pipeline 函数传入闭包参数
    pipeline(codeBlock)
    // 把定义闭包的语句去掉
    pipeline({print "hello closure"})
    // 括号也可以去掉
    pipeline {
        print "hello closure"
    }
    ```

- 闭包的另类用法。
    ```groovy
    def stage(String name, closure) {
        println name
        closure()
    }
    // 正常情况下，这样使用 stage 函数
    stage("stage name", {println "hello closure"})
    // 但是，Groovy 提供了另一种写法
    stage("stage name") {
        println "hello closure"
    }
    ```

### 6. 类

Jenkins Pipeline 中大多编写的是脚本式的 Groovy 代码，一些通用功能或者模块会使用类来定义。

- Groovy 可以使用 Import 别名。
    ```groovy
    import java.util.List as jurist
    import java.awt.List as aList
    import java.awt.WindowConstants as WC
    ```
 
- Groovy 可以引入静态方法。
    ```groovy
    import static pkg.SomeClass.foo
    foo()
    ```
  
- Groovy 检查某个对象的值是否为 null，可以使用 `?.` 安全取值。
    ```groovy
    println order?.customer?.address
    ```
