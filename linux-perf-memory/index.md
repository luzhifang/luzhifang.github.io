# Linux性能优化-内存


## 内存性能指标

<img src="/images/linux/memory-indicator-mind.png" alt="" width="90%" />

### 系统内存使用情况

- 已用内存和剩余内存很容易理解，就是已经使用和还未使用的内存。
- 共享内存是通过 tmpfs 实现的，所以它的大小也就是 tmpfs 使用的内存大小。tmpfs 其实也是一种特殊的缓存。
- 可用内存是新进程可以使用的最大内存，它包括剩余内存和可回收缓存。
- 缓存包括两部分，一部分是磁盘读取文件的页缓存，用来缓存从磁盘读取的数据，可以加快以后再次访问的速度。另一部分，则是 Slab 分配器中的可回收内存。
- 缓冲区是对原始磁盘块的临时存储，用来缓存将要写入磁盘的数据。这样，内核就可以把分散的写集中起来，统一优化磁盘写入。

### 进程内存使用情况

- 虚拟内存，包括了进程代码段、数据段、共享内存、已经申请的堆内存和已经换出的内存等。这里要注意，已经申请的内存，即使还没有分配物理内存，也算作虚拟内存。
- 常驻内存是进程实际使用的物理内存，不过，它不包括 Swap 和共享内存。
- 共享内存，既包括与其他进程共同使用的真实的共享内存，还包括了加载的动态链接库以及程序的代码段等。
- Swap 内存，是指通过 Swap 换出到磁盘的内存。

### 缺页异常

- 可以直接从物理内存中分配时，被称为次缺页异常。
- 需要磁盘 I/O 介入（比如 Swap）时，被称为主缺页异常。

### Swap 的使用情况

- 已用空间和剩余空间很好理解，就是字面上的意思，已经使用和没有使用的内存空间。
- 换入和换出速度，则表示每秒钟换入和换出内存的大小。

## 内存性能工具

<img src="/images/linux/memory-indicator-to-tool.png" alt="" width="90%" />

**为了迅速定位内存问题，我通常会先运行几个覆盖面比较大的性能工具，比如 free、top、vmstat、pidstat 等。**

具体的分析思路主要有这几步。

1. 先用 free 和 top，查看系统整体的内存使用情况。
2. 再用 vmstat 和 pidstat，查看一段时间的趋势，从而判断出内存问题的类型。
3. 最后进行详细分析，比如内存分配分析、缓存 / 缓冲区分析、具体进程的内存使用分析等。

<img src="/images/linux/memory-perf-steps.png" alt="" width="100%" />

## 查看内存使用情况

```Shell
$ free -w
              total        used        free      shared     buffers       cache   available
Mem:        8113464      214652     7745064          64       14180      139568     7689660
Swap:       2097152           0     2097152

# total 是总内存大小；
# used 是已使用内存的大小，包含了共享内存；
# free 是未使用内存的大小；
# shared 是共享内存的大小；
# buffers 是对磁盘数据的缓存，会用在读和写请求上
# cache 是文件数据的缓存，会用在读和写请求上
# available 是新进程可用内存的大小。

$ top
top - 17:13:55 up 29 min,  0 users,  load average: 0.01, 0.00, 0.00
Tasks:   9 total,   1 running,   8 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.1 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  8113464 total,  7745304 free,   213904 used,   154256 buff/cache
KiB Swap:  2097152 total,  2097152 free,        0 used.  7690156 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
   40 root      20   0  836056 140424  34740 S   0.3  1.7   0:27.98 hugo

# VIRT 是进程虚拟内存的大小，只要是进程申请过的内存，即便还没有真正分配物理内存，也会计算在内。
# RES 是常驻内存的大小，也就是进程实际使用的物理内存大小，但不包括 Swap 和共享内存。
# SHR 是共享内存的大小，比如与其他进程共同使用的共享内存、加载的动态链接库以及程序的代码段等。
# %MEM 是进程使用物理内存占系统总内存的百分比。
```

## 缓存命中率

Buffer 和 Cache 的设计目的，是为了提升系统的 I/O 性能。它们利用内存，充当起慢速磁盘与快速 CPU 之间的桥梁，可以加速 I/O 的访问速度。

Buffer 和 Cache 分别缓存的是对磁盘和文件系统的读写数据。

缓存命中率是指直接通过缓存获取数据的请求次数，占所有请求次数的百分比。命中率越高说明缓存带来的收益越高，应用程序的性能也就越好。

安装 bcc 包后可以通过 cachestat 和 cachetop 来监测缓存的读写命中情况。

