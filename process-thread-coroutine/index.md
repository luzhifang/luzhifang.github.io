# 进程线程协程的区别


## 进程与线程的区别

每个线程都是一个轻量级进程(Light Weight Process)，都有自己的唯一 PID 和一个 TGID(Thread group ID)。TGID 是启动整个进程的 thread 的 PID。

例如，当一个进程被创建的时候，它其实是一个 PID 和 TGID 数值相同线程。当线程 A 启动线程 B 时，线程 B 会有自己的唯一 PID，但它的 TGID 会从 A 继承而来。这样通过 PID 线程可以独立得到调度，而相同的 TGID 可以知道哪些线程属于同一个进程，这样可以共享资源(RAM，虚拟内存、文件等)。

从内核层面来看线程其实也是一种特殊的进程，它跟父进程共享了打开了的文件和文件系统信息，共享了地址空间和信号处理函数。

线程进程的区别主要体现在以下几个方面：

1. 根本区别：进程是操作系统资源分配的基本单位，而线程是处理器任务调度和执行的基本单位。
2. 资源开销：每个进程都有独立的代码和数据空间，程序之间的切换会有较大的开销；线程可以看做轻量级的进程，同一进程的线程共享代码和数据空间，每个线程都有自己独立的运行栈和程序计数器，线程之间切换的开销小。
3. 包含关系：如果一个进程内有多个线程，则执行过程不是一条线的，而是多条线（线程）共同完成的。
4. 内存分配：同一进程的线程共享本进程的地址空间和资源，而进程之间的地址空间和资源是相互独立的。
5. 影响关系：一个进程崩溃后，在保护模式下不会对其他进程产生影响，但是一个线程崩溃整个进程都死掉。所以多进程要比多线程健壮。
6. 执行过程：每个独立的进程有程序运行的入口、顺序执行序列和程序出口。但是线程不能独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。两者均可并发执行。
7. 上下文切换：线程切换依然需要内核进行进程切换，线程切换相比进程切换，主要节省了虚拟地址空间的切换。
8. 通信方面：进程主要通过管道、套接字、消息队列、信号量、信号、共享内存进行通信，线程主要通过锁、信号、信号量进行通信。

## 协程与线程的区别

1. 根本区别：协程并不是取代线程，而且抽象于线程之上。线程是被分割的 CPU 资源, 协程是组织好的代码流程, 协程需要线程来承载运行。
2. 资源开销：协程需要用到的寄存器更少，相比线程，协程占用的内存空间更小。
3. 包含关系：一个线程可以有多个协程。
4. 同步异步：大多数业务场景下，线程进程可以看做是同步机制，而协程则是异步。
5. 调度：线程是抢占式，而协程是非抢占式的，所以需要用户代码释放使用权来切换到其他协程，因此同一时间其实只有一个协程拥有运行权，相当于单线程的能力。
6. 上下文切换：协程切换不需要陷入内核。

## 进程、线程和协程的区别和联系

|          | 进程                                                                              | 线程                                               | 协程                                                                                 |
| -------- | --------------------------------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------ |
| 定义     | 资源分配和拥有的基本单位                                                          | 程序执行的基本单位                                 | 用户态的轻量级线程，线程内部调度的基本单位                                           |
| 切换情况 | 进程 CPU 环境 (栈、寄存器、页表和文件句柄等)的保存以及新调度的进程 CPU 环境的设置 | 保存和设置程序计数器、少量寄存器和栈的内容         | 先将寄存器上下文和栈保存，等切换回来的时候再进行恢复                                 |
| 切换者   | 操作系统                                                                          | 操作系统                                           | 用户                                                                                 |
| 切换过程 | 用户态->内核态->用户态                                                            | 用户态->内核态->用户态                             | 用户态(没有陷入内核)                                                                 |
| 调用栈   | 内核栈                                                                            | 内核栈                                             | 用户栈                                                                               |
| 拥有资源 | CPU 资源、内存资源、文件资源和句柄等                                              | 程序计数器、寄存器、栈和状态字                     | 拥有自己的寄存器上下文和栈                                                           |
| 并发性   | 不同进程之间切换实现并发，各自占有 CPU 实现并行                                   | 一个进程内部的多个线程并发执行                     | 同一时间只能执行一个协程，而其他协程处于休眠状态，适合对任务进行分时处理             |
| 系统开销 | 切换虚拟地址空间，切换内核栈和硬件上下文，CPU 高速缓存失效、页表切换，开销很大    | 切换时只需保存和设置少量寄存器内容，因此开销很小   | 直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快 |
| 通信方面 | 进程间通信需要借助操作系统                                                        | 线程间可以直接读写进程数据段(如全局变量)来进行通信 | 共享内存、消息队列                                                                   |

1、进程是资源调度的基本单位，运行一个可执行程序会创建一个或多个进程，进程就是运行起来的可执行程序

2、线程是程序执行的基本单位，是轻量级的进程。每个进程中都有唯一的主线程，且只能有一个，主线程和进程是相互依存的关系，主线程结束进程也会结束。多提一句：协程是用户态的轻量级线程，线程内部调度的基本单位

