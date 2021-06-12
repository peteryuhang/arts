## 对象的功能扩展

总所周知，JS 对 object 的依赖特别大，不管是函数还是数组，本质上其实都是对象。ES6 对现有的对象的功能进行了一些优化，可以让使用者更好地操控对象。

### 语法扩展

在 ES6 之前，我们会以下面的方式来 返回/生成 新的对象：

```javascript
function createPerson(name, age) {
    return {
        name: name,
        age: age
    };
}
```

ES6 把对象的属性对应给简化了，如果是相同名称的键值对，可以简写成一个：

```javascript
function createPerson(name, age) {
    return {
        name,
        age
    };
}
```

除了属性有改动，对象中的方法的初始化语法也做了调整，在 ES6 之前，在对象中定义方法的语法如下：

```javascript
var person = {
    name: "pyhhou",
    sayName: function() {
        console.log(this.name);
    }
};
```

ES6 可以省略 `function` 关键字，并且以这种方式定义的方法内部可以使用 `super`，这也是这种语法和传统语法唯一的区别：

```javascript
var person = {
    name: "pyhhou",
    sayName() {
        console.log(this.name);
    }
};
```

另外，为了对象中的 key 可以使用变量来定义，ES6 也对此进行了补充，类似 value，你可以使用任意的表达式来定义 key，不同的是需要加上 `[]`：

```javascript
let lastName = "last name";
let person = {
    "first name": "hou",
    [lastName]: "pyh"       // **
};

console.log(person["first name"]); // "hou"
console.log(person[lastName]);  // "pyh"
```

关于语法，另外一点变化是，ES6 之前的严格模式下，如果对象中有重名的 key，会直接报错，但是 ES6 的严格模式下下，后面的定义会覆盖前面的：

```javascript
"use strict";
var person = {
    name: "Nicholas",
    name: "Greg" // syntax error in ES5 strict mode，no error in ES6 strict mode
};
```

### 新的内置方法

在 JS 中可以用 `==` 和 `===` 来比较元素是否相等，一个弱比较，只需要保证值一样就行。另一个强比较，必须保证值和类型都相同，虽然看似很完美，但是还是有漏洞：

```javascript
console.log(+0 == -0);    // true
console.log(+0 === -0);   // true
console.log(NaN == NaN);  // false
console.log(NaN === NaN); // false
```

`+0` 和 `-0` 在 JS 中的二进制的表示上是不相同的，但是不管是 `==`，还是 `===` 都认为这两个值是相同的。另外 `NaN` 和自身比较会被认为不同。ES6 为了解决上述问题，引进了 `Object.is()` 内置方法来更准确地进行强比较：

```javascript
console.log(Object.is(+0, -0));    // false
console.log(Object.is(NaN, NaN));  // true
console.log(Object.is(5, 5));      // true
console.log(Object.is(5, "5"));    // false
```

另外 JS 中比较常见的操作是，将一个对象中的属性和方法拷贝到另一个对象中去：

```javascript
function mixin(receiver, supplier){
    Object.keys(supplier).forEach(function(key) {
        receiver[key] = supplier[key];
    });
    return receiver;
}

function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
};
var myObject = {};
mixin(myObject, EventTarget.prototype);
myObject.emit("somethingChanged");
```

每次都需要定义一个 `mixin` 函数，这其实不太方便。ES6 引进了 `Object.assign` 内置方法来做这个事情：

```javascript
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
}
var myObject = {}
Object.assign(myObject, EventTarget.prototype);
myObject.emit("somethingChanged");
```

`Object.assign` 可以接受任意多的被拷贝对象，相比之前来说，这会更加方便。

### 原型加强

在 ES6 以前，原型是不允许更改的。但是这样的话，一旦对象被初始化就没法更改对象的原型，在某些情况下不太方便。ES6 提供了 `Object.setPrototypeOf()` 方法对对象的原型进行更改：

```javascript
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// prototype is person
let friend = Object.create(person);
console.log(friend.getGreeting());
console.log(Object.getPrototypeOf(friend) === person); // true

// set prototype to dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting()); // "Woof" 
console.log(Object.getPrototypeOf(friend) === dog); // true
```

除此之外，ES6 还提供了 `super` 来对原型进行访问。在 ES6 之前，则是需要借助 `Object.getPrototypeOf(this)` 的值，并且有时还需要对 `this` 进行绑定，可见并不直观。

```javascript
let friend = {
    getGreeting() {
        // this is the same as:
        // Object.getPrototypeOf(this).getGreeting.call(this)
        return super.getGreeting() + ", hi!";
    } 
};
```

使用 `super` 的时候注意一点，方法的表示语法必须是 ES6 的这种简洁形式，否则会报错：

```javascript
let friend = {
    getGreeting: function() {
        // syntax error
        return super.getGreeting() + ", hi!";
    }
};
```

