---
title: Python Cookbook（函数）
date: 2017-11-06 23:28:04
tags: 函数
categories: Python Cookbook
---

# 函数

使用语句定义函数是所有程序的基础。本章讲述一些更加高级和不常见的函数定义与使用模式。涉及到的内容包括默认参数、任意数量参数、强制关键 字参数、注解和闭包。另外，一些高级的控制流和利用回调函数传递数据的技术在这里也会讲解到。 
<!-- more -->
## 可接受任意数量参数的函数

问题：
你想构造一个可接受任意数量参数的函数。

解决方案：
为了能让一个函数接受任意数量的位置参数，可以使用一个 * 参数。例如：
```bash
def avg(first, *rest):
    return (first + sum(rest)) / (1 + len(rest))

# Sample use
avg(1, 2) # 1.5
avg(1, 2, 3, 4) # 2.5
```
在这个例子中，rest 是由所有其他位置参数组成的元组。然后我们在代码中把它当成了一个序列来进行后续的计算。
为了接受任意数量的关键字参数，使用一个以 ** 开头的参数。比如：
```python
import html

def make_element(name, value, **attrs):
    keyvals = ['%s="%s"' % item for item in attrs.items()]
    attr_str = ''.join(keyvals)
    element = '<{name}{attrs}>{value}</{name}>'.format(
            name=name, 
            attrs = attr_str,
            value = html.escape(value)
    )
    return element

# Example
# Creats '<item size="large" quantity="6">Albatross</item>'
make_element('item', 'Albatross', size='large', quantity=6)

# Creates '<p>&lt; spam&gt;</p>'
make_element('p', '<spam>')
```
在这里，attrs 是一个包含所有被传入进来的关键字参数的字典。
如果你还希望某个函数能同时接受任意数量的位置参数和关键字参数，可以同时使用 * 和 **。比如：
```python
def anyargs(*args, **kwargs):
    print(args) # A tuple
    print(kwargs) # A dict
```
使用这个函数时，所有位置参数会被放到 args 元组中，所有关键字参数会被放到字典 kwargs 中。

讨论：
一个 * 参数只能出现在函数定义中最后一个位置参数后面，而 ** 参数只能出现在最后一个参数。有一点要注意的是,
在 * 参数后面仍然可以定义其他参数。
```python
def a(x, *args, y):		# 错误 
    pass
def b(x, *args, y, **kwargs):  # 错误
    pass
```
这就是强制关键字参数

## 只接受关键字参数的函数

问题：
你希望函数的某些参数强制使用关键字参数传递

解决方案：
将强制关键字参数放到某个 * 参数或者单个 * 后面就能达到这种效果。比如：
```python
def recv(maxsize, *, block):  # Python 3
    'Receives a message'
    pass
    
recv(1024, True) # TypeError
rece(1024, block=True) # Ok
```
利用这种技术，我们还能在接受任意多个位置参数的函数中指定关键字参数。比如：
```python
def mininum(*values, clip=None):
    m = min(values)
    if clip is not None:
        m = clip if clip > m else m
    return m
    
mininum(1, 5, 2, -5, 10) # Returns -5
mininum(1, 5, 2, -5, 10, clip=0) # Returns 0
```
讨论：
很多情况下，使用强制关键字参数会比位置参数表意更加清晰，程序也更加具有可读性。例如，考虑下如下一个函数调用：
```python
msg = recv(1024, False)
```
如果调用者对 recv 函数并不是很熟悉，那他肯定不明白那个 False 参数到底来干嘛用的。但是，如果代码变成下面这样子的话就清楚多了：
```python
msg = recv(1024, block=False)
```
另外，使用强制关键字参数也会比使用 **kwargs 参数更好，因为在使用函数 help 的时候输出也会更容易理解：
```bash
>>> help(recv)
Help on function recv in module __main__:
recv(maxsize, *, block)
    Receives a message
```
强制关键字参数在一些更高级场合同样也很有用。例如，它们可以被用来在使用 *args 和 **kwargs 参数作为输入的函数中插入参数。

## 给函数参数增加元信息

问题：
你写好了一个函数，然后想为这个函数的参数增加一些额外的信息，这样的话其他使用者就能清楚的知道这个函数应该怎么使用。

解决方案：
使用函数参数注解是一个很好的方法，它能提示程序员应该怎样正确使用这个函数。例如，下面有一个被注解了的函数：

