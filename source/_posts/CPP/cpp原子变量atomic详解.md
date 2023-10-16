---
title: cpp原子变量atomic详解
date: 2023-10-13 08:57:34
tags: atomic
categories: C++
---
# C++原子变量atomic详解
参考列表：https://zhuanlan.zhihu.com/p/599202353 https://cplusplus.com/reference/atomic/atomic/
https://blog.csdn.net/sinat_28305511/article/details/131257757
## 简介
C++中原子变量（atomic）是一种多线程编程中常用的同步机制，它能够确保对共享变量的操作在执行时不会被其他线程的操作干扰，从而避免竞态条件（race condition）和死锁（deadlock）等问题。

原子变量可以看作是一种特殊的类型，它具有类似于普通变量的操作，但是这些操作都是原子级别的，即要么全部完成，要么全部未完成。C++标准库提供了丰富的原子类型，包括整型、指针、布尔值等，使用方法也非常简单，只需要通过std::atomic\<T>定义一个原子变量即可，其中T表示变量的类型。

在普通的变量中，并发的访问它可能会导致数据竞争，竞争的后果会导致操作过程不会按照正确的顺序进行操作。

atomic对象可以通过指定不同的memory orders来控制其对其他非原子对象的访问顺序和可见性，从而实现线程安全。常用的memory orders包括：
memory_order_relaxed、
memory_order_acquire、
memory_order_release、
memory_order_acq_rel
memory_order_seq_cst等。
### C++ 6种内存序
在多线程编程中，内存序可以帮助我们更好地控制多线程程序中的数据访问顺序。C++ 11引入了6种内存序，分别是 `memory_order_relaxed`、`memory_order_consume`、`memory_order_acquire`、`memory_order_release`、`memory_order_acq_rel` 和 `memory_order_seq_cst`。这些内存序提供了不同级别的同步保证，可以帮助我们在多线程程序中实现更精细的控制。

- `memory_order_relaxed`: 最宽松的内存序，不提供任何同步保证。它只保证原子操作本身是原子的，但不保证操作之间的顺序。
- `memory_order_consume`: 消费者内存序，用于同步依赖关系。它保证了依赖于原子操作结果的后续操作将按照正确的顺序执行。
- `memory_order_acquire`: 获取内存序，用于同步对共享数据的访问。它保证了在获取操作之后对共享数据的所有读取操作都将看到最新的数据。
- `memory_order_release`: 释放内存序，用于同步对共享数据的访问。它保证了在释放操作之前对共享数据的所有写入操作都已完成，并且对其他线程可见。
- `memory_order_acq_rel`: 获取-释放内存序，结合了获取和释放两种内存序的特点。它既保证了获取操作之后对共享数据的所有读取操作都将看到最新的数据，又保证了在释放操作之前对共享数据的所有写入操作都已完成，并且对其他线程可见。
- `memory_order_seq_cst`: 顺序一致性内存序，提供了最严格的同步保证。它保证了所有线程都将看到相同的操作顺序，并且所有原子操作都将按照程序顺序执行。

## 成员函数
### 构造函数
std::atomic::atomic
>（1）默认：使对象处于未初始化状态。 atomic() noexcept = default;   <br>
（2）初始化 ：使用val初始化对象。 constexpratomic (T val) noexcept; <br>
（3）复制 [删除] ：无法复制/移动对象。 atomic (const atomic&) = delete;
```
std::atmoic<int> count(0);
```
### is_lock_free函数
is_lock_free用于检查当前atomic对象是否支持无锁操作，调用此成员函数不会启动任何数据竞争
```
bool is_lock_free() const volatile noexcept;
bool is_lock_free() const noexcept;
```
#### std::atomic_flag
std::atomic_flag 是 C++ 中的一个原子布尔类型，它用于实现原子锁操作。
1. std::atomic_flag 默认是清除状态（false）。可以使用 ATOMIC_FLAG_INIT 宏进行初始化，例如：`std::atomic_flag flag = ATOMIC_FLAG_INIT`;
2. std::atomic_flag 提供了两个成员函数 test_and_set() 和 clear() 来测试和设置标志位。test_and_set() 函数会将标志位置为 true，并返回之前的值；clear() 函数将标志位置为 false。
3. std::atomic_flag 的 test_and_set() 和 clear() 操作是原子的，可以保证在多线程环境下正确执行。
4. std::atomic_flag 只能表示两种状态，即 true 或 false，不能做其他比较操作。通常情况下，std::atomic_flag 被用作简单的互斥锁，而不是用来存储信息
   
