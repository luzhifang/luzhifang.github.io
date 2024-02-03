# Docker 实现原理


## 容器的实现原理

从本质上，容器其实就是一种沙盒技术。就好像把应用隔离在一个盒子内，使其运行。因为有了盒子边界的存在，应用于应用之间不会相互干扰。并且像集装箱一样，拿来就走，随处运行。

实现容器的核心，就是要生成限制应用运行时的边界。我们知道，编译后的可执行代码加上数据，叫做程序。而把程序运行起来后，就变成了进程，也就是所谓的应用。如果能在应用启动时，给其加上一个边界，这样不就能实现期待的沙盒吗？

在 Linux 中，实现容器的边界，主要有两种技术 Cgroups 和 Namespace。 Cgroups 用于对运行的容器进行资源的限制，Namespace 则会将容器隔离起来，实现边界。

这样看来，容器只是一种被限制的了特殊进程而已。

## Docker 容器

- 基于 Linux 内核的 Cgroup，Namespace，以及 Union FS 等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术，由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。
- 最初实现是基于 LXC，从 0.7 以后开始去除 LXC，转而使用自行开发的 Libcontainer，从 1.11 开始，则进一步演进为使用 runC 和 Containerd。
- Docker 在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护，使得 Docker 技术比虚拟机技术更为轻便、快捷。

## 为什么要用 Docker

- 更高效地利用系统资源
- 更快速的启动时间
- 一致的运行环境
- 持续交付和部署
- 更轻松地迁移
- 更轻松地维护和扩展

## 虚拟机和容器

<img src="/images/docker/docker-vm.png" alt="" width="90%" />

性能对比：

| 特性       | 容器               | 虚拟机     |
| ---------- | ------------------ | ---------- |
| 启动速度   | 秒级               | 分钟级     |
| 硬盘使用   | 一般为 MB          | 一般为 GB  |
| 性能       | 接近原生           | 弱于原生   |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |

## Namespace

- Linux Namespace 是一种 Linux Kernel 提供的资源隔离方案：
- 系统可以为进程分配不同的 Namespace；
- 并保证不同的 Namespace 资源独立分配、进程彼此隔离，即不同的 Namespace 下的进程互不干扰 。

### Pid namespace

- 不同用户的进程就是通过 Pid namespace 隔离开的，且不同 namespace 中可以有相同 Pid。
- 有了 Pid namespace, 每个 namespace 中的 Pid 能够相互隔离。

### net namespace

- 网络隔离是通过 net namespace 实现的， 每个 net namespace 有独立的 network devices, IP addresses, IP routing tables, /proc/net 目录。
- Docker 默认采用 veth 的方式将 container 中的虚拟网卡同 host 上的一个 docker bridge: docker0 连接在一起。

### ipc namespace

- Container 中进程交互还是采用 linux 常见的进程间交互方法 （interprocess communication – IPC）, 包括常见的信号量、消息队列和共享内存。
- container 的进程间交互实际上还是 host 上 具有相同 Pid namespace 中的进程间交互，因此需要在 IPC 资源申请时加入 namespace 信息 - 每个 IPC 资源有一个唯一的 32 位 ID。

### mnt namespace

mnt namespace 允许不同 namespace 的进程看到的文件结构不同，这样每个 namespace 中的进程所看到的文件目录就被隔离开了。

### uts namespace

UTS(“UNIX Time-sharing System”) namespace 允许每个 container 拥有独立的 hostname 和 domain name, 使其在网络上可以被视作一个独立的节点而非 Host 上的一个进程。

### user namespace

每个 container 可以有不同的 user 和 group id, 也就是说可以在 container 内部用 container 内部的用户执行程序而非 Host 上的用户。

## Cgroups

- Cgroups （Control Groups）是 Linux 下用于对一个或一组进程进行资源控制和监控的机制；
- 可以对诸如 CPU 使用时间、内存、磁盘 I/O 等进程所需的资源进行限制；
- 不同资源的具体管理工作由相应的 Cgroup 子系统（Subsystem）来实现 ；
- 针对不同类型的资源限制，只要将限制策略在不同的的子系统上进行关联即可 ；
- Cgroups 在不同的系统资源管理子系统中以层级树（Hierarchy）的方式来组织管理：每个 Cgroup 都可以包含其他的子 Cgroup，因此子 Cgroup 能使用的资源除了受本 Cgroup 配置的资源参数限制，还受到父 Cgroup 设置的资源限制 。

## Union FS

Union Fs 有以下特性：

- 将不同目录挂载到同一个虚拟文件系统下 （unite several directories into a single virtual filesystem）的文件系统
- 支持为每一个成员目录（类似 Git Branch）设定 readonly、readwrite 和 whiteout-able 权限
- 文件系统分层, 对 readonly 权限的 branch 可以逻辑上进行修改(增量地, 不影响 readonly 部分的)。
- 通常 Union FS 有两个用途, 一方面可以将多个 disk 挂到同一个目录下, 另一个更常用的就是将一个 readonly 的 branch 和一个 writeable 的 branch 联合在一起。

Linux 的命名空间和控制组分别解决了不同资源隔离的问题，前者解决了进程、网络以及文件系统的隔离，后者实现了 CPU、内存等资源的隔离，但是在 Docker 中还有另一个非常重要的问题需要解决 - 也就是镜像。

镜像实现就是依靠的 Union Fs。

Docker 镜像其实本质就是一个压缩包，我们可以使用下面的命令将一个 Docker 镜像中的文件导出：

```Shell
$ docker export $(docker create busybox) | tar -C rootfs -xvf -
$ ls
bin  dev  etc  home proc root sys  tmp  usr  var
```

可以看到这个 busybox 镜像中的目录结构与 Linux 操作系统的根目录中的内容并没有太多的区别，可以说 Docker 镜像就是一个文件。

Docker 中的每一个镜像都是由一系列只读的层组成的，Dockerfile 中的每一个命令都会在已有的只读层上创建一个新的层：

```Docker
FROM ubuntu:15.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

容器中的每一层都只对当前容器进行了非常小的修改，上述的 Dockerfile 文件会构建一个拥有四层 layer 的镜像：

<img src="/images/docker/docker-container-layer.png" alt="" width="90%" />

当镜像被 docker run 命令创建时就会在镜像的最上层添加一个可写的层，也就是容器层，所有对于运行时容器的修改其实都是对这个容器读写层的修改。

容器和镜像的区别就在于，所有的镜像都是只读的，而每一个容器其实等于镜像加上一个可读写的层，也就是同一个镜像可以对应多个容器。

<img src="/images/docker/docker-images-and-container.png" alt="" width="90%" />

## 参考

https://www.cnblogs.com/michael9/p/13039700.html

https://draveness.me/docker/

https://blog.csdn.net/crazymakercircle/article/details/120747767