```python
def add(x:int, y:int) -> int:
    return x + y
```
python 解释器不会对这些注解添加任何的语义。它们不会被类型检查，运行时跟没有注解之前的效果也没有任何差距。然而，对于那些阅读源码的人来讲就很有帮助啦。第三方工具和框架可能会对这些注解添加语义。同时它们也会出现在文档中。

```bash
>>> help(add)
Help on function add in module __main__:
add(x:int, y: int) -> int
>>>
```
尽管你可以使用任意类型的对象给函数添加注解（例如数字，字符串，对象实例等等），不过通常来讲使用类或者字符串会比较好点。

讨论：
函数注解只存储在函数的 \_\_annotations\_\_ 属性中。
```bash
>>> add.__annotations__
{'y': <class 'int'>, 'return': <class 'int'>, 'x': <class 'int'>}
```
尽管注解的使用方法可能有很多种，但是它们的主要用途还是文档。因为 python 并没有类型声明，通常来讲仅仅通过阅读源码很难知道应该传递什么样的参数给这个函数。这时候使用注解就能给程序员更多的提示，让他们可以争取的使用函数。 

## 返回多个值的函数

问题：
你希望构造一个可以返回多个值的函数

解决方案：
为了能返回多个值，函数直接 return 一个元组就行了。例如：
```bash
>>> def myfun():
...     return 1, 2, 3
...
>>> a, b, c = myfun()
>>> a
1
>>> b
2
>>> c
3
```
讨论：
尽管 myfun() 看上去返回了多个值，实际上是先创建了一个元组然后返回的。这个语法看上去比较奇怪，实际上我们使用的是逗号来生成一个元组，而不是用括号。比如：
```bash
>>> a = (1, 2) # With parentheses
>>> a
(1, 2)
>>> b = 1, 2 # Without parentheses
>>> b
(1, 2)
>>>
```
当我们调用返回一个元组的函数的时候，通常我们会将结果赋值给多个变量，就像上面的那样。其实这就是元组解包。返回结果也可以赋值给当个变量，这时候这个变量值就是函数返回的那个元组本身了：
```bash
>>> x = myfun()
>>> x
(1, 2, 3)
>>>
```

## 定义有默认参数的函数

问题：
你想定义一个函数或者方法，它的一个或多个参数是可选的并且有一个默认值。

解决方案：
定义一个有可选参数的函数是非常简单的，直接在函数定义中给参数指定一个默认值，并放到参数列表最后就行了。例如：
```python
def spam(a, b=42):
    print(a, b)
spam(1)    # Ok. a=1, b=42
spam(1, 2)    # Ok. a=1, b=2
```
如果默认参数是一个可修改的容器比如一个列表、集合或者字典，可以使用 None 作为默认值，就像下面这样：
```python
# Using a list as a default value
def spam(a, b=None):
    if b is None:
        b = []
    ...
```
如果你并不想提供一个默认值，而是想仅仅测试下某个默认参数是不是有传递进来，可以像下面这样写：
```python
_no_value = object()

def spam(a, b=_no_value):
    if b is _no_value:
        print('No b value supplied')
```
测试这个函数：
```bash
>>> spam(1)
No b value supplied
>>> spam(1, 2)    # b = 2
>>> spam(1, None)    # b = None
>>> 
```
仔细观察可以发现到传递一个 None 值和不传值两种情况是有差别的。