示例：
```
#include<atomic>
#include<thread>
#include<iostream>
std::atomic_flag flag = ATOMIC_FLAG_INIT; //初始状态为false
void func(int id) {
    std::cout << "Thread  " << id << " start func" << std::endl;
    while (flag.test_and_set(std::memory_order_acquire)) {
        std::cout << "Thread "<< id <<" is waiting" << std::endl;
        // 等待其他线程释放锁
    }
    std::cout << "Thread " << id << " acquired the lock." << std::endl;
    // 模拟业务处理
    //std::this_thread::sleep_for(std::chrono::seconds(1));
    flag.clear(std::memory_order_release);  // 释放锁
    std::cout << "Thread " << id << " released the lock." << std::endl;
}
void _test() {
    std::cout << flag.test_and_set(std::memory_order_acquire) << std::endl;
    flag.clear();
    std::cout << flag.test_and_set(std::memory_order_acquire) << std::endl;
    std::cout << flag.test_and_set(std::memory_order_acquire) << std::endl;
    //set后状态为true
    flag.clear();
    std::thread t1(func, 1);
    std::thread t2(func, 2);
    t1.join();
    t2.join();
}
```
输出
```
0
0
1
Thread  Thread  2 start func
Thread 2 acquired the lock.
Thread 2 released the lock.
1 start func
Thread 1 acquired the lock.
Thread 1 released the lock.
```
两个线程通过 std::atomic_flag 来争夺一个资源（即打印信息），只有一个线程能够获得锁，执行相应的操作。另一个线程需要等待锁被释放才能继续执行。使用test_and_set() 和 clear() 函数来测试和设置标志位，保证这些操作是原子的。
### store函数
std::atomic<T>::store()用于将给定的值存储到原子对象中。
```
void store(T desired, std::memory_order order = std::memory_order_seq_cst) volatile noexcept;
void store(T desired, std::memory_order order = std::memory_order_seq_cst) noexcept;
```
- `desired`：要存储的值。
- `order`：存储操作的内存顺序。默认是std::memory_order_seq_cst（顺序一致性）

存储操作的内存顺序参数：
|value|内存顺序|描述|
|------|--------|-----|
|memory_order_relaxed|无序的内存访问|不做任何同步，仅保证该原子类型变量的操作是原子化的，并不保证其对其他线程的可见性和正确性。|
|memory_order_consume|与消费者关系有关的顺序|保证本次读取之前所有依赖于该原子类型变量值的操作都已经完成，但不保证其他线程对该变量的存储结果已经可见。|
|memory_order_acquire|获取关系的顺序|保证本次读取之前所有先于该原子类型变量写入内存的操作都已经完成，并且其他线程对该变量的存储结果已经可见。|
|memory_order_seq_cst|顺序一致性的顺序|保证本次操作以及之前和之后的所有原子操作都按照一个全局的内存顺序执行，从而保证多线程环境下对变量的读写的正确性和一致性。这是最常用的内存顺序。|
memory_order_release|释放关系的顺序|保证本次写入之后所有后于该原子类型变量写入内存的操作都已经完成，并且其他线程可以看到该变量的存储结果。
*详见简介 C++6种内存序*
示例
```
 std::atomic<int> atomic_int(0);
 std::cout << "Value stored in atomic object: " << atomic_int << std::endl;
 int val = 10;
 atomic_int.store(val); //默认memory_order_seq_cst 顺序一致性
 std::cout << "Value stored in atomic object: " << atomic_int << std::endl;
 val = 5;
 atomic_int.store(val);
 std::cout << "Value stored in atomic object: " << atomic_int << std::endl;
```
```
Value stored in atomic object: 0
Value stored in atomic object: 10
Value stored in atomic object: 5
```


