# 谷歌python风格指南

## 前言
几个工具[pylint](http://pylint.pycqa.org/en/latest/)，[yapf](https://github.com/google/yapf/)，[pytype](https://github.com/google/pytype)
## 1 背景
python是谷歌使用的主要动态语言。该风格指南指出了python编程中一些该做的和不该做的行为。为了帮助读者正确地格式化代码，谷歌创建了[settings file for Vim](https://github.com/whyAtGh/styleguide/blob/gh-pages/google_python_style.vim)。对Emacs来说，默认配置没有问题。许多团队使用自动化格式工具[yapf](https://github.com/google/yapf/)来避免代码格式上的争议。

## 2 python语言规则
### 2.1 Lint
使用`pylint`检查你的代码。
#### 2.1.1 定义
`pylint`是用来检查python源代码bug和格式问题的工具。对于不那么动态的语言（C或C++），这些bug通常由编译器来捕获。由于python的动态特性，许多警告可能不正确；然而，这种假警告的情况很少出现。
#### 2.1.2 优点
可以捕获容易忽视的错误，比如拼写错误、使用未赋值的变量等等。
#### 2.1.3 缺点
`pylint`并不完美。想要充分利用它，我们有时需要：
- 围绕它来写代码
- 禁用它的警告
- 优化它
#### 2.1.4 建议
确保使用`pylint`检查你的代码。
禁用不恰当的警告，以免忽略其他问题。为了禁用警告，你可以设置一行注释:

  ```dict = 'something awful'  # Bad Idea... pylint: disable=redefined-builtin```
  
pylint警告都是用符号名称标识(`empty-docstring`)，谷歌特定的警告带有`g-`前缀。
如果从符号名称中看不出去除警告的理由，那就要做出解释。
以这种方式禁用警告有一种优点，就是我们可以轻易找到该警告并访问它。
你可以用一下方式获取`pylint`警告的列表：

```
pylint --list-msgs
```

想得到某一条特定消息(比如C6409)的详细信息：

```
pylint --help-msg=C6409
```

优先使用`pylint:disable`，而不是弃用的旧格式`pylint:disable-msg`。
对于未使用的参数的警告，我们可以通过在函数一开始删除这些变量来抑制。一定要注释一下为什么要删除这些变量，“Unused”就足够了，例如：

```python
def viking_cafe_order(spam, beans, eggs=None):
    del beans, eggs  # Unused by vikings.
    return spam + spam + spam
```

其他抑制这种警告的通用格式包括使用‘`_`’作为未使用参数的标识符、在参数前面加‘`unused`’前缀或者将它们赋值给‘`_`’。这些格式是允许的，但是已经不再推荐使用。前两种方式中断通过变量名传参的调用，最后一种方式并不强迫参数真的不能使用。
### 2.2 Imports(引用)
只能对包或者模块使用`import`，而不能对类或者函数使用。
#### 2.2.1 定义
模块间共享代码的复用机制。
#### 2.2.2 优点
名称空间的管理约定非常简单，每一个标识符的源都以一致的方式给出。`x.Obj`表示`Obj`对象定义在模块`x`中。
#### 2.2.3 缺点
模块名字依然会发生冲突，许多模块的名字很长，不方便。
#### 2.2.4 建议
- 使用`import x`来引用包和模块
- 使用`from x import y`，其中`x`是包前缀，`y`是不带前缀的模块名
- 使用`from x import y as z`，如果引入的两个包都叫`x`或者`y`的名字很长
- 使用`import y as z`，只有当`z`是标准缩写时，例如`numpy`缩写为`nm`
举个例子，模块`sound.effects.echo`可以用下面的方式引用：

```python
from sound.effects import echo
...
echo.EchoFilter(input, output, delay=0.7, atten=4)
```

引用包时不要使用相对名称，即使模块在同一个包中，也要使用完整的包名称。这可以避免两次引入同一个包。
注意引用[typing模块](#typing-imports)时有明确的豁免。
### 2.3 Packages(包)
使用模块的完整路径名位置导入每个模块。
#### 2.3.1 优点
避免模块名中的冲突。更容易找到模块。
#### 2.3.2 缺点
部署代码更加困难，因为你必须复制包的层次结构。
#### 2.3.3 建议
所有新代码都应该按其完整的包名导入每个模块。

```python
# Reference in code with complete name.
import absl.flags

# Reference in code with just module name (preferred).
from absl import flags
```

### 2.4 Exceptions(异常)
异常必须要小心使用。
#### 2.4.1 定义
异常是一种工具，它可以中断一点代码的正常控制流来处理错误或其他异常情况。
#### 2.4.2 优点
正常运行的代码控制流不会被错误处理代码干扰。当特定条件发生时，它也允许控制流跳过多层框架。例如，在N层嵌套函数中从某一步返回，而不必执行错误代码。
#### 2.4.3 缺点
可能会让控制流显得比较混乱，调用库时容易错过出错的情况。
#### 2.4.4 建议
异常必须遵循某些条件：
- 以下面的方式抛出异常：`raise MyError('Error message')`或者`raise MyError()`，不要使用两个参数的形式`raise MyError, 'Error message'`。
- 尽量使用内置的异常类。例如，如果在期望正数的地方传入了负数，就抛出`ValueError`异常。不要使用`assert`语句来验证公共API的参数值，`assert`用来确保内部的正确性，而不是用来强制正确使用什么，也不是用来指示意外事件的发生。如果后一种情况期望抛出异常，那就使用raise语句。例如，

```python
Yes:
  def ConnectToNextPort(self, minimum):
    """Connects to the next available port.  Returns the new minimum port."""
    if minimum <= 1024:
      raise ValueError('Minimum port must be greater than 1024.')
    port = self._FindNextOpenPort(minimum)
    if not port:
      raise ConnectionError('Could not connect to service on %d or higher.' % (minimum,))
    assert port >= minimum, 'Unexpected port %d when minimum was %d.' % (port, minimum)
    return port
```

```python
No:
  def ConnectToNextPort(self, minimum):
    """Connects to the next available port.  Returns the new minimum port."""
    assert minimum > 1024, 'Minimum port must be greater than 1024.'
    port = self._FindNextOpenPort(minimum)
    assert port is not None
    return port
```

- 类库或包可能会定义自己的异常类。这种情况下，自定义的异常类必须继承已有的异常类。异常名称应当以`Error`结尾，而且不应太绕口（`foo.FooError`）.`注：原文是stutter`。
- 永远不要使用捕获所有异常的`except:`语句，也不要捕获`Exception`和`StandardError`，除非你打算重新出发该异常或者在当前线程的最外层（此时记得打印一条错误信息）。Python在这方面非常宽容，`except:`会捕获拼写错误、sys.exit()调用、 Ctrl+C中断、单元测试失败以及所有你不想捕获的其它异常。
- `try`/`except`块中的代码要尽量少，`try`块代码越多，就越容易在你不希望抛出异常的地方抛出异常。这种情况下，`try`/`except`块隐藏了真正的错误。
- 使用`finally`子句执行无论`try`块是否抛出异常都会执行的代码，这对收尾工作很有用，比如关闭文件。
- 捕获异常时，使用`as`，不要用逗号。例如，

```python
try:
  raise Error
except Error as error:
  pass
```

### 2.5 Global variables(全局变量)
避免使用全局变量。
#### 2.5.1 定义
在模块级别声明的变量，或者作为类属性的变量。
#### 2.5.2 优点
偶尔很有用。
#### 2.5.3 缺点
有可能在引入包的时候改变模块的行为。因为模块在第一次被引入的时候，对全局变量的赋值就完成了。
#### 2.5.4 建议
避免使用全局变量。
模块级别的常量虽然本质上也是变量，但它们是被允许并鼓励使用的。例如，
`MAX_HOLY_HANDGRENADE_COUNT = 3`，常量必须用大写字母命名，可以带有下划线。参考下面的[命名](#naming)章节。
如果需要使用全局变量，那它们应该在模块级别声明，并且通过变量名头部添加`_`来设置为仅模块内部可见。外部的访问必须通过模块级别的共有函数来实现，参考下面的[命名](#naming)章节。
### 2.6 Nested/Local/Inner Classes and Functions（嵌套/局部/内部类和函数）
嵌套局部函数和类在闭包时很有用。内部类很有用。
#### 2.6.1 定义
一个类可以在一个方法、函数或类中定义。函数可以定义在方法或函数中。嵌套函数对封闭作用域中定义的变量拥有只读权限。
#### 2.6.2 优点
在很有限的作用域内使用的通用类和函数允许被定义。
#### 2.6.3 缺点
嵌套或局部类的实例不能被直接测试。嵌套使得外层函数更长，可读性更差。
#### 2.6.4 建议
考虑下述限制，使用它们还不错：除非闭包使用，其他情况下避免使用；不要为了不让模块调用者使用而嵌套函数，而是在模块级将它的名字加上前缀`_`，，以便在测试时仍然可以访问。
### 2.7 Comprehensions & Generator Expressions（推导式 & 生成器表达式）
在简单的情况下可以使用
#### 2.7.1 定义
在不求助于传统的循环、`map()`、`filter()`、`lambda`的情况下，列表、字典和集合推导式以及生成器表达式提供了一种简单高效的方式来生成容器类型和迭代器。
#### 2.7.2 优点
简单的推导式比字典、列表或集合生成方式更加简洁。生成器表达式可以非常高效，因为它避免了生成整个列表。
#### 2.7.3 缺点
复杂的推导式和生成器表达式会难以阅读。
#### 2.7.4 建议
简单的情况下可以使用。每一部分必须适应一行的长度：mapping表达式、`for`子句、filter表达式。多层`for`子句或filter表达式是不允许的。当情况更复杂的时候使用循环。

```python
Yes:
  result = []
  for x in range(10):
      for y in range(5):
          if x * y > 10:
              result.append((x, y))

  for x in xrange(5):
      for y in xrange(5):
          if x != y:
              for z in xrange(5):
                  if y != z:
                      yield (x, y, z)

  return ((x, complicated_transform(x))
          for x in long_generator_function(parameter)
          if x is not None)

  squares = [x * x for x in range(10)]

  eat(jelly_bean for jelly_bean in jelly_beans
      if jelly_bean.color == 'black')
```

```python
No:
  result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

  return ((x, y, z)
          for x in xrange(5)
          for y in xrange(5)
          if x != y
          for z in xrange(5)
          if y != z)
```

### 2.8 Default Iterators and Operators(默认的迭代器和运算符)
如果类型支持的话，使用默认的迭代器和运算符，像列表、字典和文件等。
#### 2.8.1 定义
容器类型，比如字典和列表，定义了默认的迭代器和成员测试运算符(“in”和“not in”)。
#### 2.8.2 优点
默认的迭代器和运算符简单高效。它们直接表达操作，没有额外的方法调用。使用默认运算符的函数是通用的。它可以和支持该运算的任何类型一起使用。
#### 2.8.3 缺点
通过方法名无法分辨对象类型(比如has\_key()表示一个字典)。这其实也是一种优势。
#### 2.8.4 建议
如果类型支持的话，使用默认的迭代器和运算符，像列表、字典和文件等。内置的类型也定义了迭代器方法。相比返回列表的方法，优先使用这些方法，除非你在迭代的时候不允许改变容器。

```python
Yes:  for key in adict: ...
      if key not in adict: ...
      if obj in alist: ...
      for line in afile: ...
      for k, v in dict.iteritems(): ...
```

```python
No:   for key in adict.keys(): ...
      if not adict.has_key(key): ...
      for line in afile.readlines(): ...
```

### 2.9 Generators(生成器)
根据需要使用生成器。
#### 2.9.1 定义
生成器函数返回一个迭代器，这个迭代器每次执行yield语句时会产生一个值。生成一个值后，生成器函数的运行状态就处于暂停状态，直到需要下一个值。
#### 2.9.2 优点
更简洁的代码。因为每次调用都保留了局部变量和控制流的状态。生成器比一次运行就生成整个列表的函数要使用更少的内存。
#### 2.9.3 缺点
没有。
#### 2.9.4 建议
生成器函数的docstring中使用“Yields:”而不是“Returns:”。
### 2.10 Lambda Functions(Lambda函数)
单行使用没问题。
#### 2.10.1 定义
Lambdas与声明相反，它在表达式中定义匿名函数。它通常被用来给更高层的函数定义回调或运算符，比如`map()`和`filter()`。
#### 2.10.2 优点
方便。
#### 2.10.3 缺点
相比局部函数，它更难阅读和调试。匿名意味着堆栈跟踪更难理解。表达能力有限，因为函数可能只包含一个表达式。
#### 2.10.4 建议
单行使用没问题。如果Lambda表达式内部的代码多于60-80个字符，那么把它定义为普通(嵌套)函数可能更好。
对于常规的运算比如乘法，使用`operator`模块中的函数而不要使用Lambda函数。例如，优先使用`operator.mul`，而不是`lambda x, y: x * y`。
### 2.11 Conditional Expressions(条件表达式)
单行使用没问题。
#### 2.11.1 定义
条件表达式(有时也叫三元运算符)是一种给if语句提供更简短语法的机制。例如，`x = 1 if cond else 2`。
#### 2.11.2 优点
比if语句更短更方便。
#### 2.11.3 缺点
比if语句更难阅读。如果表达式很长的话，条件很难定位。
#### 2.11.4 建议
单行使用没问题。其他情况下优先使用完整的if语句。
### 2.12 Default Argument Values(默认参数值)
多数情况下没有问题。
#### 2.12.1 定义 
你可以在函数的参数列表末尾给变量指定值。例如，`def foo(a, b=0):`。如果`foo`只用一个参数来调用，那么b设置为0。如果函数用两个参数的形式调用，那么b就是第二个参数的值。
#### 2.12.2 优点
经常会遇到这种情况，你有一个函数使用许多默认值，但你很少去覆盖这些默认值。默认参数值为此提供了一种简单的方式，避免因少数例外情况定义多个函数。而且，Python不支持重载方法或函数，默认参数是“伪装”重载的一个简单方式。
#### 2.12.3 缺点
默认参数只在模块载入的时候求值一次，如果参数是可变对象(例如列表或字典)，就会引发问题。如果函数改变了对象(例如在列表后追加项)，默认值就会被改变。
#### 2.12.4 建议
考虑一下建议可以使用：
在函数或方法中不要使用可变对象作为默认值。

```python
Yes: def foo(a, b=None):
         if b is None:
             b = []
Yes: def foo(a, b: Optional[Sequence] = None):
         if b is None:
             b = []
```

```python
No:  def foo(a, b=[]):
         ...
No:  def foo(a, b=time.time()):  # The time the module was loaded???
         ...
No:  def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed...
```

### 2.13 Properties（属性）
在通常你会使用简单轻量的访问或设置方法来访问或设置数据的地方换做用属性来实现。
#### 2.13.1 定义
当计算是轻量级的时候，把获取或设置属性的方法调用包装为标准属性访问的一种方法。
#### 2.13.2 优点
通过显式去除对简单属性访问的获取和设置方法，提高了代码可读性。允许计算延迟。考虑了以python的方式来维护类的接口。在性能方面，当直接访问变量合情合理时，用方法去访问属性就变得没有意义。这样也使得以后可以在不破坏接口前提下添加访问方法。
#### 2.13.3 缺点
在python2中必须要继承`object`对象。也会像操作符重载一样隐藏副作用。可能让子类感到困惑。
#### 2.13.4 建议
在通常你会使用简单轻量的访问或设置方法来访问或设置数据的地方换做用属性来实现。属性应当用`@property`修饰符来创建。如果属性本身没有被重写，那么带有属性的继承可能不明显。因此必须确保访问方法被间接调用来保证子类中重写的方法被属性调用。（使用模板方法DP）

```python
Yes: import math

     class Square(object):
         """A square with two properties: a writable area and a read-only perimeter.

         To use:
         >>> sq = Square(3)
         >>> sq.area
         9
         >>> sq.perimeter
         12
         >>> sq.area = 16
         >>> sq.side
         4
         >>> sq.perimeter
         16
         """

         def __init__(self, side):
             self.side = side

         @property
         def area(self):
             """Gets or sets the area of the square."""
             return self._get_area()

         @area.setter
         def area(self, area):
             return self._set_area(area)

         def _get_area(self):
             """Indirect accessor to calculate the 'area' property."""
             return self.side ** 2

         def _set_area(self, area):
             """Indirect setter to set the 'area' property."""
             self.side = math.sqrt(area)

         @property
         def perimeter(self):
             return self.side * 4
```

### 2.14 True/False evaluations(True/False值)
尽可能使用隐式false值。
#### 2.14.1 定义
在布尔上下文中，python将特定的值解析为`False`。快速“经验法则”就是所有“空”值都被认为是`False`，因此，`0, None, [], {}, ''`在布尔环境下都被解析为`False`。
#### 2.14.2 优点
使用python布尔值作为条件更容易阅读，更不易出错。多数情况下，运行也更快。
#### 2.14.3 缺点
对C/C++开发人员来说这有点奇怪。
#### 2.14.4 建议
尽可能使用隐式false值。例如，`if foo:`而不是`if foo != []:`。然而还有几个警告需要记住：
- 永远不要使用`==`或`!=`来比较单件(比如`None`)，使用`is`或`is not`。
- 当你真正想表示的是`if x is not None:`时，谨慎处理`if x:`。例如，当测试一个默认值为`None`的变量或参数是不是设为了其他值，这个其他值在布尔上下文中可能是false。
- 永远不要用`==`将一个布尔变量和`False`相比，使用`if not x:`。如果你想区分`False`和`None`，使用链接表达式。例如，`if not x and x is not None:`。
- 对于序列(字符串、列表、元组)，空序列就是假。因此`if seq:`和`if not seq:`分别要比`if len(seq):`和`if not len(seq):`。
- 当处理整数时，隐式的假值带来的风险可能比好处更大。比如不小心把`None`当作0处理。你可能将一个确认的整数值和这个“0”比较。

```python
Yes: if not users:
         print('no users')

     if foo == 0:
         self.handle_zero()

     if i % 10 == 0:
         self.handle_multiple_of_ten()

     def f(x=None):
         if x is None:
             x = []
```

```python
No:  if len(users) == 0:
         print('no users')

     if foo is not None and not foo:
         self.handle_zero()

     if not i % 10:
         self.handle_multiple_of_ten()

     def f(x=None):
         x = x or []
```

- 注意`'0'`(字符串0)在布尔值中是True。
### 2.15 Deprecated Language Features(弃用的语言特性)
尽可能使用字符串方法代替字符串模块。使用函数调用语法代替`apply`。当函数参数是内联lambda表达式时，无论如何使用列表推导式或`for`循环代替`filter`和`map`。使用`for`循环代替`reduce`。
#### 2.15.1 定义
python的当前版本提供了人们更愿意使用的可替代结构。
#### 2.15.2 建议
我们不使用不支持这些特性的python版本，没有理由不使用新方式 。

```python
Yes: words = foo.split(':')

     [x[1] for x in my_list if x[2] == 5]

     map(math.sqrt, data)    # Ok. No inlined lambda expression.

     fn(*args, **kwargs)
```

```python
No:  words = string.split(foo, ':')

     map(lambda x: x[1], filter(lambda x: x[2] == 5, my_list))

     apply(fn, args, kwargs)
```

### 2.16 Lexical Scoping(静态作用域)
放心使用。
#### 2.16.1 定义
嵌套的python函数可以引用封闭函数中定义的变量，但是不能给它们赋值。变量绑定是用静态作用域解析的，也是说基于静态程序文本。代码块中任何对变量的赋值都会让python将该变量的所有引用当作局部变量，即使在赋值之前使用。如果有全局声明，那么变量就被当作全局变量。该特性使用的一个例子：

```python
def get_adder(summand1):
    """Returns a function that adds numbers to a given number."""
    def adder(summand2):
        return summand1 + summand2

    return adder
```

#### 2.16.2 优点
通常会产生更加简洁优雅的代码。尤其会让有经验的Lisp和Scheme程序员感到欣慰。
#### 2.16.3 缺点
可能会导致令人困惑的bug。比如这个例子[PEP-0227](www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0227/)。

```python
i = 4
def foo(x):
    def bar():
        print(i, end='')
    # ...
    # A bunch of code here
    # ...
    for i in x:  # Ah, i *is* local to foo, so this is what bar sees
        print(i, end='')
    bar()
```
因此，`foo([1, 2, 3])`会打印`1 2 3 3`而不是`1 2 3 4`。

#### 2.16.4 建议
推荐使用。
### 2.17 Function and Method Decorators(函数和方法装饰器)
当有明显得好处时使用。避免使用`@staticmethod`，限制`@classmethod`的使用。
#### 2.17.1 定义
[Decorators for Functions and Methods](https://docs.python.org/2/whatsnew/2.4.html#pep-318-decorators-for-functions-and-methods)(也成为`@`标记符)。一个常用的装饰器是`@property`，用来把普通的方法转换为动态计算的属性。而且，装饰器语言也允许用户自定义。特别地，对某个函数`my_decorator`，下面的代码：

```python
class C(object):
    @my_decorator
    def method(self):
        # method body ...
```

与下面的代码是一样的：

```python
class C(object):
    def Methodmethod(self):
        # method body ...
    Methodmethod = MyDecoratormy_decorator(Methodmethod)
```

#### 2.17.2 优点
优雅地指定方法上的某些转换；这些转换可以消除重复代码，执行约束条件等。
#### 2.17.3 缺点
装饰器可以对函数的参数或返回值执行任意操作，可能导致令人意想不到的行为。此外，装饰器在引入的时候执行，如果装饰器代码执行失败，几乎是不可能恢复的。
#### 2.17.4 建议
有明显好处时明智地使用装饰器。装饰器应当遵循跟函数一样的引入和命名规则。装饰器的注释文档要明确说明该函数是一个装饰器。为装饰器编写测试代码。
在装饰器内部避免有外部依赖(例如，不要依赖文件、socket、数据库连接等)，因为在装饰器运行(从pydoc或其他工具引入的时候)时它们可能不可用。使用有效参数调用的decorator应该(尽可能多地)保证在所有情况下都能成功。
装饰器“顶级代码”的一种特殊情况，参考[main](#main)获取详情。
永远不要使用`@staticmethod`(除非被迫与现有库中的API进行集成)，而是编写一个模块级函数。
只有在编写命名构造函数或特定于类的例程(该例程修改必要的全局状态，如进程范围的缓存)时才使用@classmethod。
### 2.18 Threading(线程)
不要依赖内置类型的原子性。
尽管python的内置数据类型(比如字典)看上去具有原子性操作，但是在有些情况下它们并不是原子性的(例如，如果`__hash__`或者`__eq__`实现为python方法)，因此它们的原子性是靠不住的。同样，你也不应该依赖于原子性的变量赋值(因为该操作反过来依赖于字典)。
使用Queue模块的`Queue`数据类型作为线程间数据通信的优选方式。不然就使用threading模块以及它的locking其元。学习条件变量的恰当使用，这样就可以用`threading.Condition`代替更低级别的锁。
<a id="power-features"></a>
### 2.19 Power Features
避免这些特性。
#### 2.19.1 定义
python是一种极具灵活性的语言，提供了许多有趣的特性，比如自定义原类、访问字节码、动态编译、动态继承、对象父类重定义(object reparenting)、导入hacks、反射及系统内部构件的修改等。
#### 2.19.2 优点
这些都是强有力的语言特性。它们可以让你的代码更紧凑。
#### 2.19.3 缺点
使用这些“炫酷”的特性很有诱惑力，尽管它们不是绝对必要的。底层使用不常用特性的代码变得更难以阅读、理解和调试。一开始似乎不是这样(对代码原作者来说)，但是再次访问时，这些代码往往比那些更长但是更直接的代码更难理解。
#### 2.19.4 建议
在你的代码中避免出现这些特性。
推荐使用标准库模块和内部使用这些特性的类(例如，`abc.ABCMeta`、`collections.namedtuple`和`enum`)。
### 2.20 Modern Python: Python 3 and from \_\_future\_\_ imports {#modern-python}
python3已经发行了。尽管不是每个项目都是用python3，但是所有的代码都应该以面向未来的眼光来写。
#### 2.20.1 定义
python3是python语言中一个意义重大的改变。虽然现有的代码多数是用2.7写的，但是有一些简单的事情可以做来让代码更明确地表达它的意图。这样可以更充分地准备好不做任何改变就能在python3下运行。
#### 2.20.2 优点
一旦项目的所有依赖都准备好了，用python3写的代码更明显、更容易在python3下执行。
#### 2.20.3 缺点
许多人发现额外的样板文件很难看。其他人可能说，“我不在这个文件中使用该特性”，并希望进行清理。请不要这样做。最好在所有文件中使用future imports，这样后来想使用这样的特性而编辑的时候不会忘记。
#### 2.20.4 建议
##### from \_\_future\_\_ imports
推荐使用`from __future__ import`声明。所有新代码应该包含下面的语句。如果可能的话，现有代码应该更新兼容性。

```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
```

如果你还不熟悉这些，请参考：[ absolute imports](https://www.python.org/dev/peps/pep-0328/)、[new / division behavior](https://www.python.org/dev/peps/pep-0238/)和[the print function](https://www.python.org/dev/peps/pep-3105/)。
也有其他`from __future__`声明。如果合适也可以使用。我们不建议`unicode_literals`，因为它在python2.7中很多地方引入的隐式默认编码转换序列并没有明显优势。大多数代码显式使用`b''`、`u''`字节和必要的Unicode字符串常量更好。
当前、将来和过去的库。
如果你的项目需要在python2和python3都支持，如果合适的话推荐使用这些库。它们是你的代码更简洁。
### 2.21 Type Annotated Code（类型注释代码）
你可以根据[PEP-484](https://www.python.org/dev/peps/pep-0484/)使用类型提示注释python3代码，使用类型检查工具（比如[pytype](https://github.com/google/pytype)）在编译时对代码进行类型检查。
类型注释可以在源代码或[stub pyi file](https://www.python.org/dev/peps/pep-0484/#stub-files)中。不论什么时候，尽可能把注释放在源代码中。在第三方或扩展模块中使用pyi files。
#### 2.21.1 定义
类型注释（或类型提示）是针对于函数或方法参数和返回值的：

```python
def func(a: int) -> List[int]:
```

你可以用特定的注释声明变量的类型：

```python
a = SomeFunc()  # type: SomeType
```

#### 2.21.2 优点
类型提示提高了代码的可读性和可维护性。类型检查会将很多运行时错误转换为编译时错误，并减低你是使用[Power Features](#power-features)的能力。
#### 2.21.3 缺点
必须保持类型声明是最新的。许多你认为有效的代码可能有类型错误。使用[类型检查](https://github.com/google/pytype)可以降低你使用[Power Features](#power-features)的能力。
#### 2.21.4 建议
这高度依赖于项目的复杂度，试一试。

## Python Style Rules（python风格规则）
### 3.1 Semicolons（分号）
不要用分号作为语句的结束，也不要用分号将两个语句放在一行。

<a id="naming"></a>
### 3.16 命名

<a id="typing-imports"></a>
#### 3.19.12 Imports For Typing

<a id="main"></a>
### 2.17 Main
