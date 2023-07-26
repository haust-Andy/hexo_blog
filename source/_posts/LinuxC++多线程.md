---
title: LinuxC++多线程
date: 2023-07-25 16:41:37
tags: 多线程
categories: C++
---

### 线程是 CPU 调度和分派的基本单位
是处于执行期的程序以及它所管理的资源（如打开的文件、挂起的信号、进程状态、地址空间等等）的总称，从操作系统核心角度来说，进程是操作系统调度除 CPU 时间片外进行的资源分配和保护的基本单位，它有一个独立的虚拟地址空间，用来容纳进程映像(如与进程关联的程序与数据)，并以进程为单位对各种资源实施保护，如受保护地访问处理器、文件、外部设备及其他进程(进程间通信)
<br>
进程拥有独立的地址空间（包括数据区、堆区和栈区）。而线程拥有独立的栈区域，数据区和堆区都是一个进程中的几个线程共享的。
### 为什么要使用多线程
1. 避免阻塞
2. 避免 CPU 空转
3. 提升效率

## 线程的创建与运行
### pthread_create：
线程具有单独的执行流，因此需要单独定义线程的 main 函数，还需要请求操作系统在单独的执行流中执行该函数，完成该功能的函数如下。
```
#include<pthread.h>    
int pthread_create(     
pthread_t* restrict thread,  
const pthread_attr_t* restrict attr,    
void *( *start_routine)(void *),    
void* restrict arg ); 
``` 
→成功时返回 0，失败时返回其他值。
- thread    
保存新创建线程 ID 的变量地址值。线程与进程相同，也需要用于区分不同线程的 ID。
- attr  
用于传递线程属性的参数，传递 NULL 时，创建默认属性的线程。
- start routine     
相当于线程 main 函数的、在单独执行流中执行的函数地址值（函数指针）。
- arg   
通过第三个参数传递调用函数时包含传递参数信息的变量地址值


### pthread_join
调用 pthread_join 函数的进程（或线程）将进入等待状态，直到第一个参数为 ID 的线程终止为止。而且可以得到线程的 main 函数返回值，所以该函数比较有用。
```
# include<pthread.h>
int pthread_join(pthread_t thread, void ** status);
```
→成功时返回 e，失败时返回其他值。
thread 该参数值 ID 的线程终止后才会从该函数返回。status 保存线程的 main 函数返回值的
指针变量地址值



