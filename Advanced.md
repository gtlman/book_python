# <center>高级特性</center>

## <center>性能分析</center>
### GIL（Global Interpreter Lock）
- Cpython解释器的内存管理并不是线程安全的（我记得跟引用计数有关）
- 保护多线程下对Python对象的访问
- Cpython使用简单的锁机制避免多个线程同时执行字节码

### GIL影响
限制了程序的多核执行
- CPU密集型程序无法利用多核优势
- IO期间会释放GIL，因此对IO密集型的影响不大（web一般就是IO密集型的）

### 线程安全
什么是线程安全？正如数据库并发操作可能造成冲突一样，多线程之间同时对一个内存对象进行操作可能会出现冲突，例如线程A，B都要执行V = V + 1，因为这不是一个原子操作，可能出现A获取了V的值还没更新，B就更新了V的值，然后A进行的更新就会覆盖了B的操作，从而造成线程安全

为什么有多了GIL之后，还要留心线程安全问题

在一个python的进程内，不仅有xxx.py的主线程或者由该主线程开启的其他线程，还有解释器开启的垃圾回收等解释器级别的线程，而GIL保护的是就是解释器级别线程间的数据

什么操作是原子操作？

一个字节码可以完成的操作就是原子操作，通过` import dis`模块，` dis.dis(func) `可以返回`func`方法的字节码

对于非原子操作的加锁保证线程安全：
```
lock = threading.Lock()
n = [0]
def f():
    with lock:
        n[0] = n[0] + 1
        n[0] = n[0] + 1
```

### 剖析新能
1. profile/cprofile/pyflame工具
2. 二八定律，大部分时间耗费在少量代码上

#### cprofile
```
python -m cProfile [-o output_file] [-s sort_order] (-m module | myscript.py)
```

```
# test.py文件
def foo():
    for a in range(0, 101):
        for b in range(0, 101):
            if a + b == 100:
                yield a, b

for a, b in foo():
    print(a, b)
```

直接打印结果，执行命令：`python -m cProfile -s cumulative test.py`，-s参数有许多，`cumulative`参数表示根据函数累计花费的时间进行排序

```
# 可以查看到执行性能分析
         206 function calls in 0.017 seconds        #表示有206个函数调用，如果有递归调用的函数会额外表示

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.017    0.017 {built-in method builtins.exec}
        1    0.000    0.000    0.017    0.017 test.py:1(<module>)
      101    0.016    0.000    0.016    0.000 {built-in method builtins.print}  #可以发现，print函数非常耗时
      102    0.001    0.000    0.001    0.000 test.py:1(foo)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

```
# 参数解析
ncalls：表示函数调用的次数；
tottime：表示指定函数的总的运行时间，除掉函数中调用子函数的运行时间；
percall：（第一个percall）等于 tottime/ncalls；
cumtime：表示该函数及其所有子函数的调用运行的时间，即函数开始调用到返回的时间；
percall：（第二个percall）即函数运行一次的平均时间，等于 cumtime/ncalls；
filename:lineno(function)：每个函数调用的具体信息；
需要注意的是cProfile很难搞清楚函数内的每一行发生了什么，是针对整个函数来说的。
```

将运行结果输出到文件: `python -m cProfile -o cprofile.out -s cumulative test.py`，该输出文件是二进制文件，要通过` pstats `模块来进行分析
```
import pstats
p=pstats.Stats("cprofile.out")
p.sort_stats("cumulative")          #将分析数据根据cumulative模式排序
p.print_stats()                     #返回分析结果
```

返回结果与上面一致

**[Profile官方文档](https://docs.python.org/zh-cn/3/library/profile.html#the-python-profilers "profile官方文档")**


### 服务端性能优化
1. 数据结构和算法优化
2. 数据库层面：索引优化，慢查询消除，批量操作减少IO，NoSQL
3. 网络IO：批量操作，pipeline减少数据库请求次数
4. 缓存：使用内存数据库redis/memcached
5. 异步：asyncio，celery
6. 并发：gevent/多线程

## <center>generator 生成器</center>
- 生成器就是可以生成值的函数
- 当一个函数里面有了yield关键子就成了生成器
- 生成器可以挂起执行并且保持当前执行状态


## <center>协程</center>
### 基于生成器的协程
基于生成器增强实现
- 协程需要使用send(None) 或者 next(coroutine) 预激 启动
- 在 yield 处协程会暂时执行
- 单独地 yield value 会产出值给调用方
- 可以通过 coroutine.send(value)来给协程发送值，发送的值会赋值给 yield 表达式左边的变量 value = yield
- 协程执行完后（没有下一个 yield 语句）就会抛出 StopIteration 异常

### 协程装饰器
```
from functools import wraps         #wraos可以将name之类的都进行修改
def coroutine(func):
    @wraps(func)
    def primer(*args, **kwargs):
        gen = func(*args, **kwargs)
        next(gen)                   #预激协程
        return gen
    return primer