讨论：
定义带默认值参数的函数是很简单的，但绝不仅仅只是这个，还有一些东西在这里也深入讨论下。
首先，默认参数的值仅仅在函数定义的时候赋值一次。试着运行：
```bash
>>> x = 42
>>> def spam(a, b=x):
...     print(a, b)
...
>>> spam(1)
1 42
>>> x = 23 # Has no effect
>>> spam(1)
1 42
>>>
```
注意到当我们改变 x 的值的时候对默认参数值并没有影响，这是因为在函数定义的时候就已经确定了它的默认值了。
其次，默认参数的值应该是不可变的对象，比如 None、True、False、数字或字符串。特别的千万不要下面这样写代码：
```python
def spam(a, b=[]): # No!
    ...
```
如果你这么做了，当默认值在其他地方被修改后你将会遇到各种麻烦。这些修改会影响到下次调用这个函数时的默认值。比如：
```bash
>>> def spam(a, b=[]):
...     print(b)
...     return b
>>> x= spam(1)
>>> x
[]
>>> x.append(99)
>>> x.append('Yow!')
>>> x
[99, 'Yow!']
>>> spam(1) # Modified list gets returned!
[99, 'Yow!']
>>>
```
这种结果应该不是你想要的。为了避免这种情况的发生，最好是将默认值设为 None ，然后在函数里面检查它，前面的例子就是这样做的。
在测试 None 值时使用 is 操作符是很重要的，也是这种方案的关键点。有时大家会犯下下面这样的错误：
```python
def spam(a, b=None):
    if not b:     # No! Use 'b is None' instead
        b = []
    ...
```
这么写的问题在于尽管 None 值确实是被当成 False，但是还有其他的对象（比如长度为 0 的字符串、列表、元组、字典等）都会被当做 False 。因此，上面的代码会误将一些其他输入也当成是没有输入。比如：
```bash
>>> spam(1)    # OK
>>> x = []
>>> spam(1, x)    # Silent error. x value overwritten by default
>>> spam(1, 0)    # Silent error. 0 ignored
>>> spam(1, '')    # Silent error. '' ignored
```
最后一个问题比较微妙，那就是一个函数需要测试某个可选参数是否被使用者传递进来。这时候需要小心的是你不能用某个默认值比如 、 或者 值来测试用户提供的值 因为这些值都是合法的值，是可能被用户传递进来的 。因此，你需要其他的解决方案了。 

为了解决这个问题，你可以创建一个独一无二的私有对象实例，就像上面的 \_no\_value 变量那样。在函数里面，你可以通过检查被传递参数值跟这个实例是否一样来判断。这里的思路是用户不可能去传递这个实例作为输入。因此，这里通过检查这个值就能确定某个参数是否被传递进来了。
```python
_no_value = object()

def spam(a, b=_no_value):
    if b is _no_value:
        print('No b value supplied')
```
测试这个函数：
```bash
>>> spam(1)
No b value supplied
>>> spam(1, 2)    # b = 2
>>> spam(1, None)    # b = None
>>> 
```
这里对 object() 的使用看上去有点不太常见。 obejct是 python 中所有类的基类。 你可以创建 object 类的实例，但是这些实例没什么实际用处，因为它并没有任何有用的方法，也没有任何实例数据（因为它没有任何的实例字典，你甚至都不能设置任何属性值）。你唯一能做的就是测试同一性。这个刚好符合我的要求，因为我在函数中就只是需要一个同一性的测试而已。 

## 定义匿名或内联函数

问题：
你想为 sort() 操作创建一个很短的回调函数，但又不想用 def 去写一个单行函数，而是希望通过某个快捷方式以内联方式来创建这个函数。

解决方案：
当一些函数很简单， 仅仅只是计算一个表达式的值的时候，就可以使用 lambda 表达式来代替了。比如：
```bash
>>> add = lambda x, y: x + y
>>> add(2, 3)
5
>>> add('hello', 'world')
'helloworld'
>>>
```
这里使用的 lambda 表达式跟下面的效果是一样的：
```bash
>>> def add(x, y):
...     return x + y
...
>>> add(2, 3)
5
>>>
```
这里使用的 lambda 表达式跟下面的效果是一样的：
```bash
>>> def add(x, y):
...     return x + y
...
>>> add(2, 3)
5
>>>
```
lambda 表达式典型的使用场景是排序或数据 reduce 等：
```bash
>>> names = ['David Beazley', 'Brian Jones', 
...     'Raymond Hettinger', 'Ned Batchelder']
>>> sorted(names, key=lambda name: name.split()[-1].lower())
```
>PS:
>sort 与 sorted 区别：
>sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。
>list 的 sort 方法返回的是对已经存在的列表进行操作，而内建函数 sorted 方法返回的是一个新的 list，而不是在原来的基础上进行的操作。
语法
sorted 语法：
sorted(iterable[, cmp[, key[, reverse]]])
参数说明：
iterable -- 可迭代对象。
cmp -- 比较的函数，这个具有两个参数，参数的值都是从可迭代对象中取出，此函数必须遵守的规则为，大于则返回1，小于则返回-1，等于则返回0。
key -- 主要是用来进行比较的元素，只有一个参数，具体的函数的参数就是取自于可迭代对象中，指定可迭代对象中的一个元素来进行排序。
reverse -- 排序规则，reverse = True 降序 ， reverse = False 升序（默认）。

