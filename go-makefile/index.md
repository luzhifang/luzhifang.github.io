# Go Makefile使用


## Makefile 简介

Makefile 是一个工程文件的编译规则，描述了整个工程的编译和链接等规则，这些规则里包含了这些内容：

- 工程中的哪些源文件需要编译，以及如何编译；
- 需要创建哪些库文件，以及如何创建；
- 如何最终生成我们想要的可执行文件。

默认情况下，make 命令会在当前目录下按如下顺序查找 Makefile 文件：“GNUmakefile”、“makefile”、“Makefile”的文件，一旦找到，就开始读取这个文件并执行。

**建议使用“Makefile”文件名**，因为这个文件名第一个字符大写，这样有一种显目的感觉。还有一些 make 只对全小写的“makefile”文件名敏感。大多数的 make 都支持“makefile”和“Makefile”这两种默认文件名。make 也支持-f 和--file 参数来指定其它文件名，比如：make -f golang.mk 或者 make --file golang.mk。

## 如何学习 Makefile

上面，我们简单介绍了 Makefile。那么如何学习 Makefile 呢？要回答这个问题，我们先来看一下 Makefile 的组成部分，Makefile 脚本文件内容由以下三部分组成：

- 一系列规则来指定源文件编译的先后顺序。规则是 makefile 中的重要概念，它一般由目标、依赖和命令组成。
- 特定的语法规则，支持变量、函数和函数调用等。
- 操作系统中的各种命令。

学习 Makefile，其实也就是对这 3 个部分的学习，分别对应于：

- Makefile 规则
- Makefile 语法
- Shell 脚本、Linux 命令等

## Makefile 的规则

```
target ... : prerequisites ...
  command
  ...
  ...
```

- target：可以是一个 object file（目标文件），也可以是一个执行文件，还可以是一个标签（label）。对于标签这种特性，在后续的“伪目标”章节中会有叙述。

- prerequisites：生成该 target 所依赖的文件和/或 target

- command：该 target 要执行的命令（任意的 shell 命令）

这是一个文件的依赖关系，也就是说，target 这一个或多个的目标文件依赖于 prerequisites 中的文件，其生成规则定义在 command 中。说白一点就是说:

**prerequisites 中如果有一个以上的文件比 target 文件要新的话，command 所定义的命令就会被执行。**

## make 的工作方式

1. 读入所有的 Makefile。
2. 读入被 include 的其它 Makefile。
3. 初始化文件中的变量。
4. 推导隐晦规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令。

## 伪目标

GO 项目下 Makefile 的管理能力都是通过伪目标来实现的，要执行的功能在 Makefile 中以伪目标的形式存在。

在上面的 Makefile 示例中，我们定义了一个 clean 目标，这个其实是一个伪目标，也就是说我们不会为该目标生成任何文件。因为伪目标不是文件，make 无法生成它的依赖关系和决定是否要执行它，通常我们需要显式地指明这个目标为伪目标。为了避免和文件重名，在 Makefile 中可以使用.PHONY 来标识一个目标为伪目标：

```
.PHONY: clean
clean:
    rm hello.o
```

伪目标可以有依赖文件，也可以作为“默认目标”，例如：

```
.PHONY: all
all: lint test build
```

因为伪目标总是会被执行，所以其依赖总是会被决议，通过这种方式，可以达到同时执行所有依赖项的目的。

## Makefile 语法

### 书写命令

每条规则中的命令和操作系统 Shell 的命令行是一致的。make 会一按顺序一条一条的执行命令，每条命令的开头必须以 `Tab` 键开头，除非，命令是紧跟在依赖规则后面的分号后的。

`#` 是注释符

### 显示命令

```
# 当我们用 @ 字符在命令行前，那么，这个命令将不被make显示出来
@echo 正在编译XXX模块......

# 如果没有“@”，将输出命令并且换行显示echo后面的内容
echo 正在编译XXX模块......
```

