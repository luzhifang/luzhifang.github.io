# Go命令详解


## 概述

输入 go 命令我们可以看到全部命令，使用 go help <command> 可以查看某个命令的详细解释和用法

```Shell
root@luzhifang:~# go
Go is a tool for managing Go source code.

Usage:

        go <command> [arguments]

The commands are:

        bug         start a bug report
        build       compile packages and dependencies
        clean       remove object files and cached files
        doc         show documentation for package or symbol
        env         print Go environment information
        fix         update packages to use new APIs
        fmt         gofmt (reformat) package sources
        generate    generate Go files by processing source
        get         add dependencies to current module and install them
        install     compile and install packages and dependencies
        list        list packages or modules
        mod         module maintenance
        run         compile and run Go program
        test        test packages
        tool        run specified go tool
        version     print Go version
        vet         report likely mistakes in packages

Use "go help <command>" for more information about a command.
```

## go bug

输入此命名后会直接打开默认浏览器，显示 go 的 github 页面进行 bug 报告，并会自动添加系统的信息。

## go build

Build 会编译导入路径命名的包及其依赖项，但不会安装结果。

用法：go build [-o output] [-i] [build flags] [packages]

如果构建的参数是.go 文件的列表，则 build 会将它们视为指定单个包的源文件列表。

编译单个主程序包时，build 会将生成的可执行文件写入以第一个源文件命名的输出文件（'go build ed.go rx.go'write'ed'或'ed.exe'）或源代码目录（ 'go build unix / sam'写'sam'或'sam.exe'）。编写 Windows 可执行文件时会添加“.exe”后缀。

在编译多个包或单个非主包时，build 会编译包但丢弃生成的对象，仅用于检查是否可以构建包。

编译包时，构建会忽略以“\_test.go”结尾的文件。

-o 标志仅在编译单个包时允许，强制构建将结果可执行文件或对象写入命名输出文件，而不是最后两段中描述的默认行为。

-i 标志安装作为目标依赖项的软件包。

查看 Plan9 汇编代码：go build -gcflags -S main.go

逃逸分析：go build -gcflags="-m" main.go

构建标志由构建，清理，获取，安装，列表，运行和测试命令共享：

```
-a
	强制重建已经是最新的软件包。
-n
	打印命令但不运行它们。
	go build -n 查看打包执行过程 compile.exe -> buildid.exe -> link.exe
-p n
	可以并行运行
	的程序数，例如构建命令或测试二进制文件。
	默认值是可用的CPU数。
-race
	启用数据竞争检测。仅支持linux / amd64，freebsd / amd64，darwin / amd64和windows / amd64。
-msan
	支持与内存清理程序的互操作。
	仅支持在linux / amd64，linux / arm64上，
	并且仅支持Clang / LLVM作为主机C编译器。
-v
	在编译时打印包的名称。
-work
	打印临时工作目录的名称，退出时不要删除它。
-x
	打印命令。

-asmflags '[pattern =] arg list'
	传递每个go工具asm调用的参数。
-buildmode mode
	构建模式使用。有关更多信息，请参阅“go help buildmode”。
-compiler
	要使用的编译器名称，如runtime.Compiler（gccgo或gc）。
-gccgoflags '[pattern =] arg list'
	传递每个gccgo编译器/链接器调用的参数。
-gcflags '[pattern =] arg list'
	传递每个go工具编译调用的参数。
-installsuffix suffix
	在软件包安装目录的名称中使用后缀，
	为了使输出与默认构建分开。
	如果使用-race标志，则安装后缀会自动设置为race，
	或者，如果明确设置，则会附加_race。同样对于-msan
	标志。使用需要非默认编译标志的-buildmode选项
	具有类似的效果。
-ldflags '[pattern =] arg list'
	传递每个go工具链接调用的参数。
-linkshared
	链接以前使用
	-buildmode = shared 创建的共享库。
-mod mode
	模块下载模式使用：readonly或vendor。
	有关更多信息，请参阅“go help modules”。
-pkgdir dir
	从dir 安装并加载所有包，而不是通常的位置。
	例如，
	使用非标准配置构建时，请使用-pkgdir将生成的包保留在单独的位置。
-tags '标记列表' 构建期间要考虑满足的以空格分隔的构建标记列表。有关构建标记的更多信息，请参阅
	go / build包文档中的构建约束说明。
-toolexec 'cmd args'
	用于调用vet和asm等工具链程序的程序。
	例如，go命令不是运行asm，而是运行
	'cmd args / path / to / asm <asm>的参数'。
```

## go clean

删除目标文件和缓存的文件

用法：go clean [clean flags] [build flags] [packages]

Clean 从包源目录中删除目标文件。go 命令在临时目录中构建大多数对象，因此 go clean 主要关注其他工具留下的目标文件或 go build 的手动调用。

