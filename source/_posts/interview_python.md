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
- interview
categories:
- interview
---

## 语言特性

解释型语言。Python不需要在运行之前进行编译。

动态语言，不需要声明变量的类型，动态增加类方法。

适合面向对象的编程，允许类的定义和继承。

## python2和python3区别

- Python2 的默认编码是 ascii，Python 3 默认编码 UTF-8，不需要在文件顶部写 `# coding=utf-8 `。
- python2默认是按照相对路径导入模块和包，python3默认则是按照绝对路径导入
- Python3里只有新式类；Python2里面继承object的是新式类，没有写父类的是经典类，多重继承的属性搜索顺序不一样，新式类是采用广度优先搜索，旧式类采用深度优先搜索。



- Python3，新增了关键字 nonlcoal，支持嵌套函数中，变量声明为非局部变量。
- python3提供注解，但是解释器**并不会**因为这些注解而提供额外的校验，没有任何的类型检查工作。也就是说，这些类型注解加不加，对你的代码来说**没有任何影响**，好处是易懂。



- 在Python2 中，字符串有两个类型， unicode和 str，前者表示文本字符串，后者表示字节序列，python2 会自动将字符串转换为合适编码的字节字符串（utf-8/gbk...），`decode('utf-8')`之后转换为unicode，可以显示指定字符串类型为unicode类型；Python3  str 表示字符串，byte 表示字节序列，字符串默认是Unicode，不能显示指定u"xx"，转换字节序列需要`encode('utf-8')`。
- True 和 False 在 Python2 中是全局变量，分别对应 1 和 0，可以指向其它对象。 Python3  True 和 False 变为关键字，不允许再被重新赋值。
- 在Python2中，3/2是整数，在Python 3中浮点数，如果相除还想得到整数，就需要改成//相除。
- 原来在py2里，4字节以内的整数类型为int，超过就是long，而py3里没有long类型，只有int，而带来的问题是，大量整数计算时，py3要比py2占用更多内存，计算也明显更慢。
- py3里dict没有`has_key()`方法，统一使用in表达式



- Python 2中print/exec是特殊语句，Python 3中print/exec是函数，需要加上括号。
- python2 range返回列表，python3 range中返回可迭代对象，节约内存。
- Python 2 map、zip、filter函数返回list，Python3返回迭代器。
- python2中的raw_input/input函数，python3中改名为input函数，危险的input被删掉了

## python2和3代码如何兼容

- 使用 2to3 工具（python自带的转换工具）对代码检查

  查看输出信息，并修正相关问题。

- 使用python -3执行python程序

  程序在运行时会在控制台上将python2和python3不一致，同时2to3无法处理的问题提示出来

- `from __future__ import`在python2使用python的未来特性了

- import问题

  python3中“少”了很多python2的包，在大多情况下这些包之是改了个名字而已。我们可以在import的时候捕获ImportError，重新import。

- 使用python3的方式写程序

  python2中print是关键字，到了python3中print变成了函数。

- 检查当前运行的python版本

  有时候你或许必须为python2和python3写不同的代码，可以先获取版本。

- 使用six

  six 提供了一些简单的工具用来封装 Python 2 和 Python 3 之间的差异性。支持3向2的兼容。

## Python基础

### 可变数据类型和不可变数据类型

可变数据类型（引用类型）：当该数据类型对应的变量值发生了变化时，对应的内存地址不发生改变。

不可变数据类型（值类型）：当该数据类型对应的变量值发生了变化时，对应的内存地址发生了改变。

可变数据类型：list和dict；

不可变数据类型：int、float、string和tuple、bytes。

### array 与内置list 有什么区别

array 是数组, 数组是只能够保存一种类型, 初始化的时候就决定了数据类型.

而list 里面 几乎可以放任意类型

### 扁平序列

存放的都是原子级元素，此时存放的是值而不会是引用。

常见的扁平序列包括：str，bytes，bytearray, memoryview, array.array等。

### python2中xrange和range的区别

`range()`返回的是一个list对象，而xrange返回的是一个可迭代对象。

`xrange()`则不会直接生成一个list，而是每次调用返回其中的一个值，内存空间使用极少。因而性能非常好。

### 变量的作用域/查找顺序

函数作用域的LEGB顺序

L：local ，局部作用域；

E：enclosing，嵌套的父级函数的局部作用域；

G：global ，全局变量；

B：build-in， 系统固定模块里面的变量。

Python除了def/class/lambda 外，其他如: if/elif/else/ try/except for/while并不能改变其作用域。

### 内置函数

`reduce()` 函数会对参数序列中元素进行累积。

先对数据集合中的第 1、2 个元素进行操作，得到的结果再与第三个数据做函数运算。

```python
def add(x, y) :      # 两数相加
   return x + y
reduce(add, [1,2,3,4,5])  # 计算列表和：1+2+3+4+5
reduce(lambda x, y: x+y, [1,2,3,4,5]) # 使用 lambda 匿名函数
```

`filter()` 函数用于过滤序列，过滤掉不符合条件的元素，返回由符合条件元素组成的新列表。

序列的每个元素作为参数传递给函数进行判断，然后返回 True 或 False，最后将返回 True 的元素放到新列表中。

```python
def is_odd(n):
  return n % 2 == 1

newlist = filter(is_odd, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
print(newlist)
```

`map()` 会根据提供的函数对指定序列做映射。

序列中的每一个元素调用 函数，返回包含每次函数返回值的新列表（python2），python3是会返回可迭代对象的。

```python
def square(x) :      # 计算平方数
   return x ** 2
map(square, [1,2,3,4,5])  # 计算列表各个元素的平方
```

`repr() `函数将对象转化为供解释器读取的形式

```python
dict1 = {'runoob': 'runoob.com', 'google': 'google.com'};
repr(dict1)
str(dict1)

"{'google': 'google.com', 'runoob': 'runoob.com'}"
```

`vars() `函数返回对象object的属性和属性值的字典对象。

```
class Runoob:
   a = 1
print(vars(Runoob))
{'a': 1, '__module__': '__main__', '__doc__': None}
```

**ord**

一个长度为1的字符串作为参数，返回对应的 ASCII 数值，或者 Unicode 数值。

#### dir

不带参数时，返回当前范围内的变量、方法和定义的类型列表；

带参数时，返回参数的属性、方法列表。

如果参数包含方法`__dir__()`，该方法将被调用。

#### **isinstance** 

`isinstance() `判断一个对象是否是一个已知的类型，`type()`查看一个类型或变量的类型。

`type() `不会认为子类是一种父类类型。`isinstance() `会认为子类是一种父类类型。

#### raw_input、input

1、在 Python2.x 中` raw_input() `和` input()`，两个函数都存在，其中区别为:

- `raw_input()`将所有输入作为字符串看待，返回字符串类型。
-  `input()` 只能接收“数字”的输入，它返回所输入的数字的类型。

2、在 Python3.x 中 仅保留了`input()` 函数，将所有输入作为字符串处理，并返回字符串类型。

#### sort sorted

**区别**

对于一个无序的列表a，调用`a.sort()`，对a进行排序后返回None，`sort()`函数修改待排序的列表内容。

而对于同样一个无序的列表a，调用`sorted(a)`，对a进行排序后返回一个新的列表，而对a不产生影响。

**sort**

timsort是结合了合并排序和插入排序而得出的排序算法。该算法找到数据中已经排好序的块-分区，每一个分区叫一个run，然后按规则合并这些run。

