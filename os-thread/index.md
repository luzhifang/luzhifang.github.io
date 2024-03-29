# 线程


## 线程简介

一个程序只有一个执行点（一个程序计数器，用来存放要执行的指令），但多线程（multi-threaded）程序会有多个执行点（多个程序计数器，每个都用于取指令和执行）。

换一个角度来看，每个线程类似于独立的进程，只有一点区别：它们共享地址空间，从而能够访问相同的数据。

单个线程的状态与进程状态非常类似。线程有一个程序计数器（PC），记录程序从哪里获取指令。每个线程有自己的一组用于计算的寄存器。

如果有两个线程运行在一个处理器上，从运行一个线程（T1）切换到另一个线程（T2）时，必定发生上下文切换（context switch）。线程之间的上下文切换类似于进程间的上下文切换。对于进程，我们将状态保存到进程控制块（Process Control Block，PCB）。现在，我们需要一个或多个线程控制块（Thread Control Block，TCB），保存每个线程的状态。但是，与进程相比，线程之间的上下文切换有一点主要区别：地址空间保持不变（即不需要切换当前使用的页表）。

在多线程的进程中，每个线程独立运行，当然可以调用各种例程来完成正在执行的任何工作。不是地址空间中只有一个栈，而是每个线程都有一个栈。

| 线程共享资源       | 线程独享资源 |
| ------------------ | ------------ |
| 地址空间           | 程序计数器   |
| 全局变量           | 寄存器       |
| 打开的文件         | 栈           |
| 子进程             | 状态字       |
| 信号及信号服务程序 |              |

## 线程 API

### 线程创建

```C
#include <pthread.h>
int
pthread_create(      pthread_t *        thread,
               const pthread_attr_t *  attr,
                     void *             (*start_routine)(void*),
                     void *             arg);
```

该函数有 4 个参数：thread、attr、start_routine 和 arg。

1. 第一个参数 thread 是指向 pthread_t 结构类型的指针，我们将利用这个结构与该线程交互，因此需要将它传入 pthread_create()，以便将它初始化。
2. 第二个参数 attr 用于指定该线程可能具有的任何属性。一些例子包括设置栈大小，或关于该线程调度优先级的信息。
3. 第三个参数最复杂，但它实际上只是问：这个线程应该在哪个函数中运行？在 C 中，我们把它称为一个函数指针（function pointer），这个指针告诉我们需要以下内容：一个函数名称（start_routine）。
4. 第四个参数 arg 就是要传递给线程开始执行的函数的参数。

### 线程完成

```C
void *mythread(void *arg) {
    int m = (int) arg;
    printf("%d\n", m);
    return (void *) (arg + 1);
}

int main(int argc, char *argv[]) {
    pthread_t p;
    int rc, m;
    Pthread_create(&p, NULL, mythread, (void *) 100);
    Pthread_join(p, (void **) &m);
    printf("returned %d\n", m);
    return 0;
}
```

该函数有两个参数。

1. 第一个是 pthread_t 类型，用于指定要等待的线程。
2. 第二个参数是一个指针，指向你希望得到的返回值。因为函数可以返回任何东西，所以它被定义为返回一个指向 void 的指针。

### 锁

```C
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

### 条件变量

```C
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
int pthread_cond_signal(pthread_cond_t *cond);
```

## 线程的生命周期

```
                            线程被选中
●-------> 创建 -------> 就绪 <-------> 运行 -------> 退出 -------> ●
                         ↑ 时间片用尽 /
                          \         /
                    阻塞结束\       /等待IO或其他任务
                            \     /
                             \   ↓
                               阻塞
```

创建：一个新的线程被创建，等待该线程被调用执行；
就绪：时间片已用完，此线程被强制暂停，等待下一个属于它的时间片到来；
运行：此线程正在执行，正在占用时间片；
阻塞：也叫等待状态，等待某一事件(如 IO 或另一个线程)执行完；
退出：一个线程完成任务或者其他终止条件发生，该线程终止进入退出状态，退出状态释放该线程所分配的资源。

## 线程通信

线程间的通信目的主要是用于线程同步，所以线程没有像进程通信中的用于数据交换的通信机制。

### 互斥锁

互斥锁是最常见的线程同步方式，它是一种特殊的变量，它有 lock 和 unlock 两种状态，一旦获取，就会上锁，且只能由该线程解锁，期间，其他线程无法获取。

优点：使用简单；

缺点：

1. 重复锁定和解锁，每次都会检查共享数据结构，浪费时间和资源；
2. 繁忙查询的效率非常低；

### 条件变量

条件变量的方法是，当线程在等待某些满足条件时使线程进入睡眠状态，一旦条件满足，就唤醒，这样不会占用宝贵的互斥对象锁，实现高效。

条件变量允许线程阻塞并等待另一个线程发送信号，一般和互斥锁一起使用。

条件变量被用来阻塞一个线程，当条件不满足时，线程会解开互斥锁，并等待条件发生变化。一旦其他线程改变了条件变量，将通知相应的阻塞线程，这些线程重新锁定互斥锁，然后执行后续代码，最后再解开互斥锁。

### 读写锁

读写锁 也称之为 共享-独占锁，一般用在读和写的次数有很大不同的场合。即对某些资源的访问会出现两种情况，一种是访问的排他性，需要独占，称之为写操作；还有就是访问可以共享，称之为读操作。

适用场景：读写锁最适用于对数据结构的读操作次数多于写操作次数的场合。

### 信号量

信号量 和互斥锁的区别在于：互斥锁只允许一个线程进入临界区，信号量允许多个线程同时进入临界区

可以这样理解，互斥锁使用对同一个资源的互斥的方式达到线程同步的目的，信号量可以同步多个资源以达到线程同步。

### 信号

类似进程间的信号处理

## 死锁

死锁的四个必要条件

1. 资源互斥
2. 持有并等待
3. 不能抢占
4. 循环等待

死锁的应对方法

1. 顺其自然，不予理睬
2. 死锁检测与修复
3. 死锁的动态避免
4. 死锁的静态防止
5. 清除四个必要条件中的一个
   1. 资源共享
   2. 一次性抢到所有锁
   3. 允许抢占资源
   4. 顺序请求资源

## 参考

《操作系统之哲学原理第 2 版》邹恒明

《操作系统导论》

