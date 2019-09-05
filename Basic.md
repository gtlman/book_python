# <center>基础</center>

## 语法糖
### 列表，集合 和 字典推导
```
[i for i in range(10) if i % 2 == 0]    #返回[0, 2, 4, 6, 8]
(i for i in range(10) if i % 2 == 0)    #返回生成器<generator object <genexpr> at 0x000001E9FD6192A0>
{i*i for i in range(10) if i % 2 == 0}    #返回集合{0, 64, 4, 36, 16}

a = ['a', 'b']
b = [1, 2, 3]
{k: v for k, v in zip(a, b)}            #返回{'a': 1, 'b': 2}
```

## 函数
### 不可变对象
None, int, float, str, Bool, tuple, frozenset都是不可变对象
### 可变对象
list，set，dict，和其它不可变对象都是可变对象
### 参数传递
参数传递对象的**引用**(可以联想一下引用计数)

对象是可变对象时，对可变对象的改变会直接修改对象内存

对象是不可变对象时，对不可变对象的改变会重新申请一块对象内存作为结果

```
def clear_list(l):
    l = []
ll = [1,2,3]
clear_list(ll)
print(ll)               #返回[1, 2, 3]
```

### 可变对象 作为默认参数
```
def fl(l=[1]):
    l.append(1)
    print(l)
fl()            #[1, 1]
fl()            #[1, 1, 1]
```

默认参数只会计算一次，这里计算一次之后，赋予`l`参数`[1]`对象的内存地址

### `*arg` 和 `**kwargs` 可变参数
arg会把参数组成为一个tuple

```
def parg(*arg):
    print(arg)
    return arg
t = parg(1,2,3,4)   #返回(1, 2, 3, 4)
parg(*t)            #返回(1, 2, 3, 4)
```

kwargs会把参数组合为一个dict

```
def pkwargs(**kwargs):
    print(kwargs)
    return kwargs
d = pkwargs(a=1,b=2,c=3,d=4)   #返回{'a': 1, 'b': 2, 'c': 3, 'd': 4}
pkwargs(**d)                   #返回{'a': 1, 'b': 2, 'c': 3, 'd': 4}
```

## 异常
python中所有异常均继承于BaseException
### 内置异常
SystemExit          #sys.exit()会抛出该异常，python解释器捕获之后就会退出程序
KeyboardInterrupt   #Ctrl + C
GeneratorExit       #生成器退出的时候抛出

### 常见场景
- 网络请求（超时，连接错误等）
- 资源访问（权限问题，资源不存在）
- 代码逻辑（越界访问，KeyError等）

### 异常处理
```
try:
    # func                               #可能抛出异常的代码段
except (Exception1, Exception2) as e:   #可以捕获多个异常并处理
    # do something                       #异常处理代码
else:
    # pass                               #没有捕获到异常
finally:
    # pass                               #无论异常有没有捕获最后都会执行，一般处理资源的关闭和释放
```

### 自定义异常
继承Exception，为什么不是BaseException

因为如果继承BaseException，希望通过捕获BaseException来捕获业务代码异常的时候就会出现系统异常也被捕获的严重问题，该问题会导致例如ctrl+c这样的系统异常无法被解释捕获从而导致程序无法终止

raise MyException()抛出异常

## unit testing 单元测试
- 针对一个函数， 一个类这样的小单元进行正确性校验
- 从而实现自底向上保证程序的正确性

### 单元测试的意义
1. 保证逻辑正确
2. 影响设计，易测 = 高内聚，低耦合
3. 方便回归测试

### 单元测试相关库
- nose/pytest 较为常用
- mock 模块
- coverage 统计覆盖率

#### pytest
```
pip install pytest
def test():       #xxx.py下的方法，func是待测试单元
    assert func('正常值') == '预期值'
    assert func('边界值') == '预期值'
    assert func('异常值') == '预期值'
pytest xxx.py    #pytest会执行xxx.py下test方法内的所有测试，并返回结果
```
#### mock
在项目的单元测试过程中，会遇到：
1. 接口的依赖
2. 外部接口调用
3. 测试环境非常复杂

