---
title: "std::future"
date: 2022-09-10T21:24:46+08:00
tags: ['c++', 'std', 'future']
categories: ['c&c++']
draft: false
---

## 功能
类模板`std::future`提供了一个访问异步操作结果的机制:
- 异步操作(通过`std::async`、`std::packaged_task`或`std::promise`创建)可以向该异步操作的创建者提供`std::future`对象。
- 然后，异步操作的创建者可以使用各种方法来`查询`、`等待`或从`std::future`中提取值。**如果异步操作尚未提供值，这些方法可能会阻塞。**
- 当异步操作准备好向创建者发送结果时，它可以通过修改链接到创建者的`std::future`的共享状态(例如`std::promise::set_value`)来实现。

## 定义
`std::funture`定义在头文件 `<future>` 中

### 构造
```c++
templete<class T> class future;             // since c++11
templete<class T> class future<T&>;         // since c++11
templete<> class future<void>;              // since c++11
```

### 成员方法

|操作|说明|
|:---|:---|
|`operator=`|移动`future`对象|
|`share`|将共享状态从`*this`转移到`shared_future`并返回|
|`get`|返回结果|
|`valid`|检查future是否具有共享状态|
|`wait`|等待结果可用|
|`wait_for`|等待结果，如果在指定的超时时间内不可用则返回|
|`wait_until`|等待结果，如果在到达指定的时间点之前它不可用，则返回|

## 例子

```c++
#include <iostream>
#include <future>
#include <thread>

int main ()
{
    // future from a packaged_task
    std::packaged_task<int()> task([]{ return 7; });        // wrap the function
    std::future<int> f1 = task.get_future();                // get a future
    std::thread t(std::move(task));                         // launch on a thread

    // future from an async()
    std::future<int> f2 = std::async(std::launch::async, [] { return 8; });

    // future from a promise
    std::promise<int> p;
    std::future<int> f3 = p.get_future();
    std::thread( [&p]{ p.set_value_at_thread_exit(9); }).detach();

    std::cout << "Waiting..." << std::flush;
    f1.wait();
    f2.wait();
    f3.wait();
    std::cout << "Done!\nResults are: " << f1.get() << ' ' << f2.get() << ' ' << f3.get() << '\n';
    t.join();
}
```

输出结果:
```
Waiting...Done!
Results are: 7 8 9
```

```c++
#include <thread>
#include <iostream>
#include <future>

int main()
{
    std::promise<int> p;
    std::future<int> f = p.get_future();

    std::thread t([&p]{
        try {
            // code that may throw
            throw std::runtime_error("Example");
        } catch(...) {
            try {
                // store anything thrown in the promise
                p.set_exception(std::current_exception());
            } catch(...) {} // set_exception() may throw too
        }
    });

    try {
        std::cout << f.get();
    } catch(const std::exception& e) {
        std::cout << "Exception from the thread: " << e.what() << '\n';
    }
    t.join();
}
```

输出结果
```c++
Exception from the thread: Example
```

