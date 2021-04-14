---
title: python面试
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-13 19:58:48
password:
summary:
tags:
- python
- interview
categories:
- python
- interview
---

## 语言特性

解释型语言。Python不需要在运行之前进行编译。

动态语言，不需要声明变量的类型。

适合面向对象的编程，因为它允许类的定义以及组合和继承。

## Python基础

### 魔法方法

魔法方法就会在特殊的情况下被Python所调用，你可以定义自己想要的行为，而这一切都是自动发生的，它们经常是两个下划线包围来命名的（比如`__init___`,`__len__`)

`__init__`构造器，当一个实例被创建的时候初始化的方法，但是它并不是实例化调用的第一个方法。

`__new__`才是实例化对象调用的第一个方法，它只取下cls参数，并把其他参数传给`__init___`.

`__call__`让一个类的实例像函数一样被调用

`__getitem__`定义获取容器中指定元素的行为，相当于self[key]

`__getattr__`定义当用户试图访问一个不存在属性的时候的行为。

`__setattr__`定义当一个属性被设置的时候的行为

`__getattribute___`定义当一个属性被访问的时候的行为

`__del__`:删除对象执行的方法

`__str__`强调可读性，而`__repr__`强调准确性/标准性，

`__str__`的目标人群是用户，而`__repr__`的目标人群是机器，它的结果是可以被执行的；

%s调用`__str__`方法，而%r调用`__repr__`方法

`__repr__`在表示类时，是一级的，如果只定义它，那么`__str__` = `__repr__`。

而`__str__`展示类时是次级的，用户可自定义类的展示格式，如果没有定义`__repr__`，那么repr(person)将会展示缺省的定义。

### is 和 ==区别

is：比较的是两个对象的id值是否相等，也就是比较俩对象是否为同一个实例对象。是否指向同一个内存地址。

== ： 比较的两个对象的内容/值是否相等，默认会调用对象的`eq()`方法

### set去重

set的去重是通过两个函数`__hash__`和`__eq__`结合实现的。

1、当两个变量的哈希值不相同时，就认为这两个变量是不同的

2、当两个变量哈希值一样时，调用`__eq__`方法，当返回值为True时认为这两个变量是同一个，应该去除一个。返回FALSE时，不去重

set和Dict类似，均为哈希表。因此去重应该是生成hash表的过程。

### list切片

索引操作本身基于`__getitem__`和`__setitem__`

python向`__getitem__`传入了一个slice类的对象，这个类有start, stop, step三个属性，缺省值都是None。slice是Python的一个内置类

```python
a = [1,2,3,4,5,6]
x = a [1: 5]        # x = a.__getitem__(slice( 1, 5, None))
a [1: 3] = [10, 11, 12]  # a.__setitem__(slice(1, 3, None), [ 10, 11, 12 ])
del a [1: 4]        # a.__delitem__(slice(1, 4, None))
```

### 三元算子

[on true] if [expression] else [on false]

### pass

1、pass语句什么也不做，一般作为占位符或者创建占位程序，pass语句不会执行任何操作

2、保证格式、语义完整 

### 迭代器和生成器

**迭代器协议**：对象需要提供next方法，它要么返回迭代中的下一项，要么就引起一个StopIteration异常，以终止迭代。

**迭代器对象**：实现了迭代器协议的对象，表示的是一个数据流，**惰性计算**的，在需要返回下一个数据时它才会计算。

**使用**

把一个类作为一个迭代器使用需要在类中实现两个方法 `__iter__()`与 `__next__() `。

`__iter__() `方法返回一个特殊的迭代器对象， 这个迭代器对象实现了` __next__() `方法并通过 StopIteration 异常标识迭代的完成。

`__next__()` 方法（Python 2 里是 `next()`）会返回下一个迭代器对象。

**例子**

Python的内置工具（如for循环，sum，min，max函数等）使用迭代器协议访问对象



Python的内置工具(如for循环，sum，min，max函数等)使用迭代器协议访问对象

**使用了 yield 的函数被称为生成器**。**只要把一个列表生成式的[]改成()，就创建了一个生成器**

生成器是一个返回迭代器的函数，只能用于迭代操作，更简单点理解生成器就是一个迭代器。

### 缺省参数

缺省参数指在调用函数的时候没有传入参数的情况下，**调用默认的参数**，在调用函数的同时赋值时，所传入的参数会替代默认参数。

*args是不定长参数，它可以表示输入参数是不确定的，可以是任意多个。

**kwargs是关键字参数，赋值的时候是以键值对的方式，参数可以是任意多对在定义函数的时候

**位置参数：**就是在给函数传参数时，按照顺序，依次传值。

**默认参数：**就是在写函数的时候直接给参数传默认的值，调用的时候，默认参数已经有值，就不用再传值了。**默认参数一定要指向不变对象。**

**可变参数：**可变参数就是传入的参数个数是可变的，可以是0个，1个，2个，……很多个。特征：*args

**关键字参数：**允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。在调用函数时，可以只传入必选参数：特征：**kw

**命名关键字参数：**限制要传入的参数的名字，只能传我已命名关键字参数。特征：命名关键字参数需要一个特殊分隔符*，而后面的参数被视为命名关键字参数。如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符*了

参数定义的**顺序**必须是：位置参数–>默认参数–>可变参数–>命名关键字参数–>关键字参数

### zip

zip 拉链函数，

\# 将对象中对应的元素打包成一个个元组，

\# 然后返回由这些元组组成的列表迭代器。

\# 如果各个迭代器的元素个数不一致，则返回列表长度与最短的对象相同。

print(list(zip([0,1,3],[5,6,7],['a','b'])))

\# [(0, 5, 'a'), (1, 6, 'b')]

### and or

 在不加括号时候, and优先级大于or 

x or y 的值只可能是x或y. x为真就是x, x为假就是y 

x and y 的值只可能是x或y. x为真就是y, x为假就是x

v = 1 and 2 or 3 and 4 

print(v) 

 \# >>> 2 

## 面向对象

继承：将多个类的共同属性和方法封装到一个父类下面，然后在用这些类来继承这个类的属性和方法

封装：将有共同的属性和方法封装到同一个类下面

多态：Python天生是支持多态的。指的是基类的同一个方法在不同的派生类中有着不同的功能

## 字典

字典的查询、添加、删除的平均时间复杂度都是`O(1)`。因为字典是通过哈希表来实现的.

哈希算法不可避免的问题就是hash冲突，Python字典发生哈希冲突时，**会向下寻找空余位置，直到找到位置**。如果在计算key的hash值时，如果一直找不到空余位置，则字典的时间复杂度就变成了`O(n)`了，所以Python的**哈希算法**就显得非常重要了。

**python3.6之前**

- 计算key的hash值`hash(key)`，再和mask做与操作【mask=字典最小长度（DictMinSize） - 1】，运算后会得到一个数字【index】，这个index就是要插入的enteies哈希表中的下标位置

