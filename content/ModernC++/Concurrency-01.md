---
title: "Modern C++ Concurrency 01 - std::thread"
date: 2022-11-21T22:14:43+08:00
tags: ["C++", "Concurrency", "Note", "Thread"]
draft: False
---

现代C++并发编程学习01 - std::thread

`std::thread`是实现并发编程最基础的工具，使用简单，易于理解。
使用`std::thread`只需要直接实例化一个`std::thread`就可以了，可以传入函数，也可以传入lambda
```cpp
/* 传入函数到线程 */
void foo() {
    std::cout << "foo()" << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(20));
}

std::thread foo_thread(foo);
foo_thread.join();
// foo()

/* 传入函数到线程，并传入参数 */
void bar(int i) {
    i += 1;
    std::cout << "bar(), i += 1: " << i << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(300));
}
int i = 2;
std::thread bar_thread(bar, i);
bar_thread.join();
// bar(), i += 1: 3

/* 传入lambda到线程 */
std::thread lambda_thread([]() {
    std::cout << "this is lambda_thread" << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(20));
});

lambda_thread.join();
//this is lambda_thread

```

只需要在线程对象调用`join()`方法，就可以等待线程执行结果。如果不调用`join()`，则是未定义行为。
即，线程对象的持有者可能先于线程结束。

`std::thread`虽然简单，但是并不常用，一个最直接的问题是，我们需要把一些过程放到线程上异步执行，但是又希望得到执行结果。
例如开一个线程执行计算，需要获取计算结果。
使用`std::thread`并不能直接返回计算结果，一个可以尝试的方法是，传入引用，利用引用机制从参数返回结果。
```cpp
int j = 4;
/* 注意参数是引用类型 */
void bar_ref(int &i) {
    i += 1;
    std::cout << "bar_ref(), i += 1: " << i << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
}
/* 构造线程对象，传入函数方式不变，传入引用类型参数需要使用std::ref */
std::thread bar_ref_thread(bar_ref, std::ref(j));
bar_ref_thread.join();

std::cout << "j = " << j << std::endl;
// j = 5
```

虽然可以实现获取线程执行结果的返回值，但是这样并不方便使用。
`std::thread`也不是没用，不然在C++11之后的各个版本里面就会废弃掉。最多用到的场景是：线程池。

关于线程池，放到后面再说。
