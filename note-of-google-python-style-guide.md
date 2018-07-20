# 谷歌python风格指南

## 前言

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

```
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

```
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

```
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

```
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

```
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

```
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
### 2.7 Comprehensions & Generator Expressions（推导 & 生成表达式）
#### 2.7.1 
#### 2.7.2 
#### 2.7.3
#### 2.7.4 

<a id="naming"></a>
### 3.16 命名

<a id="typing-imports"></a>
#### 3.19.12 Imports For Typing