- 若index下标位置已经被占用，则会判断enteies的key是否与要插入的key是否相等

  - 如果key相等就表示key已存在，则更新value值

  - 如果key不相等，就表示hash冲突，则会继续向下寻找空位置，一直到找到剩余空位为止。

### **开放寻址法**

开放寻址法中，所有的元素都存放在散列表里，当产生哈希冲突时，通过一个探测函数计算出下一个候选位置，如果下一个获选位置还是有冲突，那么**不断通过探测函数往下找**，直到找个一个空槽来存放待插入元素。

采用哈希表，dict的哈希表里每个slot都是一个自定义的entry结构：

```c
typedef struct {
   Py_ssize_t me_hash;
   PyObject *me_key;
   PyObject *me_value;
} PyDictEntry;
```

每个entry有三种状态Active, Unused, Dummy。

- Unused:me_key == me_value == NULL，即未使用的空闲状态。

- Active:me_key != NULL, me_value != NULL，即该entry已被占用

- Dummy:me_key == dummy, me_value == NULL。

**为什么entry有Dummy状态呢？**

这是因为采用开放寻址法中，**遇到哈希冲突时会找到下一个合适的位置，**例如某元素经过哈希计算应该插入到A处，但是此时A处有元素的，通过探测函数计算得到下一个位置B，仍然有元素，直到找到位置C为止，此时ABC构成了探测链，查找元素时如果hash值相同，那么也是**顺着这条探测链不断往后找**，当删除探测链中的某个元素时，比如B，如果直接把B从哈希表中移除，即**变成Unused状态，那么C就不可能再找到了**，因为AC之间出现了断裂的现象，正是如此才出现了第三种状态Dummy，Dummy是一种类似的伪删除方式，保证探测链的连续性。

dict对象的定义为：

```c
struct _dictobject {
  PyObject _HEAD
  Py_ssize_t ma_fill; /* # Active + # Dummy */
  Py_ssize_t ma_used; /* # Active */
  Py_ssize_t ma_mask;
  PyDictEntry *ma_table;
  PyDictEntry *(*ma_lookup)(PyDictObject *mp, PyObject *key, long hash);
  PyDictEntry ma_smalltable[PyDict_MINSIZE];
};
```

**ma_fill记录Active + Dummy状态的entry数。**

**ma_used记录Active状态的entry数。**

**ma_mask等于slot总数 - 1。**

ma_smalltable是默认的slot，初始有PyDict_MINSIZE个。

ma_table初始指向ma_smalltable，如果后期扩容，则指向新的slot空间。ma_lookup为搜索函数指针。

因为一个key的哈希值很可能超过slot总数，所以作为索引时得把它约束在slot总数的范围内。而slot总数在定义的时候必须是2的乘幂，比如0x1000，所以减1之后就成了mask：0x111。再和hash做个&操作就能把索引之限制在0~0x111之间.

### **dict对象的创建**

**dict对象的创建很简单，先看看缓冲的对象池里有没有可用对象，如果有就直接用，没有就从堆上申请。**

- 计算出需要生成的字典的大小
- 初始化一个 PyDictKeysObject
  - 检查是否有缓存的字典数据可用，如果没有则申请内存重新生成一个 dk
  - 申请到的内存将内容清空
- 生成一个 PyDictObject 返回
  -  keys，values 设置到从缓冲池或者新生成一个 dict 对象

### **dict对象的插入**

如果不存在key-value则插入，存在则覆盖。

- 生成Hash
- 如果可用的entry<0，字典扩容
- 基于key、hash，查找可用哈希位置，以便于存储
  - 字典中是否有空余的值，或者如果找到了满足 hash 值与 key 相同的就将 value 设置为找到的值
- 保存key、Hash、value值

### **dict对象的删除**

dict里entry的删除更简单，算出哈希值，找到entry，将其从Active转换成Dummy，并调整table的容量。

**注意**

（1） dict的key 或者 set的值都必须是可hash的不可变对象，都是可hash的

（2）dict的内存花销大（hash映射，不可能是连续的存在内存空间中的，总有一些内存时空的，当发现内存空间中的“空”只有1/3时，便会触发扩容操作，以免引起hash冲突），但是查询速度快。

## 内存回收机制

python采用的是引用计数机制为主，标记-清除和分代收集两种机制为辅的策略。

###  **引用计数法**

**原理：**每个对象维护一个ob_ref字段，用来记录该对象当前被引用的次数，每当新的引用指向该对象时，它的引用计数ob_ref加1，每当该对象的引用失效时计数ob_ref减1，一旦对象的引用计数为0，该对象立即被回收，对象占用的内存空间将被释放。

**缺点：**不能解决对象的循环引用

### **标记清除**

Python采用了“标记-清除”算法，解决容器对象可能产生的循环引用问题。（只有容器对象才会产生循环引用的情况，比如列表、字典、用户自定义类的对象、元组等。只包含简单类型的元组不在标记清除算法的考虑之列）

**A）标记阶段，遍历所有的对象，如果是可达的（reachable），也就是还有对象引用它，那么就标记该对象为可达**；

B）清除阶段，再次遍历对象，如果发现某个对象没有标记为可达，则就将其回收。

### **分代回收**

标记清除时，应用程序会被暂停，为了减少应用程序暂停的时间。

**对象存在时间越长，越可能不是垃圾，应该越少去收集**。

在执行标记-清除算法时可以有效减小遍历的对象数，从而提高垃圾回收的速度。

给对象定义了三种世代，每一个新生对象在generation zero中，如果它在一轮gc扫描中活了下来，那么它将被移至generation one,在那里他将较少的被扫描，如果它又活过了一轮gc,它又将被移至generation two，在那里它被扫描的次数将会更少。

### **gc的扫描在什么时候会被触发呢**?

新创建的对象都会分配在年轻代，年轻代链表的总数达到上限时，Python垃圾收集机制就会被触发，把可回收的对象回收掉，而那些不会回收的对象就会被移到中年代去。

当某一世代的扫描被触发的时候，比该世代年轻的世代也会被扫描。

### **调优手段**

1.手动垃圾回收

2.调高垃圾回收阈值

3.避免循环引用

## 协程，线程和进程

### 进程

进程是系统资源分配的最小单位，进程拥有自己独立的内存空间，所有进程间数据不共享，开销大。在Python中，进程适合计算密集型任务。

#### 进程间的通信（IPC）

**1）管道（Pipe**）：通过send()和recv()来发送和接受信息，适合父子进程关系或者两个子进程之间。 

2）**有名管道（FIFO）**：有名管道也是半双工的通信方式，管道是先进先出的通信方式。 它会将自己注册到文件系统里一个文件，参数通信的进程通过读写这个文件进行通信。允许在没有亲缘关系的进程之间使用。要求读写双方必须同时打开才可以继续进行读写操作，否则打开操作会堵塞直到对方也打开。

3）**信号量（Semaphore）**：信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。 创建子进程时将信号量my_semaphore作为参数传入子进程任务函数，子进程中使用semaphore.acquire() 尝试获取信号量，semaphore.release()尝试 释放信号量。

