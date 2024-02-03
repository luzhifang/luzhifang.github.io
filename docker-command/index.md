# Docker 常用命令


## Dcoker 基本概念

Docker 包括三个基本概念：

- 镜像（Image）：Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。
- 容器（Container）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 示例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- 仓库（Repository）：仓库（Repository）类似 Git 的远程仓库，集中存放镜像文件。

三者之间的关系如下：

<img src="/images/docker/container-image-registry.png" alt="" width="90%" />

## 服务

查看 Docker 版本信息

```Shell
docker version
```

启动 Docker

```Shell
service docker start
```

重启 docker 服务

```Shell
service docker restart
```

关闭 docker 服务

```Shell
service docker stop
```

查看 Docker 信息

```Shell
docker info
```

查看命令帮助

```Shell
docker help
docker help run
```

## 容器

### 容器生命周期管理

#### docker run

创建一个新的容器并运行一个命令，语法 `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

OPTIONS 说明：

- -a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
- -d: 后台运行容器，并返回容器 ID；
- -i: 以交互模式运行容器，通常与 -t 同时使用；
- -P: 随机端口映射，容器内部端口随机映射到主机的端口
- -p: 指定端口映射，格式为：主机(宿主)端口:容器端口
- -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
- --name="nginx-lb": 为容器指定一个名称；
- --dns 8.8.8.8: 指定容器使用的 DNS 服务器，默认和宿主一致；
- --dns-search example.com: 指定容器 DNS 搜索域名，默认和宿主一致；
- -h "mars": 指定容器的 hostname；
- -e username="ritchie": 设置环境变量；
- --env-file=[]: 从指定文件读入环境变量；
- --cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定 CPU 运行；
- -m :设置容器使用内存最大值；
- --net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
- --link=[]: 添加链接到另一个容器；
- --expose=[]: 开放一个端口或一组端口；
- --volume , -v: 绑定一个卷

**示例**

使用 docker 镜像 nginx:latest 以后台模式启动一个容器,并将容器命名为 mynginx。

```Shell
docker run --name mynginx -d nginx:latest
```

使用镜像 nginx:latest 以后台模式启动一个容器,并将容器的 80 端口映射到主机随机端口。

```Shell
docker run -P -d nginx:latest
```

使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。

```Shell
docker run -p 80:80 -v /data:/data -d nginx:latest
```

绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。

```Shell
$ docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
```

使用镜像 nginx:latest 以交互模式启动一个容器,在容器内执行/bin/bash 命令。

```Shell
ubuntu@ubuntu:~$ docker run -it nginx:latest /bin/bash
root@b8573233d675:/#
```

#### docker start

启动一个或多个已经被停止的容器，语法`docker start [OPTIONS] CONTAINER [CONTAINER...]`

#### docker stop

停止一个运行中的容器，语法`docker stop [OPTIONS] CONTAINER [CONTAINER...]`

#### docker restart

重启容器，语法`docker restart [OPTIONS] CONTAINER [CONTAINER...]`

#### docker kill

杀掉一个运行中的容器，语法`docker kill [OPTIONS] CONTAINER [CONTAINER...]`

OPTIONS 说明：

- -s :向容器发送一个信号

**示例**

杀掉运行中的容器 mynginx

```Shell
ubuntu@ubuntu:~$ docker kill -s KILL mynginx
mynginx
```

#### docker rm

删除一个或多个容器，语法`docker rm [OPTIONS] CONTAINER [CONTAINER...]`

OPTIONS 说明：

- -f :通过 SIGKILL 信号强制删除一个运行中的容器。
- -l :移除容器间的网络连接，而非容器本身。
- -v :删除与容器关联的卷。

**示例**

强制删除容器 db01、db02：

```Shell
docker rm -f db01 db02
```

#### docker pause

暂停容器中所有的进程，语法`docker pause CONTAINER [CONTAINER...]`

#### docker unpause

恢复容器中所有的进程，语法`docker unpause CONTAINER [CONTAINER...]`

**示例**

暂停数据库容器 db01 提供服务。

```Shell
docker pause db01
```

恢复数据库容器 db01 提供服务。

```Shell
docker unpause db01
```

#### docker create

创建一个新的容器但不启动它，用法同 docker run

语法`docker create [OPTIONS] IMAGE [COMMAND] [ARG...]`，语法同 docker run

**示例**

使用 docker 镜像 nginx:latest 创建一个容器,并将容器命名为 myubuntu

```Shell
ubuntu@ubuntu:~$ docker create --name myubuntu nginx:latest
09b93464c2f75b7b69f83d56a9cfc23ceb50a48a9db7652ee4c27e3e2cb1961f
```

#### docker exec

在运行的容器中执行命令，语法`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`

OPTIONS 说明：

- -d :分离模式: 在后台运行
- -i :即使没有附加也保持 STDIN 打开
- -t :分配一个伪终端

**示例**

在容器 mynginx 中以交互模式执行容器内 /root/ubuntu.sh 脚本:

```Shell
ubuntu@ubuntu:~$ docker exec -it mynginx /bin/sh /root/ubuntu.sh
http://www.ubuntu.com/
```

在容器 mynginx 中开启一个交互模式的终端:

```Shell
ubuntu@ubuntu:~$ docker exec -i -t  mynginx /bin/bash
root@b1a0703e41e7:/#
```

### 容器操作

#### docker ps

列出容器，语法`docker ps [OPTIONS]`

OPTIONS 说明：

- -a :显示所有的容器，包括未运行的。
- -f :根据条件过滤显示的内容。
- --format :指定返回值的模板文件。
- -l :显示最近创建的容器。
- -n :列出最近创建的 n 个容器。
- --no-trunc :不截断输出。
- -q :静默模式，只显示容器编号。
- -s :显示总的文件大小。

**示例**

列出所有在运行的容器信息。

```Shell
ubuntu@ubuntu:~$ docker ps
CONTAINER ID IMAGE COMMAND ... PORTS NAMES
09b93464c2f7 nginx:latest "nginx -g 'daemon off" ... 80/tcp, 443/tcp myubuntu
96f7f14e99ab mysql:5.6 "docker-entrypoint.sh" ... 0.0.0.0:3306->3306/tcp mymysql
```

输出详情介绍：

- CONTAINER ID: 容器 ID。
- IMAGE: 使用的镜像。
- COMMAND: 启动容器时运行的命令。
- CREATED: 容器的创建时间。
- STATUS: 容器状态。状态有 7 种：
  - created（已创建）
  - restarting（重启中）
  - running（运行中）
  - removing（迁移中）
  - paused（暂停）
  - exited（停止）
  - dead（死亡）
- PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。
- NAMES: 自动分配的容器名称。

#### docker inspect

获取容器/镜像的元数据，语法`docker inspect [OPTIONS] NAME|ID [NAME|ID...]`

OPTIONS 说明：

- -f :指定返回值的模板文件。
- -s :显示总的文件大小。
- --type :为指定类型返回 JSON。

#### docker top

查看容器中运行的进程信息，支持 ps 命令参数，语法`docker top [OPTIONS] CONTAINER [ps OPTIONS]`

容器运行时不一定有/bin/bash 终端来交互执行 top 命令，而且容器还不一定有 top 命令，可以使用 docker top 来实现查看 container 中正在运行的进程。

#### docker attach

连接到正在运行中的容器，语法`docker attach [OPTIONS] CONTAINER`

要 attach 上去的容器必须正在运行，可以同时连接上同一个 container 来共享屏幕（与 screen 命令的 attach 类似）。

#### docker events

从服务器获取实时事件，语法`docker events [OPTIONS]`

OPTIONS 说明：

- -f ：根据条件过滤事件；
- --since ：从指定的时间戳后显示所有事件;
- --until ：流水时间显示到指定的时间为止；

#### docker logs

获取容器的日志，语法`docker logs [OPTIONS] CONTAINER`

OPTIONS 说明：

- -f : 跟踪日志输出
- --since :显示某个开始时间的所有日志
- -t : 显示时间戳
- --tail :仅列出最新 N 条容器日志

#### docker wait

阻塞运行直到容器停止，然后打印出它的退出代码，语法`docker wait [OPTIONS] CONTAINER [CONTAINER...]`

#### docker export

将文件系统作为一个 tar 归档文件导出到 STDOUT，语法`docker export [OPTIONS] CONTAINER`

OPTIONS 说明：

- -o :将输入内容写到文件。

**示例**

将 id 为 a404c6c174a2 的容器按日期保存为 tar 文件。

```Shell
ubuntu@ubuntu:~$ docker export -o mysql-`date +%Y%m%d`.tar a404c6c174a2
ubuntu@ubuntu:~$ ls mysql-`date +%Y%m%d`.tar
mysql-20160711.tar
```

#### docker port

用于列出指定的容器的端口映射，或者查找将 PRIVATE_PORT NAT 到面向公众的端口。

语法`docker port [OPTIONS] CONTAINER [PRIVATE_PORT[/PROTO]]`

#### docker stats

显示容器资源的使用情况，包括：CPU、内存、网络 I/O 等。

语法`docker stats [OPTIONS] [CONTAINER...]`

OPTIONS 说明：

- --all , -a :显示所有的容器，包括未运行的。
- --format :指定返回值的模板文件。
- --no-stream :展示当前状态就直接退出了，不再实时更新。
- --no-trunc :不截断输出。

**示例**

列出所有在运行的容器信息。

```Shell
ubuntu@ubuntu:~$  docker stats
CONTAINER ID        NAME                                    CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
b95a83497c91        awesome_brattain                        0.28%               5.629MiB / 1.952GiB   0.28%               916B / 0B           147kB / 0B          9
67b2525d8ad1        foobar                                  0.00%               1.727MiB / 1.952GiB   0.09%               2.48kB / 0B         4.11MB / 0B         2
```

输出详情介绍：

- CONTAINER ID 与 NAME: 容器 ID 与名称。
- CPU % 与 MEM %: 容器使用的 CPU 和内存的百分比。
- MEM USAGE / LIMIT: 容器正在使用的总内存，以及允许使用的内存总量。
- NET I/O: 容器通过其网络接口发送和接收的数据量。
- BLOCK I/O: 容器从主机上的块设备读取和写入的数据量。
- PIDs: 容器创建的进程或线程数。

以 JSON 格式输出：

```Shell
ubuntu@ubuntu:~$ docker stats nginx --no-stream --format "{{ json . }}"
  {"BlockIO":"0B / 13.3kB","CPUPerc":"0.03%","Container":"nginx","ID":"ed37317fbf42","MemPerc":"0.24%","MemUsage":"2.352MiB / 982.5MiB","Name":"nginx","NetIO":"539kB / 606kB","PIDs":"2"}
