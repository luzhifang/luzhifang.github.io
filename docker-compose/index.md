# Docker Compose


## Compose 简介

Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

Compose 使用的三个步骤：

- 使用 Dockerfile 定义应用程序的环境。
- 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 最后，执行 docker-compose up 命令来启动并运行整个应用程序。

## Compose 安装

Linux 上我们可以从 Github 上下载它的二进制包来使用，最新发行的版本地址：https://github.com/docker/compose/releases。

运行以下命令

```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 将可执行权限应用于二进制文件：
sudo chmod +x /usr/local/bin/docker-compose
# 创建软连接
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose version
```

## 使用

创建 Go 项目

```
mkdir docker-compose-demo
cd docker-compose-demo
go mod init docker-compose-demo
go get -u github.com/go-redis/redis
```

编辑 main.go

```Go
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
		rdb := NewRedisClient()
		pong, err := rdb.Ping().Result()
		if err != nil {
			panic(err)
		}
		fmt.Fprintf(writer, pong)
	})
	err := http.ListenAndServe(":5000", nil)
	if err != nil {
		panic(err)
	}
}

func NewRedisClient() *redis.Client {
	rdb := redis.NewClient(&redis.Options{
		// ip 需要用 ifconfig 列出来的
		Addr:     "127.0.0.1:6379",
		Password: "",
		DB:       0,
		PoolSize: 100,
	})
	return rdb
}
```

编辑 Dockerfile

```
#源镜像
FROM alpine:latest
#设置工作目录
WORKDIR /
#将服务器的go工程代码加入到docker容器中
ADD docker-compose-demo /docker-compose-demo
#暴露端口
EXPOSE 5000
#最终运行docker的命令
ENTRYPOINT  ["/docker-compose-demo"]
```

编辑 docker-compose.yml

```
# yaml 配置
version: '3'
services:
  web:
    build: .
    depends_on:
      - redis
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
    ports:
      - "6379:6379"
networks:
  some-network:
    # Use a custom driver
    driver: custom-driver-1
  other-network:
    # Use a custom driver which takes special options
    driver: custom-driver-2
```

运行以下命令

```Shell
# 如果docker基础镜像不支持cgo的话，需要加上CGO_ENABLED=0
CGO_ENABLED=0 go build -o docker-compose-demo main.go
docker-compose up
```

docker-compose 常用命令

```Shell
# 后台运行
docker-compose up -d
# 列出项目中所有容器
docker-compose ps
# 停止和删除容器和网络，会清楚数据
docker-compose down
```

## yml 配置解析

**version**

指定本 yml 依从的 compose 哪个版本制定的。

**services**

配置服务。

**build**

指定为构建镜像上下文路径：

**cap_add，cap_drop**

添加或删除容器拥有的宿主机的内核功能。

**cgroup_parent**

为容器指定父 cgroup 组，意味着将继承该组的资源限制。

**command**

覆盖容器启动的默认命令。

**container_name**

指定自定义容器名称，而不是生成的默认名称。

**depends_on**

设置依赖关系。

**image**

指定容器运行的镜像

**expose**

暴露端口，但不映射到宿主机，只被连接的服务访问。

**volumes**

将主机的数据卷或着文件挂载到容器里。

**networks**

配置容器连接的网络，引用顶级 networks 下的条目 。

## 参考

https://www.runoob.com/docker/docker-compose.html