**4）队列（Queue）**。 使用get/put在父子进程关系或者两个子进程之间通信。

5）**信号 ( signal )** ：用于通知接收进程某个事件已经发生，可以设置信号处理函数。 

6）共享内存( shared memory ) ：操作系统负责将同一份物理地址的内存映射到多个进程的不同的虚拟地址空间中。进而每个进程都可以操作这份内存。考虑到物理内存的唯一性，它属于临界区资源，需要在进程访问时搞好并发控制，比如使用信号量。我们通过一个信号量来控制所有子进程的顺序读写共享内存。 python标准库mmap，apache开源的pyarrow都是。

7）套接字( socket ) ：套接字也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同机器间的进程通信。 

**8）文件** 

（1）仅进程同步不涉及数据传输，可以使用信号、信号量；
（2）若进程间需要传递少量数据，可以使用管道、有名管道、队列；
（3）若进程间需要传递大量数据，最佳方式是使用共享内存，推荐使用pyarrow，这样减少数据拷贝、传输的时间内存代价；
（4）跨主机的进程间通信（RPC）可以使用socket通信。

**共享变量**

使用 Process 定义的多进程之间（父子或者兄弟）共享变量可以直接使用 multiprocessing 下的 Value，Array，Queue 等，如果要共享 list，dict，可以使用强大的 Manager 模块。

###  线程

线程是cpu调度执行的最小单位，依赖进程存在，一个进程至少有一个线程。在python中，线程适合IO密集型任务。

而多个线程共享内存（数据共享，共享全局变量)，从而极大地提高了程序的运行效率。

同一个进程下的线程共享程序的内存空间**（如代码段，数据集，堆栈等）**

#### 使用

继承Thread，重写run方法，通过start方法开线程

将要执行的方法作为参数传给Thread的构造方法

#### 状态

线程有五种状态:创建、就绪、运行、阻塞、死亡。 

- 调用start方法时，线程就会进入就绪状态。 

- 在线程得到cpu时间片时进入运行状态。 

- **线程调用yield方法可以让出cpu时间回到就绪状态**。 

- 线程运行时可能**由于IO、调用sleep、wait、join方法或者无法获得同步锁等原因进入阻塞**状态。 

- 当线程获得到等待的资源资源或者引起阻塞的条件得到满足时，会从阻塞状态进入就绪状态。 

- 当线程的run方法执行结束时，线程就进入死亡状态。

#### 锁

多个线程同时对一个公共资源（如全局变量）进行操作的情况，为了避免发生混乱。`threading.lock`，`acquire()`方法上锁，`release()`方法解锁

可重入锁：为了支持在同一线程中多次请求同一资源，python提供了threading.RLock。重入锁必须由获取它的同一个线程释放，同时要求解锁次数应与加锁次数相同，才能用于另一个线程。

#### 同步

阻塞线程直到子线程全部结束。

#### 守护线程

不重要线程。主线程会等所有‘重要’线程结束后才结束。

###  协程

协程是一种用户态的轻量级线程，协程的调度完全由用户控制。协程拥有自己的寄存器上下文和栈。协程调度时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快。

**如何利用多核CPU呢？最简单的方法是多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能**。

**常用模块**

greenlet：提供了切换任务的快捷方式，但是遇到io无法自动切换任务，需要手动切换

gevent：开启协程任务并切换的模块，遇到io自动切换任务。

asyncio：Python3.4

`@asyncio.coroutine`装饰器的函数称为协程函数。源码实现逻辑：判断修饰的func是否为生成器，不是的话，转换成生成器。再将生成器转换成coroutine。

`yield from`语法用于将一个生成器部分操作委托给另一个生成器。此外，允许子生成器（即yield from后的“参数”）返回一个值，该值可供委派生成器（即包含yield from的生成器）使用。并且在委派生成器中，可对子生成器进行优化。

`async`/`await`：只是`@asyncio.coroutine`和`yield from`的语法糖

**缺点**

- 无法利用多核资源：协程的本质是个单线程

- 进行阻塞（Blocking）操作（如IO时）会阻塞掉整个程序

**协程主要使用场景**

网络请求，比如爬虫，大量使用 aiohttp

文件读取， aiofile

web 框架， aiohttp， fastapi

数据库查询， asyncpg, databases

### 僵尸进程和孤儿进程 

孤儿进程： **父进程退出，子进程还在运行的这些子进程都是孤儿进程，**孤儿进程将被init 进程（进程号为1）所收养，并由init 进程对他们完成状态收集工作。

僵尸进程： 进程使用fork 创建子进程**，如果子进程退出，而父进程并没有调用wait 获取子进程的状态信息**，那么子进程的进程描述符仍然保存在系统中，这些进程是僵尸进程。

避免僵尸进程的方法：

1.用`wait()`函数使父进程阻塞

2.使用信号量，在signal handler 中调用waitpid,这样父进程不用阻塞

**3.fork 两次用孙子进程去完成子进程的任务（？？？）**

##  Global Interpreter Lock(全局解释器锁)

执行 Python 字节码时，为了保护访问 Python 对象而阻止多个线程执行的一把互斥锁。这把锁的存在主要是因为 CPython 解释器的内存管理不是线程安全的。

因为 Python 默认的解释器是 CPython，**GIL 是存在于 CPython 解释器中的**。

常见的 Python 解释器：IPython（基于Cython）、IPython、Jython（可以把 Python 代码编译成 Java 字节码，依赖 Java 平台，不存在 GIL）、IronPython（运行在微软的 .Net 平台下的 Python 解释器，可以把 Python 代码编译成 .Net 字节码，不存在 GIL）

### GIL原理

python 的线程就是 C 语言的 pthread，它是通过操作系统调度算法调度执行的。

Python 2.x 的代码执行是基于 opcode 数量的调度方式，简单来说就是每执行一定数量的字节码，或遇到系统 IO 时，会强制释放 GIL，然后触发一次操作系统的线程调度。

虽然在 Python 3.x 进行了优化，基于固定时间的调度方式，就是每执行固定时间的字节码，或遇到系统 IO 时，强制释放 GIL，触发系统的线程调度。

### 为什么会有GIL

Python 设计者在设计解释器时，可能没有想到 CPU 的性能提升会这么快转为多核心方向发展，所以在当时的场景下，设计一个全局锁是那个时代保护多线程资源一致性最简单经济的设计方案。

多核心时代来临，当大家试图去拆分和去除 GIL 的时候，发现大量库的代码和开发者已经重度依赖 GIL（默认认为 Pythonn 内部对象是线程安全的，无需在开发时额外加锁），所以这个去除 GIL 的任务变得复杂且难以实现。

## 新式类和经典类

a. 在python里凡是继承了object的类，都是新式类

b. Python3里只有新式类

c. Python2里面继承object的是新式类，没有写父类的是经典类

d. 保持class与type的统一，对新式类的实例执行`a.__class__`与`type(a)`的结果是一致的，对于旧式类来说就不一样了，经典类的type(a)返回instance，其他都是class。