```Shell
# 可以把bcc工具加到环境变量中，export PATH=$PATH:/usr/share/bcc/tools
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4052245BD4284CDD
echo "deb https://repo.iovisor.org/apt/xenial xenial main" | sudo tee /etc/apt/sources.list.d/iovisor.list
sudo apt-get update
sudo apt-get install -y bcc-tools libbcc-examples linux-headers-$(uname -r)

# cachestat 提供了整个操作系统缓存的读写命中情况。
# TOTAL ，表示总的 I/O 次数；
# MISSES ，表示缓存未命中的次数；
# HITS ，表示缓存命中的次数；
# DIRTIES， 表示新增到缓存中的脏页数；
# BUFFERS_MB 表示 Buffers 的大小，以 MB 为单位；
# CACHED_MB 表示 Cache 的大小，以 MB 为单位。
$ cachestat 1 3
   TOTAL   MISSES     HITS  DIRTIES   BUFFERS_MB  CACHED_MB
       2        0        2        1           17        279

# cachetop 提供了每个进程的缓存命中情况。
# READ_HIT 和 WRITE_HIT ，分别表示读和写的缓存命中率。
$ cachetop
11:58:50 Buffers MB: 258 / Cached MB: 347 / Sort: HITS / Order: ascending
PID      UID      CMD              HITS     MISSES   DIRTIES  READ_HIT%  WRITE_HIT%
   13029 root     python                  1        0        0     100.0%       0.0%

```

## 内存泄漏

虚拟内存分布从低到高分别是只读段，数据段，堆，内存映射段，栈五部分。其中会导致内存泄漏的是：

- 堆：由应用程序自己来分配和管理，除非程序退出这些堆内存不会被系统自动释放。
- 内存映射段：包括动态链接库和共享内存，其中共享内存由程序自动分配和管理

内存泄漏的危害比较大，这些忘记释放的内存，不仅应用程序自己不能访问，系统也不能把它们再次分配给其他应用。内存泄漏不断累积甚至会耗尽系统内存。

bcc 软件包中的 memleak 工具可以用来检测内存泄漏

```Shell
# 可以根据内存分配的调用栈，定位内存的分配位置，从而释放不再访问的内存。
$ /usr/share/bcc/tools/memleak -a -p $(pidof app)
```

避免内存泄漏，最重要的一点就是养成良好的编程习惯，比如分配内存后，一定要先写好内存释放的代码，再去开发其他逻辑。

## Swap

对于程序自动分配的堆内存，也就是我们在内存管理中的匿名页，虽然这些内存不能直接释放，但是 Linux 提供了 Swap 机制将不常访问的内存写入到磁盘来释放内存，再次访问时从磁盘读取到内存即可。

Swap 本质就是把一块磁盘空间或者一个本地文件当作内存来使用，包括换入和换出两个过程：

- 换出：将进程暂时不用的内存数据存储到磁盘中，并释放这些内存
- 换入：进程再次访问内存时，将它们从磁盘读到内存中

当 Swap 变高时，你可以用 sar、/proc/zoneinfo、/proc/pid/status 等方法，查看系统和进程的内存使用情况，进而找出 Swap 升高的根源和受影响的进程。

**Swap 优化：**

- 禁止 Swap，现在服务器的内存足够大，所以除非有必要，禁用 Swap 就可以了。随着云计算的普及，大部分云平台中的虚拟机都默认禁止 Swap。
- 如果实在需要用到 Swap，可以尝试降低 swappiness 的值，减少内存回收时 Swap 的使用倾向。
- 响应延迟敏感的应用，如果它们可能在开启 Swap 的服务器中运行，你还可以用库函数 mlock() 或者 mlockall() 锁定内存，阻止它们的内存换出。

## 内存优化思路

- 最好禁止 Swap。如果必须开启 Swap，降低 swappiness 的值，减少内存回收时 Swap 的使用倾向。
- 减少内存的动态分配。比如，可以使用内存池、大页（HugePage）等。
- 尽量使用缓存和缓冲区来访问数据。比如，可以使用堆栈明确声明内存空间，来存储需要缓存的数据；或者用 Redis 这类的外部缓存组件，优化数据的访问。
- 使用 cgroups 等方式限制进程的内存使用情况。这样，可以确保系统内存不会被异常进程耗尽。
- 通过 /proc/pid/oom_adj ，调整核心应用的 oom_score。这样，可以保证即使内存紧张，核心应用也不会被 OOM 杀死。

## 参考

https://www.eet-china.com/mp/a117035.html

[性能之巅-洞悉系统、企业与云计算](https://weread.qq.com/web/reader/6da3218071fd5aa16dabd75k8e232ec02198e296a067180)

```

```

