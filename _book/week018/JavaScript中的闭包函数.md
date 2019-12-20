## 概要
这周看了 **《JavaScript 高级程序设计》** 的第七章，讲到了 JavaScript 中的一等公民 - 函数的一些特性，在这里进行一下总结。

在一个函数中返回另外一个函数，当我们调用这个函数，我们称调用的这个函数是**外部函数**，其返回的函数是**内部函数**，但是问题来了，正常的情况下，一个函数调用结束时，其内部定义的变量会被清理，但是**如果说函数返回函数，这个被返回的函数中引用了其外部函数中定义的变量，那么这个时候这些变量是会以某种方式被保存起来供内部函数引用，这种情况下就会形成函数闭包**

你可能并不太理解这样做的目的是什么？其实闭包函数隐藏了很多的内部实现，这样调用者不需要关注额外的东西，在有些场合下频繁被使用。而且之前说过，早期的 JavaScript 是没有类的，OOP 基本上都靠函数，闭包函数其实在 OOP 中也有不少的用途，当然它也存在一些缺点，我们在使用的时候需要注意。

<br>

## 函数表达式和函数定义
先说说函数定义吧，在一般的语言中都会支持函数定义，如果你在 JavaScript 中定义一个函数，可以写成下面这个样子：
```js
function func() {
    console.log("this is a function declaration");
}

func(); // this is a function declaration
```
需要注意一点的是这里的函数名不能省，定义在调用前还是后都不受影响。

函数表达式是和函数定义有区别的东西，首先它是一个表达式，这个表达式是将一个匿名函数（**在这里给上名字不会报错，但是名字是无效的**）赋值给一个变量，这个变量就等同于函数名:

```js
const func = function () {
    console.log("this is a function expression");
}

func(); // this is a function expression
```

和之前不同的是，如果是写成函数表达式的形式，那么**调用一定是在赋值之后**，不然会报错。

函数赋值表达式是很多编译型语言所没有的，比如 Java，它确实提供了很多的便利，比如参数传递、多个函数入口等等，但是与此同时它也存在一定的缺陷，比如下面这个例子：
```js
function factorial (num) {
    if (num == 1) {
        return 1;
    }
    
    return factorial(num - 1) * num;
}

let copyOfFactorial = factorial;
factorial = null;
copyOfFactorial(10); // error
```
这个递归的实现中，函数内部调用的变量被在外部设为空，从而导致报错，解决办法就是函数内部调用自身的时候使用 **arguments.callee(num - 1)**。当然另外一种方法就是将函数在括号内定义并赋值：
```js
let copyOfFactorial = (function factorial (num) {
    if (num == 1) {
        return 1;
    }
    
    return factorial(num - 1) * num;
});
factorial = null;
copyOfFactorial(10); // 3628800
```

<br>

## 闭包的形成
在学习一些编程语言的时候，我们或多或少都会听到 **定义域** 这个词，一个变量不能在其定义域以外的地方被访问，比如说你在函数内部定义了一个变量，你不能在函数外部去访问它，不然就会报错或者是显示不存在。但是反过来，内部访问外部是行得通的，比如函数里面是可以访问函数外部的全局变量的，如果在函数内部定义了函数，那么内部的函数也是可以访问外部函数中定义的变量的。这些都可以理解，但是如果说这些东西放在了闭包函数上，你可能不太清楚的是 “**为什么函数调用结束后，返回的函数还可以访问之前的变量，其实现机制是什么？**” 这里要引入一个 **作用域链** 的概念。想想之前的原型链，你应该不难理解，就是查找一个变量的时候，先在本定义域找，然后去到外层函数的定义域，然后是外层的外层，。。。，最后是全局。返回的闭包函数会以级联的方式保存这些作用域中的变量，本层找不到就去到下一层中找，直到全局，JS 中的全局就是指 window 这个 object。

注意闭包保存的是这个环境并不是单单地固定住某个变量，比如下面的例子：
```js
function createFunctions(){ 
  let result = new Array();
  for (var i = 0; i < 10; i++){ 
    result[i] = function() {
        return i;
    };
  }
  
  return result;
}

const result = createFunctions();

console.log(result[8]()); // 10
```
这个例子可以很好地说明闭包保存的是某个环境下，而不是单单地仅仅保存变量在某个时候的值，将上面的函数略微改动一下就会是我们想要的结果：
```js
function createFunctions(){ 
  let result = new Array();
  for (var i = 0; i < 10; i++){ 
    result[i] = function(num){
        return function() {
            return num;
        }
    }(i);
  }
  
  return result;
}

const result = createFunctions();

console.log(result[8]()); // 8
```