如果 make 执行时，带入 make 参数 `-n` 或 `--just-print` ，那么其只是显示命令，但不会执行命令，这个功能很有利于我们调试我们的 Makefile，看看我们书写的命令是执行起来是什么样子的或是什么顺序的。

而 make 参数 `-s` 或 `--silent` 或 `--quiet` 则是全面禁止命令的显示。

### 命令执行

如果你要让上一条命令的结果应用在下一条命令时，你应该使用分号分隔这两条命令。

```
# 当我们执行 make exec 时，第一个例子中的cd没有作用，pwd会打印出当前的Makefile目录
exec:
    cd /home/hchen
    pwd

# cd起作用，pwd会打印出“/home/hchen”。
exec:
    cd /home/hchen; pwd
```

### 命令出错

如果一个规则中的某个命令出错了（命令退出码非零），那么 make 就会终止执行当前规则，这将有可能终止所有规则的执行。

给 make 加上 `-i` 或是 `--ignore-errors` 参数，那么，Makefile 中所有命令都会忽略错误。

还有一个要提一下的 make 的参数的是 `-k` 或是 `--keep-going` ，这个参数的意思是，如果某规则中的命令出错了，那么就终止该规则的执行，但继续执行其它规则。

可以通过在命令行前加-符号，来让 make 忽略命令的出错，比如

```
clean:
    -rm hello.o
```

### 引入其它的 Makefile

```
include <filename>
```

### 变量赋值

通过变量声明来声明一个变量，变量在声明时需要赋予一个初值，比如：`GO=go`，引用变量时可以通过`$()`或者`${}`方式引用，**建议用`$()`方式引用变量**：`$(GO)`，也建议整个 makefile 的变量引用方式要保持一致。变量会像 bash 变量一样，在使用它的地方展开。

Makefile 中一共有 4 种变量赋值方法。

1. = 最基本的赋值方法。

```
# B最后的值为：c b，而不是a b。也就是说，在用变量给变量赋值时，右边变量的取值取的是最终的变量值。
A = a
B = $(A) b
A = c
```

2. := 直接赋值，赋予当前位置的值。

```
# B最后的值为：a b。通过:=可以避免=赋值带来的一些潜在的不一致。
A = a
B := $(A) b
A = c
```

3. ?= 表示如果该变量没有被赋值，则赋予等号后的值。

4. += 表示将等号后面的值追加到前面的变量上。

### 多行变量

```
define 变量名
变量内容
...
endef
```

### 环境变量

Makefile 也像 Linux 一样支持环境变量，在 Makefile 中有 2 种环境变量：Makefile 预定义的环境变量和自定义的环境变量。

其中自定义的环境变量可以覆盖 Makefile 预定义的环境变量。默认情况下 Makefile 中定义的环境变量只在当前 Makefile 有效，如果想向下层传递（Makefile 中调用另一个 Makefile），需要使用 export 关键字来声明，如下声明了一个环境变量，并可以在下层 Makefile 中使用：

```
export USAGE_OPTIONS
```

### 特殊变量

特殊变量是 make 提前定义好的，可以在 makefile 中直接引用，特殊变量列表如下：

| 变量          | 含义                                                                                                    |
| ------------- | ------------------------------------------------------------------------------------------------------- |
| MAKE          | 当前 make 解释器的文件名                                                                                |
| MAKECMDGOALS  | 命令行中指定的目标名（make 的命令行参数）                                                               |
| CURDIR        | 当前 make 解释器的工作目录                                                                              |
| MAKE_VERSION  | 当前 make 解释器的版本                                                                                  |
| MAKEFILE_LIST | make 所需要处理的 makefile 文件列表，当前 makefile 的文件名总是位于列表的最后，文件名之间以空格进行分隔 |
| .DEFAULT_GOAL | 指定如果在命令行中未指定目标，应该构建哪个目标，即使这个目标不是在第一行                                |
| .VARIABLES    | 所有已经定义的变量名列表（预定义变量和自定义变量）                                                      |
| .FEATURES     | 列出本版本支持的功能，以空格隔开                                                                        |
| .INCLUDE_DIRS | make 查询 makefile 的路径，以空格隔开                                                                   |

