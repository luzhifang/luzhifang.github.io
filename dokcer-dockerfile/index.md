# Docker Dockerfile


## 什么是 Dockerfile

Dockerfile 是一个文本文件，记录了镜像构建的所有步骤。

## Dockerfile 构建镜像的过程

1. 从 base 镜像运行一个容器。
2. 执行一条指令，对容器做修改。
3. 执行类似 docker commit 的操作，生成一个新的镜像层。
4. Docker 再基于刚刚提交的镜像运行一个新容器。
5. 重复 2 ～ 4 步，直到 Dockerfile 中的所有指令执行完毕。

## 使用 Dockerfile 定制镜像

这里仅讲解如何运行 Dockerfile 文件来定制一个镜像，具体 Dockerfile 文件内指令详解，将在下一节中介绍，这里你只要知道构建的流程即可。

### 下面以定制一个 nginx 镜像（构建好的镜像内会有一个 /usr/share/nginx/html/index.html 文件）

在一个空目录下，新建一个名为 Dockerfile 文件，并在文件内添加以下内容：

FROM nginx
RUN echo '这是一个本地构建的 nginx 镜像' > /usr/share/nginx/html/index.html

### FROM 和 RUN 指令的作用

FROM：定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。

RUN：用于执行后面跟着的命令行命令。

**注意**：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。例如：

```Shell
FROM centos
RUN yum -y install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
以上执行会创建 3 层镜像。可简化为以下格式：
```

```Shell
FROM centos
RUN yum -y install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz
```

如上，以 && 符号连接命令，这样执行后，只会创建 1 层镜像。

### 构建镜像