另外一个比较重要的变化是，ES6 的对象的方法都含有一个 `[[HomeObject]]` 的内部槽，这个属性涵盖方法所在的对象，而普通的函数不具有这个内部槽。`super` 也是就是根据 `[[HomeObject]]` 来进行判断，以及原型中属性或者方法的索取。

<br>

## 解构赋值

解构赋值是 ES6 中特有的一个语法，它可以让代码变得更加地简洁，这主要针对的是对象和数组。

### 对象解构

举个例子，你就可以看得到区别了：

```javascript
let node = {
    type: "Identifier",
    name: "foo"
};

// ES6 之前
// let type = node.type,
//     name = node.name;

// ES6
let { type, name } = node;
```

可能从例子中看不到特别大的区别，但是你要知道，在 JS 中，从对象或者对象数组中取值是一个非常频繁的操作。如果没有一个简洁的语法，那么代码会变得非常的长，并且中间会充斥着很多重复代码，以至于难以阅读。并且这样看似简单的语法其实还是有着许多的高阶用法，我们一起来看看。

**对已经存在的变量进行赋值**：
```javascript
let node = {
    type: "Identifier",
    name: "foo"
},
type = "Literal",
name = 5;
// assign different values using destructuring
({ type, name } = node);
console.log(type); // "Identifier"
console.log(name); // "foo"
```

**默认参数**：

```javascript
let node = {
    type: "Identifier",
    name: "foo"
};
let { type, name, value = true } = node;
console.log(type); // "Identifier"
console.log(name); // "foo"
console.log(value); // true
```

**声明不同名称的变量**：

```javascript
let node = {
    type: "Identifier",
    name: "foo"
};

let { type: localType, name: localName } = node;
console.log(localType); // "Identifier"
console.log(localName); // "foo"
```

**嵌套的对象解构**：
```javascript
let node = {
    type: "Identifier",
    name: "foo",
    loc: {
        start: {
            line: 1,
            column: 1
        },
        end: {
            line: 1,
            column: 4
        }
    }
};
// extract node.loc.start
let { loc: { start: localStart }} = node;
console.log(localStart.line); // 1
console.log(localStart.column); // 1
```

总之，对象解构的规律就是，分号 `:` 左边的东西表示的是在对象当前层搜索的目标，分号右边的东西是可选项，表示的是一个标识符，用于接收赋值，如果没有，默认左边的内容作为标识符。如果右边是一个大括号，则表明要进到下一层进行搜索。

### 数组解构

数组解构和对象解构非常的类似。只不过在数组中，没有 key 需要考虑，情况相对来说会简单些：

```javascript
let colors = [ "red", "green", "blue" ];
let [ firstColor, secondColor ] = colors;
let [ , , thirdColor ] = colors;
console.log(firstColor); // "red"
console.log(secondColor); // "green"
console.log(thirdColor); // "blue"
```

利用数组的解构特性，可以很方便地交换两个元素：

```javascript
// swapping variables in ECMAScript 6
let a = 1,
    b = 2;
[ a, b ] = [ b, a ];
console.log(a); // 2
console.log(b); // 1
```

和对象解构类似的，数组解构也支持默认参数，以及嵌套解构：