e.对于多重继承的属性搜索顺序不一样新式类是采用广度优先搜索，旧式类采用深度优先搜索。

一个旧式类的深度优先的例子

```python
class A():
  def foo1(self):
     print "A"

class B(A):
  def foo2(self):
     pass

class C(A):
  def foo1(self):
     print "C"

class D(B, C):
  pass


d = D()
d.foo1()
```

**缺点：**经典类的查找顺序从左到右深度优先的规则，在访问d.foo1()的时候,D这个类是没有的..那么往上查找,先找到B,里面没有,深度优先,访问A,找到了foo1(),所以这时候调用的是A的foo1()，从而导致C重写的foo1()被绕过



## new & init

1、__new__至少要有一个参数cls，代表当前类，此参数在实例化时由Python解释器自动识别

2、__new__必须要有返回值，返回实例化出来的实例，这点在自己实现__new__时要特别注意，可以return父类（通过super(当前类名, cls)）__new__出来的实例，或者直接是object的__new__出来的实例

3、__init__有一个参数self，就是这个__new__返回的实例，__init__在__new__的基础上可以完成一些其它初始化的动作，__init__不需要返回值

4、如果__new__创建的是当前类的实例，会自动调用__init__函数，通过return语句里面调用的__new__函数的第一个参数是cls来保证是当前类实例，如果是其他类的类名，；那么实际创建返回的就是其他类的实例，其实就不会调用当前类的__init__函数，也不会调用其他类的__init__函数。

## lambda

1.lambda函数比较轻便，即用即仍，很适合需要完成一项功能，但是此功能只在此一处使用，连名字都很随意的情况下

2.匿名函数，一般用来给filter，map这样的函数式编程服务

3.作为回调函数，传递给某些应用，比如消息处理

## 类方法、类实例方法、静态方法

- 类方法: 是类对象的方法，在定义时需要在上方使用 @classmethod 进行装饰,形参为cls，表示类对象，类对象和实例对象都可调用
-  类实例方法: 是类实例化对象的方法,只有实例对象可以调用，形参为self,指代对象本身;
-  静态方法: 是一个任意函数，在其上方使用 @staticmethod 进行装饰，可以用对象直接调用，静态方法实际上跟该类没有太大关系

## numba pypy cython

python 程序性能优化的套路一般有两种：

1）jit， 即just in time compiler， 即时编译器，在运行时将某些函数编译成二进程代码，使用这种方式的有：numba 和pypy；

2）将python代码转换成c++/c代码，然后编译执行，这种方式有：cython和nuitka。总而言之，转换成c++/c代码以后编译成二进制文件执行的效率比用numba和pypy即时编译执行的效率要高。

易用性：易用性最好的无疑是Pypy，Pypy是Python的解释器，我们针对纯Python使用Pypy，除了Pypy不支持的部分库外，不需要进行任何改动。然后是Numba，Numba的基本使用方法就是给函数加一个装饰器，易用性也很高，最后是Cython，因为Cython中需要使用Python+C混合编码，如果需要移植，代价会很大

通用性：在三个方案中，Cython和Numba的兼容性都非常好，而Pypy对于部分库的支持较差（如Numpy，Scipy）。



## 字符串

避免转义给字符串加哪个字母表示原始字符串？r

 

## python2和python3区别

1、Python3 使用 print 必须要以小括号包裹打印内容，比如 print('hi')

Python2 既可以使用带小括号的方式，也可以使用一个空格来分隔打印内容，比 如 print 'hi'

2、python2 range(1,10)返回列表，python3中返回迭代器，节约内存

**3、python2中使用ascii编码，python3中使用utf-8编码**

**4、python2中unicode表示字符串序列，str表示字节序列**

**python3中str表示字符串序列，byte表示字节序列**

5、python2中为正常显示中文，引入coding声明，python3中不需要

6、python2中是raw_input()函数，python3中是input()函数

## xrange和range的区别

都在循环时使用，xrange内存性能更好，xrange用法与range完全相同，range一个生成list对象，xrange是生成器

## 可变数据类型和不可变数据类型

不可变数据类型：数值型、字符串型string和元组tuple

不允许变量的值发生变化，如果改变了变量的值，相当于是新建了一个对象，而对于相同的值的对象，在内存中则只有一个对象（一个地址），如下图用id()方法可以打印对象的id

 

 

Python数据类型分为值类型和引用类型

值类型：

对象本身不允许修改，数值的修改实际上是让变量指向了一个新的对象

包含：字符串、元组、数值，本身不允许被修改

引用类型：

对象本身可以修改，

包含：列表、字典，本身允许修改

## 深拷贝、浅拷贝

1、复制不可变数据类型， copy 、deepcopy,都指向原地址对象

 

2、复制的值是可变对象（列表和字典）

**浅拷贝copy有两种情况：**

第一种情况：复制的 对象中无复杂子对象，原来值的改变并不会影响浅复制的值，同时浅复制的值改变也并不会影响原来的值。原来值的id值与浅复制原来的值不同。

第二种情况：复制的对象中有复杂子对象（例如列表中的一个子元素是一个列表），改变原来的值中的复杂子对象的值 ，会影响浅复制的值。

 

**深拷贝deepcopy**：完全复制独立，包括内层列表和字典

浅拷贝只拷贝一层, 深拷贝拷贝全部 



## 退出Python时，是否释放全部内存？

循环引用其它对象或引用自全局命名空间的对象的模块，在Python退出时并非完全释放。另外，也不会释放C库保留的内存部分。



## 回调函数

回调函数通常和应用处于同一抽象层（因为传入什么样的回调函数是在应用级别决定的）。而回调就成了一个高层调用底层，底层再回过头来调用高层的过程。

 

我们把图中的库函数改称为**中间函数。**

在回调中，我们利用某种方式，**把回调函数像参数一样传入中间函数。可以这么理解，在传入一个回调函数之前，中间函数是不完整的**。换句话说，**程序可以在运行时，通过登记不同的回调函数，来决定、改变中间函数的行为。这就比简单的函数调用要灵活太多了**。

回调里的第三位要角，就是中间函数的调用者。绝大多数情况下，这个调用者可以和程序的主函数等同起来，但为了表示区别，我这里把它称为**起始函数**

 

**如何通信的?**

回调函数是把函数的指针(地址)作为参数传递给另一个函数，将整个函数当作一个对象，赋值给调用的函数。

## 内置函数

**reduce() 函数会对参数序列中元素进行累积。**

函数将一个数据集合（链表，元组等）中的所有数据进行下列操作：用传给 reduce 中的函数 function（有两个参数）先对集合中的第 1、2 个元素进行操作，得到的结果再与第三个数据用 function 函数运算，最后得到一个结果。

\>>>def add(x, y) :      # 两数相加

...   return x + y

... 

\>>> reduce(add, [1,2,3,4,5])  # 计算列表和：1+2+3+4+5

15

\>>> reduce(lambda x, y: x+y, [1,2,3,4,5]) # 使用 lambda 匿名函数

15

