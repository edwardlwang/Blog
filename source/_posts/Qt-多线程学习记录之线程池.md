---
title: Qt 多线程使用
date: 2026-07-14 16:41:00
tags: [Qt, C++, multithreading]
categories: Qt
---

在进行桌面应用程序开发的时候，假设应用程序在某些情况下需要处理比较复杂的逻辑，如果只有一个线程去处理，就会导致窗口卡顿，无法处理用户的相关操作。这种情况下就需要使用多线程，其中一个线程处理窗口事件，其他线程进行逻辑运算，多个线程各司其职，不仅可以提高用户体验还可以提升程序的执行效率。

在 qt 中使用了多线程，有些事项是需要额外注意的：

- 默认的线程在 qt 中称之为窗口线程，也叫主线程，负责窗口事件处理或者窗口控件数据的更新
- 子线程负责后台的业务逻辑处理，子线程中不能对窗口对象做任何操作，这些事情需要交给窗口线程处理
- 主线程和子线程之间如果要进行数据的传递，需要使用 qt 中的信号槽机制

## 1 线程类 QThread

### 1.1 常用共用成员函数

Qt中提供了一个线程类。通过这个类就可以创建子线程，Qt一共提供了两种创建子线程的方式。

```cpp
// QThread 类常用 API
// 构造函数
QThread::QThread(QObject *parent = Q_NULLPTR);

// 判断线程中的任务是不是处理完毕了
bool QThread::isFinished() const;
// 判断子线程是不是在执行任务
bool QThread::isRunning() const;

// Qt中的线程可以设置优先级
// 得到当前线程的优先级
Priority QThread::priority() const;
void QThread::setPriority(Priority priority);

优先级枚举：
QThread::IdlePriority         --> 最低的优先级
QThread::LowestPriority
QThread::LowPriority
QThread::NormalPriority
QThread::HighPriority
QThread::HighestPriority
QThread::TimeCriticalPriority
QThread::InheritPriority     --> 默认优先级，最高优先级

// 退出线程，停止底层的事件循环
// 退出线程的工作函数
void QThread::exit(int returnCode = 0);

// 调用线程退出函数之后，线程不会马上退出因为当前任务有可能还没有完成，调用这个函数是
// 等待任务完成，然后退出线程，一般情况下会在 exit() 后边调用这个函数
bool QThread::wait(unsigned long time = ULONG_MAX);
```

### 1.2 信号槽

```cpp
// 和调用 exit() 效果是一样的
// 调用这个函数之后，再调用 wait() 函数
[slot] void QThread::quit();

// 启动子线程
[slot] void QThread::start(Priority priority = InheritPriority);

// 线程退出，可能是会马上终止线程，一般情况下不使用这个函数
[slot] void QThread::terminate();

// 线程中执行的任务完成了，发出该信号
// 任务函数中的处理逻辑执行完毕了
[signal] void QThread::finished();

// 开始工作之前发出这个信号，一般不使用
[signal] void QThread::started();
```

### 1.3 静态函数

```cpp
// 返回一个指向管理当前执行线程的QThread的指针
[static] QThread *QThread::currentThread();

// 返回可以在系统上运行的理想线程数 == 和当前电脑的 CPU 核心数相同
[static] int QThread::idealThreadCount();

// 线程休眠函数
[static] void QThread::msleep(unsigned long msecs);    // 单位: 毫秒
[static] void QThread::sleep(unsigned long secs);     // 单位: 秒
[static] void QThread::usleep(unsigned long usecs);    // 单位: 微秒
```

### 1.4 任务处理函数

```cxx
// 子线程要处理什么任务，需要写到run()中
[virtual protected] void QThread::run();
```

这个 run() 是一个虚函数，如果想让创建的子线程执行某个任务，需要写一个子类让其继承 QThread，并且在子类中重写父类的 run() 方法，函数体就是对应的任务处理流程。另外，这个函数是一个受保护的成员函数，不能够在类的外部调用，如果想要让线程执行这个函数中的业务流程，需要通过当前线程对象调用槽函数 start() 启动子线程，当子线程被启动，这个 run() 函数就在线程内部被调用了。