但是单元测试应该只针对当前单元进行测试, 所有的内部或外部的依赖应该要求是稳定的, 让测试能聚焦在当前单元的实现上

例如，我们要测试A模块，然后A模块依赖于B模块的调用。但是，由于B模块的改变，导致了A模块返回结果的改变，从而使A模块的测试用例失败。对于A模块，以及A模块的用例来说，并没有变化，测试不应该失败。

此时我则可以使用mock对象模拟B模块来派出B模块对A模块的干扰

##### 基本用法
```
# client.py文件
import requests

def send_request(url):
    r = requests.get(url)
    return r.status_code
def visit_baidu():
    return send_request("http://www.baidu.com")

# test.py文件
import unittest
from unittest import mock
from . import client

class TestClient(unittest.TestCase):
    def test_success_request(self):
        success_send = mock.Mock(return_value='200')    #创建一个返回值是200的mock对象
        client.send_request = success_send              #用mock对象替换client模块的send_request方法
        self.assertEqual(client.visit_baidu(), '200')  #测试client模块的visit_baidu方法
```

###### mock.patch
mock方法

```
import unittest
from unittest import mock
from . import client

class TestClient(unittest.TestCase):
    @mock.patch('client.send_request')
    def test_success_request(self, mock_send_request): #mock_send_request就是替换的mock对象
        mock_send_request.return_value='200'           #设置mock对象的返回值
        self.assertEqual(client.visit_baidu(), '200')  #测试client模块的visit_baidu方法
```

##### 进阶使用
通过mock对象来检查方法是否被调用？调用时传入的参数情况？

```
class TestClient(unittest.TestCase):
    def test_call_send_request_with_right_arguments(self):
        client.send_request = mock.Mock()                   #替换原方法send_request为一个mock对象
        client.visit_ustack()
        self.assertEqual(client.send_request.called, True)  #判断mock对象是否被调用
        call_args = client.send_request.call_args           #获取mock对象接收到的参数
        self.assertIsInstance(call_args[0][0], str)         #判断接收的参数是否是字符串类型
```

Mock.called：表示该mock对象是否被调用过

Mock.call_args：是一个tuple对象，存储了每次该mock对象被调用时传入的参数，每个tuple元素都存储该次调用时传入的(*args,**kwargs)

`((args1, kwargs1), (args2, kwargs2), ....(argsN, kwargsN))`

##### patch类参数
```
unittest.mock.patch(target, new=DEFAULT, spec=None, create=False, spec_set=None, autospec=None, new_callable=None, **kwargs)
```

- target->str, 格式为"package.module.ClassName", 指定要替换目标对象

详情参考官方文档：**[patch](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch "patch")**
()

##### mock类参数
```
class unittest.mock.Mock(spec=None, side_effect=None, return_value=DEFAULT, wraps=None, name=None, spec_set=None, unsafe=False, **kwargs)
```

- ` return_value `：当参数` side_effect `函数返回值是` DEFAULT `时，mock对象的调用会返回` return_value `的值

- ` side_effect `：传入一个可调用对象F（例如函数），当返回值不是DEFAULT时，mock对象的调用会返回该对象F的返回值，可用作动态设置mock类的返回值

详情参考官方文档：**[Mock对象](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock "Mock")**

### 测试原理
等价类划分：正常值，边界值，异常值

## python中的作用域
本地作用域（Local）→当前作用域被嵌入的本地作用域（Enclosing locals）→全局/模块作用域（Global）→内置作用域（Built-in）

## closure 闭包
1. 必须有一个内嵌函数
2. 内嵌函数必须引用外部函数的变量
3. 外部函数的返回值必须是内嵌函数

一般函数在运行完之后会回收执行所分配的内存空间，但是闭包的外部函数不会回收，因为返回的闭包引用了外部函数的属性（垃圾回收的引用计数）

因此在结束函数前应将用不上的变量删除以节省内存

