
![](https://user-gold-cdn.xitu.io/2019/7/5/16bc073fc34a612b?w=1348&h=888&f=png&s=86520)

## 原型链
你如果熟悉一般静态语言的继承模式，比如 Java、C++，你会发现这些语言的继承其实在代码实现上面都是通过类来进行的，但是问题就是 JavaScript，具体说是在 ES5 以及之前的版本是没有类这个概念的，那么继承如何进行呢？在 JavaScript 的对象创建中，我们知道每个对象都有一个 prototype，也就是它的原型，原型其实也是一个对象，但是一般来说同一类型的对象都会指向同一个原型，这也就导致原型中的成员属性或者是成员函数对对象来说就是静态的，上一小节中我们还提到 prototype 里面有一个构造函数指针指向对应的构造函数，那么可不可以通过改变这个构造函数指针来进行继承呢？答案正是这样，**一个对象中的原型指针指向这个对象的原型，原型也是对象，那么也就是说，原型中还可以有其他的原型指针指向其他类型的原型**，这样就形成了原型链，同时派生类原型中的构造函数指针会被基类的原型指针所覆盖。这时如果要查找一个对象的属性或者方法，首先在对象中去寻找，没有就会去到原型中寻找，再没有的话，就会去到原型中的原型寻找，。。。于是你会发现这其实就是我们平时说的继承：
```javascript
function SuperType () {
  this.array = ["super"];
}

SuperType.prototype.superAction = function () {
    console.log("super action");
}

function SubType () {
  this.subArray = ["sub"];
}

SubType.prototype = new SuperType();

SubType.prototype.subAction = function () {
  console.log("sub action");
}

const instance = new SubType();

console.log(SuperType.prototype.isPrototypeOf(instance)); // true
console.log(instance instanceof SuperType);  // true
console.log(instance.array); // ["super"]
console.log(instance.subArray); // ["sub"]
instance.superAction(); // super action
instance.subAction(); // sub action
```
这里可以看到，**我们可以通过将派生类型的原型指针指向一个新的基类对象来实现继承**，继承后的派生类对象可以访问到基类和基类原型中的成员属性和成员方法，用这样的方式来实现继承虽然简单粗暴，但是基本还是可以实现继承后的基本特性，比如说多态，**如果我们在派生类型的对象中，或者是派生类对象的原型中添加基类已有的方法，根据我们刚刚讲的范围寻找（对象 -> 当前对象的原型 -> 下一层基类），你会发现这时对象中的方法会覆盖基类的同名方法，这其实就是多态**。

<br>

## 构造函数窃取
上面的原型链你不难发现其实这里面是有缺陷的
```javascript
function SuperType () {
  this.array = [];
}

function SubType () {
  this.subArray = ["sub"];
}

SubType.prototype = new SuperType();

const instance = new SubType();
const instance2 = new SubType();

instance2.array.push("super");

console.log(instance.array); // ["super"]
```
原型链的缺陷其实就是之前讲的原型的缺陷（所有的对象分享一个原型，那么一个对象改变原型会导致其他对象的原型也改变），当然这里还有一点就是，其实新建派生类的对象是不会重新创建基类对象的，也就是说上面的例子 SuperType 的对象只会在 `SubType.protoType = new SuperType()` 这个时候产生，那么所有的派生类对象分享一个基类对象，这也就会出现我们上面提到的问题。

解决办法当然很简单，就是确保每个派生类对象都会被分配一个基类的对象，代码实现如下：
```javascript
function SuperType () {
  this.array = [];
}

function SubType () {
  SuperType.call(this);
}

const instance = new SubType();
const instance2 = new SubType();

instance2.array.push("super");

console.log(instance.array); // []
```
和之前代码上的唯一区别就是在 SubType 函数中添加了一行 `SuperType.call(this)`，保证每个派生类对象都能被分配到单独的基类对象。

<br>

## 混合继承
其实上面的构造函数窃取也是有缺陷的，就是会有不必要的资源的浪费，怎么讲？如果说基类中有函数的话，那么这个函数会被复制很多次，还记得之前讲过的构造器函数创建对象的方式吗？我们可以把函数放在原型当中来避免这种不必要的资源浪费，这里同理，我们可以把对象的属性通过构造函数窃取的方式继承，而静态属性或者是一些通用的方法函数使用原型链的方式来继承：
``` javascript
function SuperType () {
  this.array = [];
}

SuperType.prototype.superAction = function () {
  console.log("super action");
}

function SubType () {
  SuperType.call(this);
}

SubType.prototype = new SuperType();

const instance = new SubType();
const instance2 = new SubType();

instance2.array.push("super");

console.log(instance.array); // []
```
这也是在实际当中最常用的一种模式，既保证了继承的灵活性，又节省了资源。

<br>

## 寄生混合继承
你可能会觉得前面讲的混合继承已经很好了，但是如果要追求极致的话，不妨看看寄生混合继承。如果你看混合继承中的代码你会发现，我们在使用原型链指定继承的基类的时候创建了一次基类对象，然后后面创建派生类对象的时候又会通过构造函数窃取创建新的基类对象，**但是现在这里的问题是，使用原型链的继承方式指定基类有必要重新创建新的基类对象吗？** 在混合继承下，我们使用构造函数窃取继承基类构造函数中的那些属性，比如之前的例子中的 `array`，我们使用原型链继承基类的原型中的静态成员和方法，这也就是说原型链仅仅是继承的基类的原型，我们并不需要重新构建一个基类对象，于是我们可以以此为突破口，在讲寄生混合继承之前，我们先讲讲什么是**寄生继承**：

```javascript
function object(o){
  function F(){}
  F.prototype = o;
  return new F();
}

function parasitic(o) {
  let another = object(o); // 创建新对象
  another.action = function () {
    console.log("action");
  }
  
  return another;
}

const person = {
  name = "Peter",
  age = 25
}

const anotherPerson = parasitic(person);
anotherPerson.action(); // action
```
寄生继承其实很好理解，就是在一个对象的基础之上，通过函数的方式构建一个新的对象，这个新的对象在保留了之前对象中的属性和方法的同时，也会被新添加新的属性和方法。把这种思想用在我们的混合继承上，在代码实现上就如下：
```javascript
function inheritPrototype(subType, superType) {
  const prototype = Object(superType.prototype);
  prototype.constructor = subType;
  subType.prototype = prototype;
}

function SuperType () {
  this.array = ["Bob"];
  this.name = "Fieer";
}

SuperType.prototype.read = function read() {
  console.log("read");
}

function SubType () {
  SuperType.call(this);
  this.subArray = ["tim"];
}

inheritPrototype(SubType, SuperType);

const instance = new SubType();
const instance2 = new SubType();

instance2.array.push("Peter");

console.log(instance instanceof SuperType);
console.log(SuperType.prototype.isPrototypeOf(instance));
console.log(instance.array); // ["Bob"]
console.log(instance2.array); // ["Bob", "Peter"]
```
当使用混合继承方式的时候，我们可以避免在指定基类的时候重新构建基类对象，节省了资源。但是这里有一点需要特别注意的是，`inheritProtoType()` 函数是将基类的原型复制一份给派生类，如果派生类需要用到原型来存一些静态成员，那么请将这些操作放在 `inheritProtoType()` 函数之后，不然之前在派生类原型上的操作会因此被覆盖。

<br>

## 总结
关于 JavaScript 中的对象的知识就介绍到这里，把这两次的内容放到一起，我们可以得到下面这个**知识地图**：


![](https://user-gold-cdn.xitu.io/2019/7/6/16bc4e29d14a505c?w=1816&h=1254&f=png&s=172811)

回到之前的问题，JavaScript 为什么能够在没有类的基础上实现面向对象？这个问题现在应该也不难解释了，一切都在这个图中，细致的去想，图中的模式也很好地展示了面向对象中的**抽象、封装、继承和多态**，只是说实现的方法和一般我们所了解的语言不太一样。从这张图中你也可以看到知识的迭代，这张图中的每个圆圈其实是一个个知识点，如果你把这些知识点换做成技术，你会发现一个新技术的出现其实是为了解决或者说是弥补之前技术的不足和缺陷，**每当我们学习一项新的技术的时候，不妨去想想这个技术是从其他的什么技术演变而来的，它为了解决什么问题，这些问题为什么之前的技术解决不了？将你的知识体系化，这样久而久之就能够融会贯通**。