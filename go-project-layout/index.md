# Go项目目录规范


## 目录结构

```
├── api
├── assets
├── build
│   ├── ci
│   └── package
├── cmd
│   └── _your_app_
├── configs
├── deployments
├── docs
├── examples
├── githooks
├── init
├── internal
│   ├── app
│   │   └── _your_app_
│   └── pkg
│       └── _your_private_lib_
├── pkg
│   └── _your_public_lib_
├── scripts
├── test
├── third_party
├── tools
├── vendor
├── web
│   ├── app
│   ├── static
│   └── template
├── website
├── .gitignore
├── LICENSE.md
├── Makefile
├── README.md
└── go.mod
```

## 开发

### `/cmd`

项目主要的应用程序。

对于每个应用程序来说这个目录的名字应该和项目可执行文件的名字匹配（例如，`/cmd/myapp`）。

有可能有多个可执行文件，如果是微服务，那么通常是只有一个可执行文件的。

通常来说，项目都应该拥有一个小的`main`函数，并在`main`函数中导入或者调用`/internal`和`/pkg`目录中的代码。

更多详情，请看[`/cmd`](https://github.com/golang-standards/project-layout/blob/master/cmd/README.md)目录中的例子。

### `/internal`

私有的应用程序代码库。这些是不希望被其他人导入的代码。请注意：这种模式是 Go 编译器强制执行的。再次注意，在项目的目录树中的任意位置都可以有`internal`目录，而不仅仅是在顶级目录中。

可以在内部代码包中添加一些额外的结构，来分隔共享和非共享的内部代码。这不是必选项（尤其是在小项目中），但是有一个直观的包用途是很棒的。应用程序实际的代码可以放在`/internal/app`目录（如，`internal/app/myapp`），而应用程序的共享代码放在`/internal/pkg`目录（如，`internal/pkg/myprivlib`）中。

### `/pkg`

外部应用程序可以使用的库代码（如，`/pkg/mypubliclib`）。其他项目将会导入这些库来保证项目可以正常运行，所以在将代码放在这里前，一定要三思而行。请注意，`internal`目录是一个更好的选择来确保项目私有代码不会被其他人导入，因为这是 Go 强制执行的。使用`/pkg`目录来明确表示代码可以被其他人安全的导入仍然是一个好方式。

如果项目确实很小并且嵌套的层次并不会带来多少价值（除非你就是想用它），那么就不要使用它。请仔细思考这种情况，当项目变得很大，并且根目录中包含的内容相当繁杂（尤其是有很多非 Go 的组件）。

微服务建议尽量不使用 pkg，公用的代码建议做成私有库（go module），除非公用的代码只在有限几个项目中可公用。

### `/api`

OpenAPI/Swagger 规范，JSON 模式文件，协议定义文件，比如 proto 文件。

更多样例查看[`/api`](https://github.com/golang-standards/project-layout/blob/master/api/README.md)目录。

### `/configs`

配置文件模板或默认配置。

将`confd`或者`consul-template`文件放在这里。

### `/third_party`

外部辅助工具，fork 的代码（可以魔改）和其他第三方工具（例如 Swagger UI）。

### `/vendor`

应用程序的依赖关系（通过手动或者使用喜欢的依赖管理工具，如新增的内置[Go Modules](https://github.com/golang/go/wiki/Modules)特性）。执行`go mod vendor`命令将会在项目中创建`/vendor`目录，注意，如果使用的不是 Go 1.14 版本，在执行`go build`进行编译时，需要添加`-mod=vendor`命令行选项，因为它不是默认选项。

构建库文件时，不要提交应用程序依赖项。

请注意，从[1.13](https://golang.org/doc/go1.13#modules)开始，Go 也启动了模块代理特性（使用`https：//proxy.golang.org`作为默认的模块代理服务器）。点击[这里](https://blog.golang.org/module-mirror-launch)阅读有关它的更多信息，来了解它是否符合所需要求和约束。如果`Go Module`满足需要，那么就不需要`vendor`目录。

## 测试

### `/test`

外部测试应用程序和测试数据。随时根据需要构建`/test`目录。对于较大的项目，有一个数据子目录更好一些。例如，如果需要 Go 忽略目录中的内容，则可以使用`/test/data`或`/test/testdata`这样的目录名字。请注意，Go 还将忽略以“`.`”或“`_`”开头的目录或文件，因此可以更具灵活性的来命名测试数据目录。

更多样例查看[`/test`](https://github.com/golang-standards/project-layout/blob/master/test/README.md)。

## 打包构建

### `/scripts`

用于执行各种构建，安装，分析等操作的脚本。

这些脚本使根级别的 Makefile 变得更小更简单（例如 https://github.com/hashicorp/terraform/blob/master/Makefile ）。

更多样例查看[`/scripts`](https://github.com/golang-standards/project-layout/blob/master/scripts/README.md)。

### `/build`

打包和持续集成。

将云（AMI），容器（Docker），操作系统（deb，rpm，pkg）软件包配置和脚本放在`/build/package`目录中。

将 CI（travis、circle、drone）配置文件和就脚本放在`build/ci`目录中。请注意，有一些 CI 工具（如，travis CI）对于配置文件的位置有严格的要求。尝试将配置文件放在`/build/ci`目录，然后链接到 CI 工具想要的位置。

## 部署

### `/deployments`

IaaS，PaaS，系统和容器编排部署配置和模板（docker-compose，kubernetes/helm，mesos，terraform，bosh）。请注意，在某些存储库中（尤其是使用 kubernetes 部署的应用程序），该目录的名字是`/deploy`。

## Web 相关

### `/web`

Web 应用程序特定的组件：静态 Web 资源，服务器端模板和单页应用（Single-Page App，SPA）。

### `/assets`

项目中使用的其他资源（图像，Logo 等）。

### `/website`

如果不使用 Github pages，则在这里放置项目的网站数据。

更多样例查看[`/website`](https://github.com/golang-standards/project-layout/blob/master/website/README.md)。

## 文档和示例

### `/docs`

设计和用户文档（除了 godoc 生成的文档）。

更多样例查看[`/docs`](https://github.com/golang-standards/project-layout/blob/master/docs/README.md)。

### `/ examples`

应用程序或公共库的示例。

更多样例查看[`/examples`](https://github.com/golang-standards/project-layout/blob/master/examples/README.md)。

## 其他

### `/init`

系统初始化（systemd、upstart、sysv）和进程管理（runit、supervisord）配置。

### `/tools`

此项目的支持工具。请注意，这些工具可以从`/pkg`和`/internal`目录导入代码。

更多样例查看[`/tools`](https://github.com/golang-standards/project-layout/blob/master/tools/README.md)。

### `/githooks`

Git 的钩子。

## 参考

https://github.com/golang-standards/project-layout