## lambda 函数
匿名函数：`f = lambda x，y : x + y + 1`，前面表示传入的参数，后面表示函数逻辑

## 函数式编程
### 高级函数
***高级函数都是返回迭代体***
```
# filter(bool_func, iter)：调用一个布尔函数bool_func来迭代遍历每个iter中的元素,返回bool_func返回值为true的元素的迭代体。
a = [1,2,3,4,5,6,7]
b = filter(lambda x: x > 5, a)
print(list(b))      #返回[6,7]
```

```
#  map(func, iter)：map函数是对一个序列的每个项依次执行函数，并将执行结果组成未迭代体返回
a = map(lambda x:x*2,[1,2,3])
print(list(a))   #返回[2, 4, 6]
```

```
#  reduce(func, iter)：reduce函数是对一个迭代体的每个项迭代调用函数，下面是求3的阶乘：
reduce(lambda x,y:x*y,range(1,4))   #返回6
```
##类
### 类方法
```
class A(object):
    def foo(self,x):
        print "executing foo(%s,%s)"%(self,x)

    @classmethod    #类方法，可以通过A.class_foo(x)调用，注意因为是类方法，因此传入的是类对象本身cls而不是实例self，虽然变量名字没什么关系
    def class_foo(cls,x):
        print "executing class_foo(%s,%s)"%(cls,x)

    @staticmethod   #出于代码结构问题，将方法挂在A实例上，不需要传入实例self，通过a.static_foo()调用
    def static_foo(x):
        print "executing static_foo(%s)"%x

```

### 类变量
```
class A(object):  
    num = 0                  #类变量
    def __init__(self, name):  
        self.name = name     #实例变量

a = A('a')
b = A('b')
```

前面有说到，python一切皆对象，是对象就允许有属性

上面的num就属于类A的属性，不过所有A的实例a，b都能访问到（这个应该跟作用域链有关），但是只有类A本身具有修改权限

## 字符串格式化 % and .format
```
#!/usr/bin/python
sub1 = "python string!"
sub2 = "an arg"

a = "i am a %s" % sub1                  # "i am a python string!"
b = "i am a {0}".format(sub1)           # "i am a python string!"

c = "with %(kwarg)s!" % {'kwarg':sub2}  # "with an arg!"
d = "with {kwarg}!".format(kwarg=sub2)  # "with an arg!"
```

## 装饰器
不改变原有方法的代码，在原有方法的基础上添加功能，就像佩戴装饰品一样的实现称为**装饰器**

关键字@能在方法对象外部添加**装饰器**

例如执行某方法的代码段前先打印一个字符串：

```
import functools
def log(function):
    @functools.wraps(function)
    def wrapper(*arg, **kw):
        print(function.__name__)
        return function(*arg, **kw)
    return wrapper

@log        #给function方法佩戴log装饰器，之后调用function()就会先打印
def function()
```

中间的`@functools.wraps(function)`给返回的新方法对象绑定原有方法对象的名称属性

## 内置数据结构

