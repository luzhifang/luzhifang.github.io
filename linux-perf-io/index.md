# Linux性能优化-IO


## 磁盘性能指标

说到磁盘性能的衡量标准，必须要提到五个常见指标，也就是我们经常用到的，使用率、饱和度、IOPS、吞吐量以及响应时间等。这五个指标，是衡量磁盘性能的基本指标。

- 使用率，是指磁盘处理 I/O 的时间百分比。过高的使用率（比如超过 80%），通常意味着磁盘 I/O 存在性能瓶颈。
- 饱和度，是指磁盘处理 I/O 的繁忙程度。过高的饱和度，意味着磁盘存在严重的性能瓶颈。当饱和度为 100% 时，磁盘无法接受新的 I/O 请求。
- IOPS（Input/Output Per Second），是指每秒的 I/O 请求数。
- 吞吐量，是指每秒的 I/O 请求大小。
- 响应时间，是指 I/O 请求从发出到收到响应的间隔时间。

## 性能工具

<img src="/images/linux/io-indicator-to-tool.png" alt="" width="90%" />

## I/O 快速分析步骤

从 I/O 延迟的案例到 MySQL 和 Redis 的案例，就会发现，虽然这些问题千差万别，但从 I/O 角度来分析，最开始的分析思路基本上类似，都是：

1. 先用 iostat 发现磁盘 I/O 性能瓶颈；
2. 再借助 pidstat ，定位出导致瓶颈的进程；
3. 随后分析进程的 I/O 行为（比如用 strace 和 lsof，`strace -p 12280 2>&1 | grep write` `lsof -p 12280`）；
4. 最后，结合应用程序的原理，分析这些 I/O 的来源。

为了缩小排查范围，我通常会先运行那几个支持指标较多的工具，如 top、iostat、vmstat、pidstat 等。

<img src="/images/linux/io-perf-steps.png" alt="" width="90%" />

## I/O 性能优化

### 应用程序优化

- 可以用追加写代替随机写，减少寻址开销，加快 I/O 写的速度。
- 可以借助缓存 I/O ，充分利用系统缓存，降低实际 I/O 的次数。
- 可以在应用程序内部构建自己的缓存，或者用 Redis 这类外部缓存系统。这样，一方面，能在应用程序内部，控制缓存的数据和生命周期；另一方面，也能降低其他应用程序使用缓存对自身的影响。
- 在需要频繁读写同一块磁盘空间时，可以用 mmap 代替 read/write，减少内存的拷贝次数。
- 在需要同步写的场景中，尽量将写请求合并，而不是让每个请求都同步写入磁盘，即可以用 fsync() 取代 O_SYNC。
- 在多个应用程序共享相同磁盘时，为了保证 I/O 不被某个应用完全占用，推荐你使用 cgroups 的 I/O 子系统，来限制进程 / 进程组的 IOPS 以及吞吐量。

### 文件系统优化

- 可以根据实际负载场景的不同，选择最适合的文件系统。比如 Ubuntu 默认使用 ext4 文件系统，而 CentOS 7 默认使用 xfs 文件系统。
- 在选好文件系统后，还可以进一步优化文件系统的配置选项，包括文件系统的特性（如 ext_attr、dir_index）、日志模式（如 journal、ordered、writeback）、挂载选项（如 noatime）等等。
- 可以优化文件系统的缓存。

### 磁盘优化

- 最简单有效的优化方法，就是换用性能更好的磁盘，比如用 SSD 替代 HDD。
- 可以使用 RAID ，把多块磁盘组合成一个逻辑磁盘，构成冗余独立磁盘阵列。这样做既可以提高数据的可靠性，又可以提升数据的访问性能。
- 针对磁盘和应用程序 I/O 模式的特征，我们可以选择最适合的 I/O 调度算法。比方说，SSD 和虚拟机中的磁盘，通常用的是 noop 调度算法。而数据库应用，我更推荐使用 deadline 算法。
- 可以对应用程序的数据，进行磁盘级别的隔离。比如，我们可以为日志、数据库等 I/O 压力比较重的应用，配置单独的磁盘。
- 在顺序读比较多的场景中，我们可以增大磁盘的预读数据，比如，你可以通过下面两种方法，调整 /dev/sdb 的预读大小。
- 可以优化内核块设备 I/O 的选项。比如，可以调整磁盘队列的长度 /sys/block/sdb/queue/nr_requests，适当增大队列长度，可以提升磁盘的吞吐量（当然也会导致 I/O 延迟增大）。
- 最后，磁盘本身出现硬件错误，也会导致 I/O 性能急剧下降，所以发现磁盘性能急剧下降时，你还需要确认，磁盘本身是不是出现了硬件错误。

## 磁盘容量

对文件系统来说，最常见的一个问题就是空间不足。当然，你可能本身就知道，用 df 命令，就能查看文件系统的磁盘空间使用情况。比如：

```Shell
$ df /dev/sda1
Filesystem 1K-blocks Used Available Use% Mounted on
/dev/sda1 30308240 3167020 27124836 11% /
```

## 磁盘 I/O 观测

iostat 是最常用的磁盘 I/O 性能观测工具，它提供了每个磁盘的使用率、IOPS、吞吐量等各种常见的性能指标，当然，这些指标实际上来自 /proc/diskstats。

iostat 的输出界面如下：

```Shell
# -d 表示显示 I/O 性能指标，-x 表示显示扩展统计（即所有 I/O 指标）
$ iostat -d -x 1

Device            r/s     w/s     rkB/s     wkB/s   rrqm/s   wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util
sda              0.01    1.02      0.03    510.55     0.00     0.02   0.00   1.99    7.56    1.09   0.00     3.95   500.45   2.74   0.28
sdb              1.39    0.12     33.91      7.73     0.50     0.39  26.29  76.99    6.65   41.15   0.01    24.44    65.65   5.90   0.89
# %util ，就是我们前面提到的磁盘 I/O 使用率；
# r/s+ w/s ，就是 IOPS；
# rkB/s+wkB/s ，就是吞吐量；
# r_await+w_await ，就是响应时间。
```

<img src="/images/linux/iostat.png" alt="" width="90%" />

## 进程 I/O 观测

观察进程的 I/O 情况，可以使用 pidstat 和 iotop 这两个工具。

```Shell
# -d 显示IO数据，-t显示线程，不加-t默认显示各进程
$ pidstat -td 1

13:39:51      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
13:39:52      102       916      0.00      4.00      0.00       0  rsyslogd
```

iotop 是一个类似于 top 的工具，你可以按照 I/O 大小对进程排序，然后找到 I/O 较大的那些进程。

```Shell
$ iotop
Total DISK READ :       0.00 B/s | Total DISK WRITE :       7.85 K/s
Actual DISK READ:       0.00 B/s | Actual DISK WRITE:       0.00 B/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
15055 be/3 root        0.00 B/s    7.85 K/s  0.00 %  0.00 % systemd-journald
```

## 参考

https://www.eet-china.com/mp/a117035.html

[性能之巅-洞悉系统、企业与云计算](https://weread.qq.com/web/reader/6da3218071fd5aa16dabd75k8e232ec02198e296a067180)

