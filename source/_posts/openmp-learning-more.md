---
title: OpenMP Learning (More)
tags:
  - openmp
  - hpc
date: 2017-05-12
description: 在上一篇教程基础上完善的中文版 OpenMP 入门教程。
---

这是在上一篇教程基础上完善的中文版 OpenMP 入门教程。在之前的基础上细化了各种 OpenMP 命令的介绍，并引入了更多例子。

<!--more-->

## 0 实验环境

- 操作系统：macOS 10.12.4
- 编译器：GNU GCC 7.1
- OpenMP：201511


## 1 基本概念

### 1.1 并行

**并行**指的是同一台机器或通过网络连接的多台机器上的多个CPU同时执行一些进程。

#### 1.1.1 并行机制

具体的实现方式主要有以下两种机制：

- **共享内存(Shared Memory)**：不同进程／线程可以直接访问同一段内存
- **消息传递(Message Passing)**：每一块CPU都只能访问它自己的内存空间，通过MPI消息传递接口通信


OpenMP是一套基于**共享内存**机制，在多CPU机器上进行并行程序设计的API。

#### 1.1.2 共享内存涉及的基本元素

- `fork`／`join` 并行模式
- 线程同步
- 将`jobs`分配给各个线程的方法
- 运行时调度与控制机制

#### 1.1.3 并行计算的应用场景

- 包含很多高吞吐量的独立工作
- 基于网格 (Mesh) 的编程：图划分算法
- 基于粒子 (Particle) 的编程：元胞划分／树划分

合适的并行算法可以很大程度上提升程序运行的速度！



### 1.2 OpenMP简介

#### 1.2.1 什么是OpenMP？

**OpenMP**是一系列已经被主流编译器支持的，用来帮助开发者通过共享内存机制并行化一些代码段的`hint` (C/C++中的`#pragma`预处理指令) 和函数的集合。

