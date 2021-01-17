**导言** <br>

> 如果你接触 JavaScript 不久，你会觉得这门语言像是一个熟悉的陌生人。它的语法和 Java 等类 C 语言很接近，但是程序运行的机制又和这些语言有明显的区别。如果把它当成是我们所熟悉的语言来写，时常会带来 “惊喜”。借着这个机会分享一下 JS 中常见的误区，这些误区都是作者本人在日常开发中踩过的坑。欢迎评论和指正。

<br>


## 位运算

JS 中常见的位运算符有 `&`、`|`、`^`、`~`、`>>`、`>>>`、`<<`。这里我并不想解释这些运算符的功能和意义。JS 的语法借鉴了 Java，如果你对 Java 有所了解，应该不会对这些符号感到陌生。我这里想说的是，这些位运算符在底层实现上有着和其他语言本质的区别。

在 Java 中，这些位运算符只能作用于整型（int）。但是 JS 中没有整型，JS 中的 `number` 其实是一个 64 位的浮点型（类似 double）。**每当你使用这些位运算符的时候，JS 底层都会把相关的操作数先转换成 int，执行完操作后，然后再将结果转换回 `number`。** 这一来一去，时间消耗并不低。

在我们的传统理解上，位运算应该是比较接近计算机底层的操作。并且，它的速度应该远在普通运算之上。但在 JS 中并不是这样。JS 中的位运算离计算机底层很远，而且运行速度很慢。在 JS 中我们应该尽量少地去使用位运算符。

<br>

## new

如果刚接触 JS 不久的话，可能会对 `new` 这个关键字产生困惑。对于有些对象的创建，使用或者不使用 `new`，貌似都能得到结果。比如下面这个例子：

```javascript
const d = Date();
console.log(d); // 'Thu Dec 31 2020 11:25:40 GMT-0800 (Pacific Standard Time)'

const dd = new Date();
console.log(dd); // 2020-12-31T19:25:52.102Z
```

上面的例子展示了，对于 `Date` 这个内置函数，好像加不加 `new` 关系都不是太大，程序均不会报错。但是最后的结果会有区别。

这里解释说明下，具体加不加 `new` 得看这个函数的具体实现。按照 JS 惯例，首字母大写的函数属于构造器函数。一般通过构造器函数创建对象，都是需要加上 `new` 的。当然，惯例仅仅是惯例，并不意味着人人都会遵守。如果加上 `new`，对象的创建就会通过函数原型中的成员来实现，并且新生成对象中的 `this` 会被绑定到新生成的对象本身。如果不加，可能也会达到预期，但是 `this` 会被绑定到全局对象，而不是新生成的对象。这样的话，很可能带来出乎意料的结果。当然，有些内置的函数在你不加 `new` 的时候是会报错的，比如：

```javascript
const a = Map();
// Uncaught TypeError: Constructor Map requires 'new'
//    at Map (<anonymous>)
```

不管如何，面对一个未知的构造器函数时，我都建议你在使用之前花些时间查看文档。看看这个函数的具体用法，如果有时间的话，也可以看看其内部的实现原理。只有在了解了这些东西以后，你才会更有把握，写出来的程序更不容易出错。

<br>

## 分号

JS 对行尾加不加分号并没有严格的要求。貌似我们加不加分号，好像都没太大区别。但某些情况下还真有区别，比如下面这个例子：

```javascript
function testSemi() {
    return
    {
    	status: 'success',
    }
}
```

上面的例子的中的 `testSemi` 函数本意是要返回一个 object 的。但是最后返回的是 `undefined`。

究其原因，JS 有一个自动插入分号的机制，如果你不加分号，它会在该加分号的地方补上分号。上面的程序会被 JS 改变成下面这样：

```javascript
function testSemi() {
    return;
    {
    	status: 'success',
    }
}
```

于是，为什么最后函数返回的是 `undefined` 就说得通了。就我而言，写 JS 程序的时候，我还是会加上分号，而且还会检查大括号的位置（左括号应该放在之前行的结束而不是下一行的开始）。看似是风格的问题，到最后会演变成程序的错误。还是养成一个好的习惯，不要偷懒。

<br>

## NaN

