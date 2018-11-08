---
title: Python Cookbook（类与对象）
date: 2017-11-23 23:29:25
tags: 类与对象
categories: Python Cookbook
---

# 类与对象

本章主要关注点的是和类定义有关的常见编程模型。包括让对象支持常见的 Python 特性、特殊方法的使用、类封装技术、继承、内存管理以及有用的设计模式。 
<!-- more -->

## 改变对象的字符串显示

问题：
改变对象实例的打印或显示输出，让它们更具可读性。

解决方案：
要改变一个实例的字符串表示，可重新定义它的 \_\_str\_\_() 和 \_\_repr\_\_() 方法。例如：
```python
class Pair:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __repr__(self):
        return 'Pair({0.x!r}, {0.y!r})'.format(self)
    def __str__(self):
        return '({0.x!r}, {0.y!r})'.format(self)
```
>如果实现了__repr__而没有定义__str__，那么对象将会表现出__str__ = __repr__

\_\_repr\_\_() 方法返回一个实例的代码表示形式，通常用来重新构造这个实例。内置的 repr() 函数返回这个字符串，跟我们使用交互式解释器显示的值是一样的。\_\_str\_\_() 方法将实例转换为一个字符串，使用 str() 或 print() 函数会输出这个字符串。比如：
```bash
>>> p = Pair(3, 4)
>>> p
Pair(3, 4)  # __repr__() output
>>> print(p)
(3, 4)      # __str__() output
>>>
```
我们在这里还演示了在格式化的时候怎样使用不同的字符串表现形式。特别来讲，!r 格式化代码指明输出使用 \_\_repr\_\_() 来代替默认的 \_\_str\_\_() 。你可以用前面的类来试着测试下: 
```bash
>>> p = Pair(3, 4)
>>> print('p is {0!r}'.format(p))
p is Pair(3, 4)
>>> print('p is {0}'.format(p))
p is (3, 4)
>>>
```
讨论：
自定义 \_\_repr\_\_() 和 \_\_str\_\_() 通常是很好的习惯，因为它能简化调试和实例输出。例如，如果仅仅只是打印输出或日志输出某个实例，那么程序员会看到实例更加详细与有用的信息。

\_\_repr\_\_() 生成的文本字符串标准做法是需要让 eval(repr(x)) == x 为真。如果实在不能这样子做，应该创建一个有用的文本表示，并用 < 和 > 括起来。比如：
```bash
>>> f = open('file.dat')
>>> f
<_io.TextIOWrapper name='file.dat' mode='r' encoding='UTF-8'>
>>>
```
如果 \_\_str\_\_() 没有被定义，那么就会使用 \_\_repr\_\_() 来代替输出。
上面的 format() 方法的使用看上去很有趣，格式化代码 {0.x} 对应的是第一个参数的 x 属性。因此，在下面的函数中，0 实际上指的就是 self 本身：
```python
def __repr__(self):
    return 'Pair({0.x!r}, {0.y!r})'.format(self)
```
作为这种实现的一个替代，你也可以使用 % 操作符， 就像下面这样：
```bash
def __repr__(self):
    return 'Pair(%r, %r)' % (self.x, self.y)
```

## 自定义字符串的格式化

问题：
你想通过 format() 函数和字符串方法使得一个对象能支持自定义的格式化。

解决方案：
为了自定义字符串的格式化，我们需要在类上面定义 \_\_format\_\_() 方法。例如：
```python
_formats = {
    'ymd': '{d.year}-{d.month}-{d.day}',
    'mdy': '{d.month}/{d.day}/{d.year}',
    'dmy': '{d.day}/{d.month}/{d.year}'
    }
    
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
        
    def __format__(self, code):
        if code = '':
            code = 'ymd'
        fmt = _formats[code] # code = 'ymd', fmt = {d.year}-{d.month}-{d.day}
        return fmt.format(d=self)
```
现在 Date 类的实例可以支持格式化操作了，如同下面这样：
```bash
>>> d = Date(2017, 11, 24)
>>> format(d)
'2017-11-24'
>>> format(d, 'mdy')
'11/24/2017'
>>> The date is {:ymd}'.format(d)
'The date is 2017-11-24'
>>> 'The date is {:mdy}'.format(d)
'The date is 11/24/2017'
```
讨论：
\_\_format\_\_() 方法给 Python 的字符串格式化功能提供了一个钩子。这里需要着重强调的是格式化代码的解析工作完全由类自己决定。因此，格式化代码可以是任何值。 例如，参考下面来自 datetime 模块中的代码: 
```bash
>>> from datetime import date
>>> d = date(2017, 11, 24)
>>> format(d)
'2017-11-24'
>>> format(d, '%A, %B, %d, %Y')
'Friday, November 24, 2017'
>>> 'The end is {:%d %b %y}. Goodbye'.format(d)
'The end is 24 Nov 2017. Goodbye'  # %b: Nov, %B: November
>>>
```
对于内置类型的格式化有一些标准的约定。可以参考 string 模块文档说明。 

## 让对象支持上下文管理协议

问题：
让对象支持上下文管理协议（ with 语句）。