```javascript
let colors = [ "red" ];
let [ firstColor, secondColor = "green" ] = colors;
console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

```javascript
let colors = [ "red", [ "green", "lightgreen" ], "blue" ];
// later
let [ firstColor, [ secondColor ] ] = colors;
console.log(firstColor);  // "red"
console.log(secondColor); // "green"
```

另外，数组解构还支持剩余参数（类似函数参数接收）：

```javascript
let colors = [ "red", "green", "blue" ];
let [ firstColor, ...restColors] = colors;
console.log(firstColor); // "red"
console.log(restColors.length); // 2
console.log(restColors[0]); // "green"
console.log(restColors[1]); // "blue"
```

数组解构和前面提到的对象解构还可以混在一起用：

```javascript
let node = {
    type: "Identifier",
    name: "foo",
    loc: {
        start: {
            line: 1,
            column: 1
        },
        end: {
            line: 1,
            column: 4
        }
    },
    range: [0, 3]
};
let {
    loc: { start },
    range: [ startIndex ]
} = node;
console.log(start.line); // 1
console.log(start.column); // 1
console.log(startIndex); // 0
```

### 参数解构

解构不仅仅可以用在变量声明和赋值上，也可以用作函数的参数接收，在 ES6 之前，函数参数的解构是麻烦的一件事：

```javascript
// properties on options represent additional parameters
function setCookie(name, value, options) {
    options = options || {};
    let secure = options.secure,
        path = options.path,
        domain = options.domain,
        expires = options.expires;
        // code to set the cookie
}
// third argument maps to options
setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```

参数解构的引入，让代码变得简洁不少：

```javascript
function setCookie(name, value, { secure, path, domain, expires }) {
    // code to set the cookie
}
setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```

但是注意几个问题，解构的参数必须存在，否则会报错，比如：

```javascript
function setCookie(name, value, { secure, path, domain, expires }) {
    // code to set the cookie
}
// error!
setCookie("type", "js");
```

当然你也可以借助默认参数来解决这个问题：

```javascript
function setCookie(name, value, { secure, path, domain, expires } = {}) {
    // code to set the cookie
}
```

<br>

## 符号

在 ES6 之前，JS 中有 5 个基础数据类型，分别是 `String`、`Number`、`Boolean`、`null`、 `undefined`。ES6 引进了一个新的基础数据类型 `Symbol`。这个数据类型是为了解决对象的属性名称的问题，在这之前，对象的属性名只能是字符串，并且没有访问权限的设定。`Symbol` 可以允许开发人员定义非字符的属性名，并且这些属性被设定成私有的，不会被一般的语句所检测到，增加了属性的隐蔽性。这样也就保证了，在遍历对象的时候，这些 Symbol 标识的特殊属性不会被遍历到。

Symbol 的创建和使用也非常的简单：

```javascript
let firstName = Symbol("first name"); 
let person = {};
person[firstName] = "pyhhou";
console.log("first name" in person); // false
console.log(person[firstName]); // "pyhhou"
console.log(firstName); // "Symbol(first name)"
```

注意，因为 `Symbol` 是基础数据类型，在创建的时候前面不能加 `new`，否则会报错。`Symbol` 的描述是被存在于一个叫做 `[[Description]]` 的内部槽中，当 `Symbol` 的 `toString` 方法被调用的时候，内部会去访问 `[[Description]]` 内部槽拿到 `Symbol` 的描述。

因为 `Symbol` 是基础类型，所以你 `typeof` 可以识别 `Symbol` 类型的变量：

```javascript
let symbol = Symbol("test symbol");
console.log(typeof symbol); // "symbol"
```

ES6 里面有一个全局共享的 Symbol，就是定义之后会被存在一个类似 global/window 的地方，如果再次被定义则直接返回：

```javascript
let uid = Symbol.for("uid");
let object = {
    [uid]: "12345"
};
console.log(object[uid]);
console.log(uid);
let uid2 = Symbol.for("uid");

console.log(uid === uid2); // true
console.log(object[uid2]); // "12345"
console.log(uid2); // "Symbol(uid)"
```

关于 Symbol，可能在一般情况下用的比较少。这是因为通常来说，用户不会直接在业务代码中使用。但是 Symbol 在 JS 内部底层还是频繁被用来实现一些常见的功能。比如下面这些：

**Symbol.hasInstance 方法**

ES6 实现 `obj instanceof Array` 其实是通过 `Arra[Symbol.hasInstance](obj)` 来做到的。当然你可以使用 `Object.defineProperty` 方法来改变这个 Symbol 所表示的东西：

```javascript
function MyObject() {
    // empty
}
Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(v) {
        return false;
    }
});
let obj = new MyObject();
console.log(obj instanceof MyObject); // false
```

**Symbol.isConcatSpreadable 方法**

JS 中的 Array 可以和其它的 array 或者是其他类型的变量组合在一起：

```javascript
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ], "brown");

console.log(colors2.length); // 5
console.log(colors2); // ["red","green","blue","black","brown"]
```

之所以只有 `Array` 类型的变量可以这样，其实是 JS 内部对 Array 做了区别对待。那如何让一般的对象也有相同的属性呢？那我们就可以使用 `Symbol.isConcatSpreadable` 符号。注意，这个符号不会默认出现于任何的的标准对象中。它是一个 `boolean` 的属性，如果为 `true` 则表示对象具有 `length` 属性，以及数字的 Key，这也就是 `Array` 对象的基本特性。

```javascript
let collection = {
    0: "Hello",
    1: "world",
    length: 2,
    [Symbol.isConcatSpreadable]: true
};
let messages = [ "Hi" ].concat(collection); 
console.log(messages); // ["hi","Hello","world"]
console.log(messages.length); // 3
```

**Symbol.toPrimitive 方法**

当你用一个 string 类型的变量和一个 object 类型的变量进行 **弱比较** 的时候，`str == obj`。这时 obj 对象需要去事先做一个类型转换，但是问题是，obj 需要知道要转换成什么样的类型。这个隐藏的类型转换过程就是由 `Symbol.toPrimitive` 方法控制的。

```javascript
function Temperature(degrees) {
    this.degrees = degrees;
}
Temperature.prototype[Symbol.toPrimitive] = function(hint) {
    switch (hint) {
        case "string":
            return this.degrees + "\u00b0"; // degrees symbol
        case "number":
            return this.degrees;
        case "default":
            return this.degrees + " degrees";
    } 
};
var freezing = new Temperature(32);
console.log(freezing + "!"); // "32 degrees!"
console.log(freezing / 2); // 16
console.log(String(freezing)); // "32°"
```