讨论：
尽管 lambda 表达式允许你定义简单函数，但是它的使用是有限制的。你只能指定单个表达式，它的值就是最后的返回值。也就是说不能包含其他的语言特性了，包括多个语句、条件表达式、迭代以及异常处理等等。
你可以不使用 lambda 表达式就能编写大部分 python 代码。但是，当有人编写大量计算表达式值的短小函数或者需要用户提供回调函数的程序的时候，你就会看到 lambda 表达式的身影了。

## 匿名函数捕获变量值

问题：
你用 lambda 定义了一个匿名函数，并想在定义时捕获到某些变量的值。

解决方案：
先看下面代码的效果：
```bash
>>> x = 10
>>> a = lambda y: x + y
>>> x = 20
>>> b = lambda y: x + y
>>>
```
a(10) 和 b(10)返回的结果是什么？如果你认为结果是 20 和 30，那么你就错了：
```bash
>>> a(10)
30
>>> b(10)
30
>>>
```
这其中的奥妙在于 lambda 表达式中的 x 是一个自由变量，在运行时绑定值，而不是定义时就绑定，这跟函数的默认值参数定义是不同的。因此，在调用这个 lambda 表达式的时候，x 的值是执行时的值。例如：
```bash
>>> x = 15
>>> a(10)
25
>>> x = 3
>>> a(10)
13
>>>
```
如果你想让某个匿名函数在定义时就捕获到值，可以将那个参数值定义成默认参数即可，就像下面这样：
```bash
>>> x = 10
>>> a = lambda y, x=x: x + y
>>> x = 20
>>> b = lambda y, x=x: x + y
>>> a(10)
20
>>> b(10)
30
>>>
```
讨论：
如果想通过在一个循环或列表推导中创建一个 lambda 表达式列表，并期望函数能在定义时就记住每次的迭代值。例如：
```bash
>>> funcs = [lambda x: x+n for n in range(5)]
>>> for f in funcs:
...     print(f(0))
...
4
4
4
4
4
>>>
```
但是实际效果是运行是 n 的值为迭代的最后一个值。现在我们用另一种方式修改一下：
```bash
>>> funcs = [lambda x, n=n: x+n for in range(5)]
>>> for f in funcs:
...     print(f(0))
...
0
1
2
3
4
>>>
```
通过使用函数默认值参数形式，lambda 函数在定义时就能绑定到值。

## 减少可调用对象的参数个数

问题：
你有一个被其他 python 代码使用的 callable 对象，可能是一个回调函数或者是一个处理器，但是它的参数太多了，导致调用时出错。

解决方案
如果需要减少某个函数参数个数，你可以使用 functools.partial()。partial() 函数允许你给一个或多个参数设置固定的值，减少接下来被调用时的参数个数。为了演示清楚，假设你有下面这样的函数：
```python
def spam(a, b, c, d):
    print(a, b, c, d)
```
现在我们使用 partial() 函数来固定某些参数值：
```bash
>>> from functools import partial
>>> s1 = partial(spam, 1) # a = 1
>>> s1(2, 3, 4)
1 2 3 4
>>> s2 = partial(spam, d=42) # d = 42
>>> s2(1, 2, 3)
1 2 3 42
>>> s3 = partial(spam, 1, 2, d=42)
>>> s3(3)
1 2 3 42
>>> s3(4)
1 2 4 42
>>>
```
>如果想改变 s2 中的 d ：
>错误写法：s2(1, 2, 3, 4)
>正确写法：s2(1, 2, 3, d=4)

可以看出 partial() 固定某些参数并返回一个新的 callable 对象。这个新的 callable 接受未赋值的参数，然后跟之前已经赋值过的参数合并起来，最后将所有参数传递给原始函数。