`NaN` 是 JS 中独有的一个关键字，表示的意思是 `Not a Number`。这个关键字表示的其实是一个计算上的错误。当你在表达式中，试着把 `String` 类型的变量转换成 `Number` 类型的变量时，如果转换失败，就会生成 `NaN`。比如：

```javascript
const a = 1 * '1';
console.log(a); // 1
const b = 0 * 'b';
console.log(b); // NaN
```

另外，任何数和 `NaN` 相互计算的结果还是 `NaN`。如果一个式子的结果是 `NaN`，那么这个式子中参与计算的变量之中有可能有 `NaN`，或者 `NaN` 是在某一步的计算产生的。`NaN` 的诡异之处在于它的性质：

```javascript
typeof NaN === 'number' // true
NaN === NaN // false
NaN !== NaN // true
```

基于上面的这些，你是不是有个疑惑，那就是如何判断一个变量是不是 `NaN`。JS 提供了一个函数来区别 `number` 和 `NaN`：

```javascript
isNaN(NaN); // true
isNaN(0); // false
isNaN('abc'); // true
isNaN('0'); // false
isNaN(Infinity) // false
```

你会发现这个 `isNaN` 只能区别 `number` 和 `NaN`。这个方法在实际当中用的并不多。实际中更多的是使用 `isFinite` 这个方法，它不仅可以排除 `NaN`，还可以排除 `Infinity`，通常用于判断一个变量能不能参与算术计算。

<br>

## 函数声明与函数表达式

JS 中的函数可以单独声明，也可以将其作为值存入到变量中。但是两种写法有一个区别，如果写成函数声明，那么你可以在函数声明之前调用这个函数，比如：

```javascript
abc(); // abc
function abc() {
    console.log("abc");
}
```

但是如果是函数表达式，那么必须在定义之后使用。因为在函数表达式中，函数是作为值存入变量中的，那么对于变量而言，只有先定义再使用。但是如果是把函数写成是函数声明的形式（类似上面的例子），JS 预处理机制会有一个上提的动作，就是把声明的函数提到全局顶上，这样在程序运行过程中，在任何地方调用之前声明的函数都是可行的。

<br>

## array.sort

之前我们提过 JS 中的数组其实是 object。对这一点不熟悉的话，可以看[上一期的文章](https://juejin.cn/post/6910689763016065032)。`sort` 是数组对象中的一个内置方法，你可以传入自定义的 `compare` 方法，让它按照一定的方式进行排序。但是很多时候，我们需要的仅仅是简单的排序，因此我们很可能会忽略这个 `compare` 方法。如果你不传入 `compare` 方法也是可行，它就会按它默认的方式进行排序。但是最后的结果很可能出乎你的意料：

```javascript
const arr = [1, 2, 11, 22, 33];
arr.sort();
console.log(arr); // [1,11,2,22,33]
```

**`sort` 方法默认的 `compare` 方法会假设待排序的元素都是 String，然后按照 String 的排序方式去对输入的变量进行排序**。于是就会产生上面例子中的结果。

在使用 `sort` 方法时，建议还是老老实实地去实现 `compare` 函数吧，即使你的需求非常的简单。

<br>

## 枚举与迭代

JS 中对枚举的定义是列出一个结构中所有的属性，包括其原型中添加的内容，对于枚举，最常见的遍历方式就是 `for in`：

```javascript
Object.prototype.objCustom = function() {};
Array.prototype.arrCustom = function() {};

const iterable = [3, 5, 7];
iterable.foo = 'hello';

for (const i in iterable) {
  console.log(i); // logs 0, 1, 2, "foo", "arrCustom", "objCustom"
}
```

另外一个很类似的东西是 **迭代**。同样是遍历，**迭代** 只会遍历可迭代的属性，常见的方式是 `for of`：

```javascript
Object.prototype.objCustom = function() {};
Array.prototype.arrCustom = function() {};

const iterable = [3, 5, 7];
iterable.foo = 'hello';

for (const i of iterable) {
  console.log(i); // logs 3, 5, 7
}
```

这里需要把 **枚举** 和 **迭代** 区别开，每次做函数遍历的时候都要想清楚。

<br>