为了减少对升序部分的回溯和对降序部分的性能倒退，将输入按其升序和降序特点进行了分区。排序的输入的单位不是一个个单独的数字，而是一个个的块-分区。其中每一个分区叫一个run。针对这些 run 序列，每次拿一个 run 出来按规则进行合并。每次合并会将两个 run合并成一个 run。合并的结果保存到栈中。合并直到消耗掉所有的 run，这时将栈上剩余的 run合并到只剩一个 run 为止。这时这个仅剩的 run 便是排好序的结果。

（0）数组长度小于某个值，直接用二分插入排序算法

（1）找到各个run，并入栈

（2）按规则合并run

0：有序部分的长度一般不会太长，当小于 minRun 时，会将此部分后面的元素插入其中，直至长度满足 minRun。通过二分法查找元素

2：会将该run在数组中的起始位置和run的长度放入栈中，然后根据先前放入栈中的run决定是否该合并run。Timsort不会合并在栈中不连续的run

先用二分查找算法/折半查找算法（binary search）找到插入的位置，然后在插入。

   例如，我们要将A和B这2个run 合并，且A是较小的run。因为A和B已经分别是排好序的，二分查找会找到B的第一个元素在A中何处插入。同样，A的最后一个元素找到在B的何处插入，找到以后，B在这个元素之后的元素就不需要比较了。这种查找可能在随机数中效率不会很高，但是在其他情况下有很高的效率。

### 魔法方法

在特殊的情况下被Python所调用的方法。

`__init__`构造器，当一个实例被创建的时候用于初始化的方法。

`__new__`实例化对象调用的第一个方法，用来创造一个类的实例的，取下cls参数，把其他参数传给`__init__`.

`__slot__`:让解释器在元组中存储实例属性，而不用字典，告诉解释器：“这个类中的所有实例属性都在这儿了！”

`__call__`让一个类的实例像函数一样被调用

`__getitem__`定义获取容器中指定元素的行为，相当于`self[key]`

`__getattr__`定义当用户试图访问一个不存在属性的时候的行为

`__setattr__`定义当一个属性被设置的时候的行为

`__getattribute___`定义当一个属性被访问的时候的行为

`__del__`删除对象执行的方法

`__str__`强调可读性，面向用户，在`print()`或者`str()`函数调用的时候才会被调用；

`__repr__`强调标准性，面向开发者。

%s调用`__str__`方法，而%r调用`__repr__`方法

`__repr__`在表示类时，是一级的，如果只定义它，那么`__str__` = `__repr__`。

```python
class Callable:
  def __call__(self, a, b):
     return a + b


func = Callable() 
result = func(2, 3) # 像函数一样调用
print(result)
```

### 读取obj.field时, 发生了什么?

1. 如果定义了`__getattribute__`，访问该方法获取属性值。逐级查找父类的`__getattribute__`
2. 对应描述符`__get__()`方法
3. 如果obj 实例有这个属性, 返回. 
4. 非数据描述符`__get__()`
5. 如果obj 的class 有这个属性, 返回. 逐级查找父类的属性
6. 执行`obj.__getattr__`方法.逐级查找父类的`__getattr__`方法

### new & init区别

1、`__new__`有参数cls，代表当前类，从而产生一个实例；`__new__`必须要有返回值，返回实例化出来的实例，可以return父类（`super(当前类名, cls)`）`__new__`出来的实例，或object的`__new__`出来的实例

2、`__init__`有参数self，完成一些初始化的动作，`__init__`不需要返回值

3、如果`__new__`创建的是当前类的实例，会自动调用`__init__`（return语句里面调用的`__new__`函数的第一个参数是cls，保证是当前类实例）；如果`__new__`返回一个已经存在的实例，`__init__`不会被调用。

4、如果我们在`__new__`函数中不返回任何对象，则`__init__`函数也不会被调用。

> Python的旧类中实际上并没有`__new__`方法。因为旧类中的`__init__`实际上起构造器的作用

### 字符串

避免转义，给字符串加r表示原始字符串。

### is 和 ==区别

is：比较俩对象是否为同一个实例对象，是否指向同一个内存地址。

== ： 比较的两个对象的内容/值是否相等，默认会调用对象的`eq()`方法

### set去重

set的去重是通过两个函数`__hash__`和`__eq__`结合实现的。

1、当两个变量的哈希值不相同时，就认为这两个变量是不同的

2、当两个变量哈希值一样时，调用`__eq__`方法，当返回值为True时认为这两个变量是同一个。返回FALSE时，不去重。

### list切片

索引操作本身基于`__getitem__`和`__setitem__`

python向`__getitem__`传入了一个`slice`的对象，这个类有start, stop, step三个属性，缺省值都是None。

```python
a = [1,2,3,4,5,6]
x = a [1: 5]        # x = a.__getitem__(slice( 1, 5, None))
a [1: 3] = [10, 11, 12]  # a.__setitem__(slice(1, 3, None), [ 10, 11, 12 ])
del a [1: 4]        # a.__delitem__(slice(1, 4, None))
```

### 三元算子

`[on true] if [expression] else [on false]`

### pass

1、一般作为占位符或者创建占位程序，pass语句不会执行任何操作

2、保证格式、语义完整 

### lambda

创建匿名函数的一个特殊语法，即用即仍，

1.一般用来给filter，map这样的函数式编程服务

2.作为回调函数

### 迭代器和生成器

#### 迭代器

**迭代器协议**： `__iter__()` 返回一个特殊的迭代器对象， 这个迭代器对象实现了 `__next__()` 并通过 `StopIteration` 异常，标识迭代的完成。

**迭代器对象**：实现了迭代器协议的对象/被`next()`函数调用并不断返回下一个值的对象称为迭代器。

**例子**

Python的内置工具（如for循环，sum，min，max函数等）使用迭代器协议访问对象

#### 生成器

**使用了 yield 的函数被称为生成器**。**只要把一个列表生成式的`[]`改成`()`，就创建了一个生成器**

生成器是一种特殊的迭代器，生成器自动实现了“迭代器协议”。

好处：不用占用很多内存，只需要在用的时候计算元素的值就行了。

生成器在迭代的过程中可以改变当前迭代值，而修改普通迭代器的当前迭代值往往会发生异常，影响程序的执行。

**yield**

yield 的作用就是把一个函数变成一个 generator，带有 yield 的函数不再是一个普通函数，Python 解释器会将其视为一个 generator。

它和普通函数不同，生成一个 generator 看起来像函数调用，**但不会执行任何函数代码，直到对其调用** `next()`（在 for 循环中会自动调用 `next()`）才开始执行。虽然执行流程仍按函数的流程执行，**但每执行到一个 yield 语句就会中断，并返回一个迭代值，下次执行时从 yield 的下一个语句继续执行**。看起来就好像一个函数在正常执行的过程中被 yield 中断了数次，每次中断都会通过 yield 返回当前的迭代值。

**激活**

- 除了 next，还可以使用 send 激活生成器，两者可以交替使用。
- 第一次当生成器处于 started 状态时，只能 send(None)，否则会报错



#### 可迭代对象

实现`__iter__`方法的对象。可迭代对象包含文件对象、序列（字符串、列表、元组、集合）、字典。

#### 判断方法

```python
from collections import Iterable, Iterator
from inspect import isgenerator

isinstance(a, Iterable)
isinstance(a, Iterator)
isgenerator(a)
```

### 装饰器

装饰器本质上是一个**Python函数或者类**，让其他函数在不做任何代码变动，从而增加额外功能，装饰器的返回值也是一个函数对象。

场景：**插入日志**、性能测试、**事务处理**、缓存、**权限校验、异常处理**。

```python
import functools

def add(a, b):
  print(a + b)
 
add  = functools.partial(add, 1)
# 输出：3
add(2)
```