### 条件语句

#### 语法：

```
<conditional-directive>
<text-if-true>
else
<text-if-false>
endif
```

其中 `<conditional-directive>` 表示条件关键字，如 `ifeq` 。这个关键字有四个，如下

| 关键字 | 释义   |
| ------ | ------ |
| ifeq   | 相等   |
| ifneq  | 不相等 |
| ifdef  | 不为空 |
| ifndef | 为空   |

#### 示例

```
ifeq ($(strip $(foo)),)
<text-if-empty>
endif

ifeq (<arg1>, <arg2>)
<text-if-true>
endif

ifdef <variable-name>
<text-if-true>
endif
```

### 函数

#### 函数定义

```
define 函数名
函数体
endef
```

例如，如下是一个自定义函数：

```
define Foo
    @echo "my name is $(0)"
    @echo "param is $(1)"
endef
```

define 本质上是定义一个多行变量，可以在 call 的作用下当作函数来使用，在其它位置使用只能作为多行变量的使用，例如：

```
var := $(call Foo)
new := $(Foo)
```

#### 预定义函数

make 编译器也定义了很多函数，这些函数叫做预定义函数，调用语法和变量类似，语法为：

```
$(<function> <arguments>)
```

或者

```
${<function> <arguments>}
```

`<function>`是函数名，`<arguments>`是函数参数，参数间以逗号（,）分割。函数的参数也可以为变量。

常用的函数，罗列如下：