在 Dockerfile 文件的存放目录下，执行[docker build](/docker-command/#docker-build)构建动作。

以下示例，通过目录下的 Dockerfile 构建一个 nginx:v3（镜像名称:镜像标签）。

**注意**：最后的 . 代表本次执行的上下文路径。

```Shell
$ docker build -t nginx:v3 .
```

## Dockerfile 指令详解

### FROM

指定 base 镜像。

### MAINTAINER

设置镜像的作者，可以是任意字符串。

### COPY

将文件从 build context 复制到镜像。COPY 支持两种形式： COPY src dest 与 COPY ["src", "dest"]。

**注意**：src 只能指定 build context 中的文件或目录。

### ADD

与 COPY 类似（同样需求下，官方推荐使用 COPY），从 build context 复制文件到镜像。不同的是，如果 src 是归档文件（tar、zip、tgz、xz 等），文件会被自动解压到 dest，同时 ADD 也支持 URL 而 COPY 不支持。

### WORKDIR

为后面的 RUN、CMD、ENTRYPOINT、ADD 或 COPY 指令设置镜像中的当前工作目录。

### RUN

在容器中运行指定的命令。有以下俩种格式：

shell 格式：

```Shell
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令。
```

exec 格式：

```Shell
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```

### CMD

类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

- CMD 在 docker run 时运行。
- RUN 是在 docker build。

作用：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

**注意**：如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

格式：

```Shell
CMD <shell 命令>
CMD ["<可执行文件或命令>","<param1>","<param2>",...]
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```

推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。

### ENTRYPOINT

类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。

但是, 如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 ENTRYPOINT 指令指定的程序。

优点：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。

**注意**：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

格式：

```Shell
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下示例会提到。

示例：

假设已通过 Dockerfile 构建了 nginx:test 镜像：

```Shell
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参
```

RUN、CMD、ENTRYPOINT 三者用法最佳实践

1. RUN：使用 RUN 指令安装应用和软件包，创建新的镜像层。
2. CMD：如果想为容器设置默认的启动命令，可使用 CMD 指令。用户可在 docker run 命令行中替换此默认命令。
3. ENTRYPOINT：如果 Docker 镜像的用途是运行应用程序或服务，比如运行一个 MySQL，应该优先使用 Exec 格式的 ENTRYPOINT 指令。CMD 可为 ENTRYPOINT 提供额外的默认参数，同时可利用 docker run 命令行替换默认参数。

### ENV

设置环境变量，环境变量可被后面的指令使用。

格式：

```Shell
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

以下示例设置 NODE_VERSION = 7.2.0 ， 在后续的指令中可以通过 $NODE_VERSION 引用：

```Shell
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
```

### ARG

构建参数，与 ENV 作用一致。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

### EXPOSE

仅仅只是声明端口。

作用：

- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
- 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

格式：

```Shell
EXPOSE <端口1> [<端口2>...]
```

### VOLUME

定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：

- 避免重要的数据，因容器重启而丢失，这是非常致命的。
- 避免容器不断变大。

格式：

```Shell
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。

### ONBUILD

用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这时执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

### USER

用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。

格式：

```Shell
USER <用户名>[:<用户组>]
```

### LABEL

LABEL 指令用来给镜像添加一些元数据（metadata），以键值对的形式，语法格式如下：

```Shell
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

比如我们可以添加镜像的作者：

```Shell
LABEL org.opencontainers.image.authors="ubuntu"
```

## 多阶段构建

编译、测试、打包等同时放到一个 Dockerfile 文件，会带来一些问题：

- 镜像层次多，镜像体积较大，部署时间变长
- 源代码存在泄露的风险

分散到多个 Dockerfile 文件又太复杂。

Docker v17.05 开始支持多阶段构建 (multistage builds)。使用多阶段构建我们就可以很容易解决前面提到的问题，并且只需要编写一个 Dockerfile：

```
FROM golang:alpine AS builder

LABEL stage=gobuilder

ENV CGO_ENABLED 0
ENV GOPROXY https://goproxy.cn,direct
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

RUN apk update --no-cache && apk add --no-cache tzdata

WORKDIR /build

ADD go.mod .
ADD go.sum .
RUN go mod download
COPY . .
COPY user-api/etc /app/etc
RUN go build -ldflags="-s -w" -o /app/user user-api/user.go


FROM scratch

COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=builder /usr/share/zoneinfo/Asia/Shanghai /usr/share/zoneinfo/Asia/Shanghai
ENV TZ Asia/Shanghai

WORKDIR /app
COPY --from=builder /app/user /app/user
COPY --from=builder /app/etc /app/etc

CMD ["./user", "-f", "etc/user-api.yaml"]
```

### 只构建某一阶段的镜像

我们可以使用 as 来为某一阶段命名，例如

```
FROM golang:alpine as builder
```

例如当我们只想构建 builder 阶段的镜像时，增加 --target=builder 参数即可

```Shell
$ docker build --target builder -t username/imagename:tag .
```

### 构建时从其他镜像复制文件

可以使用 `COPY --from=0 /go/src/github.com/go/helloworld/app .` 从上一阶段的镜像中复制文件，我们也可以复制任意镜像中的文件。

```Shell
$ COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

## 最佳实践

- 不要安装无效软件包。
- 应简化镜像中同时运行的进程数，理想状况下，每个镜像应该只有一个进程。
- 当无法避免同一镜像运行多进程时，应选择合理的初始化进程（init process）。
- 最小化层级数
  - 最新的 docker 只有 RUN， COPY，ADD 创建新层，其他指令创建临时层，不会增加镜像大小。
    - 比如 EXPOSE、VOLUME 指令就不会生成新层。
  - 多条 RUN 命令可通过连接符连接成一条指令集以减少层数。
  - 通过多段构建减少镜像层数。
- 把多行参数按字母排序，可以减少可能出现的重复参数，并且提高可读性。
- 编写 dockerfile 的时候，应该把变更频率低的编译指令优先构建以便放在镜像底层以有效利用 build cache。
- 复制文件时，每个文件应独立复制，这确保某个文件变更时，只影响该文件对应的缓存。

## 参考

[《每天 5 分钟玩转 Docker 容器技术》](https://weread.qq.com/web/reader/93d325a0719b200493d5ba9k6f4322302126f4922f45dec)

https://www.runoob.com/docker/docker-dockerfile.html

https://docs.docker.com/engine/reference/builder/

https://yeasy.gitbook.io/docker_practice/image/multistage-builds