经过partial包装之后，a参数的值被固定为了1，新的add对象（注意此处add已经是一个可调用对象）只需要接收一个参数即可。

**把原函数的部分参数固定了初始值，新的调用只需要传递其它参数。**

`@functools.wraps(func)`底层逻辑，就是把wrapped函数的属性拷贝到wrapper函数中。

```python
def outer(func):
  @functools.wraps(func)
  def inner(*args, **kwargs):
     print(f"before...")
     func(*args, **kwargs)
     print("after...")
  return inner

@outer
def add(a, b):
  """
  求和运算
  """
  print(a + b)
```

1、原函数为add。

2、@outer会去执行outer装饰器，传入add函数，返回一个inner函数。

3、执行outer函数时，加载inner函数，此时会直接执行`functools.wraps(func)`返回一个可调用对象，即partial对象。

4、此时inner的装饰器实际上是@partial，partial会被调用，传入inner函数，执行partial内部的update_wrapper函数，将func的相应属性拷贝给inner函数，最后返回inner函数。这一步并没有生成新的函数，仅仅是改变了inner函数的属性。

5、把add指向inner函数。

6、调用add实际调用的是inner函数，inner函数内部持有原add函数的引用即func。

 **总结**

1）functools.wraps 旨在消除装饰器对原函数造成的影响，即对原函数的相关属性进行拷贝。

2）wraps内部通过partial对象和update_wrapper函数实现。

3）partial是一个类，通过实现`__new__`，**自定义实例化对象过程，使得对象内部保留原函数和固定参数**，通过实现`__call__`，使得对象可以像函数一样被调用，再通过内部保留的原函数和固定参数以及传入的其它参数进行原函数调用。

#### **类装饰器**

类装饰器具有**灵活度大、高内聚、封装性**等优点。

依靠`__call__`方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。

```python
class Foo(object):
  def __init__(self, func):
     self._func = func

def __call__(self):
   print ('class decorator runing')
   self._func()
   print ('class decorator ending')

@Foo
def bar():
  print ('bar')

bar()
```

### 动态属性property

让方法像属性一样使用.

大量的@property修饰的方法在同一个类，这是不符合设计原则的，代码的分离性和可读性大大降低。建议使用属性描述符。

```python
class C(object):

  def __init__(self):
     self._x = None

  @property
  def x(self):
     """I'm the 'x' property."""
     return self._x

  @x.setter
  def x(self, value):
     if isinstance(value,numbers.Integral):
     	self._x = value

  @x.deleter
  def x(self):
     del self._x
```

使用property装饰后，x不再是一个函数，而是property类的一个实例。所以第二个函数可以使用 x.setter 来装饰，本质是调用property.setter 来产生一个新的 property实例赋值给第二个x。

第一个 x和第二个 x 是两个不同 property实例。但他们都属于同一个描述符类（property），当赋值时，就会进入 `property.__set__`，取值时，就会进入 `property.__get__`。

### 参数类型

**位置参数：**传参数时，按照顺序，依次传值。

**默认参数：**参数提供默认值。默认参数一定要指向不变对象。

**可变参数：**可变参数就是传入的参数个数是可变的。特征：*args

**关键字参数：**允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。特征：**kw

**命名关键字参数：**如果要限制关键字参数的名字，就可以用命名关键字参数。特征：命名关键字参数需要一个特殊分隔符`*`，而后面的参数被视为命名关键字参数。如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要特殊分隔符了

参数定义的**顺序**必须是：位置参数–>默认参数–>可变参数–>命名关键字参数–>关键字参数

### zip

拉链函数， 将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表迭代器。

如果各个迭代器的元素个数不一致，则返回列表长度与最短的对象相同。

```python
print(list(zip([0,1,3],[5,6,7],['a','b'])))
[(0, 5, 'a'), (1, 6, 'b')]
```

### and 和or

 在不加括号时候, and优先级大于or 

x or y：x为真是x, x为假是y 

x and y ： x为真就是y, x为假就是x

```python
v = 1 and 2 or 3 and 4 
print(v) # 2
```

### for 循环

**通过调用`iter()`方法执行（字符串，元组，字典，集合，文件）对象内部的`__iter__`方法，获取一个迭代器，然后使用迭代器协议去实现循环访问，**当元素循环完时，会触发StopIteration异常，for循环会捕捉到这种异常，终止迭代

### 深拷贝和浅拷贝

浅拷贝：在另一块地址中创建一个新的变量或容器，但是容器内的元素的地址均是源对象的元素地址的拷贝。也就是说新的容器中指向了旧的元素（ 新瓶装旧酒 ）。

深拷贝：在另一块地址中创建一个新的变量或容器，同时容器内的元素的地址也是新开辟的，仅仅是值相同而已，是完全的副本。（ 新瓶装新酒 ）。

1、复制不可变数据类型， copy /deepcopy，都指向原地址对象

2、复制的值是可变对象

**浅拷贝copy有两种情况：**

复制对象中包含的非可变数据类型：改变值，会开辟新的内存，有新的引用。原来值的改变并不会影响浅复制的值。

复制对象中包含的可变数据类型：改变原来的值，会影响浅复制的值。

**深拷贝deepcopy**

完全复制独立，包括内层列表和字典

### 参数传递

**值传递：**实参把值传递给形参，形参的改变不影响实参值。

**引用传递（地址传递）：**把实参地址传递形参，形参值的改变会影响实参的值。

- 函数中修改字典某一个键值对是有效的
- 函数中交换两个字典并无法生效

因此不是严格意义上的引用传递，而是**基于引用地址的值传递**，传递的是对象地址的拷贝。

### 闭包

**高阶函数**：函数为入参，或者函数作为返回结果。

**闭包**：在外函数中定义了内函数，内函数里使用了外函数的临时变量，并且外函数的返回值是内函数的引用。

```python
def timer(func):
  def wrapper(*args, **kwargs):
     start = time.time()
     func(*args, **kwargs) #此处拿到了被装饰的函数func
     time.sleep(2)#模拟耗时操作
     long = time.time() - start
      print(f'共耗时{long}秒。')
  return wrapper #返回内层函数的引用

@timer
def add(a, b):
  print(a+b)
 
add(1, 2) #正常调用add
```

**模块加载**

-  遇到@，执行timer函数，传入add函数 
-  生成`timer.<locals>.wrapper`函数并命名为add，其实是覆盖了原同名函数 
-  调用`add(1, 2) `
-  去执行`timer.<locals>.wrapper(1, 2) `
-  wrapper内部持有原add函数引用`(func)`，调用`func(1, 2) `
-  继续执行完wrapper函数

**带参数的装饰器**

```python
def auth(permission):
  def _auth(func):
     def wrapper(*args, **kwargs):
       print(f"验证权限[{permission}]...")
       func(*args, **kwargs)
       print("执行完毕...")
     return wrapper
  return _auth

@auth("add")
def add(a, b):
  """
  求和运算
  """
  print(a + b)
```

真正调用的是装饰后生成的新函数。

为了消除装饰器对原函数的影响，需要伪装成原函数，拥有原函数的属性。可以利用functools：

```python
def auth(permission):
  def _auth(func):
     @functools.wraps(func) # 注意此处
     def wrapper(*args, **kwargs):
       print(f"验证权限[{permission}]...")
       func(*args, **kwargs)
       print("执行完毕...")
     return wrapper
  return _auth

@auth("add")
def add(a, b):
  """
  求和运算
  """
  print(a + b)
```

#### 特殊例子

```python
def multi():
  return [lambda x : i*x for i in range(4)]

print([m(3) for m in multi()]) # [9,9,9,9]
```

