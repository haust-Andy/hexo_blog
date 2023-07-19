---
title: Linux C++ 多进程
date: 2023-07-18 09:31:54
tags: 多进程
categories: C++
---

**进程是资源分配的基本单位，是程序流的基本单位**

## 进程 ID

无论进程是如何创建的，所有进程都会从操作系统分配到 ID。此 ID 称为"进程 ID"，其
值为大于 2 的整数。1 要分配给操作系统启动后的（用于协助操作系统）首个进程，因此用
户进程无法得到 ID 值 1。
通过 ps au 指令可以查看当前运行的所有进程。令同时可以列
出 PID（进程 ID）。通过指定 a 和 u 参数列出了所有进程详细信息。

## 通过调用 fork 函数创建进程

> #include <unistd.h> pid_t fork(void); → 成功时返回进程 ID，失败时返回-1。

fork 函数将创建调用的进程副本。也就是说，并非根据完全不同的程序创建进程，
而是复制正在运行的、调用 fork 函数的进程。另外，两个进程都将执行 fork 函数调用
后的语句（准确地说是在 fork 函数返回后）。但因为通过同一个进程、复制相同的内
存空间，之后的程序流要根据 fork 函数的返回值加以区分。即利用 fork 函数的如下特
点区分程序执行流程。 <br>
父进程 ∶fork 函数返回子进程 ID。<br>
子进程 ∶fork 函数返回 0。 <br>
此处"父进程（" Parent Process）指原进程，即调用 fork 函数的主体，而"子进程（" Child
Process）是通过父进程调用 fork 函数复制出的进程。

```
#include <stdio.h>
#include <unistd.h>
int gval = 10;
int main(int argc, char* argv[])
{
    pid_t pid;
    int lval = 20;
    gval++, lval += 5;
    pid = fork();
    if (pid == 0)// if Child Process
        gval += 2, lval += 2;
    else // if Parent Process
        gval -= 2, lval -= 2;
    if (pid == 0)
        printf("Child Proc: [%d, %d] \n", gval, lval);
    else
        printf("Parent Proc: [%d, %d] \n", gval, lval);
    return 0;
}
```

父子进程拥有完全独立的内存结构

### fork 函数产生子进程的终止方式

1 传递参数并调用 exit 函数。 <br>
2 main 函数中执行 retun 语句并返回值 <br>
向 exit 函数传递的参数值和 main 函数的 return 语句返回的值都会传递给操作系统。但是
操作系统不会销毁子进程，直到该子进程的父进程 return 或者 exit。处在这种状态下的进程就是僵尸进程。

## 僵尸进程——如何销毁

父进程调用 fork 创建子进程，子进程退出后，父进程在未退出时，操作系统不会主动销毁子进程，直到父进程退出后操作系统才会一同销毁父子进程。处于子进程退出，父进程未退出阶段的，操作系统没有主动销毁的子进程就成为了僵尸进程。

此时该子进程如果不调用 wait 或 waitpid 来获取子进程的退出信息，子进程就会沦为僵尸进程。

**子进程已经退出了，父进程还在运行当中，父进程没有读取到子进程的状态，子进程就会进入僵尸状态。**

为了销毁子进程，父进程应主动请求获取子进程的返回值。

### **1.wait 函数**

> #include<sys/wait.h> pid_t wait(int\* statloc); → 成功时返回终止的子进程 ID，失败时返回-1。

调用此函数时如果已有子进程终止，那么子进程终止时传递的返回值（exit 函数的参数值、main 函数的 return 返回值）将保存到该函数的参数所指内存空间。但函数参数指向的单元中还包含其他信息，因此需要通过下列宏进行分离。

1. WIFEXITED 子进程正常终止时返回"真"（true）。
2. WEXITSTATUS 返回子进程的返回值。
   也就是说，向 wait 函数传递变量 status 的地址时，调用 wait 函数后应编写如下代码。

