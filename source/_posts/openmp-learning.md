---
title: OpenMP Learning
tags:
  - openmp
  - hpc
photos:
 - images/runners-635906_1920.jpg
date: 2017-05-10 20:32:31
---

When I start to learn HPC, I set out from OpenMP. In this series of articles, I **collect and summarize** a batch of useful tutorials about OpenMP (C/C++) for newcomers.

<!--more-->

## 1. Overview

### 1.1 Parallelism

- Two programming models for processors to collaborate
  - **Shared Memory**: directly access and change all the variables
  - **Distributed Memory**: every CPU has access to its own memory space
- Elements of Shared-memory programming
  - Fork/join threads
  - Synchronization
  - Assign/Distribute jobs to threads
  - Run-time control
- Application of Parallelism: Typical Examples
  - "High-throughput" calculations with lots of independent jobs
  - Mesh-based Problems: partition the graph
  - Particle-based Problems: partition the cells/tree
  - Parallelism speeds up your program!


### 1.2 OpenMP

- What is OpenMP?
  - OpenMP is a set of compiler hints and function calls supported by mainstream compilers to enable you to run sections of your code in parallel on a shared memory parallel computer(multicore)
  - Use `#progma` to parallel your code
- Why OpenMP?
  - Adding cores doesn’t make programs go faster, you also need to tell the program how to use the cores.
  - Simple, compiler-supported, relative high level

> Warning: OpenMP-compliant implementations is **not** responsible for providing a conforming program. OpenMP API does not cover automatic parallelization and directives to assist such parallelization.

### 1.3 MPI (Message Passing Interface)

- Suitable for both distributed-systems and single computer
- Use distributed memory
- In the form of libraries rather than compiler-supported
- Lower level, more details

## 2. Basic Usage

### 2.1 Set up

OpenMP is supported by compilers which implement corresponding APIs, so there is no need to download or install external packages. Follow these steps and enjoy your parallel coding journal.

1. Choose a compiler. Make sure your compiler is in [this list](http://www.openmp.org/resources/openmp-compilers/). Notice that if you are using clang in mac OS, it may not support OpenMP for now, and personally I install GNU GCC 7.1 with [homebrew](https://brew.sh/) to solve this problem.
2. Create a folder used to place your OpenMP project together with a file named "helloworld.cpp" in which includes `<omp.h>` and some `#pragma omp` statements to check if your compiler works.
3. Write a `makefile` for your project. You may need `-fopenmp` argument when compiling source code.
4. Learn OpenMP by coding~

- Shared and Private
  - Private: loop index
  - Shared
- Nested Loop
  - parallelized outer loop
  - how to parallelized inner loop: `#pragma opm parallel for private(i)`
- reduction operation
  - sum reduction

### 2.2 What a OpenMP Program Looks like?

1. Header file `#include <omp.h>`
2. Parallel region
3. Environment variables and functions

![parallel region](http://oowu6eof3.bkt.clouddn.com/parallel-region.png)

### 2.3 General Syntax (for C/C++)

You can use OpenMP in this way:

```C++
#pragma omp construct_name [clauses...]
{
  // do something ...
}
```

#### 2.3.1 Parallel region

Use `#pragma omp parallel [clauses...]` to create a parallel region.
By default, this will fork N threads as a team, executing codes in this region at the same time. **Clauses** are used to specify the precise behavior of the region.

Commonly-used clauses:

- `private(variable, ...)`
  - the values of private data are undefined upon entry to and exit from the specific construct
  - `lastprivate`: the last value is accessible after the construct
  - `firstprivate`: variables with values available prior to the region
- `shared(variable, ...)`
  - Each thread can read or modify shared variables
  - Race condition may take place
- `nowait`
  - Allow earlier finished threads to proceed without waiting
- `if (condition)`
  - Determine if the region should run in parallel

#### 2.3.2 Work Sharing

Just adding cores doesn’t make programs go faster, you need to tell the compiler how to use these cores.
Work-sharing methods allow you to distribute jobs to different threads by a set of APIs.

- Loop
  - Construct: `#pragma omp for [clauses...]`
  - Schedule: `schedule()`
    - static(default) / dynamic / guided / runtime
  - Notice that `#pragma omp for [clauses...]` should be inside a parallel
  region or threads will not be created.
  - You can also use `#pragma omp parallel for [clauses...]` to create threads and loop in one line.
- Sections `#pragma omp sections` and `#pragma omp section`
    - One thread executes one section
    - Threads may executes more or less than one section if the number of section is not equal to the number of processors
- `single`
  - Executed by exactly one thread
- `critical`
  - One thread at a time
  - Every thread will execute this region which differs from `single`
- `reduction`
  - Used to calculate sum or something like it
- `Barrier`
  - All threads pause at the barrier, until all threads execute the barrier


#### 2.3.3 Environment variables and functions

- Resource Query Functions
  - `omp_get_num_procs()`: get the number of processors on this computer
  - `omp_get_thread_num()`: get the thread id
  - `omp_get_num_threads()`: get the number of threads inside a parallel region
  - `omp_get_max_threads()`: get the max number of threads
- Control the number of threads
  - `#pragma omp parallel num_threads(integer)` highest priority
  - `omp_set_num_threads()` medium priority
  - `export OMP_NUM_THREADS=n` lowest priority

Explore more about OpenMP usage in official API~

### 2.4 Race Condition

When threads try to access the same address, **race condition** happens.
Notice that data race condition can be **hard to detect/debug**.

Example (try to add up a array):
{% codeblock lang:cpp %}
//  Naive Version
sum = 0
#pragma omp parallel
{
  #pragma omp for
  for (int i = 0; i < N; ++i) {
    sum += a[i];
  }
}
{% endcodeblock %}

To solve this problem, we divide total sum operation into exclusive parts, then we can add then up to achieve the final result.

{% codeblock lang:cpp %}
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
{% endcodeblock %}

Or, solve it with simpler and cleaner reduction operation.

{% codeblock lang:cpp %}
sum = 0;
#pragma omp parallel for shared(...) private(...) reduction(+:sum)
{
  for (i=0; i<n; i++) {
    sum += a[i];
  }
}
{% endcodeblock %}

## 3. Major Terminology

### Concept

- thread
- OpenMP thread
- thread-safe routine
- processor
- device
- host device
- target device


## 4. Reference and More useful websites

- [OpenMP in 30 Minutes](http://www.linux-mag.com/id/4609/)
- [intro-openmp](https://idre.ucla.edu/sites/default/files/intro-openmp-2013-02-11.pdf)
- [Openmp-Tutorial](http://www.openmp.org/resources/tutorials-articles/)
- [MSDN](https://msdn.microsoft.com/en-us/library/tt15eb9t.aspx)