讨论：
假设你有一个点的列表来表示 (x, y) 坐标元组。你可以使用下面的函数来计算两点之间的距离：
```python
points = [(1, 2), (3, 4), (5, 6), (7, 8)]

import math
def distance(p1, p2):
    x1, y1 = p1    # p1 为元组，例如：(1, 2)
    x2, y2 = p2
    return math.hypot(x2 - x1, y2 - y1)
```
现在假设你想以某个点为基点，根据点和基点之间的距离来排序所有的这些点。列表的 sort() 方法接受一个关键字参数来自定义排序逻辑，但是它只能接受一个单个参数的函数（distance() 很明显是不符合条件的）。现在我们可以通过使用 partial() 来解决这个问题: 
```bash
>>> pt = (4, 3) # 作为基点，与 points 中的点分别计算距离
>>> points.sort(key=partial(distance, pt))
>>> points
[(3, 4), (1, 2), (5, 6), (7, 8)]
>>> 
```
更进一步，partial() 通常被用来微调其他库函数所使用的回调函数的参数。例如，下面是一段代码，使用 multiprocessing 来异步计算一个结果值，然后这个值被传递给一个接受一个result 值和一个可选 logging 参数的回调函数：
```python
def output_result(result, log=None):
    if log is None:
        log.debug('Got: %r', result)

# A sample function
def add(x, y):
    return x + y
    
if __name__ == '__main__':
    import logging
    from multiprocessing import Pool
    from functools import partial
    
    logging.basicConfig(level=logging.DEBUG)
    log = logging.getLogger('test')
    
    p = Pool()
    p.apply_async(add, (3, 4), callback=partial(output_result, log=log))
    p.close()
    p.join()
```
当给 apply_async() 提供回调函数时，通过使用 partial() 传递额外的 logging 参数。而 multiprocessing 对这些一无所知———它仅仅只是使用单个值来调用回调函数。
>未完

## 将单方法的类转换为函数

问题：
你有一个除 \_\_init\_\_() 方法外只定义了一个方法的类。为了简化代码，你想将它转换成一个函数。

解决方案：
大多数情况下，可以使用闭包来将单个方法的类转换成函数。举个例子，下面示例中的类允许使用者根据某个模版方案来获取到 URL 链接地址。
```python
from urllib.request import urlopen

class UrlTemplate:
    def __init__(self, template):
        self.template = template
        
    def open(self, **kwargs):
        return urlopen(self.template.format_map(kwargs))
        
# Example user. Downlaod stock data from yahoo
yahoo = UrlTemplate('http://finance.yahoo.com/d/quotes.csv?s={name}&f={fields}')
for line in yahoo.open(names='IBM,AAPL,FB', fields='slic1v'):
    print(line.decode('utf-8'))
```
这个类可以被一个更简单的函数来代替：
```python
def urltemplate(template):
    def opener(**kwargs):
        return urlopen(template.format_map(kwargs))
    return opener
    
# Example use
yahoo = urltemplate('http://finance.yahoo.com/d/quotes.csv?s={names}&f={fields}')
for line in yahoo(names='IBM,AAPL,FB', fields='sliciv'):
    print(line.decode('utf-8'))
```
讨论：
大部分情况下，你拥有一个单方法类的原因是需要存储某些额外的状态来给方法使用。比如，定义 UrlTemplate 类的唯一目的就是先在某个地方存储模版值，以便将来可以在 open() 方法中使用。

使用一个内部函数或者闭包的方案通常会更优雅一些。简单来讲，一个闭包就是一个函数，只不过在函数内部带上了一个额外的变量环境。闭包关键点就是它会记住自己被定义时的环境。因此，在我们的解决方案中，opener() 函数记住了 template 参数的值，并在接下来的调用中使用它。

任何时候只要你碰到需要给某个函数增加额外的状态信息的问题，都可以考虑使用闭包。相比将你的函数转换成一个类而言，闭包通常是一种更加简洁和优雅的方案。

## 带额外状态信息的回调函数

问题：
你的代码中需要依赖到回调函数的使用（比如事件处理器、等待后台任务完成后的回调等），并且你还需要让回调函数拥有额外的状态值，以便在它的内部使用到。

解决方案：
这一小节主要讨论的是那些出现在很多函数库和框架中的回调函数的使用————特别是跟异步处理有关的。为了演示与测试，我们先定义如下一个需要调用回调函数的函数：
```python
def apply_async(func, args, *, callback):
    # Compute the result
    result = func(args)
    
    # Invoke the callback with the result
    callback(result)
```
实际上，这段代码可以做任何更高级的处理，包括线程、进程和定时器，但是这些都不是我们要关心的。我们仅仅只需要关注回调函数的调用。下面演示怎样使用上述代码的例子：
```bash
>>> def print_result(result):
...     print('Got:', result)
...
>>> def addd(x, y):
...     return x + y
...
>>> apply_async(add, (2, 3), callback=print_result)
Got: 5
>>> apply_async(add, ('hello', 'world', callback=print_result))
Got: helloword
>>>
```
注意到 print_result() 函数仅仅只接受一个参数 result 。不再传入其他信息。而当你想让回调函数访问其他变量或者特定环境的变量值的时候就会遇到麻烦。