OpenMP已经被集成在[主流编译器](http://www.openmp.org/resources/openmp-compilers/)的实现中，因此不需要额外安装任何库，只需要在编译时添加`-fopenmp`参数告知编译器即可。

> 单纯为计算机添加更多核不会让它跑的更快，你需要告诉它如何使用这些CPU。

#### 1.2.2 MPI (Message Passing Interface)

MPI 是通过消息传递方式实现并行的一套接口，不同于OpenMP主要在以下几个方面：

- MPI对于分布式系统和单一机器都适用，而OpenMP仅适用于单个机器
- 使用消息传递机制，分布式内存
- 以第三方库的形式提供而非编译器实现
- 相对来说在更低的level，含有更多的细节需要开发者进行处理





## 2 基本使用

### 2.1 环境配置

因为OpenMP已经被主流编译器集成，所以只需要确保使用的编译器支持OpenMP即可。（如果使用macOS系统，自带的`gcc`只是Apple`clang`的别名，而后者当前并不支持openmp，所以需要另外安装`GNU GCC`。通过`homebrew`直接安装是一个比较方便的选项。）

通过以下四个步骤即完成所有配置操作：

1. 选择／安装编译器；
2. 创建Project文件夹和相应cpp文件，在首部需要`#include <omp.h>`，也可以加入一些`#pragma omp`来检测编译器是否能够正常工作；
3. 编写`makefile`文件，注意需要在编译时加上`-fopenmp`参数；
4. 现在已经可以通过编写Demo的方式学习OpenMP～





### 2.2 OpenMP程序构成要素

- `#include <omp.h>`
- 由`#pragma omp parallel`创建的并行域
- `OpenMP`中特有的环境变量和全局函数





### 2.3 语法 (for C/C++)

C和C++中，OpenMP主要通过`#pragma`预处理指令来指定并行域以及并行域中各个线程的行为，基本格式如下：

```C++
#pragma omp construct_name [clauses...]
{
  // do something in parallel...
}
```
#### 2.3.1 并行域 (Parallel Region)

通过`#pragma omp parallel [clauses...]`来创建一块并行域。但是默认情况下，这回让程序在这里为每一个CPU创建一个线程，分别将区域内的代码执行一遍。`clauses`可以用来改变这种默认行为。

常用到的`clauses`:

- `private(variable, ...)`
  - 显式指明不会被并行域内不同线程间共享的变量
- `firstprivate`
  - 用于继承公有变量的值，修改后原变量不受影响
- `lastprivate`
  - 退出并行域时将值拷贝给同名的共享变量
- `threadprivate`
  - 在每一个线程中创建一份私有的拷贝
- `shared(variable, ...)`
  - 显式指明不会被并行域内不同线程间共享的变量
  - 可能会导致race competition
- `nowait`
  - 允许先执行完的线程继续运行
- `if (condition)`
  - 为并行域中的代码是否进入并发添加一个条件限制

**Example:**

```C++
#pragma omp parallel
{
  printf("hello openmp\n");
}

/* output
hello openmp
hello openmp
hello openmp
hello openmp
*/
```



#### 2.3.2 工作共享 (Work Sharing)

Work-sharing通过一系列的API，允许开发者告诉编译器如何安排各个线程在并行域中的行为。

##### 2.3.2.1 Loop

构造：`#pragma omp for [clauses...]`

**Example:**

```C++
//  use `for` inside a parallel region
#pragma omp parallel
{
  #pragma omp for
  for (int i = 0; i < 10; i++) {
    printf("hello openmp: %d\n", i);
  }
}

//  or do it in another way
#pragma omp parallel for
for (int i = 0; i < 10; i++) {
  printf("hello openmp: %d\n", i);
}

/* output
hello openmp: 0
hello openmp: 6
hello openmp: 3
hello openmp: 8
hello openmp: 1
hello openmp: 7
hello openmp: 4
hello openmp: 9
hello openmp: 2
hello openmp: 5
*/
```

可以看到此时由于并发运行输出的顺序发生了改变。

不过需要注意的是`#pragma omp for`一定要在并行域中才能起到并行的作用，如果单独使用则会被编译器无视切没有任何错误提示信息。

**Negative Example:**

```c++
// No parallel at all!
#pragma omp for
for (int i = 0; i < 10; i++) {
  printf("hello openmp: %d\n", i);
}

/* output
hello openmp: 0
hello openmp: 1
hello openmp: 2
hello openmp: 3
hello openmp: 4
hello openmp: 5
hello openmp: 6
hello openmp: 7
hello openmp: 8
hello openmp: 9
*/
```

##### 2.3.2.2 Section

除了在`for`中让程序并行之外，通过使用`section`将程序划分为几个部分同样可以让OpenMP为我们自动创建线程进行并行操作，基本结构如下：

```C++
#pragma omp parallel sections
{
  #pragma omp section
  {
    //  thread 0
  }

  #pragma omp section
  {
    //  thread 1
  }

  ...
}
```

这里会将每个`section`分配给不同的线程，再随机分配给不同的CPU同时执行，若`section`的数目超出了CPU数目，有的CPU会执行多个`section`；若少于CPU的数目，将导致部分CPU空闲的情况。我们可以在如下的例子中看到分配给每一个线程的分配情况。

**Example:**

```C++
#pragma omp parallel sections
{
  #pragma omp section
  { printf("section 1: %d\n", omp_get_thread_num()); }

  #pragma omp section
  { printf("section 2: %d\n", omp_get_thread_num()); }

  #pragma omp section
  { printf("section 3: %d\n", omp_get_thread_num()); }

  #pragma omp section
  { printf("section 4: %d\n", omp_get_thread_num()); }
}

/* output
section 2: 2
section 3: 0
section 1: 1
section 4: 3
*/
```

##### 2.3.2.3 Single

`single`标记的区域一定只会**被一个线程执行且只一次**

**Example:**

```C++
int b[5];
# pragma omp parallel
{
  int a = 0;
  #pragma omp single
  {
    a = 10;
  }
  #pragma omp for
  for (int i = 0; i < 5; i++) {
    b[i] = a;
  }
  #pragma omp for
  for (int i = 0; i < 5; ++i) {
    printf("%d: %d\n", b[i], omp_get_thread_num());
  }
}

/* output
0: 0
10: 3
0: 1
0: 2
0: 0
*/
```

可以看到只有进程3执行了`single`标记的代码块，将`a`的值进行了修改，而其他的均保持不变。

##### 2.3.2.4 Critical

`critical`标记的区域**每次只有一个线程在执行**，但是最终会被每个线程都执行到。（注意和`single`的区别）

可以用来避免race condition。我们用并行化累加这一经典的例子来进行详细说明，任务很简单，就是用并行的方法将存在数组`a[]`中的值累加起来输出：

**Negative Example:**

```C++
//  Naive version, this will get wrong answer
#pragma omp parallel for shared(sum, a)
for (int i = 0; i < N; ++i) {
  sum += a[i];
}
printf("sum test: %d\n", sum);
```

这个例子有时无法得到正确的结果，虽然看起来正确。只要仔细分析就会发现，在并行的`for`中，`sum`被所有线程共享，因此可能发生多个线程同时写入`sum`地址空间的问题。

针对这个问题，可以用先将整个数组拆成若干小数组进行分别求和，算出各自的`sum`，然后再用`critical`对`sum_total`的写入进行保护，累加之前各自的结果，防止多线程同时写入，作为第一种解决方案：

**Solution 1:**

```C++
//  Solution 1: use critical
#pragma omp parallel shared(N, a ,sum) private(sum_local)
{
  sum_local = 0;
  #pragma omp for
  for (int i = 0; i < N; i++) {
    sum_local += a[i];
  }
  #pragma omp critical
  {
    sum += sum_local;  // form global sum
  }
}
printf("sum test 2: %d\n", sum);
```

本例只是用来表现`critical`的作用，真正求和时可以使用`reduction`，更加简洁且同样可以达到目的。

##### 2.3.2.5 Reduction

通过`reduction`可以更方便地进行求和、减法、`and`、`or`等操作，下面使用`reduction`重新实现前面求和的操作如下：

**Example:**

```C++
#pragma omp parallel for reduction(+:sum)
for (int i = 0; i < N; i++) {
  sum += a[i];
}
printf("sum test 3: %d\n", sum);
```

这样同样可以避免race condition的产生。

##### 2.3.2.6 Barrier

通过`#pragma omp barrier`放置`barrier`，在此处所有线程都会进行等待直到当前所有线程的任务都已完成才会通过执行下一步操作。可以用于同步线程。

**Example:**

```c++
int x = 2;
#pragma omp parallel shared(x)
{
  int tid = omp_get_thread_num();
  if (tid == 0) {
    x = 5;
  } else {
    printf("[1] thread %2d: x = %d\n",tid,x);
  }
  #pragma omp barrier
  printf("[2] thread %2d: x = %d\n", tid, x);
}

/* output
[1] thread  3: x = 2
[1] thread  1: x = 2
[1] thread  2: x = 5
[2] thread  0: x = 5
[2] thread  3: x = 5
[2] thread  1: x = 5
[2] thread  2: x = 5
*/
```

本例中首先设置公有变量`x = 2`然后使用线程0将其修改成5。但是由于可能有部分线程先于线程0执行所以此时打印出的结果仍然是2，而经过了barrier后，所有线程中的x都被改成5，说明此时线程1一定得到了执行。

##### 2.3.2.7 Schedule

`schedule`可以指定调度方式（系统默认的是static），不同的调度方式以及区别列举如下：

**Static**：每个进程被赋予固定尺寸的块

![](http://oowu6eof3.bkt.clouddn.com/static-schedule.png)



**Dynamic**: 按照线程需求分配块的大小

![](http://oowu6eof3.bkt.clouddn.com/static-schedule.png)



**Guided**: 先分配较大的块，然后逐渐分配越来越小的块

![](http://oowu6eof3.bkt.clouddn.com/guided-schedule.png)



**Runtime**: 根据用户的系统环境变量来选择调度方法



### 2.4 Race Condition

总结一下，当多个线程**有可能**对同一块地址进行写入操作，或者读写同时发生的时候，就有可能产生race condition，造成难以预料的结果。

规避主要有两种方法：

- 使用`critical`保护可能产生race condition的区域
- 当可行的时候，使用`reduction`



### 2.5 环境变量与常用函数

默认OpenMP会根据CPU数目创建相应的线程并进行并行处理，通过修改环境变量`#OMP_NUM_THREADS`、调用`omp_set_num_threads()`或者使用`#pragma omp parallel num_threads(integer)`直接指定可以改变这一默认行为。其中`#pragma`优先级最高、仅对该并行域起作用；`omp_set_num_threads()`次之，对调用后的部分起作用；修改环境变量优先级最低，对系统下所有文件起作用。

其它常用到的函数主要有：

- `omp_get_num_procs()`: 获取计算机上的处理器个数
- `omp_get_thread_num()`: 获取当前线程ID
- `omp_get_num_threads()`: 获取当前并行域中的所有线程数目
- `omp_get_max_threads()`: 获取创建的最大线程数



## 3 一些主要概念的解释

- 线程 thread：具有栈和一段静态内存的运行实体
- OpenMP 线程：由OpenMP运行时系统管理的线程
- 线程安全：多个线程同时操作时仍能具备预期功能
- 处理器：实现的能够运行一个或多个OpenMP线程的硬件单元
- 设备：定义了逻辑运行引擎的实现
- host设备：让OpenMP程序开始运行的设备
- target设备：从host设备获取代码和数据的设备
- 指令 directive：定义OpenMP行为的语句



## 4 Reference

- [OpenMP in 30 Minutes](http://www.linux-mag.com/id/4609/)
- [intro-openmp](https://idre.ucla.edu/sites/default/files/intro-openmp-2013-02-11.pdf)
- [Openmp-Tutorial](http://www.openmp.org/resources/tutorials-articles/)
- [MSDN](https://msdn.microsoft.com/en-us/library/tt15eb9t.aspx)
- [CSDN blog](http://blog.csdn.net/drzhouweiming/article/details/1131537)
