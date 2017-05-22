---
title: caffe tour
tags:
---

## Overview

`caffe`中包含四个核心部分，抽象程度自低向高分别是：

- Blob
  - 承载传输数据 类似TF中的Tensor
- Layer
  - 网络基础单元
- Net
  - 搭建网络结构
- Solver
  - 求解网络的策略

通过分析`caffe`项目源代码，可以看到`caffe`程序启动后大致会经历以下几个阶段。

## 初始化过程

初始化过程中包含以下几个核心的操作：

`RegisterBrewFunction`：这个宏的作用是将函数`func`的首地址存入一个`map`数据结构中，在接受输入时，`caffe`会对命令进行解析，通过`GetBrewFunction`调用相应的函数。

```C++
//  caffe.cpp
#define RegisterBrewFunction(func) \
namespace { \
class __Registerer_##func { \
 public: /* NOLINT */ \
  __Registerer_##func() { \
    g_brew_map[#func] = &func; \
  } \
}; \
__Registerer_##func g_registerer_##func; \
}
```

目前版本的`caffe`存入的函数为：device_query / train / test / time
源码中的注释中也有提及如果需要自定义命令同样可以通过`RegisterBrewFunction`注册自定义的函数名然后进行使用。

`CreateSolver`

## 训练网络时的调用过程

`solver->Solve();` (caffe.cpp 257)
TODO caffe.cpp 266 test

## C++ Notes

`static` 函数：只能在该文件中使用
宏`#`与`##`：用于拼接字符，同时`#`会用双引号进行包裹