```

### 容器 rootfs 命令

#### docker commit

从容器创建一个新的镜像，语法`docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`

OPTIONS 说明：

- -a :提交的镜像作者；
- -c :使用 Dockerfile 指令来创建镜像；
- -m :提交时的说明文字；
- -p :在 commit 时，将容器暂停。

**示例**

将容器 a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。

```Shell
ubuntu@ubuntu:~$ docker commit -a "ubuntu.com" -m "my apache" a404c6c174a2  mymysql:v1
sha256:37af1236adef1544e8886be23010b66577647a40bc02c0885a6600b33ee28057
ubuntu@ubuntu:~$ docker images mymysql:v1
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mymysql             v1                  37af1236adef        15 seconds ago      329 MB
```

#### docker cp

用于容器与主机之间的数据拷贝。

**语法**

`docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-`

`docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH`

OPTIONS 说明：

- -L :保持源目标中的链接

**示例**

将主机/www/ubuntu 目录拷贝到容器 96f7f14e99ab 的/www 目录下。

```Shell
docker cp /www/ubuntu 96f7f14e99ab:/www/
```

将主机/www/ubuntu 目录拷贝到容器 96f7f14e99ab 中，目录重命名为 www。

```Shell
docker cp /www/ubuntu 96f7f14e99ab:/www
```

将容器 96f7f14e99ab 的/www 目录拷贝到主机的/tmp 目录中。

```Shell
docker cp  96f7f14e99ab:/www /tmp/
```

#### docker diff

检查容器里文件结构的更改，语法`docker diff [OPTIONS] CONTAINER`

## 镜像仓库

### docker login

登陆到一个 Docker 镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

语法`docker login [OPTIONS] [SERVER]`

OPTIONS 说明：

- -u :登陆的用户名
- -p :登陆的密码

**示例**

登陆到 Docker Hub

```Shell
docker login -u 用户名 -p 密码
# 登出 Docker Hub
# docker logout
```

### docker logout

登出一个 Docker 镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

语法`docker logout [OPTIONS] [SERVER]`

### docker pull

从镜像仓库中拉取或者更新指定镜像，语法`docker pull [OPTIONS] NAME[:TAG|@DIGEST]`

OPTIONS 说明：

- -a :拉取所有 tagged 镜像
- --disable-content-trust :忽略镜像的校验,默认开启

### docker push

将本地的镜像上传到镜像仓库,要先登陆到镜像仓库

语法`docker push [OPTIONS] NAME[:TAG]`

OPTIONS 说明：

- --disable-content-trust :忽略镜像的校验,默认开启

### docker search

从 Docker Hub 查找镜像，语法`docker search [OPTIONS] TERM`

OPTIONS 说明：

- --automated :只列出 automated build 类型的镜像；
- --no-trunc :显示完整的镜像描述；
- -f <过滤条件>:列出收藏数不小于指定值的镜像。

**示例**

从 Docker Hub 查找所有镜像名包含 java，并且收藏数大于 10 的镜像

```Shell
ubuntu@ubuntu:~$ docker search -f stars=10 java
NAME                  DESCRIPTION                           STARS   OFFICIAL   AUTOMATED
java                  Java is a concurrent, class-based...   1037    [OK]
anapsix/alpine-java   Oracle Java 8 (and 7) with GLIBC ...   115                [OK]
develar/java                                                 46                 [OK]
```

参数说明：

- NAME: 镜像仓库源的名称
- DESCRIPTION: 镜像的描述
- OFFICIAL: 是否 docker 官方发布
- stars: 类似 Github 里面的 star，表示点赞、喜欢的意思。
- AUTOMATED: 自动构建。

## 本地镜像管理

### docker images

列出本地镜像，语法`docker images [OPTIONS] [REPOSITORY[:TAG]]`

OPTIONS 说明：

- -a :列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；
- --digests :显示镜像的摘要信息；
- -f :显示满足条件的镜像；
- --format :指定返回值的模板文件；
- --no-trunc :显示完整的镜像信息；
- -q :只显示镜像 ID。

### docker rmi

删除本地一个或多个镜像，语法`docker rmi [OPTIONS] IMAGE [IMAGE...]`

OPTIONS 说明：

- -f :强制删除；
- --no-prune :不移除该镜像的过程镜像，默认移除；

### docker tag

标记本地镜像，将其归入某一仓库。

语法`docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]`

**示例**

将镜像 ubuntu:15.10 标记为 ubuntu/ubuntu:v3 镜像。

```Shell
root@ubuntu:~# docker tag ubuntu:15.10 ubuntu/ubuntu:v3
root@ubuntu:~# docker images   ubuntu/ubuntu:v3
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu/ubuntu       v3                  4e3b13c8a266        3 months ago        136.3 MB
```

### docker build

命令用于使用 Dockerfile 创建镜像。

语法`docker build [OPTIONS] PATH | URL | -`

OPTIONS 说明：

- --build-arg=[] :设置镜像创建时的变量；
- --cpu-shares :设置 cpu 使用权重；
- --cpu-period :限制 CPU CFS 周期；
- --cpu-quota :限制 CPU CFS 配额；
- --cpuset-cpus :指定使用的 CPU id；
- --cpuset-mems :指定使用的内存 id；
- --disable-content-trust :忽略校验，默认开启；
- -f :指定要使用的 Dockerfile 路径；
- --force-rm :设置镜像过程中删除中间容器；
- --isolation :使用容器隔离技术；
- --label=[] :设置镜像使用的元数据；
- -m :设置内存最大值；
- --memory-swap :设置 Swap 的最大值为内存+swap，"-1"表示不限 swap；
- --no-cache :创建镜像的过程不使用缓存；
- --pull :尝试去更新镜像的新版本；
- --quiet, -q :安静模式，成功后只输出镜像 ID；
- --rm :设置镜像成功后删除中间容器；
- --shm-size :设置/dev/shm 的大小，默认值是 64M；
- --ulimit :Ulimit 配置。
- --squash :将 Dockerfile 中所有的操作压缩为一层。
- --tag, -t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。
- --network: 默认 default。在构建期间设置 RUN 指令的网络模式

**示例**

使用当前目录的 Dockerfile 创建镜像，标签为 ubuntu/ubuntu:v1。

```Shell
docker build -t ubuntu/ubuntu:v1 .
```

使用 URL github.com/creack/docker-firefox 的 Dockerfile 创建镜像。

```Shell
docker build github.com/creack/docker-firefox
```

也可以通过 -f Dockerfile 文件的位置：

```Shell
$ docker build -f /path/to/a/Dockerfile .
```

### docker history

查看指定镜像的创建历史，语法`docker history [OPTIONS] IMAGE`

OPTIONS 说明：

- -H :以可读的格式打印镜像大小和日期，默认为 true；
- --no-trunc :显示完整的提交记录；
- -q :仅列出提交记录 ID。

### docker save

将指定镜像保存成 tar 归档文件，语法`docker save [OPTIONS] IMAGE [IMAGE...]`

OPTIONS 说明：

- -o :输出到的文件。

**示例**

将镜像 ubuntu/ubuntu:v3 生成 my_ubuntu_v3.tar 文档

```Shell
ubuntu@ubuntu:~$ docker save -o my_ubuntu_v3.tar ubuntu/ubuntu:v3
ubuntu@ubuntu:~$ ll my_ubuntu_v3.tar
-rw------- 1 ubuntu ubuntu 142102016 Jul 11 01:37 my_ubuntu_v3.ta
```

### docker load

导入使用 docker save 命令导出的镜像，语法`docker load [OPTIONS]`

OPTIONS 说明：

- --input , -i : 指定导入的文件，代替 STDIN。
- --quiet , -q : 精简输出信息。

**示例**

```Shell
$ docker load < busybox.tar.gz
$ docker load --input fedora.tar
```

### docker import

从归档文件中创建镜像，语法`docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]`

OPTIONS 说明：

- -c :应用 docker 指令创建镜像；
- -m :提交时的说明文字；

**示例**

从镜像归档文件 my_ubuntu_v3.tar 创建镜像，命名为 ubuntu/ubuntu:v4

```Shell
ubuntu@ubuntu:~$ docker import  my_ubuntu_v3.tar ubuntu/ubuntu:v4
sha256:63ce4a6d6bc3fabb95dbd6c561404a309b7bdfc4e21c1d59fe9fe4299cbfea39
ubuntu@ubuntu:~$ docker images ubuntu/ubuntu:v4
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu/ubuntu       v4                  63ce4a6d6bc3        20 seconds ago      142.1 MB
```

## 参考

https://cloud.tencent.com/developer/article/1772136

https://www.runoob.com/docker/docker-command-manual.html

