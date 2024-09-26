---
title: C++多线程模型
date: 2023-10-10 06:47:41
categories: [CPP]
tags: [CPP]
---

## 线程同步和线程通信

### 线程同步

1. 互斥锁
2. 条件变量
3. 互斥量
4. 信号量

### 线程通信

1. 条件变量
2. 队列
3. 原子操作
3. 条件变量和定时器

## C++标准库提供了哪些锁？

`std::mutex`：互斥锁是最基本的锁类型，用于确保一次只有一个线程可以访问共享资源。你可以使用 std::mutex 来创建一个互斥锁对象，然后使用 lock() 和 unlock() 方法来手动锁定和解锁。

`std::unique_lock：`std::unique_lock 是一个更加灵活的互斥锁包装器，它允许你在需要时手动锁定和解锁，也可以在构造函数和析构函数中自动锁定和解锁。

`std::lock_guard：`std::lock_guard 是另一个互斥锁包装器，但它只支持自动锁定和解锁。一旦 std::lock_guard 对象被创建，它会自动锁定互斥锁，并在其生命周期结束时自动解锁。

`std::recursive_mutex：`递归互斥锁允许同一线程多次获得锁。这对于某些特定的情况很有用，但要小心避免死锁。

`std::shared_mutex：`共享互斥锁（C++17引入）允许多个线程同时读取共享资源，但只有一个线程可以写入资源。这有助于提高并发性能。

`std::condition_variable：`条件变量用于线程之间的同步和通信。它们允许一个线程等待某个条件成立，然后另一个线程发出信号以通知等待线程。



## 实现线程的交替打印：

```cpp
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <thread>

const int NUM_THREADS = 3;
const int NUM_ITERATTIONS = 5;

std::mutex mtx; // 锁，用于线程同步
std::condition_variable cv; // 条件变量，用于线程通信
int current_thread = 0;

void print(int thread_num, const std::string& messages) {
    for (int i = 0; i < NUM_ITERATTIONS; ++i) {
        // 获取锁，进入临界区
        std::unique_lock<std::mutex> lock(mtx);

        // 如果不是当前线程，就等待
        while (current_thread != thread_num) {
            cv.wait(lock); // 释放锁，允许其他线程进入临界区
        }
        // 打印信息
        std::cout << messages << std::endl;

        // 更改当前线程的编号
        current_thread = (current_thread + 1) % NUM_THREADS;

        // 通知下一个线程开始打印
        cv.notify_all(); // 这里并没有释放锁，当前线程仍然持有锁
    }
}

int main() {
    std::thread threads[NUM_THREADS];
    for (int i = 0; i < NUM_THREADS; ++i) {
        threads[i] = std::thread(print, i, "Thread " + std::to_string(i));
    }
    for (int i = 0; i < NUM_THREADS; ++i) {
        threads[i].join();
    }
}
```

```bash
g++ test.cpp -o test -pthread
```

## 问题

### 为什么使用条件变量唤醒的时候要用while而不用if？
条件变量在等待期间可能会收到虚假唤醒（spurious wakeups），也就是在条件尚未满足时，线程被唤醒。

虚假唤醒是由于操作系统或底层线程库的实现细节引起的，他们可能会偶尔导致条件变量的信号量被错误唤醒。所以要循环检查，直到条件不满足。