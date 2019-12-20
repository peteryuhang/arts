最近开始学习一些基础的 JavaScript 知识，作为初学者，发现了这门语言和其他语言诸多不一样的地方，函数的参数传递就是其中一个，这些东西不像算法，有时需要巧思和严密的逻辑推导，这些东西更像是一些常识类的东西，理解起来没有任何的难度，但是知道和不知道却有着本质的差别，这主要还是靠平时的积累巩固，如果不清楚的话，可能写代码容易出问题，严重影响效率，我们一起来看看

<br>

**值传递与引用传递**

值传递（passed by value）和引用传递（passed by reference）应该是个比较基础的问题，几乎在每个语言中都存在，这里我不想说太多，可以参考 [java基本数据类型传递与引用传递区别详解](https://blog.csdn.net/javazejian/article/details/51192130)，总结下来就是 JavaScript 跟 Java 一样，存在 primitive type 跟 object，传递 object 并不是传递整个 object，而是复制并传递指向 object 的 reference，这么看来所有的传递都可以理解为值传递，就是先复制，再传递，只不过是复制 value 还是 reference 的区别

<br>

**arguments 数组**

arguments 数组是每个函数自带的数组，它里面装的是函数的参数，有以下几点我们要特别注意：
* 可以不通过变量名，而是通过 arguments 数组去访问函数参数，例如 `arguments[0]` 表示的是传进函数的第一个参数，`arguments.length` 表示的是函数接收的参数的总个数
```javascript
var f = function(num1, num2) {
    console.log(arguments[0]); // 1
    console.log(arguments.length); // 2
}

f(1, 2);
```
<br>

* 不管函数定义的时候有几个参数，调用该函数时传递参数的数量永远是随意的，即使参数的数目不匹配，系统也不会报错
```javascript
var f = function(num1, num2) {
    console.log(arguments[0]); // 1
    console.log(arguments.length); // 7
}

f(1, 2, 3, 4, 5, 6, 7);
```
<br>

* arguments 和参数名之间的变化影响是相互的，即改变对应位置的 arguments 会导致命名参数的变化，反之同理：
```javascript
var f = function(num1, num2) {
    arguments[0] = 10;
    console.log(num1);  // 10
    
    num2 = 15;
    console.log(arguments[1]); // 15
};

f(1, 2);
```
<br>

* arguments 并不是 Array 的对象，也没有一般数组的功能性函数，如果调用诸如 pop, push 一类的函数，系统会报错，若试图给 arguments 增添元素，arguments 长度不变
```javascript
var f = function(num1, num2) {
    console.log(arguments instanceof Array);  // false
    
    arguments[3] = 10;
    console.log(arguments.length);  // 2
};

f(1, 2);
```

<br>

**没有函数重载（Overloading）** 

我不太了解是不是所有的动态语言都不支持重载，但是至少在 JavaScript 里面是这样，在函数定义重名的情况下，后面的函数会覆盖前面的函数，如下：
```javascript
function addSomeNumber(num) {
    return num + 100;
}

function addSomeNumber(num, num2) {
    return num + 200;
}

console.log(addSomeNumber(1)); // 201
```

<br>

以上就是我的分享，希望对你有用，如果我有什么地方解释的不够到位，或者没有提及，有错误，还请指正。欢迎也感谢你的评论
