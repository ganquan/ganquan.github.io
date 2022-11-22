---
title: "Concurrency 02 - std::future std::promise"
date: 2022-11-22T09:34:10+08:00
tags: ["C++", "Concurrency", "Note", "Thread", "Promise", "Future"]
weight: 20
draft: false
---

上一篇举例说明了`std::thread`不方便获取线程执行信息，直白的讲，就是线程通信不够方便，所以就引入了`std::promise`和`std::future`。

`std::promise`用于存储异步行为中的信息，`std::future`用于获取这些信息。
一个也许不是很恰当的理解，可以这么看：
`std::promise`和`std::future`一起提供了一种单向的线程间通信机制，`std::promise`是这种通信机制的输入，`std::future`是这种通信机制的输出。

下面的例子比较直观的说明这种通信：
```cpp
void bar_promise(std::promise<int> &promise) {
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    promise.set_value(42);
}

int main() { 
    std::promise<int> p;
    std::thread bar_promise_thread(bar_promise, std::ref(p));
    bar_promise_thread.join();
    auto future = p.get_future();
    std::cout << "future = " << future.get() << std::endl;
    // future == 42

    return 0;
}
```
future.get()会阻塞等待promise的值准备好：
```cpp
/* future.get() 会阻塞等待直到future有值 */
int main() { 
    std::promise<int> p;
    std::thread bar_promise_thread(bar_promise, std::ref(p));
    bar_promise_thread.detach();
    /* bar_promise_thread.join(); */
    std::future<int> future = p.get_future();
    std::cout << "future = " << future.get() << std::endl;
    // future == 42

    return 0;
}

```

如果在线程中没有调用promise的set_value，那么在调用future.get()方法时，就会一直阻塞住调用者。

到这里来看，`std::promise`和`std::future`机制，在书写上，也没有比通过引用获取值的方式要简化多少。

但是Modern C++提供了另外一个工具，来更进一步方便这种通信方式的书写，就是`std::packaged_task`。
语言手册的解释是，`std::packaged_task`封装了任意的可调用(Callable)对象以异步执行，例如function、lambda、 bind expression以及其他函数对象。
`std::packaged_task`提供了get_future方法用于获取异步执行的结果。

到这里一个对比和总结就显现出来了，即：

`std::promise`和`std::packaged_task`都提供get_future方法，但是`std::promise`只是对数据的封装，而`std::packaged_task`则实现了对异步过程的封装。

所以，`std::packaged_task`的抽象程度，一下子就脱离了基础的promise和future，在抽象层次上更高了。

看下面这个例子：
```cpp

int main() {

    std::packaged_task<int(int, int)> task([](int a, int b) {
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
        return a + b;
    });

    auto future_task = task.get_future();
    task(5, 8);

    std::cout << "future = " << future_task.get() << std::endl;
    //future = 13

    return 0;
}
```

没有了`std::promise`相关的代码，书写上就变得简洁不少。


