# <center>语言特点</center>
Python是一门动态的强类型的语言

## 动态/静态语言
静态指的是在编译期就已经提前确定对象类型，类似JAVA，C这类

动态指的是在运行期（执行到相应代码段）才确定对象类型，类似JS，Python这类

### duck type (鸭子类型)
`“当一直鸟走起来像鸭子，游泳起来像鸭子，叫起来也像，那么在走/游泳/叫声这三件事情上，这只鸟可以被称为鸭子”`

动态语言更关注接口而非类型
1. 关注点在对象的行为，而不是类型
2. Python的File，StringIO，Socket对象都支持read/write方法，因此他们都是（File-like Object）
3. Python的for迭代可以用于一切定义了__iter__方法的对象，而不关心类型

### monkey patch（猴子补丁）
猴子补丁就是在运行时进行替换，其实也就是另外写一段代码专门用作重写

例如在`gevent`库中需要修改内置的阻塞`socket`模块为非阻塞的`socket`模块：

```
from gevent import monkey
print(socket.socket)        #在这里依然调用的是内置的socket.socket
monkey.patch_socket()       #在这一步动态替换了内置的socket.socket
monkey.patch_select()       #在这一步动态替换了内置的socket.select
```

### introspection 自省
运行时获取对象类型的功能
type()，isinstance()，id()，dir(), hasattr(), is，inpect模块
```
type('11')                    #<class 'str'>，返回对象类型
isinstance('11', str)         #True，判断是否属于某类或它的子类
id('11')                      #2104490588120，返回对象内存地址
dir('11')                     #tribute__', '__getitem__', '__getnewargs__', '__gt__'......返回对象属性名
hasattr('11', '__len__')      #True, 判断对象是否有该属性
'11' is '11'                  #True，判断内存地址是否相等，str是不可变对象，因此是同一个对象
[1] is [1]                    #False，List是可变对象，因此不是同一个对象
```

## 强/弱类型语言
强类型不会发生隐式类型转换（JS，PHP是弱类型）

`1 + '1'`在python会报错，JS会返回`'11'`)

## The Zen of Python（Python之禅）
Tim Peters编写的关于Python的编程准则
终端执行import this
```
The Zen of Python, by Tim Peters

Beautiful is better than ugly.                                        #优美 优于 丑陋
Explicit is better than implicit.                                     #显示 优于 隐式
Simple is better than complex.                                        #简单 优于 复杂
Complex is better than complicated.                                   #复杂 优于 更复杂
Flat is better than nested.                                           #扁平 优于 嵌套  
Sparse is better than dense.                                          #稀疏 优于 密集
Readability counts.                                                   #可读性和重要
Special cases aren't special enough to break the rules.               #特殊情况也不能破坏规定
Although practicality beats purity.                                   #实用性 优于 简洁
Errors should never pass silently.                                    #异常应被抛出
Unless explicitly silenced.                                           #除非异常要明确要求隐藏
In the face of ambiguity, refuse the temptation to guess.             #摸棱两可时，拒绝瞎猜
There should be one-- and preferably only one --obvious way to do it. #尽量找一种，最好是唯一一种明显的解决方案
Although that way may not be obvious at first unless you're Dutch.    #虽然这并不容易，因为你不是 Python 之父
Now is better than never.                                             #做也许好过不做
Although never is often better than *right* now.                      #但不假思索就动手还不如不做
If the implementation is hard to explain, it's a bad idea.            #如果你无法向人描述你的方案，那肯定不是一个好方案
If the implementation is easy to explain, it may be a good idea.      #反之亦然
Namespaces are one honking great idea -- let's do more of those!      #命名空间是一种绝妙的理念，我们应当多加利用
```