# Effective go

## 1. 介绍

**英文**：

Go is a new language. Although it borrows ideas from existing languages, it has unusual properties that make effective Go programs different in character from programs written in its relatives. A straightforward translation of a C++ or Java program into Go is unlikely to produce a satisfactory result—Java programs are written in Java, not Go. On the other hand, thinking about the problem from a Go perspective could produce a successful but quite different program. In other words, to write Go well, it's important to understand its properties and idioms. It's also important to know the established conventions for programming in Go, such as naming, formatting, program construction, and so on, so that programs you write will be easy for other Go programmers to understand.

This document gives tips for writing clear, idiomatic Go code. It augments the [language specification](https://golang.org/ref/spec), the [Tour of Go](https://tour.golang.org/), and [How to Write Go Code](https://golang.org/doc/code.html), all of which you should read first.

**中文对照**：

Go是一门新语言。尽管它借鉴了现有语言的思想，但它具有不同寻常的特性，使得高效的go程序在性质上不同于用其相关语言编写的程序。将c++或Java程序直接翻译成Go不太可能产生令人满意的结果——Java程序是用Java编写的，而不是Go。另一方面，从go的角度来思考这个问题可以产生一个成功但又截然不同的程序。换句话说，要想写好go，理解它的性质和习惯用法是很重要的。同样重要的是，要了解go编程的既定约定，比如命名、格式化、程序构造等等，这样你写的程序才会让其他go程序员容易理解。

本文提供了编写清晰、惯用的Go代码的技巧。它扩展了语言规范、go语言之旅以及如何编写go代码，所有这些都应该先阅读。

### 1.1 例子

**英文**：

The [Go package sources](https://golang.org/src/) are intended to serve not only as the core library but also as examples of how to use the language. Moreover, many of the packages contain working, self-contained executable examples you can run directly from the [golang.org](https://golang.org/) web site, such as [this one](https://golang.org/pkg/strings/#example_Map) (if necessary, click on the word "Example" to open it up). If you have a question about how to approach a problem or how something might be implemented, the documentation, code and examples in the library can provide answers, ideas and background.

**中文对照**：

Go包源代码不仅作为核心库，而且作为如何使用该语言的示例。此外，许多包包含工作的、独立的可执行示例，您可以直接从golang.org网站运行这些示例(如有必要，单击“Example”打开它)。如果您对如何处理问题或如何实现某个东西有疑问，库中的文档、代码和示例可以提供答案、想法和背景。

## 2、格式化

英文：

Formatting issues are the most contentious but the least consequential. People can adapt to different formatting styles but it's better if they don't have to, and less time is devoted to the topic if everyone adheres to the same style. The problem is how to approach this Utopia without a long prescriptive style guide.

With Go we take an unusual approach and let the machine take care of most formatting issues. The `gofmt`program (also available as `go fmt`, which operates at the package level rather than source file level) reads a Go program and emits the source in a standard style of indentation and vertical alignment, retaining and if necessary reformatting comments. If you want to know how to handle some new layout situation, run `gofmt`; if the answer doesn't seem right, rearrange your program (or file a bug about `gofmt`), don't work around it.

As an example, there's no need to spend time lining up the comments on the fields of a structure. `Gofmt` will do that for you. Given the declaration

中文对照：

格式化问题最有争议，但影响最小。人们可以适应不同的格式风格，但如果不需要，那就更好了;如果每个人都坚持同一种风格，那么花在主题上的时间就会更少。问题是如何在没有冗长规范的风格指南的情况下实现这个 乌托邦（理想中最美好的社会）。

在Go中，我们采用了一种不同寻常的方法，让机器来处理大多数格式问题。“gofmt”程序(也可以作为“go  fmt”使用，它在包级别而不是源文件级别运行)读取一个go程序，并以标准的缩进和垂直对齐方式发出源文件，保留并在必要时重新格式化注释。如果你想知道如何处理一些新的布局情况，运行' gofmt ';如果答案似乎不正确，重新排列你的程序(或提交一个关于“go fmt”的bug)，不要绕过它。

例如，不需要花时间对结构的字段进行注释。“Gofmt”会帮你的。展示说明

```go
type T struct {
    name string // name of the object
    value int // its value
}
```

`gofmt` will line up the columns:   gofmt会将将列对齐排列

```go
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

All Go code in the standard packages has been formatted with `gofmt`. （所有在标准库的go 代码都会被gofmt 格式化。）

-----------------------------------------

Some formatting details remain. Very briefly:

- Indentation

  We use tabs for indentation and `gofmt` emits them by default. Use spaces only if you must.

- Line length

  Go has no line length limit. Don't worry about overflowing a punched card. If a line feels too long, wrap it and indent with an extra tab.

- Parentheses

  Go needs fewer parentheses than C and Java: control structures (`if`, `for`, `switch`) do not have parentheses in their syntax. Also, the operator precedence hierarchy is shorter and clearer, so

  ```go
  x<<8 + y<<16
  ```

  means what the spacing implies, unlike in the other languages
  
-----------------------------------

保留一些格式化细节。非常简单：

- 缩进
  我们使用制表符进行缩进，默认情况下“gofmt”会生成它们。只有在必要时才使用空格。
  
- 行长度
  go没有行长度限制。不用担心溢出。如果感觉一行太长，用一个额外的tab把它包起来缩进。

- 括号
  

go需要比c 和java 更少的括号：控制结构(if，for, switch)的语法中没有括号。此外，操作符优先层次结构代码会更短、更清晰,所以

  `x<<8  +  y<<16`

  表示空格所表示的内容，这与其他语言不同。

  ## 3、Commentary（注释）

Go provides C-style `/* */` block comments and C++-style `//` line comments. Line comments are the norm; block comments appear mostly as package comments, but are useful within an expression or to disable large swaths of code.

Go提供C风格的' /* */ '块注释和c++风格的' // '行注释。行注释是规范;块注释主要以包注释的形式出现，但在表达式中或禁用大量代码时非常有用。

The program—and web server—`godoc` processes Go source files to extract documentation about the contents of the package. Comments that appear before top-level declarations, with no intervening newlines, are extracted along with the declaration to serve as explanatory text for the item. The nature and style of these comments determines the quality of the documentation `godoc` produces.

`godoc`程序和web 服务处理go源文件并提取关于包内容的文档。 出现在顶级声明之前的注释（没有中间换行），会和声明一起被提取出来作用于该项的解释文本。这些注释的特征和风格决定了`godoc`生成的文档的质量。

Every package should have a *package comment*, a block comment preceding the package clause. For multi-file packages, the package comment only needs to be present in one file, and any one will do. The package comment should introduce the package and provide information relevant to the package as a whole. It will appear first on the `godoc` page and should set up the detailed documentation that follows.

每个包都应该有一个*package注释*，一个位于package子句之前的块注释。对于多文件包，包注释只需要出现在一个文件中，任何一个都可以。包注释应该介绍包并提供与整个包相关的信息。它将首先出现在`godoc `页面，并应设置以下详细的文档。

```go
/*
Package regexp implements a simple library for regular expressions.
	包regexp为正则表达式实现了一个简单的库。
The syntax of the regular expressions accepted is:
	接受的正则表达式的语法是

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

If the package is simple, the package comment can be brief.

如果包很简单，则包注释可以很简短。

```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
包路径实现用于操作斜杠分隔的文件名路径的实用程序惯例。
```

Comments do not need extra formatting such as banners of stars. The generated output may not even be presented in a fixed-width font, so don't depend on spacing for alignment—`godoc`, like `gofmt`, takes care of that. The comments are uninterpreted plain text, so HTML and other annotations such as `_this_` will reproduce *verbatim* and should not be used. One adjustment `godoc` does do is to display indented text in a fixed-width font, suitable for program snippets. The package comment for the [`fmt` package](https://golang.org/pkg/fmt/) uses this to good effect.

注释不需要额外的格式，例如星条旗。生成的输出甚至可能不以固定宽度的字体显示，因此不要依赖于对齐的间距—`godoc`(如`gofmt`)负责这方面的工作。注释是未解释的纯文本，因此HTML和其他注释(如`_this_`)将逐字复制，不应该使用。`godoc`所做的一个调整是用固定宽度的字体显示缩进的文本，适合于程序代码段。`fmt`包的包注释很好地使用了这一点。