## 一般序列操作
大多数序列类型包括可改变序列（例如列表）和不可改变序列（元组）都支持以下操作
**[一般序列操作官方中文文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#typesseq-common "一般序列操作官方中文文档")**

## 可变序列操作
可变序列可以实现的一些操作
**[可变序列操作官方中文文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#typesseq-mutable "可变序列操作官方中文文档")**

### list
**[list官方中文文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#list "list官方中文文档")**

元组实现了所有 ***一般*** 和 ***可变***序列的操作。

### tuple
`class tuple([iterable])`

**[tuple官方中文文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#tuple "tuple官方中文文档")**

元组实现了所有 ***一般*** 序列的操作。

### collections.namedtuple
为元组内部的数据进行命名，变成类似字典这样的数据结构，不过依然是不能修改的

```
collections.namedtuple(typename, field_names, verbose=False, rename=False)
typename        #元组名称
field_names     #元组中元素的名称

import collections
# 两种方法来给 namedtuple 定义方法名
#User = collections.namedtuple('User', ['name', 'age', 'id'])
User = collections.namedtuple('User', 'name age id')    #这种更加方便
user = User('tester', '22', '464643123')
print(user)         #User(name='tester', age='22', id='464643123')
print(user.name)    #'tester'
user.name = '1'     #抛出异常AttributeError: can't set attribute
```

**[namedtuple官方中文文档](https://docs.python.org/zh-cn/3/library/collections.html#collections.namedtuple "namedtuple官方中文文档")**

### collections.deque
双端队列，两端都可以添加(append)和弹出(pop)，大概开销都是 O(1) 复杂度

list因为是线形存储，插入insert(0, v)和删除pop(0)的效率都非常低，deque优化了相应了操作

`class collections.deque([iterable[, maxlen]])`

如果设定了maxlen，一旦限定长度的deque满了，当新项加入时，同样数量的项就从另一端弹出。

**[deque官方中文文档](https://docs.python.org/zh-cn/3/library/collections.html#collections.deque "deque官方中文文档")**

### collections.Counter
`class collections.Counter([iterable-or-mapping])`

是dick的一个子类，用于计数可哈希对象

它是一个集合，元素像字典键(key)一样存储，它们的计数存储为值（计数可以是任何整数值，包括0和负数）
```
# 创建Counter对象
c = Counter('gallahad')                 # print(c)->Counter({'a': 3, 'l': 2, 'g': 1, 'h': 1, 'd': 1})
c = Counter({'red': 4, 'blue': 2})      # a new counter from a mapping
c = Counter(cats=4, dogs=8)             # print(c)->Counter({'dogs': 8, 'cats': 4})
```

```
# elements方法，将元素组成成一个迭代体返回
c = Counter(a=4, b=2, c=0, d=-2)    #
c.elements()                        #返回结果是一个迭代体<itertools.chain object at 0x00000275ED4B4E10>
list(c.elements())                  #['a', 'a', 'a', 'a', 'b', 'b']
```

**[Counter官方中文文档](https://docs.python.org/zh-cn/3/library/collections.html#collections.Counter "Counter官方中文文档")**

### dict
```
class dict(**kwarg)
class dict(mapping, **kwarg)
class dict(iterable, **kwarg)
```

```
a = dict(one=1, two=2, three=3)
b = {'one': 1, 'two': 2, 'three': 3}
c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
d = dict([('two', 2), ('one', 1), ('three', 3)])
e = dict({'three': 3, 'one': 1, 'two': 2})
a == b == c == d == e
#注意每个创建方式传参的顺序不同，但这里仍然返回True，因此可以知道dict的相等测试是顺序不敏感的
```

**[dict官方中文文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#dict "dict官方中文文档")**

### collections.orderedDict

### set ＆ frozenset
`set `对象是由具有唯一性的` hashable `对象所组成的无序多项集，常用于成员检测、去重以及集合计算（例如交集、并集、差集与对称差集等等）

`set `类型是可变对象，其内容可以使用` add() `和` remove() `这样的方法来改变

`frozenset `类型是不可变对象并且为` hashable `，其内容在被创建后不能再改变

```
class set([iterable])
class frozenset([iterable])
```

**[set和frozenset官方中文文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#set-types-set-frozenset "set和frozenset官方中文文档")**

### sorted
```
sorted(iterable, *, key=None, reverse=False)       #注意那个*，两个可选参数都是关键字参数
key 指定带有单个参数的函数，用于从 iterable 的每个元素中提取用于比较的键 (例如 key=str.lower，会把每个元素先变为小写再进行比较)。 默认值为 None (直接比较元素)。
```

**[内置sorted官方中文文档](https://docs.python.org/zh-cn/3/library/functions.html#sorted "内置sorted官方中文文档")**

### bisect
```
bisect.bisect_left(a, x, lo=0, hi=len(a))
在 a 中找到 x 合适的插入点以维持有序。参数 lo 和 hi 可以被用于确定需要考虑的子集（也就是左侧集合a[lo:i]他们的值小于x，右侧集合a[i:hi]他们的值都大于x
如果 x 已经在 a 里存在，那么返回的插入点会在已存在元素之前（也就是左边）

bisect.bisect_right(a, x, lo=0, hi=len(a))
如果 x 已经在 a 里存在，那么返回的插入点会在已存在元素之后（也就是右边）

bisect.insort_left(a, x, lo=0, hi=len(a))
将 x 插入到一个有序序列 a 里，并维持其有序。如果 a 有序的话，这相当于 a.insert(bisect.bisect_left(a, x, lo, hi), x)。要注意搜索是 O(log n) 的，插入却是 O(n) 的。
```

**[数组二分查找算法官方中文文档](https://docs.python.org/zh-cn/3/library/bisect.html?highlight=bisect#module-bisect "数组二分查找算法官方中文文档")**


### heapq
heapq构建的是 小根堆

要创建一个堆，可以使用list来初始化为 [] ，或者你可以通过一个函数 heapify() ，来把一个list转换成堆
```
# 方法一
def heapsort(iterable):
    h = []
    for value in iterable:
        heapq.heappush(h, value)            #逐个添加到堆中
    return [heapq.heappop(h) for i in range(len(h))]
heapsort([1, 3, 5, 7, 9, 2, 4, 6, 8, 0])    #返回[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# 方法二
h = [1, 3, 5, 7, 9, 2, 4, 6, 8, 0]
heapq.heapify(h)                            #建立小根堆
[heapq.heappop(h) for i in range(len(h))]   #逐个弹出，返回[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
**[堆队列算法官方中文文档](https://docs.python.org/zh-cn/3/library/heapq.html?highlight=heapq#module-heapq "堆队列算法官方中文文档")**


### functools.lru_cache
一个为函数提供缓存功能的装饰器，缓存 maxsize 组传入参数，在下次以相同参数调用时直接返回上一次的结果。用以节约高开销或I/O函数的调用时间

由于使用了字典存储缓存，所以该函数的固定参数和关键字参数必须是可哈希的

不同模式的参数可能被视为不同从而产生多个缓存项，例如, f(a=1, b=2) 和 f(b=2, a=1) 因其参数顺序不同，可能会被缓存两次

如果` maxsize `设置为 None ，LRU功能将被禁用且缓存数量无上限, maxsize 设置为2的幂时可获得最佳性能

为了衡量缓存的有效性以便调整 maxsize 形参，被装饰的函数带有一个 cache_info() 函数

`cache_info() `: 返回一个具名元组，包含命中次数 hits，未命中次数 misses ，最大缓存数量 maxsize 和 当前缓存大小 currsize。在多线程环境中，命中数与未命中数是不完全准确的

`cache_clear() `: 该装饰器也提供了一个用于清理/使缓存失效的函数 cache_clear()

```
# 静态 Web 内容的 LRU 缓存示例:
@lru_cache(maxsize=32)
def get_pep(num):
    'Retrieve text of a Python Enhancement Proposal'
    resource = 'http://www.python.org/dev/peps/pep-%04d/' % num
    try:
        with urllib.request.urlopen(resource) as s:
            return s.read()
    except urllib.error.HTTPError:
        return 'Not Found'

>>> for n in 8, 290, 308, 320, 8, 218, 320, 279, 289, 320, 9991:
...     pep = get_pep(n)
...     print(n, len(pep))

>>> get_pep.cache_info()
CacheInfo(hits=3, misses=8, maxsize=32, currsize=8)
```

```
# 以下是使用缓存通过 动态规划 计算 斐波那契数列 的例子。
@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)

>>> [fib(n) for n in range(16)]
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610]

>>> fib.cache_info()
CacheInfo(hits=28, misses=16, maxsize=None, currsize=16)
```

**[lru_cache算法官方中文文档](https://docs.python.org/zh-cn/3/library/functools.html?highlight=lru_cache#functools.lru_cache "lru_cache算法官方中文文档")**