为了让回调函数访问外部信息，一种方法是使用一个绑定方法来代替一个简单函数。不如，下面这个类会保存一个内部序列号，每次接受到一个 result 的时候序列号加 1:
```python
class ResultHandler:
    def __init__(self):
        self.sequence = 0
    def handler(self, result):
        self.sequence += 1
        print('[{}] Got : {}'.format(self, sequence, result))
```
使用这个类的时候，你先创建一个类的实例，然后用它的 handler() 绑定方法来做为回调函数：
```bash
>>> r = ResultHandler()
>>> apply_async(add, (2, 3), callback=r.handler)
[1] Got: 5  # r.handler(add(2, 3))
>>> apply_async(add, ('hello', 'world'), callback=r.handler)
[2] Got: helloworld
```
第二种方式，作为类的替代，可以使用一个闭包捕获状态值，例如：
```python
def make_handler():
    sequence = 0
    def handler(result):
        nonlocal sequence
        sequence += 1
        print('[{}] Got: {}'.format(sequence, result))
    return handler
```
下面是使用闭包方式的一个例子：
```bash
>>> handler = make_handler()
>>> apply_async(add, (2, 3), callback=handler)
[1] Got: 5
>>> apply_async(add, ('hello', 'world'), callback=handler)
[2] Got: helloworld
>>>
```

>用闭包或者类的方法作为回调函数，是因为可以存储序列号 sequence

还有另外一个更高级的方法，可以使用协程来完成同样的事情：
```python
def make_handler():
    sequence = 0
    while True:
        result = yield
        sequence += 1
        print('[{}] GOt: {}'.format(sequence, result))
```
对于协程，你需要使用它的 send() 方法作为回调函数，如下所示：
```bash
>>> handler = make_handler()
>>> next(handler) # Advance to the yield
>>> apply_async(add, (2, 3), callback=handler.send)
[1] Got: 5
>>> applt_async(add, ('hello', 'world'), callback=handler.send)
[2] Got: helloworld
>>>
```
讨论：
	基于回调函数的软件通常都有可能变得非常复杂。一部分原因是回调函数通常会跟请求执行代码断开。因此，请求执行和处理结果之间的执行环境实际上已经丢失了。如果你想让回调函数连续执行多步操作，那你就必须去解决如何保存和恢复相关的状态信息了。 
	至少有两种主要方式来捕获和保存状态信息，你可以在一个对象实例 （通过一个绑定方法）或者在一个闭包中保存它。两种方式相比，闭包或许是更加轻量级和自然一点，因为它们可以很简单的通过函数来构造。它们还能自动捕获所有被使用到的变量。因此，你无需去担心如何去存储额外的状态信息（代码中自动判定 ）。 
	如果使用闭包，你需要注意对那些可修改变量的操作。在上面的方案中，nonlocal 声明语句用来指示接下来的变量会在回调函数中被修改。如果没有这个声明，代码会报错。 
	而使用一个协程来作为一个回调函数就更有趣了，它跟闭包方法密切相关。某种意义上来讲，它显得更加简洁，因为总共就一个函数而已。并且，你可以很自由的修改变量而无需去使用 nonlocal 声明。这种方式唯一缺点就是相对于其他 Python 技术而已或许比较难以理解。另外还有一些比较难懂的部分，比如使用之前需要调用 next()，实际使用时这个步骤很容易被忘记。尽管如此，协程还有其他用处，比如作为一个内联回调函数的定义(下一节会讲到)。 
	如果你仅仅只需要给回调函数传递额外的值的话，还有一种使用 的 partial() 方式也很有用。在没有使用 partial() 的时候，你可能经常看到下面这种使用 lambda 表达式的复杂代码: 
```bash
>>> apply_async(add, (2, 3), callback=lambda r: handler(r, seq))
[1] Got: 5
>>>
```

## 内联回调函数

问题：
当你编写使用回调函数的代码的时候，担心很多小函数的扩张可能会弄乱程序控制流。你希望找到某个方法来让代码看上去更像是一个普通的执行序列。