| 函数名                            | 功能描述                                                                                                                                                                                                                                                                                                                                                                                                                     |
| :-------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $(origin <variable>)              | 告诉变量的“出生情况”，有如下返回值：<br><ul><li>undefined：<variable> 从来没有定义过</li><li>default：<variable> 是一个默认的定义</li><li>environment：<variable> 是一个环境变量</li><li>file：<variable> 这个变量被定义在 Makefile 中</li><li>command line：<variable> 这个变量是被命令行定义的</li><li>override：<variable> 是被 override 指示符重新定义的</li><li>automatic：<variable> 是一个命令运行中的自动化变量</li> |
| $(addsuffix <suffix>,<names...>)  | 把后缀<suffix>加到<names>中的每个单词后面，并返回加过后缀的文件名序列。                                                                                                                                                                                                                                                                                                                                                      |
| $(addprefix <prefix>,<names...>)  | 把前缀<prefix>加到<names>中的每个单词后面，并返回加过前缀的文件名序列。                                                                                                                                                                                                                                                                                                                                                      |
| $(wildcard <pattern>)             | 扩展通配符，例如：$(wildcard ${ROOT_DIR}/build/docker/\*)                                                                                                                                                                                                                                                                                                                                                                    |
| $(word <n>,<text>)                | 取字符串<text>中第<n>个单词（从一开始），并返回字符串<text>中第<n>个单词。如 <n>比<text>中的单词数要大，那么返回空字符串                                                                                                                                                                                                                                                                                                     |
| $(subst <from>,<to>,<text>)       | 把字串 <text> 中的 <from> 字符串替换成 <to>，并返回被替换后的字符串                                                                                                                                                                                                                                                                                                                                                          |
| $(eval <text>)                    | 将<text>的内容将作为 makefile 的一部分而被 make 解析和执行。                                                                                                                                                                                                                                                                                                                                                                 |
| $(firstword <text>)               | 取字符串 <text> 中的第一个单词，并返回字符串 <text> 的第一个单词                                                                                                                                                                                                                                                                                                                                                             |
| $(lastword <text>)                | 取字符串 <text> 中的最后一个单词，并返回字符串 <text> 的最后一个单词                                                                                                                                                                                                                                                                                                                                                         |
| $(abspath <text>)                 | 将<text>中的各路径转换成绝对路径，并将转换后的结果返回                                                                                                                                                                                                                                                                                                                                                                       |
| $(shell cat foo)                  | 执行操作系统命令，并返回操作结果                                                                                                                                                                                                                                                                                                                                                                                             |
| $(info <text ...>)                | 输出一段信息                                                                                                                                                                                                                                                                                                                                                                                                                 |
| $(warning <text ...>)             | 出一段警告信息，而 make 继续执行                                                                                                                                                                                                                                                                                                                                                                                             |
| $(error <text ...>)               | 产生一个致命的错误，<text ...> 是错误信息                                                                                                                                                                                                                                                                                                                                                                                    |
| $(filter <pattern...>,<text>)     | 以<pattern>模式过滤<text>字符串中的单词，保留符合模式<pattern>的单词。可以有多个模式。返回符合模式<pattern>的字串                                                                                                                                                                                                                                                                                                            |
| $(filter-out <pattern...>,<text>) | 以<pattern>模式过滤<text>字符串中的单词，去除符合模式<pattern>的单词。可以有多个模式，并返回不符合模式<pattern>的字串                                                                                                                                                                                                                                                                                                        |
| $(dir <names...>)                 | 从文件名序列<names>中取出非目录部分。非目录部分是指最後一个反斜杠（/）之后的部分。返回文件名序列<names>的非目录部分。                                                                                                                                                                                                                                                                                                        |
| $(notdir <names...>)              | 从文件名序列<names>中取出非目录部分。非目录部分是指最後一个反斜杠（/）之后的部分。返回文件名序列<names>的非目录部分。                                                                                                                                                                                                                                                                                                        |
| $(strip <string>)                 | 去掉<string>字串中开头和结尾的空字符，并返回去掉空格后的字符串                                                                                                                                                                                                                                                                                                                                                               |
| $(suffix <names...>)              | 从文件名序列<names>中取出各个文件名的后缀。返回文件名序列<names>的后缀序列，如果文件没有后缀，则返回空字串。                                                                                                                                                                                                                                                                                                                 |
| $(foreach <var>,<list>,<text>)    | 把参数<list>中的单词逐一取出放到参数<var>所指定的变量中，然后再执行<text>所包含的表达式。每一次 <text>会返回一个字符串，循环过程中<text>的所返回的每个字符串会以空格分隔，最后当整个循环结束时，<text>所返回的每个字符串所组成的整个字符串（以空格分隔）将会是 foreach 函数的返回值。                                                                                                                                        |

## Makefile 常见管理内容

- 静态代码检查(lint)：推荐用 golangci-lint。
- 单元测试(test)：运行 go test ./...。
- 编译(build)：编译源码，支持不同的平台，不同的 CPU 架构。
- 镜像打包和发布(image/image.push)：现在的系统比较推荐用 Docker/Kubernetes 进行部署，所以一般也要有镜像构建功能。
- 清理（clean）:清理临时文件或者编译后的产物。
- 代码生成（gen）：比如要编译生成 protobuf pb.go 文件。
- 部署（deploy，可选）：一键部署功能，方便测试。
- 发布（release）：发布功能，比如：发布到 Docker Hub、github 等。
- 帮助（help）:告诉 Makefile 有哪些功能，如何执行这些功能。
- 版权声明（add-copyright）：如果是开源项目，可能需要在每个文件中添加版权头，这可以通过 Makefile 来添加。
- API 文档（swagger）：如果使用 swagger 来生成 API 文档，这可以通过 Makefile 来生成。

## 参考

https://github.com/marmotedu/geekbang-go/tree/master/makefile

https://seisman.github.io/how-to-write-makefile/Makefile.pdf