这些文件包括：

- \_obj/ 旧的 object 目录，由 Makefiles 遗留
- \_test/ 旧的 test 目录，由 Makefiles 遗留
- \_testmain.go 旧的 gotest 文件，由 Makefiles 遗留
- test.out 旧的 test 记录，由 Makefiles 遗留
- build.out 旧的 test 记录，由 Makefiles 遗留
- \*.[568ao] object 文件，由 Makefiles 遗留
- DIR(.exe) 由 go build 产生
- DIR.test(.exe) 由 go test -c 产生
- MAINFILE(.exe) 由 go build MAINFILE.go 产生
- \*.so 由 SWIG 产生

参数包括：

- -i 清除关联的安装的包和可运行文件，也就是通过 go install 安装的文件
- -n 把需要执行的清除命令打印出来，但是不执行，这样就可以很容易的知道底层是如何运行的
- -r 循环的清除在 import 中引入的包
- -x 打印出来执行的详细命令，其实就是-n 打印的执行版本

## go doc

Doc 打印与其参数（包，const，func，类型，var，方法或结构字段）标识的项目相关联的文档注释。

用法：go doc [-u] [-c] [package | [package.] symbol [.methodOrField]]

参数列表：

```
-all
	显示包的所有文档。
-c
	在匹配符号时尊重大小写。
-cmd
	将命令（包main）视为常规包。
	否则，在显示程序包的顶级文档时，将隐藏程序包主导出的符号。
-src
	显示符号的完整源代码。
	这将显示其声明和定义的完整Go源，例如函数定义（包括
	正文），类型声明或封闭const块。因此输出可能包括未导出的细节。
-u
	显示未导出的符号，方法和字段的文档。
```

## go env

Env 打印 Go 环境信息。go env -w GOPROXY=https://goproxy.cn,direct -w 参数修改环境信息。

用法：go env [-json] [var ...]

默认情况下，env 将信息打印为 shell 脚本（在 Windows 上，即批处理文件）。如果给出一个或多个变量名作为参数，则 env 在其自己的行上打印每个命名变量的值。

-json 标志以 JSON 格式而不是 shell 脚本打印环境。

有关环境变量的更多信息，请参阅“go help environment”。

## go fix

更新包以使用新 API

用法：go fix [packages]

Fix 在导入路径命名的包上运行 Go fix 命令。

有关修复的更多信息，请参阅“go doc cmd/fix”。有关指定包的更多信息，请参阅“go help packages”。

要使用特定选项运行修复，请运行“go tool fix”。

## go fmt

Gofmt（重新格式化）包源

用法：go fmt [-n] [-x] [packages]

Fmt 在导入路径命名的包上运行命令'gofmt -l -w'。它打印修改的文件的名称。

有关 gofmt 的更多信息，请参阅“go doc cmd/gofmt”。有关指定包的更多信息，请参阅“go help packages”。

-n 标志打印将要执行的命令。-x 标志在执行时打印命令。

要使用特定选项运行 gofmt，请运行 gofmt 本身。

参数说明：

- -l 显示那些需要格式化的文件
- -w 把改写后的内容直接写入到文件中，而不是作为结果打印到标准输出。
- -r 添加形如“a[b:len(a)] -> a[b:]”的重写规则，方便我们做批量替换
- -s 简化文件中的代码
- -d 显示格式化前后的 diff 而不是写入文件，默认是 false
- -e 打印所有的语法错误到标准输出。如果不使用此标记，则只会打印不同行的前 10 个错误。
- -cpuprofile 支持调试模式，写入相应的 cpufile 到指定的文件

## go generate

通过处理源生成 Go 文件

用法：go generate [-run regexp] [-n] [-v] [-x] [build flags] [file.go ... | 包]

生成由现有文件中的指令描述的运行命令。这些命令可以运行任何进程，但目的是创建或更新 Go 源文件。

Go generate 永远不会通过 go build，go get，go test 等自动运行。它必须明确运行。

Go 生成扫描文件中的指令，这些指令是表单的行，

```
// go：生成命令参数...
```

## go get

下载并安装包和依赖项

旧版本执行命令后一般会下载在 GOPATH 的 src 目录下。go mod 方式下载的包源码在 GOPATH 目录下的 pkg/mod 目录下。

```
go get github.com/PuerkitoBio/goquery
```

用法：go get [-d] [-f] [-t] [-u] [-v] [-fix] [-insecure] [build flags] [packages]

