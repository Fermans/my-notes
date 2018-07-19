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
### 2.2 Imports
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
### 2.3 Packages
使用模块的完整路径名位置导入每个模块。
#### 2.3.2 优点
避免模块名中的冲突。更容易找到模块。
#### 2.3.3 缺点
部署代码更加困难，因为你必须复制包的层次结构。
#### 2.3.4 建议
所有新代码都应该按其完整的包名导入每个模块。

```
# Reference in code with complete name.
import absl.flags

# Reference in code with just module name (preferred).
from absl import flags
```


<a id="typing-imports"></a>
#### 3.19.12 Imports For Typing