```

### python3.5+原生协程
async/asait

## <center>深拷贝 与 浅拷贝
**浅拷贝**：创建新对象完全拷贝，不管是基本类型还是内存地址

**深拷贝**：拷贝所有属性指向的动态分配内存，并对子对象进行深拷贝（递归操作）

**深拷贝 的特点**：拷贝之后，内容完全一致但是对于A的修改不会影响到B（拷贝时所有的子对象都满足该条件）

**如何实现 深拷贝？**

不可变对象在进行修改时，会自动申请另一块内存空间 或者 自动指向另一个不可变对象，因此不用管（tuple？）

可变对象则需要重新new实例来申请另一块内存空间，并且子对象也要递归该处理，tuple也要归到该操作处理

**如何初始化一个二维数组？**

因为数组是可变对象，当数组元素是数组的时候，要留心**浅拷贝**，留意元素是原数组的引用还是一个新的数组
```
A = [1]，B=[2], C=[3], D=[A,B,C]
A = [2] #会同步印象数组D，要留意这是不是你想要的
```

## <center>类</center>
### 元类
一切皆对象，类也是对象

类是创建实例的模板

元类是创建类的模板

type是所有类的元类（包括自定义的继承自type的元类）

### __new__和__init__，__metaclass__
__new__方法会返回一个创建的实例,而__init__什么都不返回.

当创建一个新实例时调用__new__,初始化一个实例时用__init__.

`_metaclass__`是创建类时起作用，所以我们可以分别使用`__metaclass__, __new__和__init__`来分别在类创建,实例创建和实例初始化的时候做一些修改，最常用在ORM这种复杂的东西上


## <center>python垃圾回收
### reference counting 引用计数
1. Python一切皆对象
2. 每一个对象都有一个ob_refcnt的属性来表示该对象的引用数->引用计数器
3. del 的本质就是取消变量的引用，并使引用计数减一

**优点**：简单，实时
**缺点**：
1. 维护计数器需要消耗资源
2. 循环引用会导致内存泄漏（A引用B，B引用C，C引用A）

```
a = []
b = []
a.append(b)
b.append(a)
```
此时就会出现循环引用，即使del a ,del b也无法释放内存，因为互相引用这两块内存的计数器永远不为0

### 标记-清除机制
为了解决循环引用的BUG

无空余内存时，从寄存器和程序栈上的引用触发，遍历以对象为节点，以引用为边构成的图，把所有可以访问的对象打上标记，然后清扫一遍内存空间，把所有没标记的对象释放

### 分代技术
根据存活时间分代，存活时间越长的代，执行垃圾回收的频率就越低（很久才去收一次垃圾）

## <center>单例模式</center>
是一种常用的设计模式，通过单例模式可以保证系统中一个类只有一个实例而且该实例易于外界访问

许多时候整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为。比如在某个服务器程序中，该服务器的配置信息存放在一个文件中，这些配置数据由一个单例对象统一读取，然后服务进程中的其他对象再通过这个单例对象获取这些配置信息。这种方式简化了在复杂环境下的配置管理。

实现对象单例模式的思路是：
1. 一个类能返回对象一个引用(永远是同一个)和一个获得该实例的方法（必须是静态方法，通常命名为getInstance）
2. 当我们调用这个方法时，如果类持有的引用不为空就返回这个引用，如果类保持的引用为空就创建该类的实例并将实例的引用赋予该类保持的引用
3. 同时我们还将该类的构造函数定义为私有方法，这样其他处的代码就无法通过调用该类的构造函数来实例化该类的对象，只有通过该类提供的静态方法来得到该类的唯一实例，保证了一个类只能有一个对象

上面这种单例的实现方式我们通常称之为懒汉模式，指的是只有在需要对象的时候才会生成（getInstance方法被调用的时候才会生成）。

懒汉模式是要用再创建，避免了不必要的开销，但是在第一次使用的时候会有较高的消耗，因此有了另一种**饿汉模式**

**饿汉模式**就是在类创建的时候就提前创建单例实例

### __new__ + 类属性 实现
```
class Singleton(object):
    def __new__(cls, *args, **kw):
        if not hasattr(cls, '_instance'):
            orig = super()
            cls._instance = orig.__new__(cls, *args, **kw)  #创建类属性，懒汉模式
        return cls._instance

class MyClass(Singleton):
    a = 1
```

### import方法
作为python的模块是天然的单例模式

```
# mysingleton.py
class My_Singleton(object):
    def foo(self):
        pass

my_singleton = My_Singleton()

# to use
from mysingleton import my_singleton

my_singleton.foo()
```

### 单例的线程安全
懒汉模式不是线程安全的，当多个线程都去初始化单例对象的时候就会出现问题,多个线程对单例实例的查询`if(instance==null)`都通过了，然后各个线程都会去创建单例实例，导致产生了多个单例实例

可以通过双重校验锁来实现线程安全，也就是当线程要创建单例的时候加锁，然后获取锁之后再判断是否存在单例即可