```
if(WIFEXITED(status)){//是正常终止的吗?
puts("Normal termination!");
printf（"Child pass num∶%d"，WEXITSTATUS（status））;}//打印返回值
```

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
int main(int argc, char* argv[])
{
    int status;
    pid_t pid = fork();
    if (pid == 0){
        //创建的子进程将在此行通过 main 函数中的 return 语句终止
        sleep(30);
        return 3;
    }else{
        printf("Child PID: %d \n", pid);
        pid = fork();
        if (pid == 0){
            //创建的子进程将在此行通过调用 exit 函数终止
            exit(7);
        }else{
            printf("Child PID: %d \n", pid);
            //调用 wait。之前终止的子进程信息将保存到 status 变量，同时相关子进
            //程被完全销毁。
            wait(&status);
            //通过 WIFEXITED 宏验证子进程是否正常终止。
            //如果正常退出，则调用 WEXITSTATUS 宏输出子进程的返回值
            if (WIFEXITED(status))
                printf("Child send one: %d \n", WEXITSTATUS(status));
            //因为之前创建了 2 个进程，所以再次调用 wait 函数和宏
            wait(&status);
            if (WIFEXITED(status))
                printf("Child send two: %d \n", WEXITSTATUS(status));
            //为暂停父进程终止而插入的代码。此时可以查看子进程的状态
            sleep(30); // Sleep 30 sec.
        }
    }
    return 0;
}
```

输出：

```
<br>Child PID: 7660
<br>Child PID: 7661
<br>Child send one: 7
<br>Child send two: 3
```

从结果可以看出来，系统中并无上述结果中的 PID 对应的进程，。这是因为调用了 wait
函数，完全销毁了该进程。另外 2 个子进程终止时返回的 3 和 7 传递到了父进程。
**_调用 wait 函数时，如果没有已终止的
子进程，那么程序将阻塞（Blocking）直到有子进程终止_**，因此需谨慎调用该函数。

### 2.waitpid 函数

wait 函数会引起程序阻塞，还可以考虑调用 waitpid 函数。这是防止僵尸进程也是防止阻塞的方法。
调用 waitpid 函数时，程序不会阻塞。

> #include<sys/wait.h> pid_t waitpid(pid_t pid, int \*statloc, int options); → 成功时返回终止的子进程 ID（或 0），失败时返回-1。

- pid 等待终止的目标子进程的 ID，若传递-1，则与 wait 函数相同，可以等待任意子进
  程终止。
- statloc 与 wait 函数的 statloc 参数具有相同含义。
- options 传递头文件 sys/wait.h 中声明的常量 WNOHANG，即使没有终止的子进程也
  不会进入阻塞状态，而是返回 0 并退出函数。

```
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
int main(int argc, char* argv[])
{
    int status;
    pid_t pid = fork();
    if (pid == 0)
    {
        //调用 sleep 函数推迟子进程的执行。这会导致程序延迟 15 秒
        sleep(15);
        return 24;
    }
    else
    {
        //while 循环中调用 waitpid 函数。向第三个参数传递 WNOHANG，因此，若之前没
        //有终止的子进程将返回 0。
        if (!waitpid(-1, &status, WNOHANG)) printf("此刻无终止的子进程\n");
        while (!waitpid(-1, &status, WNOHANG))
        {
            sleep(3);
            puts("sleep 3sec.");
        }
        if (WIFEXITED(status))
            printf("Child send %d \n", WEXITSTATUS(status));
    }
    return 0;
}
```

可以看出 puts("sleep 3sec.");
共执行了 5 次。另外，这也证明 waitpid 函数并未阻塞。

## 信号处理

为了避免父进程调用 waitpid 函数后一直等待子进程终止。所以要在进程发现自己的子进程结束时，请求
操作系统调用特定函数。该请求通过 signal 函数调用完成（因此称 signal 为信号注册函数）。

> #include<signal.h>  
> void (*signal(int signo, void(_func)(int))) (int);    
> 等价于      
> typedef void(*signal_handler)(int)  
> signal_handler signal(int signo, signal_handler func)  
> 函数名 ∶signal
> 参数 ∶int signo,void(*func)(int) 
> 返回类型 ∶ 参数为 int 型，返回 void 型函数指针。

调用上述函数时，第一个参数为特殊情况信息，第二个参数为特殊情况下将要调用的函
数的地址值（指针）。发生第一个参数代表的情况时，调用第二个参数所指的函数。下面给
出可以在 signal 函数中注册的部分特殊情况和对应的常数。

- SIGALRM∶ 通过调用 alarm 函数注册的时间来使 os 收到该信号。
- SIGINT∶ 输入 CTRL+C。
- SIGCHLD∶ 子进程终止。（英文为 child）

_样例_
编写调用 signal 函数的语句完成如下请求

1. "子进程终止则调用 mychild 函数。"

> signal(SIGCHLD, mychild);

此时 mychild 函数的参数应为 int，返回值类型应为 void。对应 signal 函数的第二个参数。
<br>另外，常数 SIGCHLD 表示子进程终止的情况，应成为 signal 函数的第一个参数。

2. "已到通过 alarm 函数注册的时间，请调用 timeout 函数。"

> signal(SIGALRM, timeout);

3. "输入 CTRL+C 时调用 keycontrol 函数。"

> signal(SIGINT, keycontrol);

以上就是信号注册过程。注册好信号后，发生注册信号时（注册的情况发生时），操作系统将调用该信号对应的函数

### alarm 函数

alarm 系统调用是设置多久触发 SIGALRM 信号的函数

> #include<unistd.h>
> unsigned int alarm(unsigned int seconds);→ 返回 0 或以秒为单位的距 SIGALRM 信号发生所剩时间。

如果调用该函数的同时向它传递一个正整型参数，相应时间后（以秒为单位）将产生
SIGALRM 信号。若向该函数传递 0，则之前对 SIGALRM 信号的预约将取消。如果通过该函数
预约信号后未指定该信号对应的处理函数，则（通过调用 signal 函数）终止进程，不做任何
处理。

```
void signal_func(int sig) {
    switch (sig){
    case SIGALRM:
        printf("子线程  tid: %d, pid: %d\n", pthread_self(), getpid());
        alarm(2);//为了每隔 2 秒重复产生 SIGALRM 信号，在信号处理器中调用 alarm 函数
        break;
    case SIGINT:
        puts("CTRL+C press...\n");
        exit(0);
        break;
    }
}
int main(int argc, char* argv[]){
    //注册 SIGALRM、SIGINT 信号及相应处理器
    signal(SIGALRM, signal_func);
    signal(SIGINT, signal_func);
    //预约 2 秒后发生 SIGALRM 信号
    alarm(2);
    while (true) {
        printf("主线程  tid: %d, pid: %d\n", pthread_self(), getpid());
        sleep(10);//信号量会对sellp休眠时间产生影响
    }
    return 0;
}
```

```
运行结果：
主线程  tid: 1678316736, pid: 13241
子线程  tid: 1678316736, pid: 13241
主线程  tid: 1678316736, pid: 13241
子线程  tid: 1678316736, pid: 13241
主线程  tid: 1678316736, pid: 13241
子线程  tid: 1678316736, pid: 13241
主线程  tid: 1678316736, pid: 13241
子线程  tid: 1678316736, pid: 13241
主线程  tid: 1678316736, pid: 13241
^CCTRL+C press...
```

发生信号时将唤醒由于调用 sleep 函数而进入阻塞状态的进程。故主线程和子线程交替执行。
调用函数的主体的确是操作系统，但进程处于睡眠状态时无法调用函数。因此，产生信
号时，为了调用信号处理器，将唤醒由于调用 sleep 函数而进入阻塞状态的进程。而且，进
程一旦被唤醒，就不会再进入睡眠状态。即使还未到 sleep 函数中规定的时间也是如此。 所以，主程序的一个 while 循环运行不到 10 秒就会结束

### Sigaction 函数

——利用 Sigaction 函数进行信号处理
的 signal 足以用来编写防止僵尸进程生成的代码。介绍这个更强大的 sigaction 函数，是因为 sigaction 函数类似于 signal 函数，而且完全可以代替后者，也更稳定。之所以稳定，是因为如下原因 ∶
"signal 在 UNIX 系列的不同操作系统中可能存在区别，但 sigaction 完全相同。" 实际上现在很少使用 signal 函数编写程序，它只是为了保持对旧程序的兼容。

> #include<signal.h>
> int sigaction(int signo, const struct sigaction*act, struct sigaction*oldact);
> → 成功时返回 0，失败时返回-1

- signo 传递信号信息。
- act 对应于第一个参数的信号处理函数（信号处理器）信息。
- oldact 通过此参数获取之前注册的信号处理函数指针，若不需要则传递 0。

声明并初始化 sigaction 结构体变量，该结构体定义如下

```
struct sigaction{
    void(*sa_handler)(int); //保存信号处理函数的指  针值（地址值）
    sigset_t sa mask;//这 2 个成员用于指定信号相关的    选项和特性,一般初始化为 0
    int sa_flags;
}
```

```
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
void timeout(int sig){
    if (sig == SIGALRM)
        puts("Time out!");
    alarm(2);
}
int main(int argc, char* argv[])
{
    //为了注册信号处理函数，声明 sigaction 结构体变量并在 sa_handler 成员中保
    //存函数指针值。
    struct sigaction act;
    act.sa_handler = timeout;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    //注册 SIGALRM 信号的处理器。调用 alarm 函数预约 2 秒后发生 SIGALRM 信号。
    sigaction(SIGALRM, &act, 0);
    alarm(2);
    for (int i = 0; i < 3; i++){
        puts("wait...");
        sleep(100);
    }
    return 0;
}
```

#### **利用信号处理技术消灭僵尸进程**

子进程终止时将产生 SIGCHLD 信号

```

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <sys/wait.h>
void read_childproc(int sig)
{
    int status;
    pid_t id = waitpid(-1, &status, WNOHANG);
    if (WIFEXITED(status)){
        printf("Removed proc id: %d \n", id);
        printf("Child send: %d \n", WEXITSTATUS(status));
    }
}
int main(int argc, char* argv[])
{
    pid_t pid;
    struct sigaction act;
    act.sa_handler = read_childproc;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    //注册 SIGCHLD 信号对应的处理器。若子进程终止，则调用 read_childproc 函数。
    //处理函数中调用了 waitpid 函数，所以子进程将正常终止，不会成为僵尸进程
    sigaction(SIGCHLD, &act, 0);
    pid = fork();
    if (pid == 0){
        puts("Hi! I'm child process");
        sleep(10);
        return 12;
    }else{
        printf("Child proc id: %d \n", pid);
        pid = fork();
        if (pid == 0){
            puts("Hi! I'm child process");
            sleep(10);
            exit(24);
        }else{
            int i;
            printf("Child proc id: %d \n", pid);
            //for 循环∶为了等待发生 SIGCHLD 信号，使父进程共暂停 5 次，每次间隔 5 秒。发生
            //信号时，父进程将被唤醒，因此实际暂停时间不到 25 秒。
            for (i = 0; i < 5; i++){
                puts("wait...");
                sleep(5);
            }
        }
    }
    return 0;
}
```

运行结果

```
Child proc id: 1629
Hi! I'm child process
Child proc id: 1630
Hi! I'm child process
wait...
wait...
Removed proc id: 1629
Child send: 12
wait...
Removed proc id: 1630
Child send: 24
wait...
wait...
```

可以看出，子进程并未变成僵尸进程，而是正常终止了。
