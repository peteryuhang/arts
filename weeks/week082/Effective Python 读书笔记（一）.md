## 写在前面

最近由于工作上面的需要，拜读了 *[《Effective Python》](https://book.douban.com/subject/26709315/)*。这也是我读完的第一本 Effective 系列的丛书（之前读过 *Effective Java*，但是没有完整的读下来），读完还是有挺多收获和感触的，在此记录，也算是帮助自己复习加回顾了。

**Effective 系列的丛书的叙事架构和逻辑都是固定的**。这类书籍往往是以一门计算机语言为标题，比如 C++，Java，Python。但与大部分语言类书籍不同的是，它并不会从最基本的概念开始讲起，它也很少会去讲那些你直接可以从文档中查到的东西。这类书籍主要分享的是实际开发中遇到的问题，以及相应的解决方案。一本书里有几十个小标题，每个小标题都是一个建议。

作者叙述每个小标题（建议）的结构都是大同的。首先，作者会提出一个开发中时常会遇到的问题，当然他会花一些篇幅来解释这个问题本身，主要是为了让读者意识到问题的所在，以及问题的严重性。然后，作者会对该问题提出一个解决方案，往往最开始给出的解决方案都是最容易想到的。紧接着，他又会质疑他提出的这个解决方案，罗列出这个方案的各种漏洞与不好。根据这些东西，他又会提出新的解决方案，有可能这个解决方案会存在新的问题，那么就接着质疑并提出另一个解决方案。直到最后，作者才会给出他觉得合理的建议。

**反复地寻找解决问题的解，又反复地质疑自己找到的解。** 在这样的层层递进的方式下，读者读到的并不仅仅是枯燥的知识本身，而是学会如何去思考并寻找问题的最优解。这个思考的过程是及其珍贵的，它让你获得了很多知识以外的东西。

当然，我没有作者这样的功力，我也没有办法通过文字完全还原我的感受与收获。在这里，我就概括性地记录一些我学到的东西。如果想要有比较深刻的认识与感悟，还是建议去读原书。

<br>

## 代码风格

在团队开发中，简洁、风格一致的代码能够大大提高开发的效率。[PEP 8](https://www.python.org/dev/peps/pep-0008/) 介绍了什么样的代码是简洁的代码，以及一些比较好的规范。这里指出一些我觉得比较好的，但是时常被忽视的点：

* 关于函数之间的空行，文件中的全局函数或者是类之间的间隔为 2 个空行。如果是一个类中的方法，相互之间的间隔为 1 个空行。

* 模块导入的顺序应该是，标准库模块、第三方模块、自己本地的模块。在每一个区块中，按模块名称的字母顺序决定先后。

* 不要通过判断长度来判断空，比如 `if len(somelist) == 0`，这样的代码存在视觉干扰，不简洁。正确的做法是把空看作是 False，`if not somelist`。

* 养成习惯，使用空格而不是制表符来完成缩进。

可能对于新人来说，一开始很难把这些风格完完全全的都注意到。但是 Python 提供了像 [Pylint](https://www.biix.com/domain/pylint.com) 这样的代码风格检测工具。它能够在必要的时候提醒你，让你更好更快地养成习惯。

<br>

## 基础数据结构和语法

### 字符与字符串

在程序的数据表示中，最常见的莫过于字符串了。字符串由一个个的字符组成。但是由于存在不同的编码方式，每种语言表示字符的方式都不尽相同。

在 Python3 中，**bytes** 和 **str** 都可以用来表示字符，其中 bytes 表示的是一个 8 bits 的值，而 str 中所表示的是 Unicode 的字符。

而 Python2 中用的又是另外一套机制，**str** 和 **unicode** 用来表示字符，其中 str 表示的是一个 8 bits 的值，而 unicode 表示的是 Unicode 的字符。

由于 Python 中存在两套机制来表示字符串，如果不加限定和约束就会造成混乱。一般地都需要编解码函数来确保类型的统一：

* **编码（encode）**：把 Unicode 字符转换为二进制数据
* **解码（decode）**：把二进制数据转换为 Unicode 字符

在通常情况下，你的大部分程序用 Unicode 字符（Python3 中的 str 类型，Python2 中的 unicode 类型）来表示字符串。但当你把程序中的数据内容导出时，记得将其按照指定的方式进行编码。比如，按照 `UTF-8` 来进行编解码的话，在 Python3 中我们就有下面的编解码函数：

```python
# 解码
def to_str(bytes_or_str):
    if isinstance(bytes_or_str, bytes):
        value = bytes_or_str.decode('utf-8')
    else:
        value = bytes_or_str
    return value # Instance of str

# 编码
def to_bytes(bytes_or_str)
    if isinstance(bytes_or_str, str):
        value = bytes_or_str.encode('utf-8')
    else:
        value = bytes_or_str
    return value # Instance of bytes
```

在 Python2 中也是类似的：

```python
# 解码
def to_unicode(unicode_or_str):
    if isinstance(unicode_or_str, str):
        value = bytes_or_str.decode('utf-8')
    else:
        value = bytes_or_str
    return value # Instance of unicode

# 编码
def to_str(unicode_or_str)
    if isinstance(unicode_or_str, unicode):
        value = unicode_or_str.encode('utf-8')
    else:
        value = unicode_or_str
    return value # Instance of str
```

除了以上这些，关于这两种类型的比较上，Python2 与 Python3 还是有一些区别。在 Python2 中，unicode 类型很多时候可以等同于 str 类型（当表示的是 ASCII 时），两种类型的字符串可以通过 `+` 拼接在一起，如果两种类型数据表示的是同一个字符串，用 `==` 比较也会返回 `True`。但是在 Python3 中，bytes 和 str 被设定为完全不一样的类型，他们之间无法进行直接的拼接和比较。

<br>

### 列表

列表是动态数组，同时也是 Python 里面最常见，使用频率最高的数据结构。并且，列表有着非常灵活的使用方法，往往能让语言的表达更加的简洁。最基本的使用就不讲了，提几个比较重要的点：

* 列表有着非常灵活的范围选取，但是往往为了简洁起见，如果范围中包含开始或者结束，我们则会对其进行省略。比如，我们会用 `a[:5]` 来代替 `a[0:5]`，`a[1:]` 来代替 `a[1:len(a)]`。这样的好处是可以减少视觉干扰，让代码更容易被理解。

* 注意 `b = a[:]` 和 `b = a` 的区别。后者仅仅是地址的拷贝，而前者的 `a[:]` 则是将列表 a 浅拷贝了一份赋给 b。这里的浅拷贝的意思是，对原先列表 a 中的引用类型的数据进行地址的拷贝，非引用类型进行值的拷贝，赋值过后，a，b 指向的将会是两个不同的列表。

* 注意 `a == b` 和 `a is b` 的区别。前者是值的比较，后者是地址的比较。通过下面的例子可以很好的看出它们的区别：

    ```python
    a = [1,2]
    b = [1,2]
    print(a == b) # True
    print(a is b) # False
    a.append(3)
    print(a == b) # False
    ```

* 列表解析提供了便捷的生成列表的方法，比如下面这个例子：
    ```python
    a = [1, 2, 3, 4, 5, 6]
    even_square = [x ** 2 for x in a if a % 2 == 0]
    ```
  当然这种初始化数据结构的方式并不仅限于列表，同样适用于字典和散列：
    ```python
    name_id = {"peter":3, "Alex": 2, "Bob": 4}
    m = {id: name for name, id in name_id.items()}
    s = {id for id in name_id.values()}
    ```
  但是这里要提醒一句，**列表解析虽好，但不能滥用**。如果你发现你需要用到 2 层以上的表达的时候，尽量避免使用这样的表达方式。因为在复杂的情况下，这种方式往往会降低代码的可读性。
  
<br>

### 异常处理机制

这里顺带提一下 Python 的异常处理。Python 中的完整的异常处理机制是 **try/except/else/finally**。与其它语言不同的是，这里 Python 增加了一个 else 语句用于处理 try 中的内容成功执行的情况。

注意，仅当 try 代码块中的语句成功执行后，else 代码块才会被执行。个人感觉这样的设计更加的合理，毕竟给异常处理新增了一个分支可选项。

<br>

## 函数

### 通过异常来反映函数的运行状态而不是返回值

很多时候，我们习惯给函数的返回值加上一些意义。这样一来，函数的调用者就可以通过函数的返回值来判断函数的运行状态。

但这其实是一个误区。如果仔细思考就会发现，按这样的逻辑，函数的返回值就会有两层含义，一层用来表示运行状态，一层用来表示函数的运行结果。这势必就会有冲突，举个简单的例子，比如你用 `None` 来表示函数运行出错，但正常情况下， `None` 也有可能作为返回值返回。那这该如何是好，如果函数返回 `None`，该如何判断函数是出错还是正常退出呢？也许你会说，找一个不可能成为正常情况下的结果的值作为状态返回值不就可以了吗？这也许能够保证程序在特定的情况下不会出错，但这样的设计还是混淆了概念。

函数的最终状态只有两种，**异常** 和 **正常退出**。在正常退出的情况下，我们才会去查看函数的返回结果。而在异常发生的情况下，我们更关注的是代码出错的具体位置和原因，而要更好地反映这些信息，只有通过抛异常的方式来提醒函数的调用者。否则，函数的调用者就会认为函数的返回结果是在正常状态下返回的，是可靠的。

<br>

### 函数的传参

函数传参按理来说应该是一个普通的不能再普通的操作。但是由于需求的复杂化，加上人们总是想尝试把复杂的东西简单化，由此延伸出来了很多不一样的参数传递方式，我们循序渐进地来看一看：

* **普通传参**

  这个不用过多解释，就是按照函数定义的参数挨个传参，如下：
  
  ```python
  def log(message, value1, value2):
      ...
  
  log("log message", 1, 2)
  ```
  
  这种可以算是最原始的参数传递方式，仔细去想，它其实存在如下的问题：
  
  1. 参数的数目固定，当函数投入使用后，如果再想对这个函数增加新的参数，那么改动的范围就会变的很大，不仅要改函数本身，而且还要改动函数的调用。当一个函数被广泛使用后，这样的改动往往会变得非常的棘手。
  
  2. 在函数的调用处很难清楚地知道每个参数的具体意义，比如从 `log("log message", 1, 2)` 中我们只能知道函数的输入参数是一个字符串，两个数字。至于它们所代表的意义就不得而知。
  
  3. 在函数的使用方面，函数存在可选参数。也就是说有些参数如果调用者可传可不传递，不传递不会影响函数的正常运行。而上面的普通传参方式明显做不到。

* **位置参数**

  位置参数允许调用者传递可变数目的参数，示例如下
  ```python
  def log(message, *values):
      print(values)
  
  log("log message", 1, 2) # [1, 2]
  log("log message") # []
  ```
  
  在参数前面加上 `*` 符号表明这个参数将会是一个列表，并挨个存放此位置以及之后位置上的所有参数。在函数声明上， `*` 只可以放在最后一个参数前面。
  
  这种将多个参数合并成一个参数的方式可以有效地减少视觉干扰，同时让函数的传参更加的灵活。但是有时会产生一些难以发现的 bug，所以还是要谨慎使用。
  
* **关键字参数**

  关键字参数能够让函数的调用更加地直观，同时也可以使参数的位置更加地灵活，比如：
  ```python
  def log(message, value):
      ...
  
  log("log message", 1)
  log("log message", value=1)
  log(message="log message", value=1)
  log(value=1, message="log message")
  ```
  
  使用的时候需要注意如下两点：
  1. 如果存在非关键字参数，则必须放在关键字参数之前
  2. 每个参数名只能被指代一次
  
  另外，关键字参数也支持可选和默认值：
  
  ```python
  def log(message, value=1):
      print(value)
  
  log("log message") # 1
  log("log message", value=2) # 2
  ```
  
  对于可选的默认参数，我们在传参的时候最好也使用关键字的形式，这样能更直观地展现参数的意义，也方便对照。
  
 * **强制关键字参数**
 
   上面提到的关键字参数虽说可以让函数调用处的代码更加直观，但是关键字参数是可选的，有时候非关键字参数和关键字参数容易对人造成混淆。强制关键字参数可以很好地解决这种混淆：
   
   python3:
   ```python
   def log(message, *, value1, value2):
       print(value1, value2)
   
   log("msg", value1=1, value2=2) # 1,2
   log("msg", value2=2, value1=1) # 1,2
   log("msg", value2=2, 1) # error
   ```
   
   这里单独用 `*` 将非关键字参数与关键字参数分割开来。确切的讲，`*` 符号表示的位置参数的结束，位置参数后面的关键字参数传参的时候必须严格使用关键字参数进行传参，否则会报错。
   
   Python2 中没有 `*` 分隔符，但是也有类似的表示强制关键字参数的方法：
   
   ```python
   def log(*args, **kwargs):
       print(args, kwargs)
   
   log("msg", value1=1, value2=2) # ('msg',) {'value2': 2, 'value1': 1}
   log("msg", value2=2, value1=1) # ('msg',) {'value2': 2, 'value1': 1}
   log("msg", value2=2, 1) # SyntaxError: non-keyword arg after keyword arg
   ```
   
 上面的传参方式主要目的是让代码变得更加简洁和清晰。但是其中有一点没提到的是关于 **默认参数**，默认参数可以给函数参数更大的灵活性，但是如果默认参数给的初始值是动态的数据结构的化，很多时候会造成意想不到的问题，比如下面的代码：
 
```python
def log(msg, msgs=[]):
    msgs.append(msg)
    print msgs
    return msgs

log("log1") # ["log1"]
log("log2") # ["log1", "log2"]
log("log3") # ["log1", "log2", "log3"]
```

从示例代码可以很容易看到，默认参数的动态初始值不会因为函数的重新调用而被覆盖。如果要给动态的初始值，最好的做法是把初始值设置成 `None`，这样可以减少不必要的误解。


<br>

## 面向对象

### 使用函数作为接口

在传统的编译型语言中，例如 C++ 或者 Java。我们一谈到接口，我们往往都会马上想到 class 或是类似的结构。Python 中也有 class，但是相对于函数这样的结构来说，class 比较庞大，里面往往含有各种属性和方法，并且 class 必须先被实例化生成对象后才能被使用。与此不同的是，函数在 Python 中有很强的灵活性，比如在使用 Python 的一些内置 API 时，函数既可以作为参数传递，又可以作为回调，并且像匿名函数这样简洁的写法有些时候可以使某些过程变得清晰明了（比如排序、过滤等等）。

当然，在函数中，我们没有办法清晰地把属性和方法很好地展现出来。**那 class 生成的对象能不能也有类似像函数这样的灵活性呢？** 答案是肯定的，这得益于 `__call__` 这个特殊方法，比如下面这个例子：

```python
class CountMissing(object):
    def __init__(self):
        self.added = 0
    
    def __call__(self):
        self.added += 1
        return 0

counter = CountMissing()
counter()
assert callable(counter)
```

我们在 `CountMissing` 类中定义了 `__call__` 这个方法，于是**该类实例化出来的对象就可以像函数一样被直接调用**。每次调用这个对象其实是在调用对象中的 `__call__` 方法。

这么做的好处是，我们完全可以把对象当成函数来看待。在使用一些必须传入函数才能使用的内置方法中，例如，`sort`，`defaultdict`，传入对象也可以是方法正常运作。增加了对象的灵活性，同时也降低了代码的复杂度。

<br>

### 类中的访问权限

Python 中 class 的访问权限只有 public 和 private。private 的设定就是在正常的属性或者方法名字前面加上 2 个下划线 `__`。但是这里要说的是，**Python 对 private 的实现只是将成员名称进行了一个简单的转换**，比如下面这个例子：

```python
class MyParentObject(object):
    def __init__(self):
        self.__private_field = 10

class MyChildObject(MyParentObject):
    def get_private_field(self):
        return self.__private_field

baz = MyChildObject()
baz.get_private_field()

>> AttributeError: 'MyChildObject' object has no attribute '_MyChildObject__private_field'
```

可以看到的是，私有成员只是将名字转换成 `_类名` + `成员名` 的形式。如果知道了这层转换，我们依然可以从外部访问私有成员，比如接上面的例子：

```python
baz._MyParentObject__private_field # 10
```

我们也可以通过 `__dict__` 这个特殊方法看到对象中的所有成员：

```python
baz.__dict__ # {'_MyParentObject__private_field': 10}
```

说到这里，你可能会问 Python 中该如何更好地对类成员的访问权限进行设定呢？这里最好的做法是利用 protected 加文档的形式进行管理。把类成员设定成 protected 就是在成员名前加上单下划线 `_`，当然这个前置下划线仅仅是给类的使用者看的，在代码中并没有实际的约束力。我们需要在文档中写清楚哪些成员不应该被调用以及原因。