**filter() 函数用于过滤序列，过滤掉不符合条件的元素，返回由符合条件元素组成的新列表。**

该接收两个参数，第一个为函数，第二个为序列，序列的每个元素作为参数传递给函数进行判断，然后返回 True 或 False，最后将返回 True 的元素放到新列表中。

def is_odd(n):

  return n % 2 == 1

 

newlist = filter(is_odd, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

print(newlist)

**map() 会根据提供的函数对指定序列做映射。**

第一个参数 function 以参数序列中的每一个元素调用 function 函数，返回包含每次 function 函数返回值的新列表。

\>>>def square(x) :      # 计算平方数

...   return x ** 2

... 

\>>> map(square, [1,2,3,4,5])  # 计算列表各个元素的平方

[1, 4, 9, 16, 25]

repr() 函数将对象转化为供解释器读取的形式

\>>>s = 'RUNOOB'

\>>> repr(s)

"'RUNOOB'"

\>>> dict = {'runoob': 'runoob.com', 'google': 'google.com'};

\>>> repr(dict)

"{'google': 'google.com', 'runoob': 'runoob.com'}"

str() 函数将对象转化为适于人阅读的形式。

\>>>s = 'RUNOOB'

\>>> str(s)

'RUNOOB'

\>>> dict = {'runoob': 'runoob.com', 'google': 'google.com'};

\>>> str(dict)

"{'google': 'google.com', 'runoob': 'runoob.com'}"

vars() 函数返回对象object的属性和属性值的字典对象。

\>>>print(vars())

{'__builtins__': <module '__builtin__' (built-in)>, '__name__': '__main__', '__doc__': None, '__package__': None}

\>>> class Runoob:

...   a = 1

... 

\>>> print(vars(Runoob))

{'a': 1, '__module__': '__main__', '__doc__': None}

\>>> runoob = Runoob()

\>>> print(vars(runoob))

{}

 

ord() 函数是 chr() 函数（对于8位的ASCII字符串）或 unichr() 函数（对于Unicode对象）的配对函数，它以一个长度为1的字符串作为参数，返回对应的 ASCII 数值，或者 Unicode 数值，如果所给的 Unicode 字符超出了你的 Python 定义范围，则会引发一个 TypeError 的异常。

## 接口类与抽象类

接口类：**基于同一个接口实现的类** 刚好满足接口隔离原则 面向对象开发的思想 规范。接口隔离原则：使用多个专门的接口，而不使用单一的总接口。即客户端不应该依赖那些不需要的接口。

from abc import abstractmethod, ABCMeta

 

class Payment(object): 

  __metaclass__ = ABCMeta

  @abstractmethod

def pay(self,money):pass

 

抽象类的本质还是类，**指的是一组类的相似性，包括数据属性（如all_type）和函数属性（如read、write），而接口只强调函数属性的相似性。**

从实现角度来看，抽象类与普通类的不同之处在于：**抽象类中有抽象方法，该类不能被实例化，只能被继承，且子类必须实现抽象方法，也有其他非抽象方法**。

## 描述符

**一个实现了描述符协议的类就是一个描述符。**

**描述符协议：实现了 __get__()、__set__()、__delete__() 其中至少一个方法的类，就是一个描述符。**

 

__get__： 用于访问属性。它返回属性的值，若属性不存在、不合法等都可以抛出对应的异常。

__set__：将在属性分配操作中调用。不会返回任何内容。

__delete__：控制删除操作。不会返回内容。

描述符的作用和优势，以弥补Python动态类型的缺点。

class Score:

  def __init__(self, default=0):

​    self._score = default

 

  def __set__(self, instance, value):

​    if not isinstance(value, int):

​      raise TypeError('Score must be integer')

​    if not 0 <= value <= 100:

​      raise ValueError('Valid value must be in [0, 100]')

 

​    self._score = value

 

  def __get__(self, instance, owner):

​    return self._score

 

  def __delete__(self):

​    del self._score

​    

class Student:

  math = Score(0)

  chinese = Score(0)

  english = Score(0)

 

  def __init__(self, name, math, chinese, english):

​    self.name = name

​    self.math = math

​    self.chinese = chinese

​    self.english = english

 

 

  def __repr__(self):

​    return "<Student: {}, math:{}, chinese: {}, english:{}>".format(

​        self.name, self.math, self.chinese, self.english

​      )

**staticmethod**

class Test:

  @staticmethod

  def myfunc():

​    print("hello")

 

\# 上下两种写法等价

 

class Test:

  def myfunc():

​    print("hello")

  \# 重点：这就是描述符的体现

myfunc = staticmethod(myfunc)

每调用一次，它都会经过描述符类的 __get__

**classmethod**

class classmethod(object):

  def __init__(self, f):

​    self.f = f

 

  def __get__(self, instance, owner=None):

​    print("in classmethod __get__")

​    

​    def newfunc(*args):

​      return self.f(owner, *args)

​    return newfunc

 

class Test:

  def myfunc(cls):

​    print("hello")

​    

  \# 重点：这就是描述符的体现

  myfunc = classmethod(myfunc)

## property

property() 函数的作用是在新式类中返回属性值。

class property([fget[, fset[, fdel[, doc]]]])

class C(object):

  def __init__(self):

​    self._x = None

 

  def getx(self):

​    return self._x

 

  def setx(self, value):

​    self._x = value

 

  def delx(self):

​    del self._x

 

  x = property(getx, setx, delx, "I'm the 'x' property.")

property能让你的方法像类属性一样使用

fdel = property(lambda self: object(), lambda self, v: None, lambda self: None) # default

fget = property(lambda self: object(), lambda self, v: None, lambda self: None) # default

fset = property(lambda self: object(), lambda self, v: None, lambda self: None) # default

property 的 getter,setter 和 deleter 方法同样可以用作装饰器：

class C(object):

  def __init__(self):

​    self._x = None

 

  @property

  def x(self):

​    """I'm the 'x' property."""

​    return self._x

 

  @x.setter

  def x(self, value):

​    self._x = value

 

  @x.deleter

  def x(self):

​    del self._x

使用property装饰后，x不再是一个函数，而是property类的一个实例。所以第二个math函数可以使用 x.setter 来装饰，本质是调用property.setter 来产生一个新的 property实例赋值给第二个x。

第一个 x和第二个 x 是两个不同 property实例。但他们都属于同一个描述符类（property），当对 math 对于赋值时，就会进入 property.__set__，当对math 进行取值里，就会进入 property.__get__。仔细一看，其实最终访问的还是C实例的 _x属性。

 

## 装饰器

装饰器本质上是一个**Python函数或者类**，它可以让其他函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象。它经常用于有切面需求的场景，比如：**插入日志**、性能测试、**事务处理**、缓存、**权限校验、异常处理**等场景。装饰器是解决这类问题的绝佳设计，有了装饰器，我们就可以抽离出大量与函数功能本身无关的雷同代码并继续重用。概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能

 

partial通过实现"__new__"和"__call__"生成一个可调用对象，这个对象内部保存了被包装函数以及固定参数，这个对象可以像函数一样被调用，调用时，其实是执行了对象内部持有的被包装函数，其参数由固定参数和新传入的参数组合而来。

 

 

import functools

 

def add(a, b):

  print(a + b)

 

add = functools.partial(add, 1)

add(2)

 

输出：3

add函数原本接收两个参数a和b，经过partial包装之后，a参数的值被固定为了1，新的add对象（注意此处add已经是一个可调用对象，而非函数，下文分析源码会看到）只需要接收一个参数即可。

 

**把原函数的部分参数固定了初始值，新的调用只需要传递其它参数。**

 

@functools.wraps(func)时最底层执行的逻辑，就是把wrapped函数的属性拷贝到wrapper函数中。

def outer(func):

  @functools.wraps(func)

  def inner(*args, **kwargs):

​    print(f"before...")

​    func(*args, **kwargs)

​    print("after...")

 

  return inner

 

 

@outer

def add(a, b):

  """

  求和运算

  """

print(a + b)

1、原函数为add。

2、@outer会去执行outer装饰器，传入add函数，返回一个inner函数。

3、执行outer函数时，加载inner函数，此时会直接执行functools.wraps(func)返回一个可调用对象，即partial对象。

4、此时inner的装饰器实际上是@partial，partial会被调用，传入inner函数，执行partial内部的update_wrapper函数，将func的相应属性拷贝给inner函数，最后返回inner函数。这一步并没有生成新的函数，仅仅是改变了inner函数的属性。

5、把add指向inner函数。

6、调用add实际调用的是inner函数，inner函数内部持有原add函数的引用即func。

 

1）functools.wraps 旨在消除装饰器对原函数造成的影响，即对原函数的相关属性进行拷贝，达到装饰器不修改原函数的目的。

2）wraps内部通过partial对象和update_wrapper函数实现。

3）partial是一个类，通过实现__new__，自定义实例化对象过程，使得对象内部保留原函数和固定参数，通过实现__call__，使得对象可以像函数一样被调用，再通过内部保留的原函数和固定参数以及传入的其它参数进行原函数调用。

 

**类装饰器**

相比函数装饰器，类装饰器具有**灵活度大、高内聚、封装性**等优点。使用类装饰器还可以依靠类内部的__call__方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。

class Foo(object):

  def __init__(self, func):

​    self._func = func

 

def __call__(self):

   print ('class decorator runing')

   self._func()

   print ('class decorator ending')

 

@Foo

def bar():

  print ('bar')

 

bar()

## __call__

可调用对象：当一个类实现了"__call__"方法后，这个类的对象就能够像函数一样被调用。

class Callable:

  def __call__(self, a, b):

​    return a + b

 

 

func = Callable() 

result = func(2, 3) # 像函数一样调用

print(result)

## 闭包

**高阶函数**：接受函数为入参，或者把函数作为结果返回的函数。后者称之为嵌套函数。

 

**闭包**：如果在一个内部函数里，对在外部作用域（但不是在全局作用域）的变量进行引用，那么内部函数就被认为是闭包(closure)

 

def timer(func):

  def wrapper(*args, **kwargs):

​    start = time.time()

​    func(*args, **kwargs) #此处拿到了被装饰的函数func

​    time.sleep(2)#模拟耗时操作

​    long = time.time() - start

​     print(f'共耗时{long}秒。')

  return wrapper #返回内层函数的引用

 

@timer

def add(a, b):

  print(a+b)

 

add(1, 2) #正常调用add

 

**模块加载** ->> 遇到@，执行timer函数，传入add函数 ->> 生成timer.<locals>.wrapper函数并命名为add，其实是覆盖了原同名函数 ->> 调用add(1, 2) ->> 去执行timer.<locals>.wrapper(1, 2) ->> wrapper内部持有原add函数引用(func)，调用func(1, 2) ->>继续执行完wrapper函数

 

**带参数的装饰器**

def auth(permission):

  def _auth(func):

​    def wrapper(*args, **kwargs):

​      print(f"验证权限[{permission}]...")

​      func(*args, **kwargs)

​      print("执行完毕...")

 

​    return wrapper

 

  return _auth

 

 

@auth("add")

def add(a, b):

  """

  求和运算

  """

  print(a + b)

真正调用的是装饰后生成的新函数。打破了“不能修改原函数”的规则

为了消除装饰器对原函数的影响，我们需要伪装成原函数，拥有原函数的属性，看起来就像是同一个人一样。functools为我们提供了便捷的方式，只需这样：

def auth(permission):

  def _auth(func):

​    @functools.wraps(func) # 注意此处

​    def wrapper(*args, **kwargs):

​      print(f"验证权限[{permission}]...")

​      func(*args, **kwargs)

​      print("执行完毕...")

 

​    return wrapper

 

  return _auth

 

 

@auth("add")

def add(a, b):

  """

  求和运算

  """

  print(a + b)

 

def multi():

  return [lambda x : i*x for i in range(4)]

print([m(3) for m in multi()])

答案是[9,9,9,9]，而不是[0,3,6,9]产生的原因**是Python的闭包的延迟绑定**导致的，这意味着在闭包中的变量是在内部函数被调用的时候被查找的，因为，最后函数被调用的时候，for循环已经完成, i 的值最后是3,因此每一个返回值的i都是3,所以最后的结果是[9,9,9,9]

def multipliers():

  for i in range(4):

​    yield lambda x: i *x

## 单例模式

**应用场景**

资源共享的情况下，避免由于资源操作时导致的性能或损耗等，如logger用以记录日志文件，应用配置？？。

控制资源的情况下，方便资源之间的互相通信。如线程池等，1,网站的计数器 2,应用配置 3.多线程池 4数据库配置 数据库连接池 5.应用程序的日志应用

**第一种方法:使用装饰器**

def singleton(cls):

  instances = {}

  def wrapper(*args, **kwargs):

​    if cls not in instances:

​      instances[cls] = cls(*args, **kwargs)

​    return instances[cls]

  return wrapper

  

  

@singleton

class Foo(object):

  pass

foo1 = Foo()

foo2 = Foo()

print(foo1 is foo2) # True

**第二种方法：使用基类**

New 是真正创建实例对象的方法，所以重写基类的new 方法，以此保证创建对象的时候只生成一个实例。

但是以上的方法在多线程中会有线程安全问题，当有多个线程同时去初始化对象时，就很可能同时判断__instance is None，从而进入初始化instance的代码中(如果有__init__方法)。所以需要用**互斥锁**来解决这个问题。

class Singleton(object):

  def __new__(cls, *args, **kwargs):

​    if not hasattr(cls, '_instance'):

​      cls._instance = super(Singleton, cls).__new__(cls, *args, **kwargs)

​    return cls._instance

 

class Singleton(object):

  __instance = None

  def __new__(cls, *args, **kwargs):

​    if cls.__instance is None:

​      cls.__instance = super(Singleton, cls).__new__(cls, *args, **kwargs)

​      \# 可以在这里给实力对象绑定一些固有属性

​      \# cls.__instance.appkey = ""

​    return cls.__instance

 

class Foo(Singleton):

  pass

 

foo1 = Foo()

foo2 = Foo()

 

print(foo1 is foo2) # True

**第三种方法：元类**，元类是用于创建类对象的类**，类对象创建实例对象时一定要调用call方法，因此在调用call时候保证始终只创建一个实例即可**，type是python的元类

class Singleton**(type):**

  def __call__(cls, *args, **kwargs):

​    if not **hasattr(cls, '_instance'):**

​      cls._instance = super(Singleton, cls).**__call__(\*args, \**kwargs)**

​    return cls._instance

 

 

\# Python2

class Foo(object):

__metaclass__ = Singleton

线程安全—装饰器

  import threading

 

  def make_synchronized(func):

​    import threading

​     **func.__lock__ = threading.Lock()**

 

​    \# 用装饰器实现同步锁

​    def synced_func(*args, **kwargs):

​      **with func.__lock__:**

​        return func(*args, **kwargs)

 

​    return synced_func

 

 

  class Singleton(object):

​    __instance = None

 

​     **@make_synchronized**

​    def __new__(cls, *args, **kwargs):

​      if not cls.__instance:

​        cls.__instance = object.__new__(cls)

​      return cls.__instance

 

​    def __init__(self):

​      self.blog = "blog"

线程安全--元类

import threading

 

class MetaSingleton(type):

  **_instance_lock = threading.Lock()**

  def __call__(cls, *args, **kwargs):

​    if not hasattr(cls, '_instance'):

​      **with MetaSingleton._instance_lock:**

​         if not hasattr(cls, '_instance'):

​          cls._instance = super(MetaSingleton, cls).__call__(*args, **kwargs)

 

​    return cls._instance

  

 

class Singleton(metaclass=MetaSingleton):

  def __init__(self, name):

​    self.name = name

## 参数传递 - 引用传递&值传递

值传递：方法调用时，实际参数把它的值传递给对应的形式参数，形参的改变不影响实参值。

 

引用传递：也称地址传递,在方法调用时，实际上是把参数地址，传递给方法的形参，形参值的改变将会影响实参的值。

 

数字、字符串、元组等不可变对象类型都属于值传递

dict或者list等可变对象类型属于引用传递。



 

## 变量的作用域/查找顺序

函数作用域的LEGB顺序

1.什么是LEGB?

L： local 函数内部作用域

E: enclosing 函数内部与内嵌函数之间

G: global 全局作用域

B： build-in 内置作用

python在函数里面的查找分为4种，称之为LEGB，也正是按照这是顺序来查找的

Python除了def/class/lambda 外，其他如: if/elif/else/ try/except for/while并不能改变其作用域。定义在他们之内的变量，外部还是可以访问。

## 元类

元类是用来创建类的类。

如果类属性中定义了__metaclass__，则在创建类的时候用元类来创建，如果没有则向其父类查找__metaclass__直到都没有，则用type()创建类。

 

class MyMetaClass(type): # type实际上也是类，所以可以被继承

  \# __new__方法是一个特殊的方法用来创建并返回对象

  \# __init__方法只是用来给对象传入初始化参数

  \# 如果需要对类进行改写，则需要改写__new__方法

  def __new__(cls, name, bases, attrs)

​    new_attrs = dosomething(attrs)

​    return super(MyMetaClass, cls).__new__(cls, name, bases, new_attr)

使用了该metaclass则自动有了new_attrs方法。

元类做了什么事情呢？

 

l 拦截类的创建

l 修改类

l 返回修改后的类

应用场景

ORM



## 可重入锁

为了支持在同一线程中多次请求同一资源，python提供了“可重入锁”：threading.RLock。

重入锁必须由获取它的同一个线程释放，同时要求解锁次数应与加锁次数相同，才能用于另一个线程。



## 线程池的原理和实现

Queue-队列。因为python中Queue本身就是同步的，所以也就是线程安全的，所以我们可以放心的让多个线程共享一个Queue。 

 

任务池中存放着待执行的任务，各个线程到任务池中取任务执行，可用Queue来实现。 

## 编码和解码

l ascii  最多只能用8位来表示（一个字节），即：2**8 = 256，所以，ASCII码最多只能表示 256 个符号。

l unicode： 所有字符（无论英文、中文等）  1个字符：2个字节

l gbk：一个字符，英文1个字节，中文两个字节

l utf-8：英文1个字节、 欧洲：2个字节， 亚洲：3个字节

python 的默认编码方式为ASCII码

## for 循环机制 ？ 

for循环的本质：循环所有对象，全都是使用迭代器协议。 

for循环就是基于迭代器协议提供了一个统一的可以遍历所有对象的方法，即在遍历之前，**先调用对象的__iter__方法将其转换成一个迭代器，然后使用迭代器协议去实现循环访问，这样所有的对象就都可以通过for循环来遍历了，** 

列表，字符串，元组，字典，集合，文件对象等本质上来说都不是可迭代对象，在使用for循环的时候内部是**先调用他们内部的iter方法，使他们变成了可迭代对象，然后在使用可迭代对象的next方法依次循环元素，当元素循环完时，会触发StopIteration异常，for循环会捕捉到这种异常，终止迭代**  

 

## I/O多路复用 

 

## select,poll和epoll 

 

select，poll，epoll都是IO多路复用的机制。I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。 

## 性能优化

**字典 (dictionary) 与列表 (list)**

Python 字典中使用了 hash table，因此查找操作的复杂度为 O(1)，而 list 实际是个数组，在 list 中，查找需要遍历整个 list，其复杂度为 O(n)，因此对成员的查找访问等操作字典要比 list 更快。

**集合 (set) 与列表 (list)**

set 的 union， intersection，difference 操作要比 list 的迭代要快。因此如果涉及到求 list 交集，并集或者差的问题可以转换为 set 来操作

**循环的优化**

对循环的优化所遵循的原则是尽量减少循环过程中的计算量，有多重循环的尽量将内层的计算提到上一层。

**Lazy if-evaluation 的特性**

如果存在条件表达式 if x and y，在 x 为 false 的情况下 y 表达式的值将不再计算。因此可以利用该特性在一定程度上提高程序效率。

**字符串的优化**

在字符串连接的使用尽量使用 join() 而不是 +

当对字符串可以使用正则表达式或者内置函数来处理的时候，选择内置函数

对字符进行格式化比直接串联读取要快

由于字符串是不可变对象，当使用“+”连接字符串的时候，每执行一次“+”操作都会申请一块新的内存，然后复制上一个“+”操作的结果和本次操作的有操作符到这块内存空间中，所以用“+”连接字符串的时候会涉及内存申请和复制；join在连接字符串的时候，首先计算需要多大的内存存放结果，然后一次性申请所需内存并将字符串复制过去。

**使用列表解析（list comprehension）和生成器表达式**

在循环中，先要LOAD_ATTR，将append方法加载进来，然后CALL_FUNCTION，也就是执行；

而在列表解析中，则直接调用了LIST_APPEND命令来添加元素，就是这一个小小的区别，使得列表推导式的速度会更快

**xrange**

在循环的时候使用 xrange 而不是 range；使用 xrange 可以节省大量的系统内存，因为 xrange() 在序列中每次调用只产生一个整数元素。而 range() 將直接返回完整的元素列表，用于循环时会有不必要的开销。

**级联比较**

使用级联比较 "x < y < z" 而不是 "x < y and y < z"；

**while 1 要比 while True 更快**

由于Python2中，True/False不是关键字，因此我们可以对其进行任意的赋值，这就导致程序在每次循环时都需要对True/False的值进行检查；而对于1，则被程序进行了优化，而后不会再进行检查。

## 性能分析

python 内置了丰富的性能分析工具，**如 profile,cProfile 与 hotshot 等**。其中 Profiler 是 python 自带的一组程序，能够描述程序运行时候的性能，并提供各种统计帮助用户定位程序的性能瓶颈。

## sort sorted

区别：

对于一个无序的列表a，调用a.sort()，对a进行排序后返回a，sort()函数修改待排序的列表内容。

而对于同样一个无序的列表a，调用sorted(a)，对a进行排序后返回一个新的列表，而对a不产生影响。

 

 

**sort**

timsort是结合了合并排序（merge sort）和插入排序（insertion sort）而得出的排序算法，它在现实中有很好的效率。Tim Peters在2002年设计了该算法并在Python中使用（TimSort 是 Python 中 list.sort 的默认实现）。该算法找到数据中已经排好序的块-分区，每一个分区叫一个run，然后按规则合并这些run。

​    **TimSort** 算法为了减少对升序部分的回溯和对降序部分的性能倒退，将输入按其升序和降序特点进行了分区。排序的输入的单位不是一个个单独的数字，而是一个个的块-分区。其中每一个分区叫一个run。针对这些 run 序列，每次拿一个 run 出来按规则进行合并。每次合并会将两个 run合并成一个 run。合并的结果保存到栈中。合并直到消耗掉所有的 run，这时将栈上剩余的 run合并到只剩一个 run 为止。这时这个仅剩的 run 便是排好序的结果。

 

（0）数组长度小于某个值，直接用二分插入排序算法

（1）找到各个run，并入栈

（2）按规则合并run

 

0：有序部分的长度一般不会太长，当小于 minRun 时，会将此部分后面的元素插入其中，直至长度满足 minRun。通过二分法查找元素

2：会将该run在数组中的起始位置和run的长度放入栈中，然后根据先前放入栈中的run决定是否该合并run。Timsort不会合并在栈中不连续的run

先用二分查找算法/折半查找算法（binary search）找到插入的位置，然后在插入。

   例如，我们要将A和B这2个run 合并，且A是较小的run。因为A和B已经分别是排好序的，二分查找会找到B的第一个元素在A中何处插入（图4）。同样，A的最后一个元素找到在B的何处插入，找到以后，B在这个元素之后的元素就不需要比较了。这种查找可能在随机数中效率不会很高，但是在其他情况下有很高的效率。

## 堆 栈

在Python中，变量也称为：对象的引用。因为，变量存储的就是对象的地址。变量通过地址引用了“对象”。

**变量位于：栈内存。**

**对象位于：堆内存。**

内存空间在逻辑上分为三部分：代码区、静态数据区和动态数据区，动态数据区又分为栈区

和堆区。

**代码区：**存储方法体的二进制代码。高级调度（作业调度）、中级调度（内存调度）、低级

调度（进程调度）控制代码区执行代码的切换。

**静态数据区：**存储**全局变量、静态变量、常量，常量包括final修饰的常量和String常量**。系统自动分配和回收。

**栈区：**存储运行方法的形参、局部变量、返回值。由系统自动分配和回收。

**堆区**：new一个对象的引用或地址存储在栈区，指向该对象存储在堆区中的真实数据。

## 整数

PyIntObject就是一个对C语言中long类型的数值的扩展，出于性能考虑,对于小整数，Python使用小整数对象池small_ints缓存了[-5，257）之间的整数，该范围内的整数在Python系统中是共享的。

而超过该范围的整数即使值相同，但对象不一定是同一个，如下所示：当a与b的值都是10000，但并不是同一个对象，而值为1的时候，a和b属于同一个对象。

## 字符串

PyStringObject的ob_sstate属性用于标记字符串对象是否经过intern机制处理，intern处理后的字符串，比如"Python"，在解释器运行过程中始终只有唯一的一个字符串"Python"对应的PyStringObject对象。

**intern**

先类型检查，intern机制只处理字符串

如果该PyStringObject对象已经进行过intern机制处理，则直接返回

interned其实一个字典对象，当它为null时，初始化一个字典对象，否则，看该字典中是否存在一个key为(PyObject *)s的value，如果存在，那么就把该对象的引用计数加1，临时创建的那个对象的引用计数减1。否则，把(PyObject *)s同时作为key和value添加到interned字典中，与此同时它的引用计数减2，这两个引用计数减2是因为被interned字典所引用，但这两个引用不作为垃圾回收的判断依据，否则，字符串对象永远都不会被垃圾回收器收集了

 

上述代码中，给b赋值为"python"后，系统中创建了几个PyStringObject对象呢？答案是：2，在创建b的时候，一定会有一个临时的PyStringObject作为字典的key在interned中查找是否存在一个PyStringObject对象的值为"python"。

 

## raw_input、input

raw_input() 返回字符串,

input() 尝试执行输入的结果，输入字符串需要带引号，输入float，返回float

## 上下文管理

 

## mro

## isinstance 作用以及应用场景？

isinstance() 函数来判断一个对象是否是一个已知的类型，类似 type()。

isinstance() 与 type() 区别：

type() 不会认为子类是一种父类类型，不考虑继承关系。

isinstance() 会认为子类是一种父类类型，考虑继承关系。

如果要判断两个类型是否相同推荐使用 isinstance()。

## dir()是干什么用的？

 

## 如何判断是函数还是方法？

看他的调用者是谁，如果是类，就需要传入一个参数self的值，这时他就是一个函数，

如果调用者是对象，就不需要给self传入参数值，这时他就是一个方法

 

print(isinstance(obj.func, FunctionType))  # False

 

print(isinstance(obj.func, MethodType))  # True

## 如何在Python中管理内存？

python中的内存管理由Python私有堆空间管理。所有Python对象和数据结构都位于私有堆中。程序员无权访问此私有堆。python解释器负责处理这个问题。 

Python对象的堆空间分配由Python的内存管理器完成。核心API提供了一些程序员编写代码的工具。 

Python还有一个内置的垃圾收集器，它可以回收所有未使用的内存，并使其可用于堆空间。

 

## 什么是pickling和unpickling？

Pickle模块接受任何Python对象并将其转换为字符串表示形式，并使用dump函数将其转储到文件中，此过程称为pickling。从存储的字符串中检索原始Python对象的过程称为unpickling。

