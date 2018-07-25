# 谷歌python风格指南

## 前言
对于python的语法，我还有很多地方不是特别清楚，所以有些地方翻译得不是特别清晰，可能会看得有点头大。对这些地方请参考官方说明。  
这篇风格指南针对于python，给出了一些建议。涉及了两个方面，一个是有关python语法的规范，比如异常怎么用、什么时候用列表生成器等等，这部分是关于python语言本身的使用指南；另一个是关于python代码的风格，比如如何进行缩进、如何续行等等，这部分让代码看上去很紧凑。这两个方面对于代码的可读性、代码以后的可维护性都有很积极的意义，对于Code Review大有帮助。  
这篇风格指南只是给出了google的一些规范和建议，有许多地方并不是强制要求这么做。但正如上所说，为了代码更加紧凑、美观，增强代码可读性和可维护性。我们应当遵守这样的指导。  
文档中提到了几个工具[pylint](http://pylint.pycqa.org/en/latest/)、[yapf](https://github.com/google/yapf/)、[pytype](https://github.com/google/pytype)，可以查阅一下使用方式，可以让代码尽可能的满足本文提到的各种规范建议。

下面开始正文。
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
<a id="line-length"></a>
### 3.2 Line Length（行长度）
一行最大长度为80个字符。
例外：
- 长的导入声明
- URL、路径名称或者注释中长标记
- 不含空格的模块级长字符串常量，比如URL或路径名称，跨行显示不方便。
- pylint中disable注释。例如，`pylint: disable=invalid-name`。
不要用反斜杠来续行，除非是需要三行及以上上下文管理的`with`语句。
使用python的[小括号、大括号和中括号隐式续行](http://docs.python.org/reference/lexical_analysis.html#implicit-line-joining)。如果必要，你可以在表达式两边额外加一组小括号。

```python
Yes: foo_bar(self, width, height, color='black', design=None, x='foo',
             emphasis=None, highlight=0)

     if (width == 0 and height == 0 and
         color == 'red' and emphasis == 'strong'):
```

当一个字符串常量一行放不下，就用小括号来隐式续行。

```python
x = ('This will build a very long long '
     'long long long long long long string')
```

在注释中，如果必要，把长的URL地址放在它们自己的行里。

```python
Yes:  # See details at
      # http://www.example.com/us/developer/documentation/api/content/v2.0/csv_file_name_extension_full_specification.html
```

```python
No:  # See details at
     # http://www.example.com/us/developer/documentation/api/content/\
     # v2.0/csv_file_name_extension_full_specification.html
```

当定义一个长度跨了三行及以上的`with`表达式时，允许使用反斜杠来续行。对于两行的表达式，使用嵌套的`with`声明。

```python
Yes:  with very_long_first_expression_function() as spam, \
           very_long_second_expression_function() as beans, \
           third_thing() as eggs:
          place_order(eggs, beans, spam, beans)
```

```python
No:  with VeryLongFirstExpressionFunction() as spam, \
          VeryLongSecondExpressionFunction() as beans:
       PlaceOrder(eggs, beans, spam, beans)
```

```python
Yes:  with very_long_first_expression_function() as spam:
          with very_long_second_expression_function() as beans:
              place_order(beans, spam)
```

记住上面续行例子的元素缩进。参考[indentation](#indentation)章节的解释。
### 3.3 Parentheses（小括号）
保守使用小括号。
可以在元组两边使用小括号，但不是必须的。不要再return语句或条件语句中使用小括号，除非用小括号来隐式续行或表明一个元组。

```python
Yes: if foo:
         bar()
     while x:
         x = bar()
     if x and y:
         bar()
     if not x:
         bar()
     # For a 1 item tuple the ()s are more visually obvious than the comma.
     onesie = (foo,)
     return foo
     return spam, beans
     return (spam, beans)
     for (x, y) in dict.items(): ...
```

```python
No:  if (x):
         bar()
     if not(x):
         bar()
     return (foo)
```

<a id="indentation"></a>
### 3.4 Indentation（缩进）
使用4个空格缩进你的代码块。
不要使用tab或者混用tab和空格。在隐式续行情况下，应该按照[line length](#line-length)章节的例子来垂直对齐这些元素；或者悬挂缩进4个空格，这种情况下第一行打开的括号后面什么都不能添加。

```python
Yes:   # Aligned with opening delimiter
       foo = long_function_name(var_one, var_two,
                                var_three, var_four)
       meal = (spam,
               beans)

       # Aligned with opening delimiter in a dictionary
       foo = {
           long_dictionary_key: value1 +
                                value2,
           ...
       }

       # 4-space hanging indent; nothing on first line
       foo = long_function_name(
           var_one, var_two, var_three,
           var_four)
       meal = (
           spam,
           beans)

       # 4-space hanging indent in a dictionary
       foo = {
           long_dictionary_key:
               long_dictionary_value,
           ...
       }
```

```python
No:    # Stuff on first line forbidden
       foo = long_function_name(var_one, var_two,
           var_three, var_four)
       meal = (spam,
           beans)

       # 2-space hanging indent forbidden
       foo = long_function_name(
         var_one, var_two, var_three,
         var_four)

       # No hanging indent in a dictionary
       foo = {
           long_dictionary_key:
           long_dictionary_value,
           ...
       }
```

### 3.5 Blank Lines（空行）
顶层定义之间用两个空行相隔，方法定义之间用一个空行。
顶层定义之间用两个空行相隔，它们应当是函数或类定义。方法定义之间用一个空行相隔，`class`所在行与第一个方法之间用一个空行。在函数和方法内部通过你的判断来使用单个空行。
### 3.6 Whitespace（空格）
标点周围使用空格要遵守标准的排版规则。
括号（大中小）中不要有空格。
```python
Yes: spam(ham[1], {eggs: 2}, [])
```

```python
No:  spam( ham[ 1 ], { eggs: 2 }, [ ] )
```

逗号、分号和冒号前面不要有空格。务必在逗号、分号和冒号后面使用，除非他们在该行末尾。

```python
Yes: if x == 4:
         print(x, y)
     x, y = y, x
```

```python
No:  if x == 4 :
         print(x , y)
     x , y = y , x
```

开始参数列表、索引或切片的打开的圆括号方括号前不要有空格。

```python
Yes: spam(1)
```

```python
No:  spam (1)
```

```python
Yes: dict['key'] = list[index]
```

```python
No:  dict ['key'] = list [index]
```

二元运算符两边各放一个空格，赋值符号`=`、比较符号（`==, <, >, !=, <>, <=, >=, in, not in, is, is not`）和布尔运算符（`and, or, not`）。根据你的判断在算数运算符周围使用空格，但是要注意二元运算符周围的空格使用保持一致的方式。

```python
Yes: x == 1
```

```python
No: x<1
```

当传递关键字参数时千万不要在赋值符号`=`两边使用空格，只有[当有类型注释的时候](#typing-default-values)，定义默认参数值时才在`=`两边使用空格。否则不要在默认参数值的`=`两边使用空格。

```python
Yes: def complex(real, imag=0.0): return Magic(r=real, i=imag)
Yes: def complex(real, imag: float = 0.0): return Magic(r=real, i=imag)
```

```python
No:  def complex(real, imag = 0.0): return Magic(r = real, i = imag)
No:  def complex(real, imag: float=0.0): return Magic(r = real, i = imag)
```

不要再连续行中使用空格垂直对齐符号，这样会引发维护负担。（应用于`:`，`#`，`=`等）：

```python
Yes:
  foo = 1000  # comment
  long_name = 2  # comment that should not be aligned

  dictionary = {
      'foo': 1,
      'long_name': 2,
  }
```

```python
No:
  foo       = 1000  # comment
  long_name = 2     # comment that should not be aligned

  dictionary = {
      'foo'      : 1,
      'long_name': 2,
  }
```

### 3.7 Shebang Line（系统行）
大多数`.py`文件不需要以`#!`行开始。程序main文件以行`#!/usr/bin/python`开始，该行后可以根据[PEP-394](https://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0394/)选填一个数字后缀`2`或`3`。（指定python版本）
内核使用这一行来寻找python编译器，但是在导入模块时python会忽略该行。它只在直接执行的文件中才是必须的。
### 3.8 Comments and Docstrings（注释和文档字符串）
确认对模块、函数和方法的文档字符串以及行内注释使用合适的风格。
#### 3.8.1 Docstrings（文档字符串）
python用docstrings来注释代码。一个文档字符串就是在一个包、模块、类或函数中的第一个声明字符串。这些字符串可以被对象的`__doc__`成员自动解析并被`pydoc`使用。
对你的模块运行`pydoc`看看出现什么。文档字符串总是用3个双引号`"""`格式，参考[PEP
257](https://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0257/)。一个文档字符串应当被组织为一行摘要或一个物理行，并以一个句点、问号或者叹号结束。后面跟一个空行，然后是以与第一行第一句话光标相同位置开始的其他文档字符串。下面有更多关于格式的指导规则。
#### 3.8.2 Modules（模块）
每个文件都应包括许可版本。选择适合项目的证书版本，例如Apache 2.0、BSD、LGPL和GPL等。
#### 3.8.3 Functions and Methods（函数和方法）
在本章见中使用的"function（函数）"适用于方法、函数和生成器。
一个函数必须有一个文档字符串，除非满足下面所有的标准：
- 外部不可见
- 很短
- 很明显
文档字符串应该提供足够的信息，使得不用阅读函数的代码就可以调用函数。文档字符串应当具有描述性（`"""Fetches rows from a Bigtable."""`）,不要用命令口气（`"""Fetch rows from a Bigtable."""`）。文档字符串应当描述函数的调用语法和语义，而不是它的实现。对于复杂的代码，注释紧跟代码要比文档字符串更合适。
覆盖了基类中的方法时，可以用简单的文档字符串让读者去查阅基类方法的文档字符串，比如`"""See base class."""`。原则是在许多地方，在基方法中出现的文档字符串就没有必要再次重复。然而，如果重写的方法比被重写的方法有较大不同，或者需要提供重写后方法的细节（例如，记录额外的副作用），那么重写后的方法就需要有文档字符串，至少包括那些不同之处。
一个函数的某些方面应该文档化为几个特殊部分，下面进行罗列。每个部分以一个标题行开始，标题行以冒号结束。这些部分应当缩进两个空格，除了标题。


*Args:*
- 列出每个参数的名字。名字后面是描述，并用一个冒号和空格将两者分开。如果描述太长，一行的80个字符放不下，那就悬挂缩进2或4个空格。文件的其余部分保持一致（缩进2个就都缩进2个，4个亦然）。
- 如果代码不包含相应的类型注释，那么描述应该包含必要的类型说明。
- 如果函数接收`*foo`（可变参数类型）或者（和）`**bar`（任意关键字参数），它们就以`*foo`和`**bar`的形式列出。


*Returns:* (或者 *Yields:*，对生成器而言)
- 描述返回值的类型和语义。如果函数返回None，那么这部分不是必须的。
- 如果文档字符串以Returns或Yields开头（例如，`"""Returns row from Bigtable as a tuple of strings."""`），或者开头语足以描述返回值，那么这部分也可以忽略。

*Raises:*
- 列出所有接口相应的异常。

```python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table.  Silly things may happen if
    other_silly_variable is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
         'Zim': ('Irk', 'Invader'),
         'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
```

#### 3.8.4 Classes（类）
类定义下面应当有文档字符串来描述这个类。如果你的类里有公有属性，它们应当在属性部分进行文档化，并且跟函数的参数部分遵循一样的格式化规则。

```python {.good}
class SampleClass(object):
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```

#### 3.8.5 Block and Inline Comments（块和内联注释）
最后一个写注释的地方是代码中棘手的部分。如果在下一步 [code
review](http://en.wikipedia.org/wiki/Code_review) 中你必须要解释它的话，你应该现在就对它做一下注释。一些复杂的操作应该在开始之前就写几行注释。意图不容易阅读的代码，在该行后面加一行注释。

```python
# We use a weighted dictionary search to find out where i is in
# the array.  We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.

if i & (i-1) == 0:  # True if i is 0 or a power of 2.
```

为了提高可读性，这些注释应该离代码至少2个空格远。
另一方面，千万不要描述代码。想像一下，读代码的人如果比你还了解python...（尽管你不希望这样）

```python
# BAD COMMENT: Now go through the b array and make sure whenever i occurs
# the next element is i+1
```

#### 3.8.6 Punctuation, Spelling and Grammar（标点、拼写和语法）
注意一下标点、拼写和语法，写得好的注释比那些糟糕的更容易阅读。
注释应该跟叙事性文本一样具有可读性，有合适的大小写以及标点。许多情况下，一个完整的句子要比一个句子片段可读性更好。更短的注释，比如一行代码后面的注释，有时不那么正式，但是你应该保持你的代码风格一致。
尽管让一个代码审查员指出本应该用分号的时候用了逗号让你感到有点沮丧，但是保持源代码高度的清晰性和可读性非常重要。恰当的标点、拼写和语法大有裨益。
### 3.9 Classes（类）
如果一个类没有继承自其他类，那么让它显式继承自`object`类。这对嵌套类也一样。

```python
Yes: class SampleClass(object):
         pass


     class OuterClass(object):

         class InnerClass(object):
             pass


     class ChildClass(ParentClass):
         """Explicitly inherits from another class already."""
```

```python
No: class SampleClass:
        pass


    class OuterClass:

        class InnerClass:
            pass
```

在python2中为了让属性正常工作，继承自`object`类是必要的；同时这样也可以避免你的代码与python3的一些潜在不兼容。它也定义了一些特殊的方法，这些方法实现了对象的默认语义，包括`__new__`，`__init__`，`__delattr__`，`__getattribute__`，`__setattr__`，`__hash__`，`__repr__`，和`__str__`。
### 3.10 Strings（字符串）
使用`format`方法或者`%`操作符来格式化字符串，即便参数全是字符串。然而你需要好好判断用`+`还是`%`（或`format`）。

```python
Yes: x = a + b
     x = '%s, %s!' % (imperative, expletive)
     x = '{}, {}'.format(first, second)
     x = 'name: %s; score: %d' % (name, n)
     x = 'name: {}; score: {}'.format(name, n)
     x = f'name: {name}; score: {n}'  # Python 3.6+
```

```python
No: x = '%s%s' % (a, b)  # use + in this case
    x = '{}{}'.format(a, b)  # use + in this case
    x = first + ', ' + second
    x = 'name: ' + name + '; score: ' + str(n)
```

避免在循环中使用`+`和`+=`操作符连接字符串。因为字符串是不可变的，这种做法会创建不必要的临时对象，结果导致平方时间复杂度，而不是线性复杂度。取而代之，把每一个子字符串放到列表里面，循环结束后用`''.join`处理列表（或者把每个子字符串写到`io.ButesIO`缓存中）。

```python
Yes: items = ['<table>']
     for last_name, first_name in employee_list:
         items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
     items.append('</table>')
     employee_table = ''.join(items)
```

```python
No: employee_table = '<table>'
    for last_name, first_name in employee_list:
        employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
    employee_table += '</table>'
```

在一个文件中使用字符串引号时保持风格一致。选择`'`或`"`并一直使用。为了在字符串中避免转义符`\\`而使用另一种引号是可以的。`gpylint`强制执行这一点。

```python
Yes:
  Python('Why are you hiding your eyes?')
  Gollum("I'm scared of lint errors.")
  Narrator('"Good!" thought a happy Python reviewer.')
```

```python
No:
  Python("Why are you hiding your eyes?")
  Gollum('The lint. It burns. It burns us.')
  Gollum("Always the great lint. Watching. Watching.")
```

对跨多行的字符串来说，优先使用`"""`而不是`'''`。项目中，当前仅当对常规字符串使用`'`时，才可选择对所有跨行的非文档字符串用`'''`。当然，文档字符串要使用`"""`。注意，使用隐式续行通常更简洁，因为多行字符串不会随着程序其他部分的缩进而流动。

```python
Yes:
  print("This is much nicer.\n"
        "Do it this way.\n")
```

```python
No:
    print("""This is pretty ugly.
Don't do this.
```

### 3.11 Files and Sockets（文件和套接字）
当时用完文件和套接字时显式关闭它们。
让文件、套接字或其他类文件对象处于不必要的打开状态，会有很多隐患，包括：
- 消耗有限的系统资源，比如文件描述符。处理这种对象的代码在使用完毕后如果没有立即把资源归还给系统就会耗尽这些资源。
- 保持文件打开状态会阻止其他动作操作它们，比如移动或删除。
- 在整个程序中共享的文件或套接字在逻辑上被关闭后可能无意中被读取或写入。如果它们真的被关闭了，试图对它们进行读写会抛出异常，使这个问题及早被发现。
此外，尽管文件对象析构后文件和套接字会自动关闭，但是把文件对象的生命周期和文件的状态联系在一起不是一个好做法，有以下几个原因：
- 运行时何时运行文件的析构是没有保证的。不同的python实现使用不同的内存管理机制，例如滞后垃圾回收，可能会任意（不确定）加大对象的生命周期。
- 对文件意想不到的引用可能让它的活动时间比预想的要长。例如在异常追踪、全局作用域内等。
处理文件的优选方式是用["with"
语句](http://docs.python.org/reference/compound_stmts.html#the-with-statement):

```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print(line)
```

对于不支持"with"语句的类文件对象来说，用`contextlib.closing()`:

```python
import contextlib

with contextlib.closing(urllib.urlopen("http://www.python.org/")) as front_page:
    for line in front_page:
        print(line)
```

### 3.12 TODO Comments（TODO注释）
`TODO`注释用来修饰临时代码、短期解决方案或者足够好但是不够完美的代码。
`TODO`包含全部大写形式的字符串`TODO`，后面跟人的姓名、邮箱地址或其他标识符或者`TODO`所引用问题的最佳上下文，并用圆括号括起来。注释给出了还有什么需要去做。主要目的是让`TODO`的格式一致，以便查询关于需求的细节。`TODO`不是说提到的这个人要去解决这个问题。因此当你创建一个`TODO`的时候，几乎总是写你自己的名字。

```python
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```

如果你的`TODO`是“在未来某个时间做什么事情”，那么确保是一个非常具体的时间（“2009年11月解决”）或者是一个具体的事件（“当所有客户能处理XML响应时删除此代码”）。
### 3.13 Imports formatting（导入的格式）
导入应当在单独的行中。例如：

```python
Yes: import os
     import sys
```

```python
No:  import os, sys
```

导入语句总是放在文件的顶端，就在模块注释和文档字符串后面并且在模块全局变量和常量前面。模块组织的顺序应当是从最常用的到最不常用的：

1.  python标准库导入，例如：

    ```python
    import sys
    ```
    
2.  [第三方](https://pypi.python.org/pypi)模块或包导入，例如：
    
    ```python
    import tensorflow as tf
    ```
    
3.  代码库的子包导入，例如：
 
    ```python
    from otherproject.ai import mind
    ```
    
4.  特定应用程序导入，它们是与本文件相同的顶级子包的一部分，例如：

    ```python
    from myproject.backend.hgwells import time_machine
    ```

在每个分组内，每个包根据模块的包的全路径以字典序排序，不计大小写。可以在导入的代码块之间用空行分隔。

```python
import collections
import Queue
import sys

import argcomplete
import BeautifulSoup
import cryptography
import tensorflow as tf

from otherproject.ai import body
from otherproject.ai import mind
from otherproject.ai import soul

from myproject.backend.hgwells import time_machine
from myproject.backend.state_machine import main_loop
```

### 3.14 Statements（语句）
通常一行只有一条语句。
然而，你可以将测试结果跟测试放在相同行里，前提是整个语句在一行上。特别地，`try`/`except`不能这么做，因为`try`和`except`不能同时适应同一行。你只能对if语句这么做，前提是没有else语句。

```python
Yes:

  if foo: bar(foo)
```

```python
No:

  if foo: bar(foo)
  else:   baz(foo)

  try:               bar(foo)
  except ValueError: baz(foo)

  try:
      bar(foo)
  except ValueError: baz(foo)
```

### 3.15 Access Control（访问控制）
如果访问函数没什么意义，那你应该使用公有变量来替代访问函数，来避免python中函数调用带来的额外花销。当添加更多功能时你可以使用`property`来保持语法一致。
另一方面，如果访问变得更加复杂，或者访问变量的花销变得很大，这时候应该使用函数中调用（参考[nameing](#naming)指导）比如`get_foo()`和`set_foo()`。如果过去的行为允许使用属性访问，不要把函数调用和属性绑定在一起。通过使用旧有的方法访问变量的任何代码都应该明显中断以便让它们意识到复杂度的变化。
<a id="naming"></a>
### 3.16 命名
`module_name`, `package_name`, `ClassName`, `method_name`, `ExceptionName`, `function_name`, `GLOBAL_CONSTANT_NAME`, `global_var_name`, `instance_var_name`, `function_parameter_name`, `local_var_name`。
函数名字、变量名字和文件名应当具有描述性，避免缩写。特别地，不要使用让项目外阅读者感到模糊或不熟悉的代码，也不要通过删除单词中的字母来进行缩写。
总是使用`.py`文件扩展名，不要使用破折号。
#### 3.16.1 避免使用的名字
- 除计数器和迭代器之外的单个字符名。在`try`/`except`块中可以使用"e"作为异常标识符。
- 包或模块中使用破折号
- `__double_leading_and_trailing_underscore__`（双下划线开头和结尾的）名字（python保留）。

#### 3.16.2 命名约定
- "Internal"意味着模块内部或者类内受保护的或私有的
- 头部添加单个下划线（`_`）会使模块变量和函数受保护（不包含`from module import *`）。然而对实例变量和方法头部添加双下划线使类的变量和方法成为私有（使用名称编码）。不推荐双下划线的使用，因为它影响可读性和可测试性，而且它也不是真正的私有。
- 把相关的类和顶层方法一起放在一个模块中。跟java不一样，没有必要限制自己一个模块中只有一个类。
- 类名使用驼峰式方法，模块名使用小写和下划线 。虽然有许多老模块使用驼峰命名，但现在这种方式不再推荐因为当模块名和类名一样的时候会让人感到很困惑。（`from StringIO import StringIO`??）
- 下划线可以出现在单元测试的以`test`开头的方法名中，来区分名字中的逻辑部分，即使那些部分使用驼峰命名。一种可能的模式是`test<MethodUnderTest>_<state>`，例如`testPop_EmptyStack`是可以的。没有一种正确的方法来命名测试方法。
#### 3.16.3 文件命名
python文件名必须以`.py`作为扩展名，而且不能包含破折号。这样它们才能被导入和测试。如果你想访问一个没有扩展名的可执行文件，请使用符号链接或使用包含`exec "$0.py" "$@"`的简单bash。
#### 3.16.4 源自吉多的建议
<table rules="all" border="1" summary="Guidelines from Guido's Recommendations"
       cellspacing="2" cellpadding="2">

  <tr>
    <th>Type</th>
    <th>Public</th>
    <th>Internal</th>
  </tr>

  <tr>
    <td>Packages</td>
    <td><code>lower_with_under</code></td>
    <td></td>
  </tr>

  <tr>
    <td>Modules</td>
    <td><code>lower_with_under</code></td>
    <td><code>_lower_with_under</code></td>
  </tr>

  <tr>
    <td>Classes</td>
    <td><code>CapWords</code></td>
    <td><code>_CapWords</code></td>
  </tr>

  <tr>
    <td>Exceptions</td>
    <td><code>CapWords</code></td>
    <td></td>
  </tr>

  <tr>
    <td>Functions</td>
    <td><code>lower_with_under()</code></td>
    <td><code>_lower_with_under()</code></td>
  </tr>

  <tr>
    <td>Global/Class Constants</td>
    <td><code>CAPS_WITH_UNDER</code></td>
    <td><code>_CAPS_WITH_UNDER</code></td>
  </tr>

  <tr>
    <td>Global/Class Variables</td>
    <td><code>lower_with_under</code></td>
    <td><code>_lower_with_under</code></td>
  </tr>

  <tr>
    <td>Instance Variables</td>
    <td><code>lower_with_under</code></td>
    <td><code>_lower_with_under</code> (protected)</td>
  </tr>

  <tr>
    <td>Method Names</td>
    <td><code>lower_with_under()</code></td>
    <td><code>_lower_with_under()</code> (protected)</td>
  </tr>

  <tr>
    <td>Function/Method Parameters</td>
    <td><code>lower_with_under</code></td>
    <td></td>
  </tr>

  <tr>
    <td>Local Variables</td>
    <td><code>lower_with_under</code></td>
    <td></td>
  </tr>

</table>

虽然python支持使用前置双下划线作为名字前缀来让一些东西变为私有，但这种做法并不推荐。优先使用但下划线。它们更容易输入、阅读和从小型单元测试访问。lint警告处理对受保护成员的无效访问。
<a id="main"></a>
### 3.17 Main
即使一个文件想用做可执行文件，他也应该可以被导入。仅仅导入不应该对程序的main函数有影响。主要功能应该在`main()`函数中。
在python中，`pydoc`和单元测试要求模块能够被导入。在执行main程序之前，你的代码应当总是进行`if __name__ == '__main__'`检查。这样，当模块被导入的时候，main函数不会执行。

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

当模块被导入时，模块的所有顶层代码都会执行。当文件被`pydoc`ed时2，注意不要进行函数调用、创建对象或其他不该执行的操作。
### 3.18 函数长度
函数要轻量、聚焦。
我们承认长函数有时是很合适的，所以对于函数长度没有强硬的限制。如果一个函数超过了40行，考虑一下在不改变程序结构前提下，函数能不能分解。
即使你的长函数现在依然完美运行，几个月后可能有人需要改变它来添加新功能。这可能导致难以发现的bug。保持函数简短可以让其他人更容易理解并改变你的代码。
当你处理某些代码时，你可能会发现又长又复杂的函数。不要被改变现有代码吓住：如果处理这样的函数很艰难，或发现错误很难调试，再或者你想在几个不同的环境中复用其代码段，那么考虑分解成更轻量的函数以及更容易维护的代码段。
### 3.19 Type Annotations（类型注释）
#### 3.19.1 一般规则

* 熟悉[PEP-484](https://www.python.org/dev/peps/pep-0484/)
* 方法中不要注释`self`或者`cls`
* 如果任何其他变量或返回类型不应该被表达，使用`Any`
* 没有必要对模块中的所有函数进行注释
  - 至少注释公共接口（API）
  - 做好判断，在安全性清晰性和灵活性之间做好权衡
  - 注释容易出现类型错误的代码（先前的错误或复杂度）
  - 注释较难理解的代码
  - 从类型的角度看，当代码稳定的时候注释代码。在许多情况下，你可以注释成熟代码的所有函数，也不会丧失太多灵活性
  
#### 3.19.2 换行
遵循现有的[indentation](#3.4-indentation)规则。总是优先在变量之间换行。
注释之后，很多函数变成“一个参数占一行”。

```python
def my_method(self,
              first_var: int,
              second_var: Foo,
              third_var: Optional[Bar]) -> int:
```

然而，如果所有代码适合一行，那就一行。

```python
def my_method(self, first_var: int) -> int:
```

如果函数名、最后一个参数和返回类型放在一行太长，那就在新的一行缩进4个空格。

```python
def my_method(
    self, first_var: int) -> Tuple[MyLongType1, MyLongType1]:
```

如果返回类型跟最后一个参数放在一行不合适，更好的方式是在新的一行将所有参数缩进4个空格，并将右括号跟def对齐。

```python
Yes:
def my_method(
    self, **kw_args: Optional[MyLongType]
) -> Dict[OtherLongType, MyLongType]:
  ...
```

`pylint`允许将右括号放在新行并和左括号对齐，但是这样可读性比较差。

```python
No:
def my_method(self,
              **kw_args: Optional[MyLongType]
             ) -> Dict[OtherLongType, MyLongType]:
  ...
```

如果单个名称或类型很长，考虑对类型使用[alias](#typing-aliases)。最后的手段是在冒号后面断行，并缩进4个字符。

```python
Yes:
def my_function(
    long_variable_name:
        long_module_name.LongTypeName,
) -> None:
  ...
```

```python
No:
def my_function(
    long_variable_name: long_module_name.
        LongTypeName,
) -> None:
  ...
```

#### 3.19.3 前向声明
如果你需要在当前相同模块中使用一个类名，而这个类还没有进行定义。例如，如果你想在类定义中使用这个类，抑或你想使用一个在下方定义的类，那么使用类名的字符串形式。

```python
class MyClass(object):

  def __init__(self,
               stack: List["MyClass"]) -> None:
```

<a id="typing-default-values"></a>
#### 3.19.4 默认值
依据[PEP-008](https://www.python.org/dev/peps/pep-0008/#other-recommendations)，当结合使用参数注释和默认值时，'='两边都要添加空格（但仅对同时使用注释和默认值的参数适用）。

```python
Yes:
def func(a: int = 0) -> int:
  ...
```

```python
No:
def func(a:int=0) -> int:
  ...
```

#### 3.19.5 NoneType
在python类型系统中，NoneType是“第一类”类型，处于方便输入的目的，`None`是`Nonetype`的别名。如果一个参数可以是`None`，那么必须要显示声明。你可以使用`Union`，但如果只有一个其他类型，`Optional`更方便。

```python
Yes:
def func(a: Optional[str]) -> str:
  ...
```

```python
No:
def func(a: Union[None, str]) -> str:
  ...
```

如果一个参数的默认值是`None`，那么`Optional`变量是可选的。

```python
Yes:
def func(a: Optional[str] = None) -> str:
  ...
def func(a: str = None) -> str:
  ...
```

<a id="typing-aliases"></a>
#### 3.19.6 Type Aliases(类型别名)
你可以为复杂的类型声明别名。别名的名称应当使用驼峰命名。描述该组合类型并以`Type`结尾（对于元组使用`Types`）。如果别名只在本模块中使用，那么应该私有化（使用`_`）。
例如，如果模块与类型的名字连起来很长时：
```python {.good}
SomeType = module_with_long_name.TypeWithLongName
```

其他例子是复杂嵌套类型和函数的多个返回变量（比如元组）。
#### 3.19.7 忽略类型
你可以用特殊的注释`# type: ignore`来禁用某一行的类型检查。`pytype` 对特定的错误有禁用的功能（类似lint）。

```python {.good}
# pytype: disable=attribute-error
```

#### 3.19.8 内部变量的类型
如果内部变量的类型很难或不可能推断出来，那么你可以提供一行特殊的注释：

```python {.good}
a = SomeUndecoratedFunction()  # type: Foo
```

#### 3.19.9 元组 vs 列表
与列表只有一种类型不同，元组既可以有一种单一的类型，也可以有一组不同类型的元素。后者通常用在函数中的返回类型。

```python {.good}
a = [1, 2, 3]  # type: List[int]
b = (1, 2, 3)  # type: Tuple[int, ...]
c = (1, "2", 3.5)  # type Tuple[int, str, float]
```
<a id=="typing-type-var"></a>
#### 3.19.10 TypeVar
python系统类型也有[泛型](https://www.python.org/dev/peps/pep-0484/#generics)。工厂函数`TypeVar`是使用泛型的通用方法。例如：

```python {.good}
from typing import List, TypeVar
T = TypeVar("T")
...
def next(l: List[T]) -> T:
  return l.pop()
```

TypeVar可以添加约束条件：

```python {.good}
AddableType = TypeVar("AddableType", int, float, str)
def add(a: AddableType, b: AddableType) -> AddableType:
  return a + b
```

在`typing`模块中常见的预定义类型变量是`AnyStr`，它主要用于可以是`bytes`或`unicode`的参数。

```python {.good}
AnyStr = TypeVar("AnyStr", bytes, unicode)
```

#### 3.19.11 字符串类型
当注释接收字符串参数或返回字符串的函数时，避免使用`str`，因为在python2和python3中它是不一样的。在Python 2中， `str`
是`bytes`；在Python 3中，它是`unicode`。所以尽可能显式使用。

```python {.bad}
No:
def f(x: str) -> str:
  ...
```

处理byte数组的代码，就用`bytes`：

```python {.good}
def f(x: bytes) -> bytes:
  ...
```

处理Unicode数组的代码就用`Text`：

```python {.good}
from typing import Text
...
def f(x: Text) -> Text:
  ...
```
如果数组既可以是byte又可以是Unicode就用`Union`：


```python {.good}
from typing import Text, Union
...
def f(x: Union[bytes, Text]) -> Union[bytes, Text]:
  ...
```
如果函数的字符串类型总是相同的，例如上面代码的参数类型和返回类型一样，这时使用[AnyStr](#typing-type-var)。

这样编写代码将简化代码移植到python3的过程。
<a id="typing-imports"></a>
#### 3.19.12 Imports For Typing（类型导入）
`typing`模块中的类，总是导入类本身。这时允许你在一行代码中从`typing`模块中导入多个类。例如：

```python {.good}
from typing import Any, Dict, Optional
```
考虑到以这种方式从`typing`模块中导入类会在本地名称空间中添加其中的项目。所以`typing`中出现的名字都简单地当作关键字处理。因此就不要在你的python代码中再次定义，无论使用与否。如果类型和模块中已有的名字产生冲突，那么用`import x as y`方式导入。


```python {.good}
from typing import Any as AnyType
```
如果需要在运行时避免类型检查所需的附加导入，则可使用条件导入。不推荐这种模式，可以选择重构代码以允许顶层导入等替代方法。如果终究还是使用了这种模式，那么条件导致的类型需要使用字符串`'sketch.Sketch'`而不是`sketch.Sketch`。这是为了与注释表达式实际求值的python3前向兼容。仅仅类型注释才会需要的导入放在`if typing.TYPE_CHECKING:`块中。
- 只定义跟typing相关的实体，包括别名。否则会出现运行时错误，因为运行时模块不会被导入。
- 这部分代码块应刚好放在普通导入之后
- 在typing的导入列表中不应有空行
- 像普通列表一样给这个列表排序，但是把有关typing模块的导入放在最后。
- `google3`模块也有一个`TYPE_CHECKING`常量。如果你不想在运行时导入`typing`，可以用它来作为替代。

```python {.good}
import typing
...
if typing.TYPE_CHECKING:
  import types
  from MySQLdb import connections
  from google3.path.to.my.project import my_proto_pb2
  from typing import Any, Dict, Optional
```
#### 3.19.13 Circular Dependencies（循环依赖）
由typing引起的循环依赖属于代码异味。这样的代码最好拿来重构。尽管技术上可以做到循环依赖，但是编译系统不允许这么做，因为每个模块都必须依赖于另一个。
用`Any`替换引发循环依赖的导入。给[alias](#typing-aliases)一个有意义的名字，并使用模块中真实的类型名称（Any中的任何属性都是Any）。别名定义与最后一行导入隔开一行。

```python {.good}
from typing import Any

some_mod = Any  # some_mod.py imports this module.
...

def my_method(self, var: some_mod.SomeType) -> None:
  ...
```

## 4 Parting Words（写在最后的话）
*保持一致*。

如果你在编辑代码，那么花几分钟看一下你的代码，然后确定它的风格。如果算数运算符两旁都有空格，你应该保持这样。如果代码注释周围有哈希标记，你写的代码也应该要有。

风格指南的意义是提供一个编码词汇表，因此人们会更关注于要说什么而不是要怎么说上面。我们在这里给出总体的风格指南，让人们了解这个词汇表；但是自己的风格也很重要。如果你在一个文件中添加的代码与文件中原有的代码风格相差太大，那么读者读代码的时候就会云里雾里，避免这样。