闭包的延迟绑定导致的，在**闭包中的变量是在内部函数被调用的时候被查找的**，最后函数被调用的时候，for循环已经完成， i 的值最后是3，因此每一个返回值的i都是3，所以最后的结果是[9,9,9,9]

```python
# [0, 3, 6, 9]
def multipliers():
  for i in range(4):
     yield lambda x: i *x
```

### 上下文管理

在一个类里，实现了`__enter__`和`__exit__`的方法，这个类的实例就是一个上下文管理器。

**基本使用语法**

```pyt
with EXPR as VAR:
    BLOCK
```

**为什么要使用上下文管理器？**

一种更加优雅的方式，操作（创建/获取/释放）资源，如文件操作、数据库连接；处理异常；

**使用contextlib**

```python
import contextlib

@contextlib.contextmanager
def open_func(file_name):
    # __enter__方法
    print('open file:', file_name, 'in __enter__')
    file_handler = open(file_name, 'r')

    try:
        yield file_handler
    except Exception as exc:
        # deal with exception
        print('the exception was thrown')
    finally:
        print('close file:', file_name, 'in __exit__')
        file_handler.close()

        return

with open_func('/Users/MING/mytest.txt') as file_in:
    for line in file_in:
        1/0
        print(line)
```

### 编码和解码

#### 编码类型

- ascii ：一个字节表示一个字符，最多只能表示 256 个符号，是针对英语字符与二进制位之间的关系的统一规定。
- unicode：将世界上所有的符号都纳入其中，每一个符号都给予一个独一无二的编码，用于解决乱码问题。只是一个符号集，它只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储。
- utf-8：互联网上使用最广的一种 Unicode 的实现方式，完成了统一的编码方式。UTF-8 最大的一个特点，就是它是一种变长的编码方式。它可以使用1~4个字节表示一个符号，根据不同的符号而变化字节长度。一般来说，**英文字符1个字节、 欧洲字符2个字节， 中文字符3个字节**
  - 对于单字节的符号，字节的第一位设为`0`，后面7位为这个符号的 Unicode 码。因此对于英语字母，UTF-8 编码和 ASCII 码是相同的。
  - 对于`n`字节的符号（`n > 1`），第一个字节的前`n`位都设为`1`，第`n + 1`位设为`0`，后面字节的前两位一律设为`10`。剩下的没有提及的二进制位，全部为这个符号的 Unicode 码。
- gbk：英文字符1个字节，中文字符两个字节

在计算机内存中，统一使用Unicode编码，当需要保存到硬盘或者需要传输的时候，就转换为UTF-8编码。

python2 的默认编码方式为ASCII码，python3默认的文件编码是UTF-8

> Python的字符串类型是`str`，在内存中以Unicode表示，一个字符对应若干个字节。如果要在网络上传输，或者保存到磁盘上，就需要把`str`变为以字节为单位的`bytes`。
>
> 如果我们从网络或磁盘上读取了字节流，那么读到的数据就是`bytes`。要把`bytes`变为`str`，就需要用`decode()`方法；
>
> 如果要在网络上传输，或者保存到磁盘上，就需要把`str`变为以字节为单位的`bytes`
>
> `len()`函数计算的是`str`的字符数，如果换成`bytes`，`len()`函数就计算字节数
>
> 如果没有特殊业务要求，请牢记仅使用`UTF-8`编码

#### unicode、utf-8和utf-16的区别

Unicode 是字符集，UTF-8 是编码规则

- 字符集：为每一个字符分配一个唯一的 ID（学名为码位 / 码点 / Code Point）
- 编码规则：将「码位」转换为字节序列的规则（编码/解码 可以理解为 加密/解密 的过程）

广义的 Unicode 是一个标准，定义了一个字符集以及一系列的编码规则，即 Unicode 字符集和 UTF-8、UTF-16、UTF-32 等等编码……

Unicode 字符集为每一个字符分配一个码位，例如「知」的码位是 30693，记作 U+77E5（30693 的十六进制为 0x77E5）。

UTF-8 顾名思义，是一套以 8 位为一个编码单位的可变长编码。会将一个码位编码为 1 到 4 个字节。

utf-16是用两个字节来编码所有的字符。

### pickling和unpickling？

模块 pickle 实现了对一个 Python 对象结构的二进制序列化和反序列化。

 "pickling" 是将 Python 对象及转化为一个字节流的过程

"unpickling" 将字节流转化回一个对象层次结构。

Pickle 协议和 JSON 间有着本质的**不同**：

- JSON 是一个文本序列化格式，而 pickle 是一个二进制序列化格式；
- JSON 是我们可以直观阅读的，而 pickle 不是；
- JSON在Python之外广泛使用，而pickle则是Python专用的；
- JSON 只能表示 Python 内置类型的子集，不能表示自定义的类；但 pickle 可以表示大量的 Python 数据类型。

### 说一下`namedtuple`的用法和作用

只有属性没有方法的类，用于组织数据，称为**数据类**。

在Python中可以用`namedtuple`（命名元组）来替代这种类。

```python
from collections import namedtuple

Card = namedtuple('Card', ('suite', 'face'))
card1 = Card('红桃', 13)
card2 = Card('草花', 5)
print(f'{card1.suite}{card1.face}')
print(f'{card2.suite}{card2.face}')
```

**命名元组与普通元组一样是不可变容器，**一旦将数据存储在`namedtuple`的顶层属性中，数据就不能再修改了，

对象上的所有属性都遵循“一次写入，多次读取”的原则。

和普通元组不同的是，命名元组中的数据有访问名称，可以**通过名称而不是索引来获取保存的数据**

**命名元组的本质就是一个类，所以它还可以作为父类创建子类。**

除此之外，命名元组内置了一系列的方法，例如，可以通过`_asdict`方法将命名元组处理成字典，也可以通过`_replace`方法创建命名元组对象的浅拷贝。

```python
class MyCard(Card):
    
    def show(self):
        faces = ['', 'A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K']
        return f'{self.suite}{faces[self.face]}'


print(Card)    # <class '__main__.Card'>
card3 = MyCard('方块', 12)
print(card3.show())    # 方块Q
print(dict(card1._asdict()))    # {'suite': '红桃', 'face': 13}
print(card2._replace(suite='方块'))    # Card(suite='方块', face=5)
```

## 面向对象

继承：将多个类的共同属性和方法封装到一个父类下面，然后在用这些类来继承这个类的属性和方法

封装：将有共同的属性和方法封装到同一个类下面

多态：Python天生是支持多态的。指的是基类的同一个方法在不同的派生类中有着不同的功能

### 新式类和经典类

Python3里只有新式类；Python2里面继承object的是新式类，没有写父类的是经典类

**区别**

- 新式类 保持class与type的统一，对新式类的实例执行`a.__class__`与`type(a)`的结果是一致的
- 旧式类的`type(a)`返回instance。
- 多重继承的属性搜索顺序不一样，新式类是采用广度优先搜索，旧式类采用深度优先搜索。

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

**缺点：**经典类的查找顺序是深度优先的规则，在访问`d.foo1()`的时候,D->B->A,找到了`foo1()`,调用A的`foo1()`，导致C重写的`foo1()`被绕过

### 类方法、类实例方法、静态方法

- 类方法: 是类对象的方法，使用 @classmethod 进行装饰，形参有cls，表示类对象
- 类实例方法: 是类实例化对象的方法，形参为self，指代对象本身;
- 静态方法: 是一个任意函数，使用 @staticmethod 进行装饰

  > 实例方法只能通过实例对象调用；
  >
  > 类方法和静态方法可以通过类对象或者实例对象调用，
  >
  > 使用实例对象调用的类方法或静态方法，最终通过类对象调用。