解决方案：
为了让一个对象兼容 with 语句，你需要实现 \_\_enter\_\_() 和 \_\_exit\_\_() 方法。例如，考虑如下的一个类，它能为我们创建一个网络连接：
```python
from socket import socket, AF_INET, SOCK_STREAM

class LazyConnection:
    def __init__(self, address, family=AF_INET, type=SOCK_STREAM):
        self.address = address
        self.family = family
        self.type = type
        self.sock = None
        
    def __enter__(self):
        if self.sock is not None:
            raise RuntimeError('Already connected')
        self.sock = socket(self.family, self.type)
        self.sock.connect(self.address)
        return self.sock
    
    def __exit__(self, exc_ty, exc_val, tb):
        self.sock.close()
        self.sock = None
```
这个类的关键特点在于它表示了一个网络连接，但是初始化的时候并不会做任何事情（比如它并没有建立一个连接）。连接的建立和关闭是使用 with 语句自动完成的。例如：
```python
from functools import partial

conn = LazyConnection(('www.python.org', 80))
# Connection closed
with conn as s:
    # conn.__enter__() executes: connection open
    s.send(b'GET /index.html' HTTP/1.0\r\n')
    s.send(b'Host: www.python.org\r\n')
    s.send(b'\r\n')
    resp = b''.join(iter(partial(s.resv, 8192), b''))
    # conn.__exit__() executes: connection closed
```
讨论：
编写上下文管理器的主要原理是你的代码会放到 with 语句块中执行。当出现 with 语句的时候，对象的 \_\_enter\_\_() 方法被触发，它返回的值（如果有的话）会被赋值给 as 声明的变量。然后，with 语句块里面的代码开始执行。最后，\_\_exit\_\_() 方法被触发进行清理工作。
不管 with 代码块中发生什么，上面的控制流都会执行完，就算代码块中发生了异常也是一样的。事实上，\_\_exit\_\_() 方法的第三个参数包含了异常类型、异常值和追溯信息（如果有的话）。\_\_exit\_\_() 方法能自己决定怎样利用这个异常信息，或者忽略它 并返回一个 值。如果 \_\_exit\_\_() 返回 True ，那么异常会被清空，就好像什么都没发生一样， with 语句后面的程序继续在正常执行。 
还有一个细节问题就是 LazyConnection 类是否允许多个 with 语句来嵌套使用连接。很显然，上面的定义中一次只能允许一个 socket 连接，如果正在使用一个 socket 的时候又重复使用 with 语句，就会产生一个异常了。不过你可以像下面这样修改下上 面的实现来解决这个问题: 
```python
from socket import socket, AF_INET, SOCK_STREAM

class LazyConnection:
    def __init__(self, address, family=AF_INET, type=SOCK_STREAM):
        self.address = address
        self.family = family
        self.type = type
        self.connections = []
        
    def __enter__(self):
        sock = socket(self.family, self.type)
        sock.connect(self.address)
        self.connections.append(sock)
        return sock
        
    def __exit__(self, exc_ty, exc_val, tb):
        self.connecitons.pop().close()
        
# Example use

from functools import partial

conn = LazyConnection(('www.python.org', 80))
with conn as s1:
    pass
    with conn as s2:
        pass
        # s1 and s2 are independent sockets
```
在第二个版本中，LazyConnection 类可以被看做是某个连接工厂。在内部，一个列表被用来构造一个栈。每次 \_\_enter\_\_() 方法执行的时候，它复制创建一个新的连接 并将其加入到栈里面。 \_\_exit\_\_() 方法简单的从栈中弹出最后一个连接并关闭它。这 里稍微有点难理解，不过它能允许嵌套使用 with 语句创建多个连接，就如上面演示的那样。 

## 创建大量对象时节省内存方法

问题：
你的程序要创建大量（可能上百万）的对象，导致占用很大的内存。

解决方案：
对于主要是用来当成简单的数据结构的类而言，你可以通过给类添加 \_\_slots\_\_ 属性来极大的减少实例所占用的内存。比如：
```python
class Date:
    __slots__ = ['year', 'month', 'day']
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
```
当你定义 \_\_slots\_\_ 后，Python 就会为实例使用一种更加紧凑的内部表示。实例通过一个很小的固定大小的数组来构建，而不是为每个实例定义一个字典，这跟元组或列表很类似。在 \_\_slots__ 中列出的属性名在内部被映射到这个数组的指定小标上。

>使用 slots 一个不好的地方就是我们不能再给实例添加新的属性了，只能使用 \_\_slots\_\_ 中定义的那些属性名。

讨论：
使用 slots 后节省的内存会跟存储属性的数量和类型有关。不过，一般来讲，使用到的内存总量和将数据存储在一个元组中差不多。为了给你一个直观认识，假设你不使用 slots 直接存储一个 Date 实例，在 64 位的 Python 上面要占用 428 字节，而如果使用了 slots ，内存占用下降到 156 字节。如果程序中需要同时创建大量的日期实例，那么这个就能极大的减小内存使用量了。 

尽管 slots 看上去是一个很有用的特性，很多时候你还是得减少对它的使用冲动。 Pthon 的很多特性都依赖于普通的基于字典的实现。另外，定义了 slots 后的类不再支持一些普通类特性了，比如多继承。大多数情况下，你应该只在那些经常被使用到的用作数据结构的类上定义 slots （比如在程序中需要创建某个类的几百万个实例对象）。 

关于 \_\_slots\_\_ 的一个常见误区是它可以作为一个封装工具来防止用户给实例增加新的属性。尽管使用 slots 可以达到这样的目的，但是这个并不是它的初衷。\_\_slots\_\_ 更多的是用来作为一个内存优化工具。 

## 在类中封装属性名

问题：
你想封装类的实例上面的“私有”数据，但是 Python 语言并没有访问控制。

解决方案：
程序员不去依赖语言特性去封装数据，而是通过遵循一定的属性和方法命名规约来达到这个效果。第一个约定是任何以单下划线 开头的名字都应该是内部实现。比如: 
```python
class A:
    def __init__(self):
        self._internal = 0 # An internal attribute
        self.public = 1 # A public attribute
        
    def public_method(self):
        '''
        A public method
        '''
        pass
        
    def _internal_method(self):
       pass
```
Python 并不会真的阻止别人访问内部名称。但是如果你这么做肯定是不好的，可能会导致脆弱的代码。同时还要注意到，使用下划线开头的约定同样适用于模块名和模块级别函数。例如，如果你看到某个模块名以单下划线开头（比如 socket ），那它就是内部实现。类似的，模块级别函数比如 sys.\_getframe() 在使用的时候就得加倍小心了。
你还可能会遇到在类定义中使用两个下划线（\_\_）开头的命名。
比如：
```python
class B:
    def __init__(self):
        self.__private = 0
        
    def __private_method(self):
        pass
    
    def public_method(self):
        pass
        self.__private_method()
```
使用双下划线开始会导致访问名称变成其他形式。比如，在前面的类 B 中，私有属性会被分别重命名为 \_B\_\_private 和 \_B\_\_private\_method 。这时候你可能会问这样重命名的目的是什么，答案就是继承——这种属性通过继承是无法被覆盖的。比如:
```python
class C(B):
    def __init__(self):
        super().__init__()
        self.__private = 1 # Does not override B.__private
        
    # Does not override B.__private_method()
    def __private_method(self):
        pass
```
这里，私有名称 \_\_private 和 \_\_private\_mehtod 被重命名为 \_C\_\_private 和 \_C\_\_private\_method , 这个跟父类 B 中的名称是完全不同的。 

