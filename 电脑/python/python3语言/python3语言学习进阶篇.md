<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline17">1. 类的高级知识</a>
<ul>
<li><a href="#orgheadline5">1.1. 类内部的字典</a>
<ul>
<li><a href="#orgheadline1">1.1.1. <code>__dict__</code> 值</a></li>
<li><a href="#orgheadline2">1.1.2. 类按键取值</a></li>
<li><a href="#orgheadline3">1.1.3. 类按键赋值</a></li>
<li><a href="#orgheadline4">1.1.4. 字典的索引删除</a></li>
</ul>
</li>
<li><a href="#orgheadline9">1.2. 属性管理</a>
<ul>
<li><a href="#orgheadline6">1.2.1. 属性赋值</a></li>
<li><a href="#orgheadline7">1.2.2. 属性删除</a></li>
<li><a href="#orgheadline8">1.2.3. 属性获取</a></li>
</ul>
</li>
<li><a href="#orgheadline15">1.3. 数学运算</a>
<ul>
<li><a href="#orgheadline10">1.3.1. 一般加法</a></li>
<li><a href="#orgheadline11">1.3.2. 右侧加法</a></li>
<li><a href="#orgheadline12">1.3.3. 增强加法</a></li>
<li><a href="#orgheadline13">1.3.4. 一般减法</a></li>
<li><a href="#orgheadline14">1.3.5. 其他数学运算符一览</a></li>
</ul>
</li>
<li><a href="#orgheadline16">1.4. 逻辑运算</a></li>
</ul>
</li>
<li><a href="#orgheadline18">2. 深入理解python3的迭代</a></li>
<li><a href="#orgheadline19">3. 模块包</a></li>
<li><a href="#orgheadline21">4. 文件处理高级知识</a>
<ul>
<li><a href="#orgheadline20">4.1. 文件对象的seek方法</a></li>
</ul>
</li>
<li><a href="#orgheadline22">5. 与c语言或c++语言编写的模块集成</a></li>
<li><a href="#orgheadline23">6. PEP8推荐的风格</a></li>
</ul>
</div>
</nav>

测试代码和模块文件分开。

# 类的高级知识<a id="orgheadline17"></a>

## 类内部的字典<a id="orgheadline5"></a>

### `__dict__` 值<a id="orgheadline1"></a>

类和模块都支持对其内部属性或者方法的点的引用方式，这种引用是根据类和模块的 `__dict__` 这个字典值来的，然后属性继承就是向上搜索链接的字典。python中的类和实例实际就是带有链接的字典，其内的方法和属性都是字典值，其都可以通过 `__dict__` 的字典索引语法来引用。如这样的语法：X.\_\_dict\_\_['data']

类的字典比如下面这个：

```python
class A():
    def __init__(self,a):
        self.a=a

    def fun2(self,what):
        print('fun',what)

class B(A):
    def __init__(self):
        self.d=5
    b=2
    def fun3(self):
        print('fun3')
```

其 `__dict__` 值如下所示：

    >>> B.__dict__
    mappingproxy({'__doc__': None, '__init__': <function B.__init__ at 
    0xb7096adc>, 'fun3': <function B.fun3 at 0xb7096a4c>, 
    '__module__': '__main__', 'b': 2})

我们看到除了class定义的继承关系之外，类B内部定义的所有属性都存储在 `__dict__` 字典值里面了。我们可以猜测类的继承关系python是通过另外一种机制管理的，而且有一个通用类，这个通用类上有很多方法就是后面要谈到的那些内置的方法。

点运算索引是加上继承搜索机制了的，这里的 `__dict__` 的值只是本实例或者本类存储的属性。

`__dict__` 这个字典值对于描述类或实例的存在状态很重要，下面有三个python的内置的类的方法，你可以通过重新定义它们来获得你想要的效果。

### 类按键取值<a id="orgheadline2"></a>

`__getitem__` 方法支持类或实例以Class['x']这样的格式引用其内字典某个（或者某一些）索引的值。

### 类按键赋值<a id="orgheadline3"></a>

`__setitem__` 方法支持类或实例以Class['x']=3这样的格式修改其内字典的某个（或者某一些）索引的值。

### 字典的索引删除<a id="orgheadline4"></a>

`__delitem__` 方法支持类或实例以del Class['x']这样的格式删除字典的某个（或者某一些）索引。

关于上面三个内置方法都以下面这个例子来讲解了：

```python
class GClass():
    def __getitem__(self,key):
        return self.__dict__[key]
    def __setitem__(self,key,value):
        self.__dict__[key] = value
    def __delitem__(self,key):
        del self.__dict__[key]

    def __repr__(self):
        return str(self.__dict__)
```

    >>> test = GClass()
    >>> print(test)
    {}
    >>> test['a'] = 1
    >>> print(test)
    {'a': 1}
    >>> test['a'] = 2
    >>> print(test)
    {'a': 2}
    >>> del test['a']
    >>> print(test)
    {}

实际上上面三个内置方法不一定是对类内部字典的操作，也可以是类其中其他数据的操作，而且经过修改它们也支持如同列表的切片操作。

## 属性管理<a id="orgheadline9"></a>

`__getattr__(self,name)`  返回self.name的值。 

### 属性赋值<a id="orgheadline6"></a>

`__setattr__(self,name,value)`  和self.name = value动作关联。

### 属性删除<a id="orgheadline7"></a>

`__delattr__(self,name)`  del self.name

### 属性获取<a id="orgheadline8"></a>

`__getattribute__(self,name)` 不推荐使用。

## 数学运算<a id="orgheadline15"></a>

### 一般加法<a id="orgheadline10"></a>

X + other ,  `__add__(self,other)`

### 右侧加法<a id="orgheadline11"></a>

所谓加法是X+other，如果是右侧加法，则为radd，然后公式是：other+X。一般不区分左右的就用上面的一般加法。

other + X , `__radd__(self,other)`

### 增强加法<a id="orgheadline12"></a>

X +=other ， `__iadd__(self.other)`

### 一般减法<a id="orgheadline13"></a>

X - other ,  `__sub__(self,other)`

同上面情况一样类似的还有rsub和isub。

### 其他数学运算符一览<a id="orgheadline14"></a>

然后其他数学运算符下面简要列表之：

-   **\*:** 乘法， `__mul__(self,other)` ，下面的类似的都有右侧运算和增强运算，不再赘述了。
-   **//:** 整除， `__floordiv__` ，下面类似的参数都是self和other，不再赘述了。
-   **/:** 除法 ， `__div__`
-   **%:** 取余, `__mod__`
-   **\*\*:** 开方， `__pow__`
-   **<<:** 左移运算， `__lshift__`
-   **>>:** 右移运算， `__rshift__`
-   **&:** 位与， `__and__`
-   **|:** 位或， `__or__`
-   ^ 位异或 (异或的逻辑是相同取0，不同取1。) ， `__xor__`

类似的右侧运算名字前面加上r，增强运算名字前面加上i，不赘述了。

## 逻辑运算<a id="orgheadline16"></a>