闭包确实是一个很巧妙的实现，但是建议还是在一般情况下不要使用，因为从上面的分析你也可以看到，闭包其实是比一般的函数多使用了很多资源，当然合适的情况下是可以使用的，只是说要结合情况来考虑。

<br>

## this 关键字
首先我们回顾一下，在 OOP 中，JavaScript 通过函数构建对象的时候，这里的 **this** 指代的是当前的对象。但是千万不要认为说所有的函数或者对象中的 this 都是指的是当前对象，对于全局函数（区分在 object 中定义的函数）来说，在 nonstrict mode 下，this 指的是 window 对象，strict mode 下，this 是 undefined 的。
```js
const name = "The window";
const object = {
    name : "My Object",
    getNameFunc : function() {
        return function () {
            return this.name;
        }
    }
}

console.log(object.getNameFunc()()); // The window
```
**当一个函数被调用的时候，其自带两个变量，分别是 this 和 arguments**，arguments 是用来存储传入参数的，之前也有提到过，但是这里想说的是内部函数无法获取外部函数的 this 和 arguments，原因也很好理解，这两个参数是跟着函数走的，上面的例子中返回的匿名函数并不隶属于某个对象，因此返回的 this 会在 nonstrict mode 下指向全局的 window 对象，如果想让其指向当前的外部对象，只需要做一点点改变即可：
```js
const name = "The window";
const object = {
    name : "My Object",
    getNameFunc : function() {
        const that = this;
        return function () {
            return that.name;
        }
    }
}

console.log(object.getNameFunc()()); // My Object
```
你可以看到，如果内部函数想要获取 this 或者 arguments 中的信息，我们需要将这两个变量的值传递到另一个变量中即可，这个值就可以通过函数闭包被保存。另外关于对象中的 this 还有一个比较 tricky 的地方：
```js
const name = "the window";

const object = {
    name : "my object",
    
    getName : function() {
        return this.name;
    }
};

console.log(object.getName());      // my object
console.log((object.getName)());    // my object
console.log((object.getName = object.getName)()); // the window
```
后面通过对象调用函数的地方，前面两个应该好理解，直接通过对象调用函数，函数中的 this 指代的是当前的对象，但是第三个语句，括号中先将对象中的函数重新自身赋值了一下，**因为 this 是跟着函数走的，如果重新赋值的话，this 也会更新，并不会被保存**，所以 this 更新后指向了全局的 window 对象。

<br>

## 闭包引起的内存泄漏
先来看下面的代码：
```js
function assignHandler() {
    const element = document.getElementById("someElement");
    element.onclick = function() {
        alert(element.id);
    };
}
```
这里在 element 上面创建了一个事件，是以闭包的形式，当我们调用外部函数 assignHandler 后使用 element.onclick，这时 element.onclick 就是闭包函数，这个闭包会保存外部函数 assignHandler 的 context，你会发现这里有着一个相互引用，就是 element 中有匿名函数，然后匿名函数又引用了 element，按照 JS 的回收机制，一个内存只要是有变量引用那么这个内存就是有效的，不会被回收，只要这个闭包函数存在，那么 assignHandler 中的 element 变量就不会被 JS 中的垃圾回收器发现并清理，这样就会造成内存的泄漏。好的做法如下：
```js
function assignHandler() {
    const element = document.getElementById("someElement");
    const id = element.id;
    
    element.onclick = function() {
        alert(id);
    };
    
    element = null;
}
```
另外有一点需要强调的是，闭包有整个外部函数 context 的引用，当不需要 element 时，我们还是需要将 element 设置为 null，让垃圾回收器来清理

<br>

## 模拟块定义域
Java 和 C++ 中有 **块定义域** 这么一个东西，就是 {} 内部的区域，但是在 JavaScript 中没有这么一个定义域，JavaScript 中的定义域基本上是由函数来划分的，比如下面的例子：