在多线程环境下使用原子变量和操作时，需要使用适当的内存顺序来保证数据的正确性和一致性。因此，store()函数中的order参数可以用来指定不同的内存顺序。如果不确定如何选择内存顺序，请使用默认值std::memory_order_seq_cst，它是最常用和最保险的。

### load函数
load函数用于获取原子变量的当前值。它有以下两种形式：
```
T load(memory_order order = memory_order_seq_cst) const noexcept;
operator T() const noexcept;
```
其中，第一种形式是显式调用load函数，第二种形式是通过重载类型转换运算符实现隐式调用。

load函数的参数memory_order表示内存序，也就是对原子变量的读操作要遵循哪种内存模型。C++中定义了多种内存序，包括：
- memory_order_relaxed：最轻量级的内存序，不提供任何同步机制。
- memory_order_acquire：在本线程中，所有后面的读写操作必须在这个操作之后执行。
- memory_order_release：在本线程中，该操作之前的所有读写操作必须在这个操作之前执行。 
- memory_order_seq_cst：最严格的内存序，保证所有线程看到的读写操作的顺序都是一致的。
  
使用load函数时，如果不指定memory_order，则默认为memory_order_seq_cst。 <br>
load函数的返回值类型为T，即原子变量的类型。在使用load函数时需要指定类型参数T。如果使用第二种形式的load函数，则无需指定类型参数T，程序会自动根据上下文推断出类型。

示例
```
 std::atomic<int> atomic_int(10);
 int x = atomic_int.load(std::memory_order_relaxed); // get value atomically
 int xx = int(atomic_int.load());
 std::cout << x << std::endl;
 std::cout << xx << std::endl;
```
输出
```
10
10
```
### exchange函数
访问和修改包含的值，将包含的值替换并返回它前面的值。
```
template< class T >
T exchange( volatile std::atomic<T>* obj, T desired );
```
其中，obj参数指向需要替换值的atomic对象，desired参数为期望替换成的值。如果替换成功，则返回原来的值。

整个操作是原子的（原子读-修改-写操作）：从读取（要返回）值的那一刻到此函数修改值的那一刻，该值不受其他线程的影响。
用例
```

std::atomic<bool> ready(false);
std::atomic<bool> winner(false);
void count1m(int id) {
    while (!ready) {}                  // wait for the ready signal
    for (int i = 0; i < 1000000; ++i) {}   // go!, count to 1 million
    if (!winner.exchange(true)) { std::cout << "thread #" << id << " won!\n"; }
};

void _test() {
    //exchange
    std::vector<std::thread> threads;
    std::cout << "spawning 10 threads that count to 1 million...\n";
    for (int i = 1; i <= 10; ++i) threads.push_back(std::thread(count1m, i));
    ready = true;
    for (auto& th : threads) th.join();
}
```
输出
```
thread #6 won!
```
### compare_exchange_weak函数
这个函数的作用是比较一个值和一个期望值是否相等，如果相等则将该值替换成一个新值，并返回true；否则不做任何操作并返回false。

>bool compare_exchange_weak (T& expected, T val,memory_order sync = memory_order_seq_cst) volatile noexcept;    
bool compare_exchange_weak (T& expected, T val,memory_order sync = memory_order_seq_cst) noexcept;  
bool compare_exchange_weak (T& expected, T val,memory_order success, memory_order failure) volatile noexcept;   
bool compare_exchange_weak (T& expected, T val,memory_order success, memory_order failure) noexcept;

参数说明
- expected：期望值的地址，也是输入参数，表示要比较的值；
- val：新值，也是输入参数，表示期望值等于该值时需要替换的值；
- success：表示函数执行成功时内存序的类型，默认为memory_order_seq_cst；
- failure：表示函数执行失败时内存序的类型，默认为memory_order_seq_cst。

该函数的返回值为bool类型，表示操作是否成功。