讨论：
上面提到有两种不同的编码约定（单下划线和双下划线）来命名私有属性，到底哪种方式好呢？大多数而言，你应该让你的非公共名称以单下划线开头。但是，如果你清楚你的代码会涉及到子类，并且有些内部属性应该在子类中隐藏起来，那么才考虑使用双下划线方案。
>还有一点要注意的是，有时候你定义的一个变量和某个保留关键字冲突，这时候可以使用单下划线作为后缀，例如：

```python
lambda_ = 2.0 # Trailing to avoid clash with lambda keyword
```
这里我们并不使用单下划线前缀的原因是它避免误解它的使用初衷（如使用单下划线前缀的目的是为了防止命名冲突而不是指明这个属性是私有的）。通过使用单下划线后缀可以解决这个问题。 

## 创建可管理的属性

问题：
你想给某个实例 attribute 增加除访问与修改之外的其他处理逻辑，比如类型检查或合法性验证。

解决方案：
自定义某个属性的一种简单方法是将它定义为一个 property 。例如，下面的代码定义了一个 property，增加对一个属性简单的类型检查：
```python
class Person:
    def __init__(self, first_name):
        self.first_name = first_name
        
    # Getter function
    @property
    def first_name(self):
        reutrn self._first_name
        
    # Setter function
    @first_name.setter
    def first_name(self, value):
        if not isinstance(value, str):
            raise TypeError('Expected a string')
        self._first_name = value
        
    # Deleter function(optional)
    @first_name.deleter
    def first_name(self):
        raise AttributeError("Can't delete attribute")
```
>上述代码中有三个相关联的方法，这三个方法的名字都必须一样。 

第一个方法是一个 getter 函数，它使得 first_name 成为一个属性。其他两个方法给 first_name 属性添加了 setter 和 deleter 函数。需要强调的是只有在 fist_name 属性被创建后， 后面的两个装饰器 @first_name.setter 和 @first_name.deleter 才能被定义。

property 的一个关键特征是它看上去跟普通的 attribute 没什么两样，但是访问它的时候会自动触发 getter、setter 和 deleter 方法。例如：
```bash
>>> a = Person('Guido')
>>> a.first_name # Calls the getter
'Guido'
>>> a.first_name = 42 # Calls the setter
Traceback(most recent call last):
    raise TypeError('Expected a string')
TypeError: Expected a string
>>> del a.first_name
Traceback(most recent call last):
AttributeError: can't delete attribute
>>>
```
在实现一个 property 的时候，底层数据（如果有的话）仍然需要存储在某个地方。 因此，在 get 和 set 方法中，你会看到对 \_first\_name 属性的操作，这也是实际数据保存的地方。另外，你可能还会问为什么 \_\_init\_\_() 方法中设置了 self.first_name 而不是 self._first_name。在这个例子中，我们创建一个 property 的目的就是在设置 attribute 的时候进行检查。因此，你可能想在初始化的时候也进行这种类型检查。通过设置 self.first_name，自动调用 setter 方法，这个方法里面会进行参数的检查， 否则就是直接访问 self._first_name 了。 
还能在已存的 get 和 set 方法基础上定义 property。例如：
```python
class Person:
    def __init__(self, first_name):
        self.set_first_name(first_name)
        
    # Getter function
    def get_first_name(self):
        return self._first_name
        
    # Setter function
    def set_first_name(self, value):
        if not isinstance(value, str):
            raise TypeError('Expected a string')
        self._first_name = value
        
    # Deleter function(optional)
    def del_first_name(self):
        raise AttributeError("Can't delete attribute")
    
    # Make a property from existing get/set methods
    name = property(get_first_name, set_first_name, del_first_name)
```
讨论：
一个 property 属性其实就是一系列相关绑定方法的集合。如果你去查看拥有 property 的类，就会发现 property 本身的 fget、fset 和 fdel 属性就是类里面的普通方法。比如：
```bash
>>> Person.first_name.fget
<function Person.first_name at 0x>
>>> Person.first_name.fset
<function Person.first_name at 0x>
>>> Person.first_name.fdel
<function Person.first_name at 0x>
>>>
```
通常来讲，你不会直接取调用 fget 或者 fset，它们会在访问 property 的时候自动被触发。 
只有当你确实需要对 attribute 执行其他额外的操作的时候才应该使用到 property。有时候一些从其他编程语言（比如 Java ）过来的程序员总认为所有访问都应该通过 getter 和 setter ，所以他们认为代码应该像下面这样写:
```python
class Person:
    def __init__(self, first_name):
        self.first_name = first_name
        
    @property
    def first_name(self):
        return self._first_name
        
    @property.setter
    def first_name(self, value):
        self._first_name = value
```
不要写这种没有做任何其他额外操作的 property。首先，它会让你的代码变得很臃肿，并且还会迷惑阅读者。其次，它还会让你的程序运行起来变慢很多。最后，这样的设计并没有带来任何的好处。特别是当你以后想给普通 attribute 访问添加额外的处理逻辑的时候，你可以将它变成一个 property 而无需改变原来的代码。因为访问 attribute 的代码还是保持原样。
Properties 还是一种定义动态计算 attribute 的方法。这种类型的 attribute 并不会被实际的存储，而是在需要的时候计算出来。比如: 
```python
import math
class Circle:
    def __init__(self, radius):
        self.radius = radius
        
    @property
    def area(self):
        return math.pi * self.radius ** 2
        
    @property
    def diameter(self):
        return self.radius * 2
    
    @property
    def perimeter(self):
        return 2 * math.pi * self.radius
```
在这里，我们通过使用 properties ，将所有的访问接口形式统一起来，对半径、直径、周长和面积的访问都是通过属性访问，就跟访问简单的 properties 是一样的。如果不这样做的话，那么就要在代码中混合使用简单属性访问和方法调用。
下面是使用实例：
```bash
>>> c = Circle(4.0)
>>> c.radius
4.0
>>> c.area # Notice lack of ()
50.265...
>>> c.perimeter # Notice lack of ()
25.132...
>>>
```
尽管 properties 可以实现优雅的编程接口，但有些时候你还是会想直接使用 getter 和 setter 函数。例如:
```bash
>>> p = Person('Guido')
>>> p.get_first_name()
'Guido'
>>> p.set_first_name('Larry')
>>> 
```
>这种情况的出现通常是因为 Python 代码被集成到一个大型基础平台架构或程序中。例如，有可能是一个 Python 类准备加入到一个基于远程过程调用的大型分布式系统中。这种情况下，直接使用 get/set 方法（普通方法调用）而不是 property 或许会更容易兼容。 

