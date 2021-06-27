---
title: python多线程同步
top: false
cover: false
toc: true
mathjax: true
date: 2021-06-26 12:46:40
password:
summary:
tags:
categories:
---

## Condition相关函数

**acquire()** — 线程锁，注意线程条件变量Condition中的所有相关函数使用必须在**acquire()** **/release()** 内部操作；

**release()** — 释放锁；

**wait(timeout)** — 线程挂起(阻塞状态)，直到收到一个notify通知或者超时才会被唤醒继续运行（超时参数默认不设置，可选填，类型是浮点数，单位是秒）。wait()必须在已获得Lock前提下才能调用，否则会触发RuntimeError；

**notify(n=1)** — 通知其他线程，那些挂起的线程接到这个通知之后会开始运行，缺省参数，默认是通知一个正等待通知的线程,最多则唤醒n个等待的线程。notify()必须在已获得Lock前提下才能调用，否则会触发RuntimeError，notify()不会主动释放Lock；

**notifyAll()** — 如果wait状态线程比较多，notifyAll的作用就是通知所有线程；

## 实操

### 两个线程交替打印1-10

```python
import threading

x = 0
con = threading.Condition()

def print_num(mode):
    global x
    con.acquire()
    while x < 100:
        if x % 2 == mode:
            con.wait()
        print(threading.get_ident(),x)
        x += 1
        con.notify()
    con.release()

if __name__ == '__main__':
    t1 = threading.Thread(target=print_num, args=[0])
    t2 = threading.Thread(target=print_num, args=[1])
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```

### 三个线程同时启动，优先级相同，分别打印A、B、C，保证打印顺序为ABC。

```python
import threading

x = 0
flag = ['A', 'B', 'C']
con = threading.Condition()

def print_char(to_print):
    global flag
    global x
    con.acquire()
    while flag[x] != to_print:
        # print(flag[x]+'!='+to_print)
        con.wait()
    print(flag[x])
    x += 1
    # 需要通知所有其他线程
    con.notifyAll()
    con.release()

if __name__ == '__main__':
    t1 = threading.Thread(target=print_char, args=['A'])
    t2 = threading.Thread(target=print_char, args=['B'])
    t3 = threading.Thread(target=print_char, args=['C'])
    t3.start()
    t2.start()
    t1.start()
    t1.join()
    t2.join()
    t3.join()
```