\subsection{bool函数支持}
bool(X)  \verb+\_\_bool\_\_(self)+

\subsection{类之间的相等判断}
\href{http://www.informit.com/articles/article.aspx?p=453682}{参考网站}。

这里先总结下is语句和==判断和isinstance和id还有type函数，然后再提及python类的内置方法\verb+\_\_eq\_\_+。

python是一个彻头彻尾的面向对象的语言，python内部一切数据都是对象，对象就有类型type的区别。比如内置的那样对象类型：

\begin{Verbatim}
>>> type('abc')
<class 'str'>
>>> type(123)
<class 'int'>
>>> type([1,2,3])
<class 'list'>
\end{Verbatim}

对象除了有type类型之外，还有id属性，id就是这个对象具体在内存中的存储位置。

当我们说lst=[1,2,3]的时候，程序具体在内存中创建的对象是[1,2,3]，而lst这个变量名不过是一个引用。然后我们看下面的例子：

\begin{Verbatim}
>>> x=[1,2,3]
>>> y=[1,2,3]
>>> type(x)
<class 'list'>
>>> type(y)
<class 'list'>
>>> id(x)
3069975884
>>> id(y)
3062209708
>>> x==y
True
>>> x is y
False
\end{Verbatim}

type函数返回对象的类型，id函数返回对象具体在内存中的存储位置，而==判断只是确保值相等，is语句返回True则更加严格，需要对象在内存上（即id相等）完全是同一个东西。

对象之间的类型比较可以用如下语句来进行比较：

\begin{Verbatim}
>>> x=10
>>> type(x) == int
True
>>> type(x) == type(0)
True
\end{Verbatim}

不过不是特别好用，比如假设fun是你自己定义的一个函数，用type(fun) == function就会出错，然后type比较还要小心NoneType和其他空列表类型不同，而且type比较并没有将类的继承考虑进去。

一般推荐isinstance函数来进行类型比较，请参考\href{http://stackoverflow.com/questions/1549801/differences-between-isinstance-and-type-in-python}{这个网站}的说明。推荐使用types模块的特定名字来判断类型，具体如下：

\begin{description}
\item[types.NoneType] None这个值的类型
\item[types.TypeType] type对象。
\item[types.BooleanType] 还可以使用\textbf{bool}。
\item[types.IntType] 还可以使用\textbf{int}，类似的有\textbf{long}，\textbf{float}。
\item[types.ComplexType] 复数类型
\item[types.StringType] 字符串类型，还可以使用\textbf{str}。
\item[types.TupleType] 元组，还可以使用\textbf{tuple}，类似的有\textbf{list}，\textbf{dict}。
\item[types.FunctionType] 定义的函数类型，此外还有\textbf{types.LambdaType}。

值得一提的是print等内置函数不是FunctionType而是BuiltinFunctionType。
\begin{Verbatim}
>>> import types
>>> isinstance(print,types.FunctionType)
False
>>> isinstance(print,types.BuiltinFunctionType)
True
\end{Verbatim}

\end{description}

\begin{large}
更多内容请参见\href{https://docs.python.org/3.4/library/types.html}{types模块的官方文档}。
\end{large}

\subsubsection{\\\_{}\\\_{}eq\\\_{}\\\_{}方法}
\verb+\_\_eq\_\_+方法定义了两个对象之间A == B的行为。
比如下面：

\begin{tcbpython}[]
def __eq__(self,other):
    if self.__dict__.keys() == other.__dict__.keys():
        for key in self.__dict__.keys():
            if  not self.__dict__.get(key)==other.__dict__.get(key):
                return False
        return True
    else:
        return False
\end{tcbpython}

定义了这样的\verb+\_\_eq\_\_+方法之后，我们运行==语句，如果两个对象之间内置字典键和值都是一样的，那么就返回True。

\begin{Verbatim}
>>> test=GClass()
>>> test.a=1
>>> test2=GClass()
>>> test2.a=1
>>> test == test2
True
>>> test is test2
False
\end{Verbatim}

如果我们不重定义\verb+\_\_eq\_\_+方法，似乎test和test2会从原始的object类继承\verb+\_\_eq\_\_+方法，然后它们比较返回的是False，我想可能是这两个实例内部某些值的差异吧，但应该不是基于id。

\subsection{比较判断操作}
类似上面的==比较操作，还有如下比较判断操作和对应的内置方法可以重定义。

\begin{itemize}
\item X != Y ，行为由\verb+__ne__(self,other)+定义。
\item X >= Y ，行为由\verb+__ge__(self,other)+定义。
\item X <= Y ，行为由\verb+__le__(self,other)+定义。
\item X > Y ，行为由\verb+__gt__(self,other)+定义。
\item X < Y ，行为由\verb+__lt__(self,other)+定义。
\end{itemize}

\subsection{in语句}
如下所示：

\begin{tcbpython}[]
    def __in__(self,other):
        for key in self.__dict__.keys():
            if not self.__dict__.get(key) == other.__dict__.get(key):
                return False
        return True
\end{tcbpython}

提供了what in X语句的支持，上面的例子是基于类其内字典的内容而做出的判断。

\section{强制类型变换}
所包含的内置方法有：

\begin{Verbatim}
__int__(self)   返回整型
__long__(self)  长整型
__float__(self)  浮点型
__complex__(self)  复数型
__str__(self)  字符型
__oct__(self)  八进制
__hex__(self) 十六进制
__index__(self) 切片操作
\end{Verbatim}

\section{len函数}
由\verb+\_\_len\_\_(self)+提供支持。

\section{copy方法和deepcopy方法}
X.copy()  由\verb+\_\_\_copy\_\_(self)+提供。

X.deepcopy()  由\verb+\_\_\_deepcopy\_\_(self)+提供。
\section{with语句支持}
在打开文件那里谈及的with open(&#x2026;) as f的这类语句是由以下两个内置方法提供的：\verb+\_\_enter\_\_(self)+和\verb+\_\_exit\_\_(self,&#x2026;)+，exit的还有其他一些参数这里忽略了，enter的返回值会赋值给with中的as后面的变量。

\section{函数调用}
请看下面的例子：

\begin{tcbpython}[]
class Position():
    def __init__(self,x=0,y=0):
        self.x = x
        self.y = y
    def __call__(self,x,y):
        self.x = x
        self.y = y
    def __repr__(self):
        return '('+str(self.x)+ ',' + str(self.y)+')'
\end{tcbpython}

\begin{Verbatim}
>>> p1=Position()
>>> print(p1)
(0,0)
>>> p1(4,5)
>>> print(p1)
(4,5)
>>> 
\end{Verbatim}

\verb+\_\_call\_\_(self,args)+方法支持类或者实例以X(args)或者instance(args)这样的形式调用这个函数。

\section{和迭代操作有关}
\subsection{\\\_{}\\\_{}next\\\_{}\\\_{}方法}
比如文件对象本身就是可迭代的，调用\verb+\_\_next\_\_+方法就返回文件中下一行的内容，到达文件尾也就是迭代越界了返回：\textbf{StopIteration}异常。

\subsection{next函数}
next函数比如next(f)等价于\verb+f.\_\_next\_\_()+，其中f是一个文件对象。

\begin{Verbatim}
>>> for line in open('removeduplicate.py'):
...  print(line,end='')
... 
#!/usr/bin/env python3
#-*-coding:utf-8-*-
#此处一些内容省略。
    
>>> f=open('removeduplicate.py')
>>> next(f)
'#!/usr/bin/env python3\n'
\end{Verbatim}

所以你可以通过定义类的\verb+\_\_next\_\_+方法来获得这个类对于next函数时的反应。

\subsection{iter函数}
\uwave{文件对象}，\uwave{map对象}，\uwave{zip对象}，\uwave{filter对象}，\uwave{生成器对象}自身已经带有了\verb+\_\_next\_\_+方法，所以可以直接用next函数。不过虽然序列（列表，元组，字典，ranges对象\footnote{range函数的返回，也属于序列类型。}）等是可迭代对象，但是没有\verb+\_\_next\_\_+方法。

iter函数是调用的 \verb+\_\_iter\_\_+ 方法，其返回的是一个列表可迭代对象。

\begin{Verbatim}
>>> list=[1,2,3]
>>> next(list)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'list' object is not an iterator
>>> i=iter(list)
>>> next(i)
1
>>> i.__next__()
2
\end{Verbatim}

下面是for语句的while实现版本：

\begin{Verbatim}
>>> list=[1,2,3]
>>> iter=iter(list)
>>> while True:
...    try:
...        x=next(iter)
...    except StopIteration:
...        break
...    print(x)
... 
1
2
3
\end{Verbatim}

range对象也可以通过iter函数来生成一个可迭代对象。

\subsection{重构字典的iter函数}
我们可以通过重新定义字典类的\verb+\_\_iter\_\_+函数来获得一个新类，这个类用iter函数处理之后的迭代器返回的是经过排序的字典的键。

\begin{tcbpython}[]
class SortedDict(dict):
    def __init__(self,dict={}):
        super().__init__(dict)

    def __iter__(self):
        self._keys = sorted(self.keys())
        for i in self._keys:
            yield i

dict02 = SortedDict()
dict02['a'] = 1
dict02['b'] = 1
dict02['c'] = 1
dict02['d'] = 1
x = iter(dict02)
\end{tcbpython}

这个\verb+\_\_iter\_\_+函数就是所谓的生成器函数，需要返回一个生成器对象。然后经过iter函数处理之后就能调用next函数来逐渐获得经过排序之后的值了。

\section{当对象内存存储回收时的操作}
当对象内存存储被回收时，python最后将执行一个内置方法\verb+\_\_del\_\_+，这个一般不推荐使用。

\section{静态方法}

\begin{Verbatim}
class Test:
#    @staticmethod
    def hello():
        print('aaa')

test=Test()
test.hello()
\end{Verbatim}

在上面的例子中，我们希望创造一个函数，这个函数和self实例没有关系（这里指这个函数将不接受self这个默认参数了）。如上所示，hello函数只是希望简单打印一小段字符，如上面这样的代码是错误的，如果我们在这个函数上面加上\textbf{@staticmethod}，那么上面这段代码就不会报错了，

\begin{Verbatim}
class Test:
    @staticmethod
    def hello():
        print('aaa')

test=Test()
test.hello()
\end{Verbatim}

这样在类里面定义出来的函数叫做这个类的静态方法，静态方法同样可以继承等等，而静态方法通常使用最大的特色就是不需要建立实例，即可以直接从类来调用，如下所示：

\begin{Verbatim}
class Test:
    @staticmethod
    def hello():
        print('aaa')

Test.hello()
\end{Verbatim}

静态方法的使用比如pyqt中的

\begin{Verbatim}
QtGui.QFileDialog.getOpenFileName(......)
\end{Verbatim}

就是一个静态方法，可以通过直接调用这个方法来弹出询问打开文件的窗口，并不需要先实例化一个对象，然后通过self.what等类似的形式来调用。

\section{装饰器}
python语言的装饰器概念算是比较高级的概念了，不过并不是那种冷门的用的很少的概念，比如在前面的静态方法中就使用装饰器的概念：

\begin{tcbpython}[]
    @staticmethod
    def what():
        pass
\end{tcbpython}

装饰器的作用机制就是对接下来的函数进行进一步的封装，比如上面的例子就是：

\begin{tcbpython}[]
    def what():
        pass
    what = staticmethod(what)
\end{tcbpython}

可见装饰器并不是一个什么神秘的难懂的概念，同样你可以定义自己的函数，这个函数处理某个函数对象，并对其进行某种封装。

类似的装饰器还有类的方法装饰器\verb+@classmethod+，在pyqt中有槽的装饰器\verb+@pyqtSlot()+和\verb+@pyqtSlot(int, str)+等，第一个例子接下来你定义的槽只接受self这个参数，第二个例子接下来你定义的槽除了接受self参数外，还接受一个int类型参数和一个str类型参数。

\subsection{自定义装饰器}

\begin{tcbpython}[]
def print1(f):
    print('1',f)
    return f

@print1
def print3(c):
    print(c)

print3('c')#image print1(print3)('c')
\end{tcbpython}

比如上面的print1函数就做成了一个装饰器函数，后面的print3函数可以理解为 \verb+print3=print1(print3)+ 。

\subsection{多个装饰器}

\begin{tcbpython}[]
def print1(f):
    print('1',f)
    return f

def print2(f):
    print('2',f)
    return f

@print2
@print1
def print4(c):
    print(c)

print4('c')#image print2(print1(print4))(c)
\end{tcbpython}

类似上面的多个装饰器就可以简单理解为:
print4 = print2(print1(print4))

\subsection{装饰器带上参数}
在前面的例子中，我们就可以简单将装饰器函数理解为一个接受函数对象返回返回函数对象的函数，这很直观和简单，要有限考虑这样的模型。实际上装饰器也是可以带上自己的参数的，这需要通过什么函数的闭包结构才能完成，如下面的例子所示:

\begin{tcbpython}[]
def print1(f):
    print('1',f)
    return f

def print2(b):
    def test(f):
        print('2',f,b)
        return f
    return test

@print2('b')
@print1
def print4(c):
    print(c)

print4('c')#image print2(print1(print4))('b')(c)
\end{tcbpython}

所谓闭包结构简单来说就是函数里面套函数的结构。前面在介绍nolocal关键词的时候说道，如果函数里面的嵌套函数的某个变量加上声明关键词nolocal，那么（如果嵌套函数内没有定义该本地变量），则该变量名是对应嵌套函数外面的自由变量的（自由变量在函数生存期具有记忆能力）。

上面例子可以理解为\verb+print2(print1(print4))('b')(c)+ 这样一个过程。首先执行print1(print4)，然后返回print4，这很直观。然后表达式变为\verb+print2(print4)('b')(c)+，这里紧跟着的参数应该由python的装饰器解释程序做的，具体我还有点困惑&#x2026;&#x2026;，总之这里装饰器函数的额外的参数是由print2这个最外层的函数接收的，然后具体返回的test函数对象还是如同前面一样只接受目标函数对象。前面这样的\verb+print2(print1(print4))('b')(c)+ 描述可能并没很好地反映python的内部处理机制了，也许更好的描述是:

装饰器的参数是通过函数的闭包机制以自由变量的形式引入进去的。

\section{类方法}
还有一个装饰器有时也会用到， \verb+@classmethod+ ，叫什么类方法装饰器。其和前面的静态方法一样也可以不新建实例，而直接通过类来调用。其和静态方法的区别就是静态方法在调用的时候没有任何默认的第一参数，而类方法在调用的时候默认第一参数就是调用的那个类\footnote{参考了\href{http://stackoverflow.com/questions/136097/what-is-the-difference-between-staticmethod-and-classmethod-in-python}{这个网站}。}。

\begin{Verbatim}
class Test:
    @classmethod
    def hello(cls):
        print('from class:', cls, 'saying hello')

Test.hello()
\end{Verbatim}

\begin{Verbatim}
from class: <class '__main__.Test'> saying hello
\end{Verbatim}

\section{多重继承的顺序问题}
我们来看下面这个例子：

\begin{tcbpython}[]
class B1():x='B1'
class B2():x='B2'
class B3():x='B3'
class B(B1,B2,B3):x='B'
class A1():x='A1'
class A2():x='A2'
class A(A1,A2):x='A'
class D(B,A):x='D'
test=D()
print(test.x)
\end{tcbpython}

\begin{fig}{多重继承}
\caption{多重继承}
\label{fig:多重继承}
\end{fig}

你可以测试一下上面这个例子，首先当然结果是D自己的x被先查找，然后返回\emph{'D'}，如果你把类D的x定义语句换成pass，结果就是\emph{'B'}。这说明这里程序的逻辑是如果test实例找不到x，那么再找D，D找不到再接下来找D继承自的父类，首先是B，到目前为止，没什么新鲜事发生。

然后我们再把B的x赋值语句换成pass，这时的结果是\emph{'B1'}，也没什么好惊讶的。然后类似的一致操作下去，我们会发现python的值的查找顺序在这里是：D，B，B1，B2，B3，A，A1，A2。

于是我们可以总结道：恩，类的多重继承就是深度优先法则，先把子类或者子类的子类都查找完，确认没有值之后再继续从左到右的查找。

一般情况来说这么理解是没有问题的，但是在编程界多重继承中有个有名的问题——菱形难题。

\subsection{菱形难题}
参考资料： \href{http://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem}{维基百科菱形难题}

\begin{fig}{菱形难题}
\caption{菱形难题}
\label{fig:菱形难题}
\end{fig}

菱形难题即在如上的类的继承中，如果C和A都有同名属性x，那么D会调用谁的呢？读者测试下面的例子：

\begin{tcbpython}[]
class E():x='E'
class F():x='F'
class G():x='G'
class A(F,G):x='A'
class B(E,F):x='B'
class D(B,A):pass
test=D()
print(test.x)
\end{tcbpython}

此时运行结果到DBE都没有什么出奇的， 接下来要某是DBEF\footnote{E当然也检查过了，否则E有没有值是无法确认的。}，要某是DBEA，这里程序的结果是\emph{'A'}。这里的情况确实比较纠结，如果没有这个F作为菱形难题的交叉点，似前面的层次分明，那么简单的理解为深度优先即可，这里python3的选择是\emph{'A'}，不清楚为什么要这么选择。

我们再来看这个例子：

\begin{tcbpython}[]
class E():x='E'
class F():x='F'
class G():x='G'
class A(F,G):x='A'
class B(F,E):pass
class D(B,A):pass
test=D()
print(test.x)
\end{tcbpython}

此时结果是\emph{'A'}，连E都被跳过去了，变成了彻底的横向优先原则。

程序出现菱形难题之后，情况变得不可琢磨了。上面的三个情况

\begin{Verbatim}
D(B(B1 B2 B3) A(A1 A2)) → D B B1 B2 B3 A A1 A2

D(B(E F) A(F G)) → D B E A F G

D(B(F E) A(F G)) → D B A F E G
\end{Verbatim}

就是这样的，总之这是很冷门的领域了。。简单的理解就是深度搜索，类似flatten函数处理过，然后如果遇到某个子元在下一个平行级别的子元中也含有，那么本子元会被略过，做个记号，分叉跳过去跑到A那里，执行完那个子元之后，又会重新调到之前的操作点上。python怎么弄这么古怪的逻辑。。

\subsection{super()}
super是python3新加入的特性，按照官方文档，有两种用法：

第一种是如果是单继承的类的系统，super()这种形式就直接表示父类的意思。然后用super().什么什么的来引用父类的某个变量或方法，\uwave{值得一提的是原父类的self参量会默认加进去了}，详细请看下面的调试例子。

第二种是多重继承的，搜索顺序和多重继承的搜索顺序相同，也就是从左到右。请注意调试下面的例子，如果调用c.d就会返回错误，说明调用的是类A的构造函数。

\begin{tcbpython}[]
class A():
    def __init__(self,a):
        self.a=a

    def fun(self):
        print('fun')

    def fun2(self,what):
        print('fun',what)

class B():
    def __init__(self):
        self.d=5
    b=2
    def fun3(self):
        print('fun3')

class C(A,B):
    def __init__(self):
        super().__init__(3)
        super().fun()
        super().fun2('what')
        super().fun3()
        print(super().b)

c=C()
print(c.a,c.b)
\end{tcbpython}

\begin{Verbatim}
fun
fun what
fun3
2
3 2
\end{Verbatim}

其中A类定义的fun函数在写的函数上通常有个self参数，而\emph{super()}这种调用形式在意义上表示其的父类，同时默认第一个参数就是self。为了理解你可以和self做个比较，比如self.fun()就是调用的实例的fun函数，默认的第一个参数是self。使用super()在类的编写中引用本类的父类的属性和方法是很便捷的，自带支持类的多重继承功能。比如上面的例子中fun3能被调用是因为多重继承的机制在这里，所以它会逐个找父类。然后c.d会出错，因为这里初始化是用的A类的构造函数。

\section{给某个对象动态加载一个方法}
这里主要参考了\href{http://stackoverflow.com/questions/962962/python-changing-methods-and-attributes-at-runtime}{这个网页} 。

具体原理还是很简单的，那就是构建一个函数对象，然后将这个对象赋值给某个对象。但这里的函数对象如果要接受self参数的话，其作为类的方法还是需要一些特殊的处理的。

\begin{Verbatim}
class Test():
    pass

test = Test()

def hello(self):
    print("hello")

import types
test.hello = types.MethodType(hello,Test)

test.hello()
\end{Verbatim}

上面的types.MethodType是用来构建一个类的方法的，其第一个参数是具体的函数对象，第二个参数是对应的类或实例。

然后上面的例子继续优化就是如下的形式:

\begin{Verbatim}
import types

class Test():
    @classmethod
    def removeVariable(cls,name):
        return delattr(cls,name)

    @classmethod
    def addMethod(cls,func):
        return setattr(cls,func.__name__,types.MethodType(func,cls))

def hello(self):
    print("hello")

test = Test()

Test.addMethod(hello)

test.hello()
\end{Verbatim}

你看到了这里的addMethod是作用于本类的，当然你也可以选择作用于本实例:

\begin{Verbatim}
import types

class Test():
    @classmethod
    def removeVariable(cls,name):
        return delattr(cls,name)

    @classmethod
    def addMethod(cls,func):
        return setattr(cls,func.__name__,types.MethodType(func,cls))

    def addMethod2(self,func):
        return setattr(self,func.__name__,types.MethodType(func,self))

def hello(self):
    print("hello")

test = Test()

test.addMethod2(hello)

test.hello()
\end{Verbatim}

这样这个函数就只加在本实例上面了，这用处不太大。

# 深入理解python3的迭代<a id="orgheadline18"></a>

\label{sec:深入理解python3的迭代}
在python中一般复杂的代码运算效率就会低一点，如果完成类似的工作但你可以更简单的语句那么运算效率就会高一点。当然这只是python的一个设计理念，并不尽然，但确实很有意思。

程序结构中最有用的就是多个操作的重复，其中有迭代和递归还有一般的循环语句。递归函式感觉对于某些特殊的问题很有用，然后一般基于数据结构的不是特别复杂的操作重复用迭代语句即可，最后才考虑一般循环语句。

迭代语句中for语句运算效率最低，然后是map函数（不尽然），然后是列表解析。所以我们在处理问题的时候最pythonic的风格，运算效率最高的就是列表解析了，如果一个问题能够用列表解析解决那么就用列表解析解决，因为python的设计者的很多优化工作都是针对迭代操作进行的，然后python3进一步深化了迭代思想，最后python中的迭代是用c语言来实现的（你懂的）。

可是让我们反思一下为什么列表解析在问题处理的时候如此通用？比如说range函数或者文件对象或者列表字符串等等，他们都可以称之为可迭代对象。可迭代对象有内置方法\verb+\_\_next\_\_+这个我们之前有所谈及，可迭代对象最大的特色就是有一系列的元素，然后这一系列的元素可以通过上面的内置方法逐个调出来，而列表解析就是对这些调出来的元素进行了某个表达式操作，然后将其收集起来。这是什么？我们看下面这张图片：

\begin{fig}{列表解析}
\caption{列表解析}
\label{fig:列表解析}
\end{fig}

这张图片告诉我们列表解析和数学上所谓的集合还有函数的定义非常的类似，可迭代对象就好像是一个集合（有顺序或者没顺序都行），然后这些集合中的所有元素经过了某个操作，这个操作似乎就是我们数学中定义的函数，然后加上过滤条件，某些元素不参加运算，这样就生成了第二个可迭代对象（一般是列表也可以是字典什么的。）

有一个哲学上的假定，那就是我们的世界一切问题都可以用数学来描述，而一些数学问题都可以用函数即如上的信息操作过滤流来描述之。当然这不尽然，但我们可以看到列表解析在一般问题处理上是很通用的思想。

不过我们看到有限的元素的集合问题适合用迭代，但无限元素的集合问题也许用递归或者循环更适合一些。然后我们又想到集合的描述分为列举描述（有限个元素的列举）和定义描述。比如说1<x<10，x属于整数，这就定义了一个集合。那么我们就想到python存在这样的通过描述而不是列举（如列表一样）的集合吗？range函数似乎就是为了这样的目的而生的，比如说range(10)就定义了[0,10)这一系列的整数集合，range函数生成一个range对象，range对象是一个可迭代对象，我们可以把它看作可迭代对象中的描述集合类型吧。这时我们就问了，既然0<=x<10这样的整数集合可以通过描述来实现，那么更加复杂的函数描述可不可以实现呢？我们可不可以建立更加复杂的类似range对象的描述性可迭代对象呢？

\section{生成器函数}
一般函数的定义使用return语句，如果使用yield语句，我们可以构建出一个生成器函数，

\begin{Verbatim}
>>> def test(x):
...    for i in range(x):
...        yield 2*i+ 1
... 
>>> test(3)
<generator object test at 0xb704348c>
>>> [x for x in test(3)]
[1, 3, 5]
>>> [x for x in test(5)]
[1, 3, 5, 7, 9]
\end{Verbatim}

这个test函数叫做什么生成器函数，返回的是什么generator object，生成器对象？anyway，通过yield这样的形式定义出来的生成器函数返回了一个生成器对象和range对象类似，都是描述性可迭代对象，里面的元素并不立即展开，而是请求一次运算一次，所以这种编程风格对内存压力很小，主要适合那些迭代元素特别多的时候的情况吧。

上面的test函数我们就可以简单理解为2x+1，其中0<=x<n（赋的值）。

下面给出一个问题作为练习：描述素数的生成器函数。
这是网上流行的素数检验函数，效率还是比较高的了。

\begin{tcbpython}
def isprime(n):
    if n ==2:
        return True
    #按位与1，前面一定都是0个位数如果是1则
    #是奇数则返回1则真则假，如果是偶数则返回
    #0则假则真
    elif n<2 or not n & 1:
        return False
    #埃拉托斯特尼筛法...
#查一个正整数N是否为素数，最简单的方法就是试除法，
#将该数N用小于等于N**0.5的所有素数去试除，
#若均无法整除，则N为素数
    for x in range(3,int(n**0.5)+1,2):
        if n % x == 0:
            return False
    return True
\end{tcbpython}

然后我们给出两种形式的素数生成器函数，其中prime2的意思是范围到（to）那里。而prime(n)的意思是到第几个素数。我们知道生成器函数是一种惰性求值运算，然后yield每迭代一次函数运算一次（即产生一次yield），但这种机制还是让我觉得好神奇。

\begin{tcbpython}
def prime2(n):
    for x in range(n):
        if isprime(x):
            yield x

def prime(n):
    i=0
    x=1
    while i<n:
        if isprime(x):
            i +=1
            yield x
        x +=1
\end{tcbpython}

在加载这些函数之后我们可以做一些检验：

\begin{Verbatim}
>>> isprime(479)
True
>>> [x for x in prime2(100)]
[2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, ........]
>>> [x for x in prime2(1000) if 100< x < 200]
[101, 103, 107, 109, 113, 127, 131, 137, 139, 149, .......]
>>> len([x for x in prime2(10000) if -1 < x < 3572])
500
>>> [x for x in prime(1)]
[2]
>>> [x for x in prime(2)]
[2, 3]
\end{Verbatim}

\section{map和filter函数}
按照之前的迭代模式的描述，虽然使用常见的列表解析格式(for 语句)就可以完成对某个集合中各个元素的操作或者过滤，不过python中还有另外两个函数来实现类似的功能，map对应对集合中各个元素进行某个函数操作（可以接受lambda函式），而filter则实现如上所述的过滤功能。然后值得一提的是python3之后map函数和filter函数返回都是一个可迭代对象而不是列表，和range函数等其他可迭代对象一样可用于列表解析结构。

\subsection{map函数}
这里列出一些例子，具体编程还是先考虑列表解析模式，可能会在某些情况下需要用到map函数？

\begin{Verbatim}
>>> map(abs, [-2,-1,0,1,2])
<map object at 0xb707dccc>
>>> [x for x in map(abs, [-2,-1,0,1,2])]
[2, 1, 0, 1, 2]
>>> [x for x in map(lambda x : x+2, [-2,-1,0,1,2])]
[0, 1, 2, 3, 4]
\end{Verbatim}

map函数还可以接受两个可迭代对象的协作参数模式，这个学过lisp语言的会觉得很眼熟，不过这里按照我们的理解也是很便捷的。具体就是第一个可迭代对象取出一个元素作为map的函数的第一个参数，然后第二个可迭代对象取出第二个参数，然后经过函数运算，得到一个结果，这个结果如果不列表解析的话就是一个map对象（可迭代对象），然后展开以此类推。值得一提的是两个可迭代对象的深度由\uwave{最短}的那个决定，请看下面的例子：

\begin{Verbatim}
>>> [x for x in map(lambda x,y : x+y, [-2,-1,0,1,2],[-2,-1,0,1,2])]
[-4, -2, 0, 2, 4]
>>> [x for x in map(lambda x,y : x+y, [-2,-1,0,1,2],[-2,-1,0,1])]
[-4, -2, 0, 2]
\end{Verbatim}

\subsection{filter函数}
同样和上面的谈及的类似，filter函数过滤一个可迭代对象然后产生一个可迭代对象。类似的功能可以用列表解析的后的if语句来实现。前面谈到map函数的时候提及一般还是优先使用列表解析模式，但filter函数这里有点不同，因为列表解析后面跟个if可能有时会让人困惑，这时推荐还是用filter函数来进行可迭代对象的过滤操作。

filter函数的基本逻辑是只有return True（用lambda表达式就是这个表达式的值为真，具体请参看python的逻辑小知识和布尔值的一些规则\ref{sec:布尔值}）的时候元素才被收集起来，或者说是过滤出来。这里强调True是因为如果你的函数没有return值那么默认的是return None，这个时候元素也是不会过滤出来的。

请参看下面的例子来理解：\sidenote{这里位运算与就是控制个位数是1那么就是奇数，这种方式更加的节省计算。}

\begin{Verbatim}
>>> [x for x in filter(lambda x:x&1,[1,2,3,5,9,10,155,-20,-25])]
[1, 3, 5, 9, 155, -25]
>>> [x for x in filter(lambda x:not x&1,[1,2,3,5,9,10,155,-20,-25])]
[2, 10, -20]
\end{Verbatim}

当然你也可以传统的编写函数：

\begin{Verbatim}
>>> def even(n):
...    if n % 2 ==0:
...         return True

>>> [x for x in filter(even,[1,2,3,5,9,10,155,-25])]
[2, 10]
\end{Verbatim}

\subsection{zip函数}
这里就顺便把zip函数也一起提了，zip函数同样返回一个可迭代对象，它接受任意数目的可迭代对象，然后逐个取出可迭代对象元素构成一个元组成为自己的一个元素（待迭代出来）。和map函数类似迭代深度由\uwave{最短}的那个可迭代对象决定。

\begin{Verbatim}
>>> zip(['a','b','c'],[1,2,3,4])
<zip object at 0xb7055e6c>
>>> [x for x in zip(['a','b','c'],[1,2,3,4])]
[('a', 1), ('b', 2), ('c', 3)]
>>> list(zip(['a','b','c'],[1,2,3,4]))
[('a', 1), ('b', 2), ('c', 3)]
>>> dict(zip(['a','b','c'],[1,2,3,4]))
{'c': 3, 'b': 2, 'a': 1}
\end{Verbatim}

\subsubsection{字典到列表}
这个例子似乎使用价值不大，只是说明zip函数接受任意数目参数的情况。y.items()解包之后是4个参数传递给zip函数，而zip函数的封装逻辑就是如果有人问我，我就把你们这些迭代对象每个取出一个元素，然后用元组包装之后返回。

\begin{tcbpython}[]
x1 = ['a','b','c','e']
x2 = [1,2,3,4]
y = dict(zip(x1,x2))
print('列表到字典：',y)
new_x1,new_x2 = zip(*y.items())
print(new_x1,new_x2)
\end{tcbpython}

\begin{Verbatim}
列表到字典： {'b': 2, 'c': 3, 'a': 1, 'e': 4}
('b', 'c', 'a', 'e') (2, 3, 1, 4)
\end{Verbatim}

这个例子如果到更加复杂的情况，我们可以跳过字典形式，来个数据映射对：

\begin{Verbatim}
>>> x1 = ['a','b','c','e']
>>> x2 = ['red','yellow','red','blue']
>>> x3 = [1,2,3,4]
>>> list(zip(x1,x2,x3))
[('a', 'red', 1), ('b', 'yellow', 2), ('c', 'red', 3), ('e', 'blue', 4)]
>>> new_x1,new_x2,new_x3 = zip(*list(zip(x1,x2,x3)))
>>> new_x1
('a', 'b', 'c', 'e')
>>> new_x2
('red', 'yellow', 'red', 'blue')
>>> new_x3
(1, 2, 3, 4)
\end{Verbatim}

当然对于多属性数据问题一般还是推荐使用类来处理，不过某些情况下可能不需要使用类，就这样简单处理之。

值得一提的是这种数据存储形式和sql存储是一致的，而且不知道你们注意到没有，这似乎实现了矩阵的转置功能。

# 模块包<a id="orgheadline19"></a>

多个模块py文件组成一个多文件夹目录的整体就是一个模块包。

模块包这部分知识是我们理解前人编写的各个有用的模块包的基础，同时以后我们自己要编写大型的项目也是一个人编写一个模块，一个模块对应一个任务或者一个功能的形式展开的，然后多个模块合并成一个大型的模块包。以前我们都是编写的不超过一百行的小python代码，不过就是对于大型的项目也不意味着我们要找一个大型的编辑器，然后一写就是上万行。在模块包的合理布局下，我们完全还可以轻松的一次编写也就那么一两百行的小代码，最后各个模块组合起来，就是一个宏大的系统了。

\section{\\\_{}\\\_{}init\\\_{}\\\_{}文件}
well，模块包和简单的模块在管理上多出来的唯一的一个内容就是你需要在每个文件夹里面加一个\verb+\_\_init\_\_.py+文件，文件的内容就是空白都没关系，但必须要有。

现在我们新建一个文件夹，\verb+mymodule+。然后进入mymodule文件夹，新建一个空白文件\verb+\_\_init\_\_.py+，然后新建一个文件\verb+mymod.py+，然后里面定义了一个简单的myfun函数，没什么意义，就是打印了一段信息。然后我们在当前目录下进入python3的eval模式就可以开始测试你的这个新模块包了，因为sys.path是默认自带搜索当前工作目录的。

你的模块包必须要有一个\verb+\_\_init\_\_.py+文件，这个文件实际上就是暗指的你的这个模块，比如说这里是mymodule。不管你是import还是使用from语句，只要调用mymodule，你的\verb+\_\_init\_\_.py+都将执行一次。

你的模块包如果还有其他py文件\footnote{这个py文件就是子模块的概念。}，那么这个文件在你的\verb+\_\_init\_\_.py+文件没有任何配置的情况下只能通过：

\begin{tcbpython}[]
import mymodule.mymod
\end{tcbpython}

这样的语句来引入进来。

在一般情况下，\verb+\_\_init\_\_.py+文件是空白的或者里面填上一些类和函数来表示mymodule自身的配置，这满足了一般的需求了。然后模块包里面的py文件在使用时都用mymodule.mymod来引用进来。

\verb+\_\_init\_\_.py+文件如果是空白我们不用关心它到底起了什么作用，但如果它不是空白，里面有一些代码，比如定义了一些函数和类，因为每个模块包下的\verb+\_\_init\_\_.py+在python首次导入时都会执行一次这个文件内部的代码。所以当你import mymodule之后，mymodule里面的函数和类就可以通过mymodule.what来使用了。

\subsection{如果里面有import语句}
如果import的是外部的模块方面本文件某些函数或类的调用这自不必说。这里讨论的是这种情况，有的时候你的模块包的某些子模块，你不希望import mymodule之后还需要import mymodule.mymod才能使用mymod.py文件里面的内容，你希望马上就能够使用，那么你需要在\verb+\_\_init\_\_.py+文件里面使用import语句了。

比如在这里\verb+\_\_init\_\_.py+文件加上一句话：

\begin{tcbpython}
import mymodule.mymod
\end{tcbpython}

这样在简单的import mymodule之后，你就可以直接使用mymodule.mymod.myfun来引用mymod.py文件里面的myfun函数了，而之前还需要额外的执行import mymodule.mymod命令一次。

\subsection{如果里面有from语句}
前面谈到from语句的前面和import语句是没有区别的，除了额外的引入变量名操作。在这里就是你厌倦了mymodule.mymod.myfun这类常常的引用的方式，你就想简单点就是mymodule.myfun，那么你可以在\verb+\_\_init\_\_.py+文件中加上如下语句：

\begin{tcbpython}[]
from mymodule.mymod import myfun
\end{tcbpython}

这样就把myfun这个变量名引入到mymodule（也就是\verb+\_\_init\_\_.py+文件）里面去了。此时mymodule点的下面引用就多了一些函数和类的定义了，在你简单import mymodule之后。

读者应该猜到了，如果现在你在外面eval模式下使用from mymodule import \*，那么就可以直接调用myfun命令了。

\subsection{\\\_{}\\\_{}all\\\_{}\\\_{}变量}
\verb+\_\_init\_\_.py+文件还有一个高级功能，这个高级功能多少取代了前面谈论的一切，是一种新的管理模式。

如果外围eval调用使用的import mymodule语句，那么\\\_{}\\\_{}all\\\_{}\\\_{}变量将不会被读取。只有在外围eval调用使用from mymodule import \*这样的语句情况下，\\\_{}\\\_{}all\\\_{}\\\_{}变量才会被读取。同时要特别提醒的是，如果\verb+\_\_init\_\_.py+文件里面没有\\\_{}\\\_{}all\\\_{}\\\_{}变量，那么情况就是我们上面讨论的，如果有\\\_{}\\\_{}all\\\_{}\\\_{}变量，并且使用了from mymodule import \*语句，那么\verb+\_\_init\_\_.py+文件里面的其他import和from语句都将失效（其他语句还是有效的）。

表面上看\\\_{}\\\_{}all\\\_{}\\\_{}变量的引入似乎使得事情变得混乱不堪了，不过我们外围可以使用这样的语法：

\begin{tcbpython}[]
import mymodule
from mymodule import *
\end{tcbpython}

这样from\*语句就负责\\\_{}\\\_{}all\\\_{}\\\_{}变量部分，import mymodule则负责其他import和from语句的管理，两者并无冲突。此时mymodule，mymodule.mymod和mymod2都是可以引用的。

但是在这里最大的问题是当你使用from mymodule import \*之后，mymod2被提到和mymodule同等级的状态，这多少不够美观，我们宁愿用mymodule.myfun或者mymodule.mymod2这样的形式，单独使用mymod2多少有点主次不分了。总的说来不推荐使用\\\_{}\\\_{}all\\\_{}\\\_{}变量，当然也不推荐使用from mymodule import \*这样的语法。

\emph{例外}：在保证mymodule这个主入口不被侵犯的情况下，某些子模块的子模块（这里我们在mymodule文件夹里面新建一个mymod3文件夹，然后新建文件\verb+\_\_init\_\_.py+，这个\verb+\_\_init\_\_.py+里面通过\verb+\_\_all\_\_+代入了mymod4，然后新建mymod4.py，里面定义一个简单地打印函数。）的情况可以使用。

如上面说明的，在我们对项目模块包内部文档管理更加美观的要求下，现在在import mymodule之后，模块的模块的模块&#x2026;&#x2026;都可以通过\\\_{}\\\_{}all\\\_{}\\\_{}变量来优化，从而外围可以直接通过mymodule.mymod4这样的二级引用格式来引用，如果我们不使用这种技术，按照常规手段，那么我们需要mymodule.mymod3.mymod4这样的格式来引用，这太过于复杂了。

\section{模块中的帮助信息}
well，大家都知道，我们编写的模块主要是给别人看得，给别人用的，所以多写点帮助信息吧，这个没人嫌你写得多的。其他\\#{}下的注释就不用说了，这里主要讲一下其他的帮助信息。

还是跟着上面的例子来：

\begin{tcbpython}
"""mymod.py
这是一个测试模块"""

def myfun():
    """myfun函数
    用于打印测试"""

    print('myfun is found')
\end{tcbpython}

\begin{tcbpython}
"""mymodule
我在mymodule文件夹的__init__.py文件里面"""

print('mymodule already import')

from mymodule import mymod
__all__ = ['mymod']
\end{tcbpython}

然后是测试代码：

\begin{tcbpython}
import os,sys
sys.path.append(os.environ['HOME']+'/pymf')
from pyconfig import *

import  mymodule
\end{tcbpython}

具体测试查看情况如下所示，其中help命令显示的内容就不粘贴在这里了，请读者自己查看之，还是很有意思的。

\begin{Verbatim}
mymodule already import
>>> dir(mymodule)
['__all__', '__builtins__', '__cached__', '__doc__', ......]
>>> dir(mymodule.mymod)
['__builtins__', '__cached__', '__doc__', '__file__', ......, 'myfun']
>>> print(mymodule.__doc__)
mymodule
我在mymodule文件夹的__init__.py文件里面
>>> print(mymodule.mymod.__doc__)
mymod.py
这是一个测试模块
>>> print(mymodule.mymod.myfun.__doc__)
myfun函数
    用于打印测试
>>> help(mymodule)

>>> help(mymodule.mymod)

>>> help(mymodule.mymod.myfun)

\end{Verbatim}

类和模块的这些文档信息都存放在\verb+\_\_doc\_\_+变量里面的。

# 文件处理高级知识<a id="orgheadline21"></a>

接下来的例子如果涉及到文件的请自己随便创建一个对应文件名的文件，内容随意了。

\section{一行行的操作}
因为文件对象本身是可迭代的，我们简单迭代文件对象就可以对文件的一行行内容进行一些操作。比如：

\begin{tcbpython}
f = open('removeduplicate.py')

for line in f:
    print(line,end='')
\end{tcbpython}

这个代码就将打印这个文件，其中end=''的意思是取消\verb+\n+，因为原来的行里面已经有\verb+\n+了。

然后代码稍作修改就可以在每一行之前加上>>>这个符号了。 

\begin{tcbpython}
f = open('removeduplicate.py')

for line in f:
    print('>>>',line,end='')
\end{tcbpython}

什么？这个输出只是在终端，没有到某个文件里面去，行，加上file参数。然后代码变成如下：

\begin{tcbpython}
import sys

f = open('removeduplicate.py')
pyout=open(sys.argv[1] ,"w")

for line in f:
    print('>>>',line,end='',file=pyout)

pyout.close()
f.close()
\end{tcbpython}

这样我们就制作了一个小python脚本，接受一个文件名然后输出这个文件，这个文件的内容就是之前我们在终端中看到的。

\section{整个文件的列表解析}
python的列表解析（迭代）效率是很高的，我们应该多用列表解析模式。

\subsection{readlines方法}
文件对象有一个readlines方法，能够一次性把整个文件的所有行字符串装入到一个列表中。然后我们再对这个列表进行解析操作就可以直接对整个文件的内容做出一些修改了。不过不推荐使用readlines方法了，这样将整个文件装入内存的方法具有内存爆炸风险，而迭代版本更好一点。

\subsection{文本所有某个单词的替换}
这里举一个例子，将removeduplicate.py文件接受进来，然后进行列表解析，将文本中的newlist全部都替换为list2。

\begin{tcbpython}
import sys

pyout=open(sys.argv[1] ,"w")

print(''.join([line.replace('newlist','list2') 
for line in open('removeduplicate.py')]),file=pyout)

pyout.close()
\end{tcbpython}

我们可以看到这种列表解析风格代码更加具有python风格和更加的简洁同时功能是异常的强大的。

从这里起我们看到如果需要更加复杂的文本处理技巧就需要学习正则表达式和re模块了，请参见re模块这一小节\ref{sec:re模块}。

## 文件对象的seek方法<a id="orgheadline20"></a>

文件对象的seek方法就三种用法，参看了 [这个网页](http://www.cnblogs.com/paisen/p/3492768.html) 。

-   f.seek(p,0) 就是从文件头开始移动p个字节。注意移动之后如果你执行 `readline()` 那么返回的字符串是从当前位置到行尾。第二个参数默认是这个行为。

-   f.seek(p,1) 基于当前位置再移动p个字节。

-   f.seek(0,2) 这个就是移动到文件尾。然后如果打开默认是 `b` 的话，那么还是可以用某个p负值来表示从文件尾向前移动了多少个字节的。

# 与c语言或c++语言编写的模块集成<a id="orgheadline22"></a>

ref <https://github.com/yasoob/intermediatePython/blob/master/python_c_extension.rst>

\section{安装和配置}
\subsection{通过apt安装}

\begin{tcbbash}[]
sudo apt-get install swig
\end{tcbbash}

在Ubuntu14.04这将安装swig2.0版本。

\subsection{从github下载最新版安装}
从github上下载最新版本：

\begin{tcbbash}[]
git  clone  https://github.com/swig/swig
\end{tcbbash}

安装需要的前提软件：

\begin{tcbbash}[]
sudo apt-get install autotools-dev
sudo apt-get install automake
sudo apt-get install byacc
sudo apt-get install yodl
\end{tcbbash}

安装：

\begin{tcbbash}[]
./configure
make
sudo make install
\end{tcbbash}

\subsection{安装后的配置}
需要确认安装了python3-dev，好支持\verb+#include python.h+：

\begin{tcbbash}[]
sudo apt-get install python3-dev
\end{tcbbash}

\section{beginning}
\subsection{手工编译}
找到源码的[Examples]→[python]→[simple]文件夹。简单的编译过程如下：

\begin{tcbbash}[]
swig -python example.i

gcc -fpic -c example.c example_wrap.c -I/usr/include/python3.4m

gcc -shared example.o example_wrap.o -o _example.so
\end{tcbbash}

可以通过如下命令查看具体\textbf{-I}的引用地址：

\begin{tcbbash}[]
python3-config --includes
\end{tcbbash}

简单的使用如下：

\begin{Verbatim}
>>> import example
>>> example.gcd(100,25)
25
>>> example.cvar.Foo
3.0
>>> 
\end{Verbatim}

\subsection{通过setuptools安装}
更加简便的处理方式是用setuptools模块来自动处理这一些，包括build到安装egg文件。

\begin{tcbpython}[]
from setuptools import setup ,Extension

setup(
    name = 'example',
    version = '0.01',
    ext_modules = [Extension('_example', ['example.i','example.c'],
    swig_opts=['-modern', '-I../include'])],
    py_modules = ['example']
)
\end{tcbpython}

上面的swig\\\_{}opts推荐加上。

编写自己的一个函数：

\begin{tcbcode}{c}
#include <stdio.h>
void hello(){
    printf("hello, world\n");
}
\end{tcbcode}

修改example.i文件：

\begin{tcbcode}{c}
%module example

%inline %{
extern int    gcd(int x, int y);
extern double Foo;
extern void hello();
%}
\end{tcbcode}

重新用setuptools安装一下。

\begin{Verbatim}
>>> import example
>>> example.hello()
hello, world
\end{Verbatim}

# PEP8推荐的风格<a id="orgheadline23"></a>

[参考文档](http://intermediate-and-advanced-software-carpentry.readthedocs.org/en/latest/structuring-python.html) 
use four spaces (NOT a tab) for each indentation level;
use lowercase, \_-separated names for module and function names, e.g.
my\_module;

use CapsWord style to name classes, e.g. MySpecialClass;
use ‘\_’-prefixed names to indicate a “private” variable that should
not be used outside this module, , e.g. \_some\_private\_variable;

all and any are two new functions in Python that work with iterables (e.g. lists, generators, etc.). any returns True if any element of the iterable is True (and False otherwise); all returns True if all elements of the iterable are True (and False otherwise).
>>> x = [ True, False ]
>>> print any(x)
True
>>> print all(x)
False
>>> y = [ True, True ]
>>> print any(x)
True
>>> print all(x)
False

<http://intermediate-and-advanced-software-carpentry.readthedocs.org/en/latest/pyparsing-presentation.html>