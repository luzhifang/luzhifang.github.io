# Go包管理


## 包管理演进

### GOPATH 模式

版本：最初版本就有了，1.13 之后不推荐使用了

用法：GOPATH 是配置 Go 开发环境时所设置的一个环境变量。历史版本的 go 语言开发时，需要将代码放在 GOPATH 目录的 src 文件夹下。go get 命令获取依赖，也会自动下载到 GOPATH 的 src 下。

缺点：go get 命令使用时，没有版本选择机制，拉下来的依赖代码都会默认当前最新版本

### vendor 机制

版本：1.5 之后

用法：每个项目都可以有一个 vendor/ 目录来存放项目所需版本依赖的拷贝（vendor 目录需在 GOPATH 之内）。

缺点：基于该机制的管理工具虽然丰富，但是不同版本工具之间不兼容，无法协作，各种工具还都有学习成本。

### Go Mod

版本：Go 1.11 和 1.12 需要设置 GO111MODULE=on 才能强制使用 Go Mod，1.13 之后默认在 GOPATH 之外开始使用 Go Mod

用法：后面细讲

## Go Mod

### Go Mod 环境变量

```
GO111MODULE="auto" # auto/on/off三种值，可通过go env -w GO111MODULE=on设置
GOPROXY="https://goproxy.io,direct" # 设置 Go 模块代理，go env -w GOPROXY=https://goproxy.cn,direct
GONOPROXY="" # 禁用代理的模块，逗号隔开，不建议使用
GOSUMDB="sum.golang.org" # 默认值为：sum.golang.org，设置了代理会走代理的，设置为off则禁用校验模块版本
GONOSUMDB="" # 禁用校验的模块，逗号隔开，不建议使用
GOPRIVATE="" # 私有模块，禁用代理和校验，go env -w GOPRIVATE="*.test.com,github.com/test/pkg"
```

### Go Mod 操作命令

```
download # 下载依赖包到本地缓存GOPATH/src/pkg/mod
edit #编辑 go.mod 文件一般直接用 ide 编辑就行
graph # 打印模块依赖图
init # 初始化go.mod文件，例：go mod init test.com/demo1
tidy # 拉取缺少的模块，移除不用的模块
vendor # 从mod cache中拷贝到项目的vendor，使用vendor：go build -mod vendor
verify # 验证依赖是否正确
why # 解释为什么需要依赖
```

### go.mod 文件

```
# 指定当前项目的模块导入路径为 test.com/demo1
module test.com/demo1

# 指定go版本
go 1.14

# 引入包
require (
  github.com/jinzhu/gorm v1.0.0
)

# 排除一个特定的模块版本
exclude (
	github.com/google/uuid v1.1.0
)

# 替换一个无法下载的包，或者引入一个本地的包
replace (
    github.com/google/uuid v1.1.1 => ../uuid
)
```

### 依赖的下载和更新

```Shell
# go get 默认下载最新版本
# go get 命令使用的版本是 git 中的 tag
# go get 之后会在go.mod的require中增加一项，并会更新go.sum
go get github.com/jinzhu/gorm@v1.0.0

# 使用-u表示更新版本
go get -u github.com/jinzhu/gorm@v1.0.0
```

### go.sum 文件

1. 用途：防止下载依赖包被恶意篡改
2. 生成过程：如果本地计算出的依赖包版本哈希值与 GOSUMDB 服务器给出的哈希值不一致，go 命令将拒绝向下执行，也不会更新 go.sum 文件。