### 如何判断是函数还是方法？

如果是以函数的形式定义或者是静态方法，一定是函数

如果是类方法，一定是方法。

实例方法是方法，如果类直接调用实例方法，则是函数（直接调用运行是有问题的）。

```python
from types import MethodType,FunctionType
print(isinstance(obj.func, FunctionType)) 
print(isinstance(obj.func, MethodType))  
```

### 接口类与抽象类

```python
class Operate_database():    # 接口类1
    def query(self, sql):
        raise NotImplementedError

    def update(self, sql):
        raise NotImplementedError

from abc import ABCMeta,abstractmethod
class Operate_database(metaclass=ABCMeta):    # 接口类2
    @abstractmethod
    def query(self, sql):
        pass

    @abstractmethod
    def update(self, sql):
        pass
```

 Python 原生仅支持抽象类，不支持接口类，abc模块就是用来实现抽象类的。

若是类中所有的方法都没有实现，则认为这是一个接口，

若是有部分方法实现，则认为这是一个抽象类。

抽象类和接口类都仅用于被继承，不能被实例化.

### 描述符

- 一个实现了`__get__()`、`__set__()`、`__delete__() `其中至少一个方法的类就是一个描述符。
- 只实现了`__get__()`的称作非数据描述符
- 实现了`__get__()`和`__set__()`方法的称作数据描述符。

`__get__`： 用于访问属性。它返回属性的值，若属性不存在、不合法等都可以抛出对应的异常。

`__set__`：将在属性分配操作中调用。

`__delete__`：控制删除操作。

描述符的作用和优势，以弥补Python动态类型的缺点。

```python
class Score:
  def __init__(self, default=0):
     self._score = default

  def __set__(self, instance, value):
     if not isinstance(value, int):
       raise TypeError('Score must be integer')

     if not 0 <= value <= 100:
       raise ValueError('Valid value must be in [0, 100]')

     self._score = value

  def __get__(self, instance, owner):
     return self._score

  def __delete__(self):
     del self._score    

class Student:
  math = Score(0)
  chinese = Score(0)
  english = Score(0)

  def __init__(self, name, math, chinese, english):
     self.name = name
     self.math = math
     self.chinese = chinese
     self.english = english
```

**staticmethod**

```python
class Test:
  @staticmethod
  def myfunc():
     print("hello")

# 上下两种写法等价
class Test:
  def myfunc():
     print("hello")
  # 重点：这就是描述符的体现
  # 每调用一次，它都会经过描述符类的 __get__
  myfunc = staticmethod(myfunc)
```

**classmethod**

```python
class classmethod(object):
  def __init__(self, f):
     self.f = f
  def __get__(self, instance, owner=None):
     print("in classmethod __get__")
     def newfunc(*args):
       return self.f(owner, *args)
     return newfunc

class Test:
  def myfunc(cls):
     print("hello")
  # 重点：这就是描述符的体现
  myfunc = classmethod(myfunc)
```

### 元类

**元类是用来创建类的类。**

如果类属性中定义了`__metaclass__`，则在创建类的时候用元类来创建；

如果没有则向其父类查找`__metaclass__`。

如果都没有，则用`type()`创建类。

```python
# metaclass是类的模板，所以必须从`type`类型派生：
class ListMetaclass(type):
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return type.__new__(cls, name, bases, attrs)
    
class MyList(list, metaclass=ListMetaclass):
    pass
```

传入关键字参数`metaclass`时，指示Python解释器在创建`MyList`时，要通过`ListMetaclass.__new__()`来创建

在此，我们可以修改类的定义，比如，加上新的方法，然后，返回修改后的定义。

`__new__()`方法接收到的参数依次是：

1. 当前准备创建的类的对象；
2. 类的名字；
3. 类继承的父类集合；
4. 类的方法集合。

**元类作用**

- 拦截类的创建
- 修改类
- 返回修改后的类

**应用场景**

ORM：所有的类都只能动态定义，因为只有使用者才能根据表的结构定义出对应的类来。

>元类中的`__call__`会在你每次实例化的时候调用, 其实和`Foo.__new__`是一样的, 如果你的Foo定义了`__new__`, 元类中的`__call__`便不会执行
>
>元类中的`__new__`会在加载类的时候执行一次，在创建类的时候会调用类的`__new__`或者元类的`__call__`

## 字典

字典的查询、添加、删除的平均时间复杂度都是`O(1)`。因为字典是通过哈希表来实现的.

- 计算key的hash值`hash(key)`，再和mask做与操作【mask=字典最小长度（DictMinSize） - 1】，运算后会得到一个数字【index】，这个index就是要插入的enteies哈希表中的下标位置

- 若index下标位置已经被占用，则会判断enteies的key是否与要插入的key是否相等

  - 如果key相等就表示key已存在，则更新value值

  - 如果key不相等，就表示hash冲突，则会继续向下寻找空位置，一直到找到剩余空位为止。

### **开放寻址法**

开放寻址法中，所有的元素都存放在散列表里，当产生哈希冲突时，通过一个探测函数计算出下一个候选位置，如果下一个获选位置还是有冲突，不断通过探测函数往下找，直到找个一个空槽来存放待插入元素。

> 开放寻址法中解决冲突的方法有：线行探查法、平方探查法、双散列函数探查法

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

用开放寻址法中，**遇到哈希冲突时会找到下一个合适的位置，**例如ABC构成了探测链，查找元素时如果hash值相同，那么也是**顺着这条探测链不断往后找**，当删除探测链中的某个元素时，如果直接把B从哈希表中移除，即变成Unused状态，那么C就不可能再找到了，因此需要Dummy保证探测链的连续性。

dict对象的定义为：

```c
struct _dictobject {
  PyObject _HEAD
  Py_ssize_t ma_fill; /* # Active + # Dummy */
  Py_ssize_t ma_used; /* # Active */
  Py_ssize_t ma_mask; //slot -1
  PyDictEntry *ma_table;
  PyDictEntry *(*ma_lookup)(PyDictObject *mp, PyObject *key, long hash); // 搜索函数指针
  PyDictEntry ma_smalltable[PyDict_MINSIZE]; //默认的slot
};
```

### **dict对象的创建**

dict对象的创建很简单，先看看缓冲的对象池里有没有可用对象，如果有就直接用，没有就从堆上申请。

### **dict对象的插入**

如果不存在key-value则插入，存在则覆盖。

- 生成Hash
- 如果可用的entry<0，字典扩容
- 基于key、hash，查找可用哈希位置，以便于存储
  - 字典中是否有空余的值，或者如果找到了满足 hash 值与 key 相同的,就将 value 设置为找到的值
- 保存key、Hash、value值

### **dict对象的删除**

算出哈希值，找到entry，将其从Active转换成Dummy，并调整table的容量。

**注意**

（1） dict的key 或者 set的值都必须是可hash的不可变对象，都是可hash的

（2）当发现内存空间中的“空”只有1/3时，便会触发扩容操作。

### OrderedDict实现

使用了双向循环链表+哈希的方法

1. 哈希用来快速获取LinkNode，从而获取value，也支持快速定位
2. 双向链表可以快速删除一个元素，使用哈希可以实现基于key快速查找
3. 循环链表可以使得处理变得更加简单，不需要保存dummy的head和tail，以快速获取收尾节点。只需要保存实际的head，通过head.prev来快速定位尾结点。

## 整数

