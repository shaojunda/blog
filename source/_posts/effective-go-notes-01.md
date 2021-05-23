---
title: Effective go 笔记-01
date: 2020-11-29 19:59:46
tags:
  - golang
  - programing
keywords:
  - golang
  - effective-go
  - commentary
  - naming conventions
  - semicolons

description: effective-go 笔记01
---

想要写好 Go 代码不仅需要理解 Go 的特性和风格，还需要了解 Go 的命名、格式化以及项目结构等约定。

## 格式化（Formatting）

Go 使用 `go fmt` 格式化代码，这样开发者就不用纠结诸如缩进等问题了

before:
```golang
type T struct {
    name string // name of the object
    value int // its value
}
```
after:
```golang
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

## 注释（Commentary）

每个 package 最好都有一个 `package comment`，对于有多个文件的 package 只需要在其中一个文件中加入 `package comment` 就可以了。
* 多行注释

```golang
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```
<!--more-->
* 单行注释

如果一个包非常简单那么可以用单行注释来简单说明一下这个包的作用。

```golang
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```

在一个包中，任何紧接在顶层声明之前的注释都可以作为该声明的文档注释。每一个对外暴露的方法（首字母大写的方法）都应该有相应的注释。文档注释最好是完整的句子，第一句话应该是单句的摘要，以所声明的名称开头。
```golang
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {}
```

使用文档注释可以方便我们检索文档，找到对应包下的某个方法。

```shell
$ go doc -all regexp | grep -i parse
    Compile parses a regular expression and returns, if successful, a Regexp
    MustCompile is like Compile but panics if the expression cannot be parsed.
    parsed. It simplifies safe initialization of global variables holding
$
```

可以使用一条简单文档注释来对一组声明进行注释。

```golang
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```
分组有可以表示出元素之间的关系，比如一组变量是由一个 `mutex` 保护。

```golang
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```

## 命名规范（Naming conventions）

命名不仅仅在 Go 中很重要在其他语言中也很重要，在 Go 中它甚至会有语义上的影响：一个名字在包外的可见性取决于它的第一个字符是否是大写的。

###  Package names

当导入一个 package 之后这个 package 的名字将成为你调用它内容的访问器

```golang
import "bytes"

bytes.Buffer
```

好的 package 名称应该是「简短且易于理解的」，通常来说 package 名称应该是：
* 小写的单字名称且不应使用下划线或驼峰记法
* package 名称是源目录最里层的名称，比如： src/encoding/base64 package 名称是 base64

package 名称可以简化其内部内容的命名，比如 buffered reader type 在 `bufio` package 中只叫 `Reader` 而不是 `BufReader`，因为使用者是这样引用的 `bufio.Reader`，这已经足够清晰且简洁地表达意图了。且 `bufio.Reader` 也不会和 `io.Reader` 混淆，因为可以通过 package 名称进行区分。再比如用来创建 `ring.Ring` 对象的方法可以是 `ring.NewRing` 更简洁一点可以直接是 `ring.New` 应为 package 名称刚好也是「ring」。另一个简洁的命名是 `once.Do(setup)` 其可读性已经很好，使用 once.DoOrWaitUntilDone(setup) 并不会使其变得更好。长命名并不会使其变得更容易理解。一份有用的说明文档通常比额外的长名字更有价值。

### 存取器（Getters and Setters）

Go 没有对存取器提供自动的支持，需要我们手动添加。假设你有一个对象
```golang
type Car struct {
  owner string
  color string
}
```

对于 owner getter 的命名，叫 `Owner()` 就很好，对于 owner setter 的命名可以是 `SetOwner()`。


### 接口命名（interface names）

按照约定，只包含一个方法的接口应当以该方法的名称加上 -er 后缀来命名，如 Reader、Writer、 Formatter、CloseNotifier 等。

### 驼峰命名法

最后，Go 中约定使用驼峰记法 MixedCaps 或 mixedCaps 的方式来对多单词名称进行命名。

## 分号（Semicolons）

Go 虽然也是使用分号来结束语句，但是不需要在源码中手动输入，词法分析器会自动加上分号。如果在一行中写多个语句，需要用分号隔开。
```golang
// init statement
// condition expression
// post statement
for i := 0; i < 10; i++ {

}

if num := 9; num < 0 {
    fmt.Println(num, "is negative")
}
```
不能将一个控制结构（if、for、switch 或 select）的左打括号不能另起一行因为词法分析器会在大括号前面插入分号，这可能引起不需要的效果。
```golang
// 应该是：
if i < f() {
    g()
}
// 而不是：
if i < f()  // wrong!
{           // wrong!
    g()
}

```

Reference: [Effective Go](https://golang.org/doc/effective_go.html)
