---
layout: post
title: "Golang 工程管理与业务实践"
date: 2021-05-16 12:00:00
updated: 2021-05-16 12:00:00
copyright: true
categories:
  - Golang
tags:
  - Golang
---
# 写在前面
范峥老师从 Golang 的依赖管理的历史和演进过程讲起，详细的阐述了 go mod 的工作原理并讲述了一些常用工具和方法，紧接着，通过几个 case 让我们直观的了解到实际工作中可能遇到的问题及其解决方案。

# Golang 工程和依赖管理基本机制
## Golang 管理机制的演进
### 第一代：GOPATH

![](/uploads/in-post/go-dependency/gopath.png)
<!-- more -->
#### 特点
- 所有依赖库按路径组织在 $GOPATH/src 下
- 编译时直接使用 $GOPATH/src 下的代码
- go get 会下载代码到 $GOPATH/src 下
#### 缺点
- 依赖库的版本控制很麻烦，需要通过控制依赖库代码下载做版本控制
- 不同环境下依赖库版本不一致

### 第二代：govendor

![](/uploads/in-post/go-dependency/govendor.png)
#### 特点
- 所有依赖库按路径组织在项目的 vendor 目录下
- 编译时使用 vendor 中的代码
- 通过 govendor 工具来更新 vendor 下的代码
#### 缺点
虽然 govendor 通过 vendor 目录解决了不同环境下依赖库版本不一致的问题，但是依然没有解决依赖库版本控制麻烦的问题。

### 第三代：go mod

![](/uploads/in-post/go-dependency/gomod.png)
#### 特点
- 所有依赖代码按路径和版本号组织在$GOPATH/pkg/mod 目录下
- 采用 go.mod 文件描述依赖项的版本
- 通过 go get/mod 等命令管理依赖

通过 go.mod 文件，我们可以轻易的描述出依赖库的版本，并且编译时会使用 mod 目录下的代码，此目录的代码是根据 go.mod 文件描述进行下载的，保证了不同环境依赖库的一致性

## go mod 工作原理
### 版本表达

![](/uploads/in-post/go-dependency/gomod-version.png)

go.mod 的版本有两种表达方式：
1. semver 语义化表达：
- 每个语义版本都采用`v 主版本号。次版本号。修订版本号`
- **v** 所有版本号都以`v`开头
- **主版本号** 如果主版本号更新，将意味着 API 版本不再向下兼容
- **次版本号** 增加了新特性，并且向下兼容
- **修订版本号** 向下兼容的 bug 修复
2. 基于 commit 的伪版本号表达：
- 每个伪版本号都采用`v 基本版本-commit UTC 时间-commit hash 前 12 位`
- **v** 所有版本号都以`v`开头
- **commit UTC 时间** 时间使用 yyyymmddhhmmss 格式
- **commit hash** commit hash 使用前 12 位

### 常用命令

![](/uploads/in-post/go-dependency/command.png)
使用 go mod 的前提是已经初始化了项目，在项目根目录下使用`go mod init`即可完成初始化。

有了模块定义，然后执行 `go mod tidy` 会自动生成依赖，填充 go.mod, go.sum 文件

之后我们可以使用 `go list -m all` 查看当前项目正在使用的 package 版本，然后执行 `go get xxx/xxx` 来更新指定的 package

### 版本选择算法

![](/uploads/in-post/go-dependency/version-select.png)

### 实际使用中存在的问题

![](/uploads/in-post/go-dependency/indirect.png)

实际使用过程中，由于不是所有的库都是使用 go mod 进行版本控制，我们的 go.mod 中可能会有如上图的两个标记：
1. // indirect
表示非本项目所直接依赖，但在本项目中指定该依赖项的版本，可能原因：
- 依赖项没有使用 go mod
- 不是所有依赖项都在 go.mod 中
- 手动为依赖的依赖指定较新的版本
2. +incompatible
表示该依赖未使用 go mod 管理依赖
- 不影响使用，但该依赖的依赖的版本没有被显式指定
- 不再认为不同 major 版本号为不同项目

# 工程和依赖管理常见问题
## case 1 执行 go get –u 后项目编译不通过

![](/uploads/in-post/go-dependency/case1.png)
## case 2 为什么一些工程不使用 go mod？

![](/uploads/in-post/go-dependency/case2.png)
## case 3 部分项目不使用 go mod 导致的复杂场景
最终参与编译的 X 是 v1 还是 v2？

![](/uploads/in-post/go-dependency/case3.png)
## case 4 惨遭删 tag/branch/commit

![](/uploads/in-post/go-dependency/case4.png)
## case 5 循环依赖陷阱

![](/uploads/in-post/go-dependency/case5.png)

循环依赖产生的原因是两个 package 之间不能互相 import，但两个不同工程的不同 package 之间可以互相 import

循环依赖一旦形成，内部所有依赖的所有版本都会一直保留，一旦其中一个进行了修改，就很容易出现问题

**经验：公共库之间应明确分工，避免大杂烩和循环依赖**