- -d 标志指示在下载软件包后停止; 也就是说，它指示不安装软件包。
- -f 标志仅在设置-u 时有效，强制 get -u 不验证每个包是否已从其导入路径隐含的源控制存储库中检出。如果源是原始的本地分支，这可能很有用。
- -fix 标志指示 get 在解析依赖项或构建代码之前在下载的包上运行修复工具。
- -insecure 标志允许从存储库中提取并使用不安全的方案（如 HTTP）解析自定义域。谨慎使用。
- -t 标志指示 get 还下载构建指定包的测试所需的包。
- -u 标志指示 get 使用网络更新命名包及其依赖项。默认情况下，get 使用网络检出丢失的包，但不使用它来查找现有包的更新。
- -v 标志启用详细进度和调试输出。

## go install

编译并安装包和依赖项，执行后可执行文件自动安装到 GOPATH/bin 目录下。

```
go install main.go
```

用法：go install [-i] [build flags] [packages]

安装编译并安装导入路径命名的包。

-i 标志也会安装命名包的依赖项。

## go list

列出包或模块

用法：go list [-f format] [-json] [-m] [list flags] [build flags] [packages]

列表列出了命名包，每行一个。最常用的标志是-f 和-json，它们控制为每个包打印的输出形式。

## go mod

go mod 是 go modules 的简写，用于对 go 包的管理。详细信息可以看[go 包管理](/go-mod)

从 go1.11 开始实现了 modules 管理，mod 相比以前的方式，优点主要体现在：

- 项目不需要放在 GOPATH 下的 src 目录了，可以在任意目录。
- 自动下载第三方包和依赖包。
- 第三方包会指定版本号。
- 项目内会生成一个 go.mod 文件，文件内指定包依赖关系。
- 有一些第三方包不存在了或转移到其他地方，不需要改代码，只需用 replace 命令替换即可。

## go run

go run 用于编译并运行源码文件，由于包含编译步骤，所以 go build 参数都可用于 go run，在 go run 中只接受 go 源码文件而不接受代码包。

```
go run main.go
```

## go test

测试包

用法：go test [build / test flags] [packages] [build / test flags＆test binary flags]

- go test 重新编译每个包以及名称与文件模式“\* \_test.go”匹配的任何文件。
- 声明具有后缀“\_test”的包的测试文件将被编译为单独的包，然后链接并与主测试二进制文件一起运行。
- 将忽略名为“testdata”的目录，使其可用于保存测试所需的辅助数据。

Go 测试以两种不同的模式运行：

1. 第一种称为本地目录模式，在没有包参数的情况下调用 go test 时发生（例如，'go test'或'go test -v'）。在此模式下，go test 将编译当前目录中的包源和测试，然后运行生成的测试二进制文件。在此模式下，禁用缓存（下面讨论）。包测试完成后，go test 打印一条摘要行，显示测试状态（'ok'或'FAIL'），包名称和已用时间。

2. 第二种叫做包列表模式，在使用显式包参数调用 go test 时发生（例如'go test math'，'go test。/ ...'，甚至'go test。'）。在此模式下，go test 编译并测试命令行中列出的每个包。如果包测试通过，则 go test 仅打印最终的'ok'摘要行。如果包测试失败，则 go test 打印完整的测试输出。如果使用-bench 或-v 标志调用，则即使传递包测试，go test 也会打印完整输出，以显示请求的基准测试结果或详细日志记录。

参数列表：

```
-args
    将命令行的其余部分（-args之后的所有内容）
    传递给测试二进制文件，取消解释并保持不变。
    由于此标志占用命令行的其余部分，
    因此包列表（如果存在）必须出现在此标志之前。

-c
    将测试二进制文件编译为pkg.test但不运行它
    （其中pkg是包的导入路径的最后一个元素）。
    可以使用-o标志更改文件名。

-exec xprog
    使用xprog运行测试二进制文件。行为与
    'go run'中的行为相同。有关详细信息，请参阅“go help run”。

-i
    安装作为测试依赖项的包。
    不要运行测试。

-json
    将测试输出转换为适合自动处理的JSON。
    有关编码详细信息，请参阅“go doc test2json”。

-o file
    将测试二进制文件编译为指定文件。
    测试仍然运行（除非指定了-c或-i）。
```

## go version

go version 可以查看当前 go 的版本

## go tool

运行指定的 go 工具

用法：go tool [-n] command [args...]

Tool 运行由参数标识的 go 工具命令。没有参数，它打印已知工具列表。

-n 标志使工具打印将要执行但不执行它的命令。

## go vet

检查包中可能出现的错误

用法：go vet [-n] [-x] [-vettool prog] [build flags] [vet flags] [packages]

-n 标志打印将要执行的命令。-x 标志在执行时打印命令。

go vet 支持的构建标志是控制包解析和执行的构建标志，例如-n，-x，-v，-tags 和-toolexec。有关这些标志的更多信息，请参阅“go help build”。

## 参考

https://zhuanlan.zhihu.com/p/161494871

https://www.cnblogs.com/sunsky303/p/10788982.html