注意，compare_exchange_weak函数是一个弱化版本的原子操作函数，因为在某些平台上它可能会失败并重试。如果需要保证严格的原子性，则应该使用compare_exchange_strong函数。
示例：
```
#include <iostream>       // std::cout
#include <atomic>         // std::atomic
#include <thread>         // std::thread
#include <vector>         // std::vector

// a simple global linked list:
struct Node { int value; Node* next; };
std::atomic<Node*> list_head (nullptr);

void append (int val) {     // append an element to the list
  Node* oldHead = list_head;
  Node* newNode = new Node {val,oldHead};

  // what follows is equivalent to: list_head = newNode, but in a thread-safe way:
  while (!list_head.compare_exchange_weak(oldHead,newNode))
    newNode->next = oldHead;
}

int main ()
{
  // spawn 10 threads to fill the linked list:
  std::vector<std::thread> threads;
  for (int i=0; i<10; ++i) threads.push_back(std::thread(append,i));
  for (auto& th : threads) th.join();

  // print contents:
  for (Node* it = list_head; it!=nullptr; it=it->next)
    std::cout << ' ' << it->value;
  std::cout << '\n';

  // cleanup:
  Node* it; while (it=list_head) {list_head=it->next; delete it;}

  return 0;
}
``` 
###  compare_exchange_strong函数
这个函数的作用和compare_exchange_weak类似，都是比较一个值和一个期望值是否相等，并且在相等时将该值替换成一个新值。不同的是，compare_exchange_strong会保证原子性，并且如果比较失败则会返回当前值。
>bool compare_exchange_strong(T& expected, T desired,  
                             memory_order success = memory_order_seq_cst,  
                             memory_order failure = memory_order_seq_cst) noexcept;

- expected：期望值的地址，也是输入参数，表示要比较的值；
- desired：新值，也是输入参数，表示期望值等于该值时需要替换的值；
- success：表示函数执行成功时内存序的类型，默认为memory_order_seq_cst；
- failure：表示函数执行失败时内存序的类型，默认为memory_order_seq_cst。

该函数的返回值为bool类型，表示操作是否成功。

注意，compare_exchange_strong函数保证原子性，因此它的效率可能比compare_exchange_weak低。在使用时应根据具体情况选择适合的函数。

### 专业化支持的操作
| | |
|-|-|
|fetch_add|添加到包含的值并返回它在操作之前具有的值|
|fetch_sub|从包含的值中减去，并返回它在操作之前的值|
|fetch_and|读取包含的值，并将其替换为在读取值和 之间执行按位 AND 运算的结果|
|fetch_or|读取包含的值，并将其替换为在读取值和 之间执行按位 OR 运算的结果|
|fetch_xor|读取包含的值，并将其替换为在读取值和 之间执行按位 XOR 运算的结果|
使用示例
```
// atomic::load/store example
#include <iostream> // std::cout
#include <atomic> // std::atomic, std::memory_order_relaxed
#include <thread> // std::thread
//std::atomic<int> count = 0;//错误初始化
std::atomic<int> count(0); // 准确初始化
void set_count(int x)
{
	std::cout << "set_count:" << x << std::endl;
	count.store(x, std::memory_order_relaxed); // set value atomically
}
void print_count()
{
	int x;
	do {
		x = count.load(std::memory_order_relaxed); // get value atomically
	} while (x==0);
	std::cout << "count: " << x << '\n';
}
int main ()
{
	std::thread t1 (print_count);
	std::thread t2 (set_count, 10);
	t1.join();
	t2.join();
	std::cout << "main finish\n";
	return 0;
}
```
## 总结
原子操作在多线程中可以保证线程安全，而且效率会比互斥量好些。

原子变量支持的基本操作有：

- 加法：a += n或者a.fetch_add(n)
- 减法：a -= n或者a.fetch_sub(n)
- 与、或、异或运算：a &= b、a |= b、a ^= b或者a.fetch_and(b)、a.fetch_or(b)、a.fetch_xor(b)
- 自增、自减运算：++a、--a、a++、a--或者a.fetch_add(1)、a.fetch_sub(1)
- 交换：a.exchange(b)返回原来的值，将a设置为b
- 比较并交换：a.compare_exchange_strong(b, c)或者a.compare_exchange_weak(b, c)，如果a的值等于b，则将a设置为c，返回true，否则返回false。

尽管原子变量是多线程编程中非常重要的同步机制，但是它也存在一些局限性。具体来说，原子变量只能保证单个变量的原子性操作，而不能保证多个变量之间的同步。此外，原子变量也无法解决数据竞争（data race）等问题，因此在使用时需要注意避免这些问题的出现。