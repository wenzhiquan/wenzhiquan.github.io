---
layout: post
title: "Golang 开发：单元测试与业务实践"
date: 2021-05-18 12:00:00
updated: 2021-05-18 12:00:00
copyright: true
categories:
  - Golang
tags:
  - Golang
---
# 说在前面
云浩老师以具体的 Golang 代码实例讲起，清晰明了地讲解了 Golang 的代码规范，然后讲解了各类 linter 的优缺点，并给出了一些最佳实践，为我们写出更加规范和一致的代码提供了指导。

# Golang 代码规范
## 为什么我们需要代码规范
说到代码规范，其实就是不同语言的代码风格不同，为了便于理解和统一而形成的编码命名等的建议。每个语言的代码风格，大部分来自创始团队的共同”审美”，不同的语言规范甚至有完全相反的规定，但是语言风格和代码规范没有对错之分，他的作用是形成最大共识，减少沟通和理解的成本。

<!-- more -->

## 命名规范
### package
-   只由小写字母组成。不包含大写字母和下划线等字符。
-   简短并包含一定的上下文信息。例如 time、list、http 等。
-   不能是含义模糊的常用名，或者与标准库同名。例如不能使用 util 或者 strings。
-   包名能够作为路径的 base name，在一些必要的时候，需要把不同功能拆分为子包。例如应该使用 encoding/base64 而不是 encoding_base64 或者 encodingbase64。

以下规则按照先后顺序尽量满足：
-   不使用常用变量名作为包名。例如使用 bufio 而不是 buf。
-   使用单数而不是复数。例如使用 encoding 而不是 encodings。
-   谨慎地使用缩写。例如使用 fmt 在不破坏上下文的情况下比 format 更加简短，以及一些需要用多个单词表达上下文的命名可以使用缩写，例如使用 strconv 而不是 stringconversion。

### function
Function 的命名应该遵循如下原则：

-   对于可导出的函数使用 MixedCaps，对于内部使用的函数使用 mixedCaps。
-   函数名不携带包名的上下文信息。例如使用 http.Server 而不是 http.HTTPServer，因为包名和函数名总是成对出现的。
-   函数名尽量简短：
    -   当名为 foo 的包某个函数返回类型 Foo 时，往往可以省略类型信息而不导致歧义。例如使用 time.Now() 以及 time.Parse()，两者返回的都是 time.Time 类型。
-   当名为 foo 的包某个函数返回类型 T 时（T 并不是 Foo），可以在函数名中加入类型信息。例如使用 time.ParseDuration() 返回的是 time.Duration 类型。
-   当名为 foo 的包某个函数返回类型 Foo，且 Foo 是其所有方法的入口时，可以使用 New() 来命名而不导致歧义。例如使用 list.New() 返回的是* list.List 类型。

### receiver
Receiver 的命名应该遵循如下原则：

-   不要使用面向对象编程中的常用名。例如不要使用 self、this、me 等。
-   一般使用 1 到 2 个字母的缩写代表其原来的类型。例如类型为 Client，可以使用 c、cl 等。
-   在每个此类型的方法中使用统一的缩写。例如在其中一个方法中使用了 c 代表了 Client，在其他的方法中也要使用 c 而不能使用诸如 cl 的命名。

### interface
Interface 的命名应该遵循如下原则：

-   对于只有一个方法的 interface，通常将其命名为方法名加上 er。例如 Reader 和 Writer。
-   interface 的方法不要占用一些惯用名，除非此方法具有同样的作用。例如 Read、Write、Flush、String、ServeHTTP。

### variable
Variable 的命名应该遵循如下原则：