Python使用**小整数对象池**small_ints缓存了[-5，257）之间的整数，该范围内的整数在Python系统中是共享的，值相同就属于同一个对象。

对于同一个代码块中值不在`small_ints`缓存范围内的整数，如果同一个代码块中已经存在一个值与其相同的整数对象，那么就直接引用该对象，否则创建新的`int`对象。

## 字符串

Python解释器中使用了 intern（字符串驻留）的技术来提高字符串效率，值同样的字符串对象仅仅会保存一份，放在一个字符串储蓄池中，是共用的。

**简单原理**

维护一个字符串存储池，这个池子是一个字典结构

如果字符串已经存在于池子中，直接返回之前创建好的字符串对象，

如果不存在，则构造一个字符串对象并加入到池子中去。

> 在shell中，并非全部的字符串都会采用intern机制。仅仅包括下划线、数字、字母的字符串才会被intern。不能超过20个字符。因为如果超过xx个字符的话，解释器认为这个字符串不常用，不用放入字符串池中。
>
> 字符串拼接时，运行时拼接，不会intern；例如"hell" + "o"在编译时完成拼接的才会intern

### 字符串不等

**字符串打印出来看着一样，但是判断却是False**？

如果两个字符串末尾有其他符号，比如回车‘\n’，print的时候无法发现的

**==判断是 True ，is 判断却是 False?******

字符串来自不同的内存块，内存地址不一样

**is判断是False，用id判断却是True**

```python
In [4]: foo.bar is Foo.bar
Out[4]: False

In [5]: id(foo.bar) == id(Foo.bar)
Out[5]: True
```

foo.bar本身并不是简单的名字，而是表达式的计算结果，是一个 method object，method object只是一个临时的中间变量而已，对临时的中间变量做id是没有意义的。

只有你能保证对象不会被销毁的前提下，你才能用 id 来比较两个对象。

## 堆 栈

在Python中，变量也称为：对象的引用。变量存储的就是对象的地址。

**变量位于：栈内存。**

**对象位于：堆内存。**

内存空间在逻辑上分为三部分：代码区、静态数据区和动态数据区，动态数据区又分为栈区和堆区。

**代码区：**程序中的代码数据、二进制数据、方法数据等等程序运行需要的预加载数据。

**静态数据区：**存储**全局变量、静态变量**。

**栈区：**存储变量。存储运行方法的形参、局部变量、返回值。由系统自动分配和回收。

**堆区**：对象真实数据。

## 内存回收机制

python采用的是引用计数机制为主，标记-清除和分代收集两种机制为辅的策略。

###  **引用计数法**

**原理：**每个对象维护一个ob_ref字段，用来记录该对象当前被引用的次数，每当新的引用指向该对象时，它的引用计数加1，每当该对象的引用失效时，计数减1，一旦对象的引用计数为0，该对象立即被回收，占用的内存空间将被释放。

**缺点：**不能解决对象的循环引用

### 标记清除

解决容器对象可能产生的循环引用问题。（只有容器对象才会产生循环引用的情况，比如列表、字典、用户自定义类的对象、元组等）

**A）标记阶段，遍历所有的对象，如果是可达的（reachable），也就是还有对象引用它，那么就标记该对象为可达**；

B）清除阶段，再次遍历对象，如果发现某个对象没有标记为可达，则就将其回收。

### 分代回收

标记清除时，应用程序会被暂停，为了减少应用程序暂停的时间。

**对象存在时间越长，越可能不是垃圾，应该越少去收集**。

给对象定义了三种世代，每一个新生对象在0代中，如果它在一轮gc扫描中活了下来，那么它将被移至1代，在那里他将较少的被扫描，如果它又活过了一轮gc，它又将被移至2代，在那里它被扫描的次数将会更少。

### **gc的扫描在什么时候会被触发呢**?

年轻代链表的总数达到上限时。

当某一世代的扫描被触发的时候，比该世代年轻的世代也会被扫描。

### **调优手段**

1.手动垃圾回收

2.调高垃圾回收阈值

3.避免循环引用

## 退出Python时，是否释放全部内存？

进程退出的时候，资源最终都会释放掉，这是操作系统负责的。

如果是一段程序运行结束之后：

1. CPython会通过引用计数立即释放引用数量为0的对象（其它版本解释器并不保证）；循环引用的对象会在下一次GC时释放，除非有两个对象都带有`__del__`析构函数，且直接或间接循环引用。这种情况下，所有循环引用的对象都无法被释放。原因在于无法确定`__del__`的执行顺序。
2. 全局引用的对象无法被回收，但也不只是模块中直接或间接保存的对象，还包括未退出的线程使用的对象，解释器缓存的小整数和字符串，还有C模块里间接引用的对象等等。
3. C扩展直接通过malloc分配的内存自然无法通过gc来回收，但一般如果存在没有被回收的内存说明是有内存泄漏的，这属于实现的bug

## 协程，线程和进程

### 进程

进程是系统资源分配的最小单位，进程拥有自己独立的内存空间，所有进程间数据不共享，开销大。在Python中，进程适合计算密集型任务。

#### 进程间的通信（IPC）

**1）管道（Pipe**）：通过`send()`和`recv()`来发送和接受信息，适合父子进程关系或者两个子进程之间。 

2）**有名管道（FIFO）**：有名管道也是半双工的通信方式。 将自己注册到文件系统里一个文件，通过读写这个文件进行通信。允许在没有亲缘关系的进程之间使用。要求读写双方必须同时打开才可以继续进行读写操作，否则打开操作会堵塞直到对方也打开。

3）**信号量（Semaphore）**：信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。 创建子进程时将信号量my_semaphore作为参数传入子进程任务函数，子进程中使用semaphore.acquire() 尝试获取信号量，semaphore.release()尝试 释放信号量。

**4）队列（Queue）**。 使用get/put在父子进程关系或者两个子进程之间通信。

5）**信号 （signal）**：用于通知接收进程某个事件已经发生，可以设置信号处理函数。 

6）共享内存（shared memory）：操作系统负责将同一份物理地址的内存映射到多个进程的不同的虚拟地址空间中。进而每个进程都可以操作这份内存。需要在进程访问时做好并发控制，比如使用信号量。 python标准库mmap，apache开源的pyarrow都是。

7）套接字（socket）：套接字也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同机器间的进程通信。 

**8）文件** 

（1）仅进程同步不涉及数据传输，可以使用信号、信号量；
（2）若进程间需要传递少量数据，可以使用管道、有名管道、队列；
（3）若进程间需要传递大量数据，最佳方式是使用共享内存，推荐使用pyarrow，这样减少数据拷贝、传输的时间内存代价；
（4）跨主机的进程间通信（RPC）可以使用socket通信。

**共享变量**

使用 Process 定义的多进程之间（父子或者兄弟）共享变量可以直接使用 multiprocessing 下的 Value，Array，Queue 等，如果要共享 list，dict，可以使用强大的 Manager 模块。

###  线程

线程是cpu调度执行的最小单位，依赖进程存在，一个进程至少有一个线程。在python中，线程适合IO密集型任务。

同一个进程下的线程共享程序的内存空间**（如代码段，全局变量，堆等）**

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

#### 线程池的工作原理

减少线程本身创建和销毁造成的开销，属于典型的空间换时间操作。

创建和释放线程涉及到大量的系统底层操作，开销较大，如果变成预创建和借还操作，将大大减少底层开销。

- 在应用程序启动后，线程池创建一定数量的线程，放入空闲队列中。这些线程最开始都处于阻塞状态，不会消耗CPU，占用少量的内存。
- 当任务到来后，从队列中取出一个空闲线程，把任务派发到这个线程中运行，并将标记为已占用。
- 当线程池中所有的线程都被占用后，可以选择自动创建一定数量的新线程，用于处理更多的任务，也可以选择让任务排队等待直到有空闲的线程可用。
- 在任务执行完毕后，线程并不退出结束，而是继续保持在池中等待下一次的任务。
- 当系统比较空闲时，大部分线程长时间处于闲置状态时，线程池可以自动销毁一部分线程，回收系统资源。

线程池组成部分：

1. 线程池管理器：用于创建并管理线程池。
2. 工作线程和线程队列：线程池中实际执行的线程以及保存这些线程的容器。
3. 任务接口：将线程执行的任务抽象出来，形成任务接口，确保线程池与具体的任务无关。
4. 任务队列：线程池中保存等待被执行的任务的容器。

###  协程

协程是一种用户态的轻量级线程，用户自己来编写调度逻辑的。

协程拥有自己的寄存器上下文和栈。协程的切换都在用户空间内进行，不需要进行系统调用。对CPU来说，协程其实是单线程，所以CPU不用去考虑怎么调度、切换上下文，这就省去了CPU的切换开销，所以协程在一定程度上又好于多线程

协程调度时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，基本没有内核切换的开销，可以不加锁的访问全局变量，上下文的切换非常快。

> 不管是进程还是线程，每次阻塞、切换都需要陷入系统调用，先让CPU跑操作系统的调度程序，然后再由调度程序决定该跑哪一个进程/线程。
>
> ----
>
> 协程（Coroutine）又称微线程，属于用户级线程。 gevent 就是一种协程实现方式，除了 gevent 还有 asyncio。用户级线程就是在一个内核调度实体上映射出来的多个用户线程，用户线程的创建、调度和销毁完全由用户程序控制, 对内核调度透明：内核一旦将 cpu 分配给了线程，该 cpu 的使用权就归该线程所有，线程可以再次按照比如时间片轮转等常规调度算法分配给每个微线程，从而实现更大的并发自由度，但所有的微线程只能在该 cpu 上运行，无法做到并行。
>
> 把协程看作这些映射出来的“微线程”。用户程序控制的协程需要解决线程的挂起和唤醒、现场保护等问题，然而区别于线程的是协程不需要处理锁和同步问题，因为多个协程是在一个用户级线程内进行的，但需要处理因为单个协程阻塞导致整个线程（进程）阻塞的问题。



**如何利用多核CPU呢？**

最简单的方法是多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

每个进程有各自独立的GIL，互不干扰，这样就可以真正意义上的并行执行，所以在python中，多进程的执行效率优于多线程

**常用模块**

greenlet：提供了切换任务的快捷方式，但是遇到io无法自动切换任务，需要手动切换

gevent：开启协程任务并切换的模块，遇到io自动切换任务。

asyncio：`@asyncio.coroutine`装饰器的函数称为协程函数。

`yield from`语法用于将一个生成器部分操作委托给另一个生成器。

`async`/`await`：`@asyncio.coroutine`和`yield from`的语法糖

**缺点**

- 无法利用多核资源：协程的本质是个单线程

- 进行阻塞（Blocking）操作（如IO时）会阻塞掉整个程序

**协程主要使用场景**

网络请求，比如爬虫，大量使用 aiohttp

文件读取， aiofile

web 框架， aiohttp， fastapi

数据库查询， asyncpg, databases

**协程优于线程**

- python 线程调度方式是，每执行 100 个字节码或者遇到阻塞就停止当前线程，然后进行一个系统调用，让 os 内核选出下一个线程。但是协程只会在阻塞的时候，切换到下一个协程。因此线程的切换存在很多是无效的切换，当线程数量越大，这种因为调度策略的先天不足带来的性能损耗就越大。
- 线程需要进行系统调用，协程不需要。系统调用需要进入内核态，无效的调度会让这部分开销显得更大
- 协程可以自主调度，而线程只能决定合适退出，但是下一个线程是谁则依赖于操作系统。

### 僵尸进程和孤儿进程 

孤儿进程： **父进程退出，子进程还在运行的这些子进程都是孤儿进程，**孤儿进程将被init 进程（进程号为1）所收养，并由init 进程对他们完成状态收集工作。

僵尸进程： 进程使用fork 创建子进程**，如果子进程退出，而父进程并没有调用wait 获取子进程的状态信息**，那么子进程的进程描述符仍然保存在系统中，这些进程是僵尸进程。

避免僵尸进程的方法：

1.用`wait()`函数使父进程阻塞，这个`wait()`操作就负责回收子进程，这样也不会产生僵尸进程。但这样做有个致命的问题，wait是阻塞的，如果进行wait，主进程就什么都做不了。

2.使用信号量，子进程退出时向父进程发送SIGCHILD信号，在signal handler 中调用waitpid，这样父进程不用阻塞

3.fork 两次用孙子进程去完成子进程的任务，孙子进程刚产生，它的父亲就退出了，成了孤儿进程，由init收养

##  Global Interpreter Lock

Python 默认的解释器是 CPython，**GIL 是存在于 CPython 解释器中的**。

 **CPython 解释器的内存管理**不是线程安全的。执行 Python 字节码时，引入了为了保护访问 Python 对象而阻止多个线程执行的一把互斥锁GIL。某个线程想要执行，必须先拿到GIL锁，并且在一个python进程中，GIL只有一个。拿不到通行证的线程，就不允许进入CPU执行。

**因此，同一时刻，只有一个线程在运行，其它线程只能等待，即使是多核CPU**，**也没办法让多个线程「并行」地同时执行代码，只能是交替执行**，因为多线程涉及到上线文切换、锁机制处理（获取锁，释放锁等），所以，多线程执行不快反慢。

尽管存在 GIL，但 python 多线程仍然不是线程安全的，对于共享状态的场合仍然需要借助锁同步。

常见的 Python 解释器：IPython（基于Cython）、Jython（Python 代码编译成 Java 字节码，依赖 Java 平台，不存在 GIL）、IronPython（ .Net 平台下的 Python 解释器，Python 代码编译成 .Net 字节码，不存在 GIL）

### GIL原理

python 的线程就是 C 语言的 pthread，它是通过操作系统调度算法调度执行的。

Python 2.x 的代码执行是基于 opcode 数量的调度方式，简单来说就是每执行一定数量的字节码，或遇到系统 IO 时，会强制释放 GIL，然后触发一次操作系统的线程调度。

 Python 3.x 基于固定时间的调度方式，就是每执行固定时间的字节码，或遇到系统 IO 时，强制释放 GIL，触发系统的线程调度。

### 为什么会有GIL

90年代单核 CPU 还是主流，多线程的应用场景也不多，大部分时候还是以单线程的方式运行，单线程不要涉及线程的上下文切换，效率反而比多线程更高（在多核环境下，不适用此规则），设计一个全局锁是**那个时代保护多线程资源一致性最简单经济的设计方案**。

多核心时代来临，当大家试图去拆分和去除 GIL 的时候，发现大量库的代码和开发者已经重度依赖 GIL（默认认为 Python内部对象是线程安全的，无需在开发时额外加锁），所以这个去除 GIL 的任务变得复杂且难以实现。

> CPython 解释器在创建变量时，首先会分配内存，然后对该变量的引用进行计数，这称为 引用计数。如果变量的引用数变为 0，这个变量就会从内存中释放掉。而当多个线程内共享一个变量时，CPython 锁定引用计数的关键就在于使用了 GIL，它会谨慎地控制线程的执行情况，无论同时存在多少个线程，解释器每次只允许一个线程进行操作。

### GIL的实现是线程不安全?为什么?

单核情况下:

![](gil.png)

> 1. 到第5步的时候，可能这个时候python正好切换了一次GIL(据说python2.7中，每100条指令会切换一次GIL),执行的时间到了，被要求释放GIL,这个时候thead 1的count=0并没有得到执行，而是挂起状态，count=0这个上下文关系被存到寄存器中.
> 2. 然后到第6步，这个时候thead 2开始执行，然后就变成了count = 1,返回给count，这个时候count=1.
> 3. 然后再回到thead 1，这个时候由于上下文关系，thead 1拿到的寄存器中的count = 0，经过计算，得到count = 1，经过第13步的操作就覆盖了原来的count = 1的值，所以这个时候count依然是count = 1，所以这个数据并没有保护起来。

python2.x和3.x都是在执行IO操作的时候，强制释放GIL，使其他线程有机会执行程序。

## 性能优化

### python为什么慢

- GIL

  无论同时存在多少个线程，解释器每次只允许一个线程进行操作。

- 解释型语言而不是编译型语言

  Python 都会解释字节码并本地执行。.NET 和 Java 都是 JIT 编译的，JIT 会允许在运行时进行优化，缺点是启动慢。

- 动态类型的语言

  类型比较和类型转换消耗的资源是比较多的，每次读取、写入或引用变量时都会检查变量的类型

### lru_cache装饰器

为函数提供缓存功能。在下次以相同参数调用时直接返回上一次的结果，要求参数可hash。

### 性能分析

Python 标准库提供了同一分析接口的两种不同实现：

1. 建议使用 [`cProfile`](https://docs.python.org/zh-cn/3/library/profile.html#module-cProfile) ；这是一个 C 扩展插件，因为其合理的运行开销，所以适合于分析长时间运行的程序。
2. [`profile`](https://docs.python.org/zh-cn/3/library/profile.html#module-profile) 是一个纯 Python 模块（[`cProfile`](https://docs.python.org/zh-cn/3/library/profile.html#module-cProfile) 就是模拟其接口的 C 语言实现），但它会显著增加配置程序的开销。如果你正在尝试以某种方式扩展分析器，则使用此模块可能会更容易完成任务

支持输出：调用次数、在指定函数中消耗的总时间（不包括调用子函数的时间）、指定的函数及其所有子函数（从调用到退出）消耗的累积时间、函数运行一次的平均时间

### 加速python运行

1. 优化代码和算法
   -  避免全局变量
   - 避免模块和函数属性访问：对于频繁访问的变量`sqrt`，通过将其改为局部变量可以加速运行。
   - 避免类内属性访问：通过将需要频繁访问的类内属性赋值给一个局部变量，可以提升代码运行速度。
   - 避免数据复制：交换值时不使用中间变量、避免无意义的数据复制、字符串拼接用join而不是+
   - 利用if条件的短路特性
   - 使用numba.jit
   - 循环优化： 用for循环代替while循环、使用隐式for循环代替显式for循环
   - 选择合适的数据结构：如果有频繁的新增、删除操作，新增、删除的元素数量又很多时，list的效率不高。此时，应该考虑使用collections.deque。

2. 使用 PyPy

   PyPy通过使用一种 Just-in-time（JIT，即时编译）技术来实现的。CPython 使用解释来执行代码，虽然这一做法提供了很大的灵活性，但速度也变得慢了下来。使用 JIT，你的代码是在运行程序时即时编译的。它结合了 Ahead-of-time（AOT，提前编译）技术的速度优势（由 C 和 C++ 等语言使用）和解释的灵活性。另一个优点是 JIT 编译器可以在运行时不断优化代码。代码运行的时间越长，它就会变得越优化。

3. 使用线程

4. 使用 Asyncio
5. 同时使用多个处理器

使用多个进程，使用分布式方案等。

## 单例模式

### **使用装饰器**

```python
def singleton(cls):
  instances = {}
  def wrapper(*args, **kwargs):
     if cls not in instances:
       instances[cls] = cls(*args, **kwargs)
     return instances[cls]

  return wrapper

@singleton
class Foo(object):
  pass

foo1 = Foo()
foo2 = Foo()

print(foo1 is foo2) # True
```

### 使用new

New 是真正创建实例对象的方法，所以重写基类的new 方法，以此保证创建对象的时候只生成一个实例。

但是以上的方法在多线程中会有线程安全问题，当有多个线程同时去初始化对象时，就很可能同时判断__instance is None，从而进入初始化instance的代码中。所以需要用**互斥锁**来解决这个问题。

```python
class Singleton(object):
  def __new__(cls, *args, **kwargs):
     if not hasattr(cls, '_instance'):
       cls._instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
     return cls._instance


class Singleton(object):
  __instance = None
  def __new__(cls, *args, **kwargs):
     if cls.__instance is None:
       cls.__instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
     return cls.__instance

class Foo(Singleton):
  pass

foo1 = Foo()
foo2 = Foo()

print(foo1 is foo2) # True
```

### 用装饰器实现同步锁

```python
def make_synchronized(func):
    import threading
    func.__lock__ = threading.Lock()
    
    def synced_func(*args, **kwargs):
        with func.__lock__:
            return func(*args, **kwargs)
    
    return synced_func

class Singleton(object):
    __instance = None
    @make_synchronized
    def __new__(cls, *args, **kwargs):
        if not cls.__instance:
            cls.__instance = object.__new__(cls)
        return cls.__instance
```

### 双重检查线程安全单例模式

```python
class SingletonSample(object):
    _instanceLock = threading.Lock()
 
    @classmethod
    def get_instance(cls):
        # 初次检查，避免锁竞争
        if not hasattr(cls, "_instance"):
            with cls._instanceLock:
                # 获取到锁后再次判断，避免重复创建
                if not hasattr(cls, "_instance"):
                    cls._instance = super(SingletonSample, cls).__new__(cls)
                    print cls._instance
        return cls._instance
```

### classmethod

```python
import time
import threading
class Singleton(object):
     _instance_lock = threading.Lock() 
    def __init__(self):
        time.sleep(1)        
    @classmethod
    def instance(cls, *args, **kwargs):
        with Singleton._instance_lock: # 加锁
            if not hasattr(Singleton, '_instance'):
                Singleton._instance = Singleton(*args, **kwargs)
            return Singleton._instance
```

### 元类

元类是用于创建类对象的类，类对象创建实例对象时一定要调用call方法，因此在调用call时候保证始终只创建一个实例即可，type是python的元类

```python
class Singleton(type):
    def __call__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            cls._instance = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instance


# Python2
class Foo(object):
	__metaclass__ = Singleton
```

### 线程安全--元类

```python
import threading

class MetaSingleton(type):
    _instance_lock = threading.Lock()
    def __call__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            with MetaSingleton._instance_lock:
                if not hasattr(cls, '_instance'):
                    cls._instance = super(MetaSingleton, cls).__call__(*args, **kwargs)

        return cls._instance
    
  
class Singleton(metaclass=MetaSingleton):
    def __init__(self, name):
        self.name = name
```

## select,poll和epoll 

 select，poll，epoll都是IO多路复用的机制。I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。 

## docker和virtualenv技术具体有什么不同？

- virtualenv是python的版本和库管理器，virtualenv虚拟python运行环境
- docker是虚拟化整个系统环境工具，docker不仅可以跑python，还可以跑其他的需要进程环境隔离的程序。

环境的特点有二：

- Python版本固定。即使系统的Python升级了，虚拟环境中的仍然不受影响，保留开发状态。
- 所有Python软件包，都只在这个环境生效。一旦退出，则回到用户+系统的默认环境中。

这两个特点，由两个小手段实现。

- 改变当前Shell的`PATH`。
- 改变Python运行时的`sys.path`。