最后一点，不要像下面这样写有大量重复代码的 property 定义: 
```python
class Person:
    def __init__(self, first_name, last_name):
        self.first_name = first_name
        self.last_name = last_name
        
    @property
    def first_name(self):
        return self._first_name
        
    @first_name.setter
    def first_name(self, value):
        if not isinstance(value, str):
            raise TypeError('Expected a string')
        self._first_name = value
        
    # Repeated property code, but for a different name(bad!)
    @property
    def last_name(self):
        return self._last_name
        
    @last_name.setter
    def last_name(self, value):
        if not isinstance(value, str):
            raise TypeError('Expected a string')
        self._last_name = value
```
重复代码会导致臃肿、易出错和丑陋的程序。好消息是，通过使用装饰器或闭包，有很多种更好的方法来完成同样的事情。可以参考 1.9 和 元编程（1.21）的内容。

## 调用父类方法

问题：
你想在子类中调用父类的某个已经被覆盖的方法。

解决方案：
为了调用父类（超类）的一个方法，可以使用 super() 函数，比如：

```python
# Python 3 里 object 已经做为所有东西的基类
class A:
    def spam(self):
        print('A.spam')
        
class B(A):
    def spam(self):
        print('B.spam')
        super().spam()    # Call parent spam()
        
# Python 2
class A(object):
    def spam(self):
        print('A.spam')
        
Class B(A):
    def spam(self):
        print('B.spam')
        super(B, self).spam()
        
# 采用新式类，要求最顶层的父类一定要继承于object，这样就可以利用 super() 函数来调用父类的 init() 等函数.// Python 2 
```
[更多关于 Python 类.](http://www.runoob.com/python/python-object.html)

super() 函数的一个常见用法是在 \_\_init\_\_() 方法中确保父类被正确的初始化了：
```python
class A:
    def __init__(self):
        self.x = 0
        
class B(A):
    def __init__(self):
        super().__init__()
        self.y = 1
```
super() 的另外一个常见用法出现在覆盖 Python 特殊方法的代码中，比如：
```python
class Proxy:
    def __init__(self, obj):
        self._obj = obj
        
    # Delegate attribute lookup to internal obj
    def __getattr__(self, name):
        return getattr(self._obj, name)
        
    # Delegate attribute assignment
    def __setattr__(self, name, value):
        if name.startswith('_'):
            super().__setattr__(name, value) # Call original __setattr__
        else:
            setattr(self._obj, name, value)
```
在上面代码中，\_\_setattr\_\_() 的实现包含一个名字检查。如果某个属性名以下划线 (_) 开头，就通过 super() 调用原始的 \_\_setattr\_\_() ，否则的话就委派给内部的代理对象 self.\_obj 去处理。就算没有显式的指明某个类的父类，super() po仍然可以有效的工作。 

讨论：
有时候你会看到像下面这样直接调用父类的一个方法：
```python
class Base:
    def __init__(self):
        print('Base.__init__')
        
class A(Base):
    def __init__(self):
        Base.__init__(self)
        print('A.__init__')
```
尽管对于大部分代码而言这么做没什么问题，但是在更复杂的涉及到多继承的代码中就有可能导致很奇怪的问题发生。比如，考虑如下的情况:
```python
class Base:
    def __init__(self):
        print('Base.__init__')
        
class A(Base):
    def __init__(self):
        Base.__init__(self)
        print('A.__init__')
        
class B(Base):
    def __init__(self):
        Base.__init__(self)
        print('B.__init__')
        
class C(A, B):
    def __init__(self):
        A.__init__(self)
        B.__init__(self)
        print('C.__init__')
```
如果你运行这段代码就会发现 Base.\_\_init\_\_() 被调用两次，如下所示: 
```bash
>>> c = C()
Base.__init__
A.__init__
Base.__init__
B.__init__
C.__init__
```
在代码中使用 super() 就可以调用一次 Base.\_\_init\_\_() ：
```python
class Base:
    def __init__(self):
        print('Base.__init__')
        
class A(Base):
    def __init__(self):
        super().__init__()
        print('A.__init__')

class B(Base):
    def __init__(self):
        super().__init__()
        print('B.__init__')
        
class C(A, B):
    def __init__(self):
        super().__init__()    # only one call to super() here
        print('C.__init__')
```
运行这个新版本后，你会发现每个 \_\_init\_\_() 方法只会被调用一次了:
```bash
>>> c = C()
Base.__init__
B.__init__
A.__init__
C.__init__
```
>Python 是如何实现继承的。对于你定义的每一个类，Python 会计算出一个所谓的方法解析顺序（MRO）列表。这个 MRO 列表就是一个简单的所有基类的线性顺序表。例如:

```bash
>>> c.__mro__
(<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>,
<class '__main__.Base'>, <class '__main__.Object'>)
```
为了实现继承，Python 会在 MRO 列表上从左到右开始查找基类，直到找到第一个匹配这个属性的类为止。
而这个 MRO 列表的构造是通过一个 C3 线性化算法来实现的。它实际上就是合并所有父类的 MRO 列表并遵循如下三条准则:

* 子类会先于父类被检查
* 多个父类会根据它们在列表中的顺序被检查
* 如果对下一个类存在两个合法的选择，选择第一个父类

你所要知道的就是 MRO 列表中的类顺序会让你定义的任意类层级关系变得有意义。 

当你使用 super() 函数时，Python 会在 MRO 列表上继续搜索下一个类。只要每个重定义的方法统一使用 super() 并只调用它一次，那么控制流最终会遍历完整个  MRO 表，每个方法也只会被调用一次。这也是为什么在第二个例子中你不会调用两次 Base.\_\_init\_\_() 的原因。 

super() 并不一定去查找某个类在 MRO 中下一个直接父类，你甚至可以在一个没有直接父类的类中使用它。例如，考虑如下这个类: 
```python
class A:
    def spam(self):
        print('A.spam')
        super().spam()
```
如果你试着直接使用这个类就会出错：
```bash
>>> a = A()
>>> a.spam()
A.spam
Traceback:
AttributeError: 'super' obejct has no attribute 'spam'
```
但是，如果你使用多继承的话：
```bash
>>> class B:
...     def spam(self):
...         print('B.spam')
>>> class C(A, B):
...     pass
...
>>> c = C()
>>> c.spam()
A.spam()
B.spam()
>>>
```
你可以看到在类 A 中使用 super().spam() 实际上调用的是跟类 A 毫无关系的类 B 中的 spam() 方法。这个用类 C 的 MRO 列表就可以完全解释清楚了: 
```bash
>>> C.__mro__
(<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>,
<class '__main__.Object'>)
```
详见 P246

## 子类中扩展 property

问题：
在子类中，你想要扩展定义在父类中的 property 的功能。

解决方案：
定义一个 property ：
```python
class Person:
    def __init__(self, name):
        self.name = name
        
    # Getter function
    @property
    def name(self):
        return self._name
        
    # setter function
    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError('Expected a string')
        self._name = value
        
    # Deleter function
    @name.deleter
    def name(self):
        raise AttributeError("Can't delete attribute")
```
示例类（它继承自 Person 并扩展了 name 属性的功能）：
```python
class SubPerson(Person):
    @property
    def name(self):
        print('Getting name')
        return super().name
        
    @name.setter
    def name(self, value):
        print('Setting name to', value)
        super(SubPerson, SubPerson).name.__set__(self, value)
        
    @name.deleter
    def name(self):
        print('Deleting name')
        super(SubPerson, SubPerson).name.__delete__(self)
```
使用这个新类：
```bash
>>> s = SubPerson('Guido')
setting name to Guido
>>> s.name
Getting name
'Guido'
>>> s.name = 'Larry'
setting name to Larry
>>> s.name = 42
Traceback:
	raise TypeError('Expected a string')
TypeError: Expected a string
>>>
```
如果你仅仅只想扩展 property 的某一个方法，可以像下面这样写：
```python
class SubPerson(Person):
     @Person.name.getter
     def name(self):
         print('Getting name')
         return super().name
```
or 修改 setter 方法：
```python
class SubPerson(Person):
    @Person.name.setter
    def name(self, value):
        print('Setting name to ', value)
        super(SubPerson, SubPerson).name.__set__(self, value)
```
讨论：
在子类中扩展一个 property 可能会引起很多不易察觉的问题，因为一个 property 其实是 getter 、setter 和 deleter 方法的集合，而不是单个方法。因此，但你扩展一个 property 的时候，你需要先确定你是否要重新定义所有的方法还是说只修改其中某一个。 

在第一个例子中，所有的 property 方法都被重新定义。在每一个方法中，使用了 super() 来调用父类的实现。在 setter 函数中使用 super(SubPerson, SubPerson).name.\_\_set\_\_(self, value) 的语句是没有错的。为了委托给之前定义的 setter 方法，需要将控制权传递给之前定义的 name 属性的 \_\_set\_\_() 方法。 不过，获取这个方法的唯一途径是使用类变量而不是实例变量来访问它。这也是为什么我们要使用 super(SubPerson, SubPerson) 的原因。

如果你只想重定义其中一个方法，那只使用 @property 本身是不够的。比如：
```python
class SubPerson(Person):
    @property    # Doesn't work
    def name(self):
        print('Getting name')
        return super().name
```
如果你试着运行会发现 setter 函数整个消失了: 
```bash
>>> s = SubPerson('Guido')
Traceback:
	self.name = name
AttributeError: can't set a attribute
```
修改代码：
```python
class SubPerson(Person):
    @Person.name.getter
    def name(self):
        print('Getting name')
        return super().name
```
这么写后，property 之前已经定义过的方法会被复制过来，而 getter 函数被替换。 然后它就能按照期望的工作了:
```bash
>>> s = SubPerson('Guido')
>>> s.name
Getting name
'Guido'
>>> s.name = 'A_name'
Getting name
'A_name'
>>> s.name = 42
Traceback:
	raise TypeError('Expected a string')
TypeError: Expected a string
>>>
```
在这个特别的解决方案中，我们没办法使用更加通用的方式去替换硬编码的 Person 类名。
>如果你不知道到底是哪个基类定义了 ，那你只能通过重新定义所有 property 并使用 super() 来将控制权传递给前面的实现。 

## 创建新的类或实例属性

问题：
你想创建一个新的拥有一些额外功能的实例属性类型，比如类型检查。

解决方案：
如果想创建一个全新的实例属性，可以通过一个描述器类的形式来定义它的功能。例子：
```python
# Descriptor attribute for an integer type-checked attribute
class Integer:
    def __init__(self, name):
        self.name = name
        
    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            return instance.__dict__[self.name]    # p 为实例，x 为 self.name (dict 的 key )所以为 p.x
    def __set__(self, instance, value):
        if not isinstance(value, int):
            raise TypeError('Expected an int')
        instance.__dict__[self.name] = value
    
    def __delete__(self, instance):
        del instance.__dict__[self.name]
```
一个描述器就是一个实现了三个核心的属性访问操作( get, set, delete )。这些方法接受一个实例作为输入，之后相应的操作实例底层的字典。

>为了使用一个描述器，需将这个描述器的实例作为类属性放到一个类的定义中。

例如：
```python
class Point:
    x = Integer('x')	# self.name = 'x'
    y = Integer('y')	# self.name = 'y'
    
    def __init__(self, x, y):
        self.x = x		# Integer('x') = x
        self.y = y		# Integer('y') = y
```
当你这样做后，所有对描述器属性(比如 x 或 y 的访问会被 \_\_get\_\_()、\_\_set\_\_() 和 \_\_delete\_\_() 方法捕获到。例如:
```bash
>>> p = Point(2, 3)
>>> p.x # Calls Point.x.__get__(p.Point)
2
>>> p.y = 5 # Calls Point.y.__set__(p, 5)
>>> p.x = 2.3 # Calls point.x.__set__(p, 2.3)
Traceback:
raise TypeError('Expected an int')
TypeError: Expected an int
>>>
```
作为输入，描述器的每一个方法会接受一个操作实例。为了实现请求操作，会相应的操作实例底层的字典( \_\_dict\_\_ 属性) 。描述器的 self.name 属性存储了在实例字典中被实际使用到的 key 。 

讨论：
描述器可实现大部分 Python 类特性中的底层魔法，包括 @classmethod 、@staticmethod 、@property ，甚至是 \_\_slots\_\_特性。
通过定义一个描述器，你可以在底层捕获核心的实例操作（get, set, delete），并且可完全自定义它们的行为。

描述器的一个比较困惑的地方是它只能在类级别被定义，而不能为每个实例单独定义。因此：
```python
# Does not work
class Point:
    def __init__(self, x, y):
        self.x = Integer('x')
        self.y = Integer('y')
        self.x = x
        self.y = y
```
同时，\_\_get\_\_() 方法实现起来比看上去要复杂得多：
```python
# Descriptor attribute  for an integer type-checked attribute
class Integer:
    
    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            return instance.__dict__[self.name]
```
\_\_get\_\_() 看上去有点复杂的原因归结于实例变量和类变量的不同。如果一个描述器被当做一个类变量来访问，那么 instance 参数被设置成 None。这种情况下，标准做法就是简单的返回这个描述器本身即可（尽管还可以添加其他的自定义操作)。
例如:
```bash
>>> p = Point(2, 3)
>>> p.x		# Calls Point.x.__get__(p, Point)
2
>>> Point.x		# Calls Point.x.__get__(None, Point)
<__main__.Integer object at ...>
>>>
```
## 使用延迟计算属性

问题：
将一个只读属性定义成一个 property ，并且只在访问的时候才会计算结果。但是一旦被访问后，你希望结果值被缓存起来，不用每次都去计算。
解决方案：
定义一个延迟属性的一种高效方法是通过使用一个描述器类，如下所示:
```python
class lazyproperty:
    def __init__(self, func):
        self.func = func
        
    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            value = self.func(instance)  # value = math.pi * self.radius ** 2
            setattr(instance, self.func.__name__, value)   # 要传入 instance ?
            return value
```
在类中使用它：
```python
import math

class Circle:
    def __init__(self, radius):
        self.radius = radius
        
    @lazyproperty
    def area(self):
        print('Computing area')
        return math.pi * self.radius ** 2
        
    @lazyproperty
    def perimeter(self):
        print('Computing perimeter')
        return 2 * math.pi * self.radius
```
在交互环境演示：
```bash
>>> c = Circle(4.0)
>>> c.radius
4.0
>>> c.area
Computing area
50.265...
>>> c.area
50.265...
>>> c.perimeter
Computing perimeter
25.132...
>>> c.perimeter
25.132...
>>>
```
讨论：
很多时候，构造一个延迟计算属性的主要目的是为了提升性能。例如，你可以避免计算这些属性值，除非你真的需要它们。这里演示的方案就是用来实现这样的效果的，只不过它是通过以非常高效的方式使用描述器的一个精妙特性来达到这种效果的。 
当一个描述器被放入一个类的定义时，每次访问属性时它的\_\_get\_\_()、\_\_set\_\_()和 \_\_delete\_\_() 方法就会被触发。不过，如果一个描述器仅仅只定义了一个 \_\_get\_\_() 方法的话，它比通常的具有更弱的绑定。
特别地，只有当被访问属性不在实例底层的字典中时 \_\_get\_\_() 方法才会被触发。(what does it mean？)

lazyproperty 类利用这一点，使用 \_\_get\_\_() 方法在实例中存储计算出来的值，这个实例使用相同的名字作为它的 property 。这样一来，结果值被存储在实例字典中并且以后就不需要再去计算这个 property 了。可以尝试更深入的例子来观察结果: 
```python
>>> c = Circle(4.0)
>>> # Get instance vairable
>>> vars(c)
{'radius': 4.0}

>>> # Compute area and observe variable afterward
>>> c.area
Computing area
50.265
>>> vars(c)
{'area': 50.265, 'radius': 4.0}

>>> # Notice access doesn't invoke property anymore
>>> c.area
50.265

>>> # Delete the variable and see property trigger again
>>> del c.area
>>> vars(c)
{'radius': 4.0}
>>> c.area
Computing area
50.265
>>>
```
这种方案有一个小缺陷就是计算出的值被创建后是可以被修改的。例如:
```python
>>> c.area
Computing area
50.265
>>> c.area = 25
>>> c.area
25
>>>
```
如果你担心这个问题，那么可以使用一种稍微没那么高效的实现，就像下面这样:
```python
def lazyproperty(func):
    name = '_lazy' + func.__name__
    @property    # 在 lazyproperty 装饰器里再用 property 装饰
    def lazy(self):
        if hasattr(self, name):
            return getattr(self, name)
        else:
            value = func(self)
            setattr(self, name, value)
            return value
    return lazy
```
如果你使用这个版本，就会发现现在修改操作已经不被允许了:
```python
>>> c = Circle(4.0)
>>> c.area
Computing area
50.265
>>> c.area
50.265
>>> c.area = 25
...
AttributeError: Can't set attribute
>>>
```
然而，这种方案有一个缺点就是所有 get 操作都必须被定向到属性的 getter 函数上去。这个跟之前简单的在实例字典中查找值的方案相比效率要低一点。

## 简化数据结构的初始化

问题：
你写了很多仅仅用作数据结构的类，不想写太多烦人的 \_\_init\_\_() 函数

解决方案：
可以在一个基类中写一个公用的 \_\_init\_\_() 函数：
```python
import math

class Structure1:
    # Class variable that specifies expected field
    _fields = []
    
    def __init__(self, *args):
        if len(args) != len(self._fields):
            raise TyepError('Expected {} arguments'.format(len(self._fields)))
        # Set the arguments
        for name, value in zip(self._fields, args):
            setter(self, name, value)
```
然后使你的类继承自这个基类：
```python
# Example class definitions
class Stock(Structure1):
    _fields = ['name', 'shares', 'price']
    
class Point(Structure1):
    _fields = ['x', 'y']
    
class Circle(Structure1):
    _fields = ['radius']
    
    def area(self):
        return math.pi * self.radius ** 2
```
使用这些类的示例：
```bash
>>> s = Stock('ACME', 50, 91.1)
>>> p = Point(2, 3)
>>> c = Circle(4.5)
>>> s2 = Stock('ACME', 50)
Traceback:
TypeError: Expected 3 arguments
```
如果还想支持关键字参数，可以将关键字参数设置为实例属性：
```python
class Structure2:
    _fields = []
    
    def __init__(self, *args, **kwargs):
        if len(args) > len(self._fields):
            raise TypeError('Expected {} arguments.format(len(self._fields))')
        # Set all of the positional arguments
        for name, value in zip(self._fields, args):
            setattr(self, name, value)
        # Set the remaining keyword arguments
        for name in self._fields[len(args):]:    # [len(args):] 取位置参数后面的关键字参数
            setattr(self, name, kwargs.pop(name))
        # Check for any remaining unknown arguments
        if kwargs:
            raise TypeError('Invalid arguments()s: {}'.format(','.join(kwargs)))
            
# Example use
if __name__ == '__main__':
    class Stock(Structure2):
        _fields = ['name', 'shares', 'price']
    
    s1 = Stock('ACME', 50, 91.1)
    s2 = Stock('ACME', 50, price=91.1)
    s3 = Stock('ACME', shares=50, price=91.1)
    # s3 = Stock('ACME', shares=50, price=91.1, aa=1)
```
你还能将不在 \_fields 中的名称加入到属性中去：
```python
class Structure3:
    # Class variable that specifies expected fields
    _fields = []
    
    def __init__(self, *args, **kwargs):
        if len(args) != len(self._fields):
            raise TypeError('Expected {} arguments'.format(len(self._fields)))
        # Set the arguments
        for name, value in zip(self._fields, args):
            setattr(self, name, value)
        
        # Set additional arguments (if any)
        # Python 3    type(extra_args) = <class 'set'>
        extra_args = kwargs.keys() - self._fields  
        for name in extra_args:
            setattr(self, name, kwargs.pop(name))    # value = kwargs.pop(name)
        
        if kwargs:
            raise TypeError('Duplicate values for {}'.format(','.join(kwargs)))
            
# Example use
if __name__ == '__main__':
    class Stock(Structure3):
        _fields = ['name', 'shares', 'price']
         
    s1 = Stock('ACME', 50, 91.1)
    s2 = Stock('ACME', 50, 91.1, date='8/2/2012')
```

## 定义接口或者抽象基类

问题：
你想定义一个接口或抽象类，并且通过执行类型检查来确保子类实现了某些特定的方法

解决方案：
使用 abc 模块可以很轻松的定义抽象基类：
```python
from abc import ABCMeta, abstractmethod

class IStream(metaclass=ABCMeta):    # Python 3
    @abstractmethod
    def read(self, maxbytes=-1):
        pass
    @abstractmethod
    def write(self, data):
        pass
```
抽象类的一个特点是它不能直接被实例化，比如你想像下面这样做是不行的：
```python
a = IStream()    # TypeError: Can't instantiate abstract class
						# IStream with abstract methods read, write
```
抽象类的目的就是让别的类继承它并实现特定的抽象方法：
```python
class SocketStream(IStream):
    def read(self, maxbytes=-1):
        pass
    def write(self, data):
        pass
```
抽象基类的一个主要用途是在代码中检查某些类是否为特定类型，实现了特定接口：
```python
def serialize(obj, stream):
    if not isinstance(stram, IStream):
        raise TypeError('Expected a IStream')
    pass
```
除了继承这种方式外，还可以通过注册方式来让某个类实现抽象基类：
```python
import io

# Register the built-in I/O classes as supporting our interface
IStream.register(io.IOBase)

# Open a normal file and type check
f = open('foo.txt')
isinstance(f, IStream) # Returns True
```
@abstractmethod 还能注解静态方法、类方法和 properties 。你只需要保证这个注解紧靠在函数定义前即可：
```python
class A(metaclass=ABCMeta):
    @property
    @abstractmethod
    def name(self):
        pass
        
    @name.setter
    @abstractmethod
    def name(self, value):
       pass
       
    @classmethod
    @abstractmethod
    def method(cls):
        pass
    
    @staticmethod
    @abstractmethod
    def method2():
        pass
```
讨论：
标准库中有很多用到抽象基类的地方。collections 模块定义了很多跟容器和迭代器（序列、映射、集合等）有关的抽象基类。numbers 库定义了跟数字对象（整数、浮点数、有理数等）有关的基类。io 库定义了很多跟 I/O 操作相关的基类。

你可以使用预定一的抽象类来执行更通用的类型检查，例如：
```python
import collections

# Check if x is a sequence
if isinstance(x, collections.Sequence):
...

# Check if x is iterable
if isinstance(x, collections.Iterable):
...

# Check if x has a size
if isinstace(x, collections.Sized):
...

# Check if x is a mapping
if isinstace(x, collections.Mapping):
```
尽管 ABCs 可以让我们很方便的做类型检查，但是我们在代码中最好不要过多的使用它。因为 Python 的本质是一门动态编程语言，其目的就是给你更多灵活性，强制类型检查或让你代码变得更复杂，这样做无异于舍本求末。

## 实现数据模型的类型约束

问题：
定义某些在属性赋值上面有限制的数据结构。

解决方案：
在这个问题中，你需要在对某些实例属性赋值时进行检查。所以你要自定义属性赋值函数，这种情况下最好使用描述器。

下面的代码使用描述器实现了一个系统类型和赋值验证框架：
```python
# Base class.Uses a descriptor to set a value
class Descriptor:
    def __init__(self, name=None, **opts):
        self.name = name
        for key, value in opts.items():
            setattr(self, key, value)
            
    def __set__(self, instance, value):
        instance.__dict__[self.name] = value
        
# Descriptor for enforcing types
class Typed(Descriptor):
    expected_type = type(None)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.expected_type):
            raise TypeError('expected ' + str(self.expected_type))
        super().__set__(instance, value)
        
# Descriptor for enforcing values
class Unsigned(Descriptor):
    def __set__(self, instance, value):
        if value < 0:
            raise ValueError('Expected >= 0')
        super().__set__(instance, value)
        
class MaxSized(Descriptor):
    def __init__(self, name=None, **opts):
        if 'size' not in opts:
            raise TypeError('missing size option')
        super().__init__(name, **opts)
        
    def __set__(self, instance, value):
        if len(value) >= self.size:
            raise ValueError('size must be < ' + str(self.size))
        super().__set__(instance, value)
```
这些类就是你要创建的数据模型或类型系统的基础构建模块。下面就是我们实际定义的各种不同的数据类型:
```python
class Integer(Typed):
    expected_type = int
    
class UnsignedInteger(Integer, Unsigned):
    pass
    
class Float(Typed):
    expected_type = float
    
class UnsignedFloat(Float, Unsigned):
    pass
    
class String(Typed):
    expected_type = str
    
class SizedString(String, MaxSized):
    pass
```
然后使用这些自定义数据类型，我们定义一个类：
```python
class Stock:
    # Specify constraints
    name = SizedString('name', size=8)
    shares = UnsignedInteger('shares')
    price = UnsignedFloat('price')
    
    def __init__(self, name, shares, price):
        self.name = name
        self.shares = shares
        self.price = price
```
测试这个类的属性赋值约束，可发现对某些属性的赋值违法了约束是不合法的：
```bash
>>> s.name
'ACME'
>>> s.shares = 75
>>> s.shares = -10
Traceback:
raise ValueError('Expected >= 0')
ValueError: Expected >= 0
>>> s.price = 'a lot'
Traceback:
raise TypeError('expected ' + str(self.expected_type))
TypeError: expected <class 'float'>
>>> s.name = 'ABRACADABRA'
Traceback:
raise ValueError('size must be < ' + str(self.size))
ValueError: size must be < 8
```
还有一些技术可以简化上面的代码，其中一种是使用类装饰器：
```python
# Class decorator to apply constraints (类的装饰器)
def check_attribute(**kwargs):
    def decorate(cls):
        for key, value in kwargs.items():
            if isinstance(value, Descriptor):
                value.name = key
                setattr(cls, key, value)
            else:
                setattr(cls, key, value(key))
        return cls
    return decorate
    
# Example
@check_attribute(name=SizedString(size=8),
                 shares=UnsignedInteger,
                 price=UnsignedFloat)             
class Stock:
    def __init__(self, name, shares, price):
        self.name = name
        self.shares = shares
        self.price = price
```
另外一种方式是使用元类：

```python
# 对这个元类不太明白
# A metaclass that applies checking
class checkedmeta(type):
    def __new__(cls, clsname, bases, methods):
        # Attach attribute names to the descriptors
        for key, value in methods.items():
            if isinstance(value, Descriptor):
                value.name = key
        return type.__new__(cls, clsname, bases, methods)
        
# Example
class Stock2(metaclass=checkedmeta):
    name = SizedString(size=8)
    shares = UnsignedInteger()
    price = UnsignedFloat()
    
    def __init__(self, name, shares, price):
        self.name = name
        self.shares = shares
        self.price = price
```
## 实现自定义容器

问题：
实现一个自定义的类来模拟内置的容器类功能，比如列表和字典。但是你不确定到底要实现哪些方法。

解决方案：
collections 定义了很多抽象基类，当你想自定义容器类的时候它们会非常有用。比如你想让你的类支持迭代，那就让你的类继承 collections.Iterable 即可：
```python
import collections
class A(collections.Iterable):
    pass
```
不过你需要实现 collections.Iterable 所有的抽象方法，否则会报错：
```bash
>>> a = A()
Traceback (most recent call last):
TypeError: Can't instantiate abstract class A with abstract methods __iter__
>>>
```
你只要实现 \_\_iter\_\_() 方法就不会报错了
你可以先试着去实例化一个对象，在错误提示中可以找到需要实现哪些方法：
```python
>>> import collections
>>> collections.Sequence()
Traceback (most recent call last):
TypeError: can't instantiate abstract class Sequence with abstract methods __getitem__, __len__
>>>
```
示例（继承自上面 Sequence 抽象类，并且实现元素按照顺序存储）：
```python
class SortedItems(collections.Sequence):
    def __init__(self, initial=None):
        self._items = sorted(initial) if initial is not None else []
    
    # Required sequence methods
    def __getitem__(self, index):
        return self._items[index]
        
    def __len__(self):
        return len(self._items)
        
    # Method for adding an item in the right location
    def add(self, item):
        bisect.insort(self._items, item)
        
items = SortedItems([5, 1, 3])
print(list(items))
print(items[0], items[-1])
items.add(2)
print(list(items))
```
可以看到，SortedItems 跟普通的序列没什么两样，支持所有常用操作，包括索引、迭代、包含判断，甚至是切片操作。
这里面使用到了 bisect 模块，它是一个在排序列表中插入元素的高效方式。可以保证元素插入后还保持顺序。

讨论：
使用 collections 中的抽象基类可以确保自定义的容器实现了所有必要的方法。并且还能简化类型检查。自定义容器会满足大部分类型检查需要，如下：
```bash
>>> items = SortedItems()
>>> import collections
>>> isinstance(items, collections.Iterable)
True
>>> isinstance(items, collections.Sequence)
True
>>> isinstance(items, collections.Container)
True
>>> isinstance(items, collections.Sized)
True
>>> isinstance(items, collections.Mapping)
False
>>>
```
colections 中很多抽象类会为一些常见容器操作提供默认的实现，这样一来你只需要实现那些你最感兴趣的方法即可。假设你的类继承自 collections.MutableSequence,  如下：
```python
class Items(collections.MutableSequence):
    def __init__(self, initial=None):
        self._items = list(initial) if initial is not None else []
    
    # Required sequence methods
    def __getitem__(self, index):
        print('Getting:', index)
        return self._items[index]
        
    def __setitem__(self, index, value):
        print('Setting:', index, value)
        self._items[index] = value
        
    def __delitem__(self, index):
        print('Deleing:', index)
        del self._items[index]
    
    def insert(self, index, value):
        print('Inserting:', index, value):
        self._items.insert(index, value)
        
    def __len__(self):
        print('The length of items:')
        return len(self._items)
```
如果你创建 Items 的实例，你会发现它支持几乎所有的核心列表方法（如 append()、remove()、count() 等）。演示：
```python
>>> a = Item([1, 2, 3])
>>> len(a)
The length of items:
3
>>> a.append(4)
Inserting: 3 4
>>> a.count(2)
Getting:0
Getting:1
Getting:2
Getting:3
Getting:4
Getting:5
2
>>> a.remove(3)
Getting:0
Getting:1
Getting:2
Deleting:2
>>>
```
## 属性的代理访问