```js
function outputNumbers(count) {
  for (var i = 0; i < count; i++) {
    console.log(i);
  }
  
  var i;
  console.log(i); // 10
}

outputNumbers(10);
```
JavaScript 并不会告诉你，你在何处定义过这个变量，以及你是否定义过这个变量。**我们可以使用匿名函数来模拟块定义域**：
```js
function outputNumbers(count) {
    (function() {
        for (var i = 0; i < count; ++i) {
            console.log(i);
        }
    })();
    
    console.log(i); // error
}
```
其实这里的想法很简单，就是把 `(function() { // block code })` 来代替 {}，括号里的函数是匿名的，也就是说即使这里有闭包函数，闭包并不会保存这个匿名函数的信息，这样使用并不会造成什么不好的影响。尝试过后我发现，**在 ES5 之后引入了 let 和 const 这两个东西，如果将最开始的 for 循环中的 var 用 let 来替代的话，i 是不会在 for 循环外部访问到了**。但是看到前辈们能够想到用函数来解决这个问题，也是学到了不少，同时也体现了函数在 JavaScript 中的不可忽视的地位。

<br>

## 构造函数的私有成员
之前讲过我们可以通过构造函数来创建对象，但是 JavaScript 中的构造函数就只是一个构造函数，它并不会和一个类来绑定，我们也不能直接对函数里面的成员来设定访问权限，但是我们可以通过闭包来间接地来实现：
```js
function MyObject(){
  // private variables and functions 
  var privateVariable = 10;
  function privateFunction(){ 
    return false;
  }
  
  // privileged methods 
  this.publicMethod = function (){
    privateVariable++;
    return privateFunction();
  };
}
```
这里定义了一个 MyObject 构造函数，当用这个构造函数创建对象的时候，privateVariable 和 privateFunction() 这两个成员因为没有被 this 指代，因此是不属于对象的成员，但是在对象的成员 publicMethod 中是可以通过闭包来访问 privateVariable 和 privateFunction()，这也就实现了我们说的私有变量对内部成员公有，对对象私有。

我们也可以通过 **块作用域** 的方式来定义：
```js
(function() {
  let privateVariable = 10;
  
  function privateFunction() {
    return false;
  }
  
  MyObject = function() {};
  
  MyObject.prototype.publicMethod = function() {
    privateVariable++;
    return privateFunction();
  }
})();
```
这里，我们在匿名函数里面定义了 MyObject 变量，但是注意的是这个 MyObject 变量并没有被 var, let 或者是 const 声明，这种情况下，MyObject 成了全局变量，但是也请注意的是，如果是在 strict mode 下是会报错的。还有需要注意的是，如果使用这种方式来创建对象，当创建多个对象，这些对象利用闭包公有这个匿名函数里面的私有成员，一个对象对这些成员的改变会影响到其他的对象。

<br>

## 块模式
有一个设计模式叫做 **单例模式**，对于 Java 和 C++ 一类的语言来说，单例模式意味着一个类只会生成一个对象，用在合适的场景会对资源的利用非常有效率。传统的 JS 中实现单例模式非常简单，直接使用对象赋值的形式：
```js
const singleton = {
    name : ...,
    method: function() {
        ...
    }
};
```
**但是通过前面提到的块作用域可以借助闭包使单例模式中返回的 instance 增加私有成员的访问**：
```js
let singleton = function(){
    //private variables and functions 
    const privateVariable = 10;
    function privateFunction() { 
        return false;
    }
    
    //privileged/public methods and properties 
    return {
        publicProperty: true, 
        publicMethod : function(){
            privateVariable++;
            return privateFunction();
        }
    };
}();
```
这里你可以看成，在匿名函数中定义的成员都是私有成员，最后通过对象赋值形式返回的对象里面定义的都是公有成员。这里匿名函数只会被调用一次，然后将结果返回给 singleton 这个变量，当然 singleton 可以被传递被使用多次。

上面一种实现方式是通过对象赋值的形式来返回参数的，按对象赋值的形式创建的对象都是 Object 的实例，块模式还有另外一种实现方式：
```js
var singleton = function(){
    var privateVariable = 10;
    function privateFunction(){ 
        return false;
    }

    var object = new Person();
    
    object.publicProperty = true;
    object.publicMethod = function(){ 
        privateVariable++;
        return privateFunction();
    };
    
    return object; 
}();
```
其实原理上面是一样的，只是说这种方式可以返回特定的对象，而不仅仅是 Object 类型的对象。

<br>

## 总结
上面讲到的内容全是围绕着 “**闭包**” 和 “**函数**” 这两个东西展开的，函数是 JavaScript 中的一等公民，应用非常的灵活且广泛，可以作为值和参数传递，可以作为返回值返回，可以作为类，还可以用来生成 JavaScript 中所没有的块定义域，闭包是指当函数作为参数返回时，函数会记录其外层定义域中的变量，配上闭包这个东西，函数的功能又增进了，可以间接地实现私有成员了。作为使用了多年 Java 的我，看到这些东西不禁感叹原来函数还可以这么玩。