解决方案：
通过使用生成器和协程可以使得回调函数内联在某个函数中。为了演示说明，假设你有如下所示的一个执行某种计算任务然后调用一个回调函数的函数：
```python
def apply_async(func, args, *, callback):
    # Compute the result
    result = func(*args)
    
    # Invoke the callback with the result
    callback(result)
```
接下来让我们看一下下面的代码，它包含了一个 Async 类和一个 inlined_async 装饰器：
```python
from queue import Quene
from functools import wraps

class Async:
    def __init__(self, func, args):
        self.func = func
        self.args = args

def inlined_async(func):
    @wraps(func)
    def wrapper(*args):
        f = func(*args)
        result_queue = Queue()
        result_queue.put(None)
        while True:
            result = result_queue.get()
            try:
                a = f.send(result)
                apply_async(a.func, a.args, callback=result_queue.put)
            except StopIteration:
                break
    return wrapper
```
这两个代码片段允许你使用 yield 语句内联回调步骤。比如：
```python
def add(x, y):
    return x + y
    
@inlined_async
def test():
    r = yield Async(add, (2, 3))
    print(r)
    r = yield Async(add, ('hello', 'world'))
    print(r)
    for n in range(3):
        r = yield Async(add, (n, n))
        print(r)
    print('Goodbye')
```
如果你调用 test()， 你会得到类似如下的输出：
```bash
5
helloworld
0
2
4
Goodbye
```
你会发现，除了那个特别的装饰器和 yield 语句外，其他地方并没有出现任何的回调函数（其实是在后台定义的）。

讨论：
详见 P226

## 访问闭包中定义的变量

问题：
你想要扩展函数中的某个闭包，允许它能访问和修改函数的内部变量。

解决方案：
通常来讲，闭包的内部变量对于外界来讲是完全隐藏的。但是，你可以通过编写访问函数并将其作为函数属性绑定到闭包上来实现这个目的。例如:
```python
def sample():
    n = 0
    # Closure function
    def func():
        print('n=', n)
    # Accessor methods for n
    def get_n():
        return n
    
    def set_n(value):
        nonlocal n
        n = value
        
    # Attach as function attributes
    func.get_n = get_n
    func.set_n = set_n
    return func
```
使用例子：
```bash
>>> f = sample()
>>> f()
n = 0
>>> f.set_n(10)
>>> f()
n = 10
>>> f.get_n()
10
>>>
```
讨论：
为了说明清楚它如何工作的，有两点需要解释一下。首先，nonlocal 声明可以让我们编写函数来修改内部变量的值。其次，函数属性允许我们用一种很简单的方式将访问方法绑定到闭包函数上，这个跟实例方法很像（尽管并没有定义类）。
还可以进一步的扩展，让闭包模拟类的实例。你要做的仅仅是复制上面的内部函数到一个字典实例中并返回它即可。例如：
```python
import sys
class ClosureInstance:
    def __init__(self, locals=None):
        if locals is None:
            locals = sys._getframe(1).f_locals
        
        # Update instance dictionary with callables
        self.__dict__.update((key, value) for key, value in locals.items()
                            if callable(value))
        # Redirect special methods
        def __len__(self):
            return self.__dict__['__len__']()
            
# Example use
def Stack():
    items = []
    def push(item):
        items.append(item)
        
    def pop():
        return items.pop()
        
    def __len__():
        return len(items)
        
    return ClosureInstance()
```
下面是一个交互式会话演示它是如何工作的：
```bash
>>> s = Stack()
>>> s
<__main__.ClosureInstance object at 0x...>
>>> s.push(10)
>>> s.push(20)
>>> len(s)
2
>>> s.pop()
20 
>>> s.pop()
10
>>>
```
有趣的是，这个代码运行起来会比一个普通的类定义要快很多。你可能会像下面这样测试它跟一个类的性能对比:
```python
class Stack2:
    def __init__(self):
        self.items = []
    
    def push(self, item):
        self.items.append(item)
        
    def pop(self):
        return self.items.pop()
     
    def __len__(self):
        return len(self.items)
```
如果这样做，你会得到类似如下的结果：
```bash
>>> from timeit import timeit
>>> # Test involving closures
>>> s = Stack()
>>> timeit('s.push(1);s.pop()', 'from __main__ import s')
0.9874754269
>>> # Test involving a class
>>> s = Stack2()
>>> timeit('s.push(1);s.pop()', 'from __main__ import s')
1.0707052160
>>>
```
结果显示，闭包的方案运行起来要快大概 8%，大部分原因是因为对实例变量的简化访问，闭包更快是因为不会涉及到额外的 self 变量。 
总体上讲，在配置的时候给闭包添加方法会有更多的实用功能，比如你需要重置内部状态、刷新缓冲区、清除缓存或其他的反馈机制的时候。

