## 为什么要学 ES6

这里说的 ES6 其实是 JavaScript 在 2015 发行的一个版本，全称是 ECMAScript6。这个版本对 JavaScript 这门语言的各个方面都产生了巨大的变化。总所周知，JavaScript 这门语言自从诞生以来就遭受无数的诟病。在这门语言的设计之初，设计者仅仅是将其定位为一门简单的脚本语言，并没有引进很先进的设计原理。但是没有想到的是浏览器的产生，以及随后的 NodeJS 都推动了 JavaScript 的发展。使用场景广了，使用者多了，自然而然这门语言存在的潜在问题都一一浮出水面。ES6 就是为了解决这些问题，并且让其保持向下兼容。

学习 ES6 其实可以很好地了解 JavaScript 这门语言。了解这门语言在此之前存在着哪些问题，为什么这些问题会被称之为问题？ES6 又是如何解决这些问题的。

另外关于 ES6 的这些知识，我主要是看了 [Understanding ECMAScript 6](https://book.douban.com/subject/26864806/) 这本书。作者的叙述简洁明了，并且举的例子也很简单易懂。这篇文章主要是从书中摘抄一些个人觉得比较重要的内容。

<br>

## 块绑定

在 ES6 之前，JS 中的变量只有两个作用域，分别是 **全局** 和 **函数**。在讲作用域之前，我们需要知道 JS 在运行之前其实有一个类似预编译的操作，**这个操作会把所有的变量或者函数的声明都进行上提**。注意这里仅仅是变量的声明上提，并不包括变量的值绑定，比如下面的代码：

```javascript
function getValue(condition) {
    if (condition) {
        var value = "blue";
        // other code
        return value;
    } else {
        // value exists here with a value of undefined
        return null;
    }
    // value exists here with a value of undefined
}
```

如果你对 JS 不了解的话，你可能会简单地认为变量 `value` 的声明仅仅发生于 `condition` 为真的情况，但是由于变量声明上提，上面的代码其实和下面是等同的：

```javascript
function getValue(condition) {
    var value;
    if (condition) { 
        value = "blue";
        // other code
        return value;
    } else {
        return null;
    }
}
```

基于这些，ES6 之前，关于变量的声明和值绑定存在以下几个问题：

* 没有块作用域，变量名的混乱很容易造成问题。
* 由于变量上提，导致变量在出现之前就允许被使用。这会对使用者造成混淆

为了解决上面这两个问题，ES6 引进了块作用域（Block Scopes），也可以被称作是**词法作用域**（lexical Scopes）。同时为了向下兼容老版本，声明有词法作用域的变量，我们需要采用 `let` 和 `const`。这两个关键字在使用上并没有过多的要说的，它们的出现也解决了上面提到的两个问题。

这里解释一下 **TDZ**（temporal dead zone），对于用 `let` 以及 `const` 声明的变量，在作用域内，但在变量的声明之前的这块区域就被称为 TDZ，在这个区域内去访问变量都会导致一个访问错误（这里并不是说变量还没有声明，而是说在词法环境中，变量在 TDZ 中是不可被访问的）。

基于上面讨论的这些，ES6 之后推荐的做法是，一般情况下用 `const` 声明变量或者函数，如果这个变量要改变就使用 `let`。这样做可以避免一些潜在的问题。

<br>

## 字符串

### 字符访问

在 ES6 之前，JS 字符串会被默认是一个 16 bits 的序列，每个字符会被 16 bits （code unit）来表示。而且所有和字符串相关的内置方法也都是基于这一点。但是 Unicode 的出现让这个变成了问题，因为 Unicode 的表示会超过 16 bits。比如下面这个例子：

```javascript
let text = "𠮷";
console.log(text.length);  // 2
console.log(text.charCodeAt(0));  // 55362
console.log(text.charCodeAt(1));  // 57271
```

这里的单个字符在 JS 看来其实是两个字符，因为这个字符其实是被两个 16 bits 的单元表示。

ES6 针对字符串中的字符访问提供了一些方法，比如 `codePointAt`，`codePointLength`, 以及 `fromCodePoint`： 

```javascript
let text = "𠮷a";
console.log(text.charCodeAt(0)); // 55362
console.log(text.charCodeAt(1)); // 57271
console.log(text.charCodeAt(2)); // 97

console.log(text.codePointAt(0)); // 134071
console.log(text.codePointAt(1)); // 57271
console.log(text.codePointAt(2)); // 97

console.log(codePointLength("abc")); // 3
console.log(codePointLength("𠮷bc")); // 3

console.log(String.fromCodePoint(134071)); // "𠮷"
```

同时，我们也可以通过 `codePointAt` 来简单的判断是否能用一个 16 bits 的序列表示

```javascript
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}
console.log(is32Bit("𠮷")); // true
console.log(is32Bit("a")); // false
```

### 内置函数

在 ES6 之前检查一个字符串中是否含有某个子串，只能通过 `indexOf`，这有时显得不那么自然。ES6 提供了 `includes`、`startsWith`、`endsWith` 来做这个事情，它们的用法从方法名就能知道，会更加地简单明了：

```javascript
let msg = "Hello world!";
console.log(msg.startsWith("Hello")); // true
console.log(msg.endsWith("!")); // true
console.log(msg.includes("o")); // true

console.log(msg.startsWith("o")); // false
console.log(msg.endsWith("world!")); // true
console.log(msg.includes("x")); // false

console.log(msg.startsWith("o", 4)); // true
console.log(msg.endsWith("o", 8)); // true
console.log(msg.includes("o", 8)); // false
```

另外一个实用的内置方法是 `repeat`，顾名思义就是把一个字符串进行重复形成一个新的字符串：

```javascript
console.log("x".repeat(3)); // "xxx"
console.log("hello".repeat(2)); // "hellohello"
console.log("abc".repeat(4)); // "abcabcabcabc"
```

这个内置方法主要用途是格式化一些文本的输出内容。

### 模版字面量

模版字面量提供 DSL，就是为某项特定任务而产生的语言或语法，相比一般的编程语言，它的目的更明确。模版字面量主要是为了解决 ES6 之前的一些字符串表示的问题：

* 字符串无法很好地多行表示
* 字符串的格式化的问题
* HTML 转译问题

模版字面量的用法也很简单，其实就是用 \` 代替普通的引号：

```javascript
let message = `Hello world!`;
console.log(message); // "Hello world!"
console.log(typeof message); // "string"
console.log(message.length); // 12
```

同样，模版字面量也可以很方便地处理字符串格式化：

```javascript
let count = 10,
    price = 0.25,
    message = `${count} items cost $${(count * price).toFixed(2)}.`; 
console.log(message); // "10 items cost $2.50."
```

另外，很重要的一点是，模版字面量也属于 JS 的表达式，因此可以相互嵌套表示：

```javascript
let name = "pyhhou",
    message = `Hello, ${
        `my name is ${ pyhhou }`
    }.`;
console.log(message); // "Hello, my name is pyhhou."
```

模版字面量也有一个自定义的用法，就是标签模版。允许用户根据模版字面量的信息，自定义最后的字符串的内容和形式：

```javascript
function passthru(literals, ...substitutions) {
    let result = "";
    // run the loop only for the substitution count
    for (let i = 0; i < substitutions.length; i++) {
        result += literals[i];
        result += substitutions[i];
    }
    // add the last literal
    result += literals[literals.length - 1];
    return result;
}

let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`; 
console.log(message); // "10 items cost $2.50."
```

有一些内置的模版函数，比如 `Shring.raw` 可以让字符串不对特殊符号进行转译：

```javascript
let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1); // "Multiline
                       // string"
console.log(message2); // "Multiline\\nstring"
```

<br>

## 函数

### 默认参数

在 ES6 之前，JS 的函数都是不支持默认参数的。这给函数的传参和参数判断带来了诸多不便。ES6 开始支持默认参数。默认参数给 JS 原先的一些机制带来了些许变化，一起来看看。

在原先的版本中，`arguments` 对象保存着当前函数的所有参数。并且在非严格模式下，参数和 `arguments` 对象也是关联起来的：

```javascript
function mixArgs(first, second) {
    console.log(first === arguments[0]); // true
    console.log(second === arguments[1]); // true
    first = "c";
    second = "d";
    console.log(first === arguments[0]); // true
    console.log(second === arguments[1]); // true
}
mixArgs("a", "b");
```

但是，ES6 中的默认参数和 `arguments` 对象的关系会表现的像之前严格模式下的形式：

```javascript
// not in strict mode
function mixArgs(first, second = "b") {
    console.log(arguments.length); // 1
    console.log(first === arguments[0]); // true
    console.log(second === arguments[1]); // false
    first = "c";
    second = "d"
    console.log(first === arguments[0]); // false
    console.log(second === arguments[1]); // false
}
mixArgs("a");
```

其实，函数参数的传递和变量的声明以及值绑定是类似的，比如下面这些情况都是允许的：

```javascript
function add(first, second = first) {
    return first + second;
}
console.log(add(1, 1)); // 2
console.log(add(1)); // 2
```

```javascript
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1)); // 2
console.log(add(1)); // 7
```

也正因为如此，参数同样存在我们之前提到过的 TDZ 现象，比如下面这些代码就会报错：

```javascript
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1)); // 2
console.log(add(undefined, 1)); // throws an error
```

### 无名参数

在实际的开发中，我们往往会向函数传入多个参数，并且这些参数的个数不确定。虽然在 ES6 之前，`arguments` 看似可以解决这样的问题。但是这种做法并不能很好地保证代码的简洁度，而且这样的代码不易维护和扩展。ES6 引进了剩余参数来解决这个问题：

```javascript
function pick(object, ...keys) {
    let result = Object.create(null);
    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }
    return result;
}
```

关于剩余参数，有两点需要注意的：

* 剩余参数必须存在于函数参数列表的最后，否则会报语法错误
* 在对象的字面量赋值中，剩余参数不能被用在 setter 函数中

### 澄清函数的双重作用

在 ES6 之前，函数既可以被普通调用，也可以被用作对象的生成。在被用来生成对象的时候，可以在函数之前加上 `new`。之所以有这样的双重特性，是因为一般的函数内部都存在有 2 个内部属性（内部槽），分别是 `[[Call]]` 和 `[[Constructor]]`。`[[Call]]` 负责函数的调用和执行。当在函数调用前面加上了 `new`，那么 `[[Constructor]]` 就会被调用，这个内部槽主要负责生成新的对象，并执行函数体，然后将 `this` 绑定到新生成的对象。

在 ES6 之前，我们需要通过 `instanceof` 来决定函数是否用了 `new`：

```javascript
function Person(name) {
    if (this instanceof Person) {
        this.name = name; // using new
    } else {
        throw new Error("You must use new with Person.");
    }
}
var person = new Person("Nicholas");
var notAPerson = Person("Nicholas"); // throws an error
```

但这并不严谨，还是可以通过其它的方式避开这样的判断：

```javascript
function Person(name) {
   if (this instanceof Person) {
       this.name = name; }
   else {
       throw new Error("You must use new with Person.");
   }
}
var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael"); // works!
```

ES6 增加了 `new.target` 这个元属性，这个属性只有在函数使用了 `new` 时才会有值存在：

```javascript
function Person(name) {
    if (typeof new.target !== "undefined") {
        this.name = name;
    } else {
        throw new Error("You must use new with Person.");
    }
}
var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael"); // error!
```

需要注意的是，在函数之外使用 `new.target` 的话将会产生语法错误。

### 箭头函数

这可以算是 ES6 在函数上的一个最大革新。箭头函数其实可以看作是 JS 的匿名函数，并且它有着简洁的语法。另外，箭头函数抛弃了很多传统函数中容易造成混淆的设计，比如箭头函数没有 `arguments` 对象，箭头函数也不能改变 `this` 的绑定，当然也不能被用作构造器来生成新对象。

一起来看看箭头函数都有哪些性质：

* 不存在 `this`、`super`、`arguments`、以及 `new.target` 的绑定
    
    箭头函数中的上述属性的值是外层函数的值

* 不能够在被调用时添加 `new`

    箭头函数没有 `[[Constructor]]` 内部槽

* 没有原型


箭头函数的表现形式会比传统函数简洁不少：

```javascript
let getTempItem = id => ({ id: id, name: "Temp" });

// effectively equivalent to:
let getTempItem = function(id) {
    return {
        id: id,
        name: "Temp"
    };
};
```

箭头函数其实也是为了解决传统函数中的 `this` 的绑定问题，比如下面的代码就因为内层函数默认会自身绑定 `this` 而报错：

```javascript
let PageHandler = {
    id: "123456",
    init: function() {
        document.addEventListener("click", function(event) {
            this.doSomething(event.type); // error
        }, false);
    },
    doSomething: function(type) {
        console.log("Handling " + type + " for " + this.id);
    }
};
```

将内层函数更换为箭头函数就能够很好地解决这个问题：

```javascript
let PageHandler = {
    id: "123456",
    init: function() {
        document.addEventListener("click",
            event => this.doSomething(event.type),
            false
        );
    },
    doSomething: function(type) {
        console.log("Handling " + type + " for " + this.id);
    }
};
```

当然，虽然箭头函数和普通函数相比有这些不同，你仍然可以对箭头函数使用类似 `call()`、`apply()`、`bind()` 这些操作，只是说 `this` 将不会被绑定了：

```javascript
var sum = (num1, num2) => num1 + num2;
console.log(sum.call(null, 1, 2)); // 3
console.log(sum.apply(null, [1, 2])); // 3

var boundSum = sum.bind(null, 1, 2);
console.log(boundSum()); // 3
```

可见，和传统的函数相比，箭头函数被设计成一个轻量级的结构，用法简洁并且不会占用太多的资源。


### 函数调用优化

当在函数结尾的地方调用其他的函数，这里可以将当前函数移出函数调用栈，以此可以节省栈空间，比如下面这个例子：

```javascript
function doSomething() {
    return doSomethingElse();   // tail call
}
```

ES6 在严格模式中，提供了这样的优化：

```javascript
"use strict";
function doSomething() {
    // optimized
    return doSomethingElse();
}
```

注意，这里需要注意 “尾调用” 这个概念，有些调用虽然存在于函数的尾部，但并不属于尾调用：

```javascript
"use strict";
function doSomething() {
    // not optimized - no return
    doSomethingElse();
}
```

```javascript
"use strict";
function doSomething() {
    // not optimized - must add after returning
    return 1 + doSomethingElse();
}
```

```javascript
"use strict";
function doSomething() {
    var num = 1,
    func = () => num;
    // not optimized - function is a closure
    return func();
}
```

尾函数优化在一般的函数调用中起到的效果并不明显，但是如果是递归调用，区别就会非常的大，比如下面这个例子：

```javascript
function factorial(n, p = 1) {
    if (n <= 1) {
        return 1 * p;
    } else {
        let result = n * p;

        // optimized
        return factorial(n - 1, result);
    }
}
```

如果有尾调用优化的加持，即使输入参数特别大，上述函数还是不会产生栈溢出的问题。