-   对于可导出的变量使用 MixedCaps，对于内部使用的变量使用 mixedCaps。
-   [缩略词](https://github.com/golang/go/wiki/CodeReviewComments#initialisms) 全大写，但当其位于变量开头且不需要导出时，使用全小写。例如使用 ServeHTTP 而不是 ServeHttp，以及使用 XMLHTTPRequest 或者 xmlHTTPRequest。
-   简洁胜于冗长。例如在循环中，使用 i 代替 sliceIndex。
-   变量距离其被使用的地方越远，则需要携带越多的上下文信息。例如全局变量在其名字中需要更多的上下文信息，使得在不同地方可以轻易辨认出其含义。

### error
#### 错误变量的命名和位置
当某种错误多次出现或者需要在其他地方捕获时，我们需要显式地申明这个变量。错误变量名统一以 err 开头，如果需要暴露给外部使用，则以 Err 开头。

-   如果某文件仅存在一个错误，将错误变量放入该代码块开头。
-   如果某文件存在多个错误，当文件代码量较小的时候，放在文件的开头位置；当文件代码量较大时，不同种类的错误放在相应的代码块开头。
-   多个错误变量的申明应该与普通变量分隔开。

```
var errProofFailed = errors.New("invalid transparency proof")
```

#### 错误的内容
错误内容不应该包含大写字母（驼峰式变量或者缩写的情况例外），不要使用标点符号（例如。）作为结尾。

-   对于表意缺少足够上下文的错误，可以考虑在内容的前方加入 package 的名字作为标注。随后再加入一个冒号和一个空格作为与内容的分割。**如果某个 package 中的某错误加入 package 名字标注，需要对 package 中的每个 error 加上此标注来统一格式**。
-   不要使用中文等非 ascii 字符来作为内容，也不要使用拼音。
```
var (
   errNotEnough = errors.New("gif: not enough image data")
   errTooMuch   = errors.New("gif: too much image data")
   errBadPixel  = errors.New("gif: invalid pixel value")

)
```

#### 自定义错误类型
一般当错误比较复杂且有着共同的上下文信息时，可以考虑引入自定义错误，此时自定义错误需要放在 error.go 文件中。自定义错误类型的命名应该以 Error 结尾。
```
 // PathError records an error and the operation and file path that caused it.
type PathError struct {
   Op   string
   Path string
   Err  error
}

func (e *PathError) Error() string { return e.Op + " " + e.Path + ": " + e.Err.Error() }

func (e *PathError) Unwrap() error { return e.Err }
```

### comments
注释建议使用英文撰写。

#### package

每个 package 都需要有包注释，用于介绍包的作用和使用方法等信息。

-   开头为 Package foo 。
-   注释需要位于文件顶部，且只在一个文件中加入，内容较少的可以放在与包名同名的文件中，内容较多的放在 doc.go 文件中。
```
// Package log implements a simple logging package. It defines a type, Logger,
// with methods for formatting output. It also has a predefined 'standard'
// Logger accessible through helper functions Print[f|ln], Fatal[f|ln], and
// Panic[f|ln], which are easier to use than creating a Logger manually.
// That logger writes to standard error and prints the date and time
// of each logged message.  package log
```

#### exported names

对于所有 exported names 都需要加入注释，包括变量、常量、自定义类型、函数（包括结构体中导出的函数）等。**注释以其名字作为前缀**，特殊地，类型的注释允许以冠词 (The、A、An) 加上其名字为前缀。
```
// Flags returns the output flags for the standard logger.
// The flag bits are Ldate, Ltime, and so on.
func Flags() int {   return std.Flags()   }
```

#### 代码段注释

如果注释位于单独一行，则需要以大写开头（缩略词或变量名除外）并以。结尾。
```
if l.flag&(Lshortfile|Llongfile) != 0 {
 // Release lock while getting caller info - it's expensive.
 l.mu.Lock()
 ...
 l.mu.Unlock()
}
```

如果位于语句后方，则不要以大写开头，且结尾不需要加入标点符号。
```
const Ldate = 1 << iota // the date in the local time zone: 2009/01/23
```
# Linter 实践
## linter 工具
### 检查工具
- golint:Go 团队提供的风格检查工具，用于检查**代码风格**的错误部分。
- go vet:Go 语言 SDK 的 go 执行程序携带的工具，用于**检查程序正确性**。例如 unsafe.Pointer 的错误使用、unreachable code 等。
- gofmt:Go 语言 SDK 自带的 **format 工具**, 用于整理 Go 代码格式。例如整理缩进、给结尾加空等。

### golint
https://github.com/golang/lint

常见的检查点：comment s 风格、变量风格、 error string 等

问题一：年久失修，存在一些影响使用的问题，例如对于 error string 的处理，不支持以当前环境变量开头。

问题二：golint 的结果并不ー定正确，并且无法解決 false positives 问题

### go vet
https://golang.org/cmd/vet/

常见的正确性检查点：
1. 检查 HTTP Body 是否被正确 Closed
2. 原子操作的误用
3. 检查 unsafe.Pointer 的正确使用

### gofmt
https://golang.org/pkg/cmd/gofmt/

Go 团队提供的用于检查和修复代码格式错误的工具。

## linter 实践

### 只使用官方工具
只使用官方工具的流程
1. 开发者使用 gofmt 格式化代码
2. 开发者使用 golint 检查风格问题并修复
3. 开发者使用 go vet 检查可疑的正确性问题
4. CICD 使用 gofmt、 golint、 go vet 检查代码

存在的问题
1. 这些 linters 提示的问题不ー定正确，存在 false positives 的问题
2. 不同的功能分散在多个工具，不够便捷。没有统一的配置。
3. gofmt 和 go vet 在不同 GO SDK 的内容不同，容易造成不一致

### golangci-lint
https://github.com/golangci/golangci-lint

1. 解决了 false positives 问题，使用 //nolint 等方式来屏蔽错误提示
2. 统一的配置文件，解决了不同工具的配置文件问题
3. 整个过程只解析一遍 AST，也可以复用文件缓存，大大加快了速度
4. 可插拔 linter 配置自由选配 linters，自身携带了大部分常用 linters

## 最佳实践

### 项目启动阶段
1. 下载 golangci-lint
2. 将部门共享的配置下载到本地或者使用 golangci-lint bconfig 下载默认配置
### 正式开发阶段
1. 项目开发时，尽量解决 IDE 中所有带有波浪线或者变色的代码段
2. 每次编辑完一个文件，将文件 format 一次
3. 提交 commit 前，运行 golangci-lint，解決相关的问题
4. 解決所有 golangci-lint 问题后将代码 push 到仓库分支
### Merge Request 阶段
1. 在代码仓库阶段配置好 golangci-lint
2. 解决所有问题后，再请求 review
