## JavaScript 中的数组

对大多数编程语言来说，数组都是基础中的基础。理解数组的性质及其用法是学好一门编程语言的必经之路。这篇文章提及数组并不是想解释说明在 JavaScript 中如何使用数组，而是想将 JavaScript 中的数组和常见编程语言（如 Java，C，Python）中的数组进行对比，找出那些不一样的地方。因为 **JavaScript 中数组的使用语法和 Java 等类 C 语言是及其相似的，但性质截然不同**。有些时候我们会把在其他语言上的编程思想和习惯带到 JavaScript 中来，导致最后的结果和我们想的完全不一样。如果花些时间来了解下这些东西，相信很多坑是完全可以避免的。好了，废话不多说，我们开始今天的话题。

一般说来，数组意味着一段连续的存储空间，我们可以通过数组的下标快速找到所对应的元素。这个是常识，甚至可以说是在我们脑海中根深蒂固的常识。有些时候我们还会用数组来优化时间和空间。但遗憾的是，数组在 JavaScript 中并不是这样。JavaScript 中并不存在我们上面所提到的数组，**JavaScript 中的数组并不是数组，而是对象**。什么意思？我们来看一个例子：

```javascript
var nums = ['zero', 'one', 'two', 'three'];
```

`nums` 是一个 “数组”，JavaScript 内部会将这个 “数组” 做一些转换，变成类似下面这个样子：

```javascript
{
    '0': 'zero', '1': 'one',
    '2': 'two', '3': 'three'
}
```

你可能会说了，最后这东西不就是 JavaScript 中的 Object 吗，为什么要做这样的转换，这样数组不就没意义了吗？

的确，这样看来，这里的数组不再是连续存储空间了，也不能帮我们优化时间空间。但是你别忽视了 object 的优势所在，object 里面可以存放函数，这样会让数组的使用更加灵活，比如下面的数组常见操作也解释的通了：

```
var nums = [];
nums.push('one');
nums.sort();
nums.map(...);
...
```

这里的 `push`, `sort`, `map` 等方法都是通过原型链内置在 “数组” 中的一些方法，这里我并不想过多的解释原型链是什么，但还是说一下这里的数组和对象的区别。每当我们通过类似 `var a = {};` 的语法去生成一个对象时，生成的对象并不是空的，而是下面这样：

```javascript
{
    "_proto_": {...}
}
```

当你通过类似 `a[name]` 或者是 `a.name` 这样的语法从对象中通过 key 值寻找变量或者函数时，Javascript 首先会在当前对象中寻找，找不到的话，它就会去 `_proto_` 这个子对象结构中寻找，这个结构中存放着很多对象内置的方法。然而，当我们使用类似的语法 `var b = [];` 去生成一个数组时，JavaScript 还是会生成类似上面的对象结构，不一样的是 `_proto_` 中的内容，对于数组来说，`_proto_` 中会包含一些普通对象所没有的方法，比如之前提到的 `push`，`map`。

这样设计的好处是，我们可以通过一些手段往 `_proto_` 这个结构中放入自定义的数组。这样的结果是，往后我们创建的所有数组都可以访问到我们放入的自定义方法。这个特性用的好的话可以很大程度上减少重复的代码。

这里说了这么多，主要是想告诉你，JavaScript 中的数组其实是对象，不应该把它看作是正常意义上的数组，这个是前提。下面我们看一下数组使用的误区。

<br>

## 常见误区

### 误区一：for in

对于 JavaScript 中的循环遍历你了解多少？受其它语言的影响，我们更喜欢使用下面的增强 for 循环写法，因为这样写会比较的简洁：

```javascript
const object = { a: 1, b: 2, c: 3 };

for (const property in object) {
  console.log(`${property}: ${object[property]}`);
}
```

**这样的写法用在对象上并没有错，但是用在数组上就需要多加注意了**。我们之前反复提到过，数组其实是对象。那么如果是对象的话，元素的遍历顺序按理来说就不会得到保证。另外，如果你往数组或者对象原型中添加了方法或者变量，那么这种遍历方式也会将其遍历到，比如：

```javascript
Object.prototype.objCustom = function() {};
Array.prototype.arrCustom = function() {};

const iterable = [3, 5, 7];
iterable.foo = 'hello';

for (const i in iterable) {
  console.log(i); // logs 0, 1, 2, "foo", "arrCustom", "objCustom"
}
```

所以，如果使用 `for in` 的遍历方式去遍历数组，结果可能会和你想的不太一样。为此 JavaScript 又提供了一种 `for of` 的遍历方式，这种遍历方式是遍历对象中的 value 而不是 key，而且可以保证顺序，比较适用于数组：

```javascript
Object.prototype.objCustom = function() {};
Array.prototype.arrCustom = function() {};

const iterable = [3, 5, 7];
iterable.foo = 'hello';

for (const i of iterable) {
  console.log(i); // logs 3, 5, 7
}
```

当然，就我个人看来，`for of` 这种设计并不能说是有多好，它的出现完全是为了弥补 `for in` 的不足。我更推荐的还是传统的 3 段式的写法：

```javascript
Object.prototype.objCustom = function() {};
Array.prototype.arrCustom = function() {};

const iterable = [3, 5, 7];
iterable.foo = 'hello';

for (let i = 0; i < iterable.length; i += 1) {
  console.log(iterable[i]); // logs 3, 5, 7
}
```

这种写法家喻户晓，你不需要去记那些特殊的不熟悉的用法，更不会担心它会出错。

<br>

### 误区二：数组优化

在其它的一些编程语言中，数组可以说是离计算机底层很近的一个数据结构。通过数组我们可以对程序做一些时间和空间上的优化。但是这个想法放在 JavaScript 上是行不通的，原因之前也讲了，JavaScript 的数组本质上是对象，**它离计算机底层并不近，反倒是更远**。

总的来说，JavaScript 这个语言并不适合海量数据的处理，使用的时候要尽量避免大量数据的操作。

<br>

### 误区三：arguments 数组

如果你还不知道 `arguments` 数组是什么，可以看我之前写的一篇文章，[关于 JavaScript 函数传参的那点事](https://juejin.cn/post/6844903805574709261)。简单说来，`arguments` 是 JavaScript 函数中隐藏的一个东西，类似于 `this`，它里面存放的是传入函数的参数。但我想说的是，其实 `arguments` 并不是 JavaScript 中的数组，它并没有包含数组原型，如果你要对它使用一些常见的数组方法，可能要大失所望了。`arguments` 最多只能算是一个特殊的对象。

<br>

### 误区四：length

length 其实是 JavaScript 中数组的一个特殊属性（区别于普通对象）。**length 的大小有时并不能直接反应数组所包含的元素个数**，比如下面这些例子：

```javascript
const arr = [];
console.log(arr.length);  // length 的初始值为 0
arr[10000] = "js"; // 我们对应存入一个元素，指定一个下标（key）
console.log(arr.length);  // 10001
console.log(arr[3]); // undefined
arr.length = 0;
console.log(arr[10000]);  // undefined
```

上面这个例子包含的信息很多，我们总结一下：
* JavaScript 中的 length 仅仅是一个属性，它可以被任意改变。正常情况下 (不手动更改)，`length = 存在的元素的最大下标 + 1`。手动更改 length 会将数组中下标大于或等于 length 的元素剔除。
* JavaScript 中的数组元素下标可以任意指定，即使该下标前面并不存在任何元素。
* JavaScript 数组操作并没有越界这一说法，如果下标找不到对应的元素，会直接返回 `undefined`

在知道了数组是对象的前提下，上面这些点理解起来应该并不费劲。

<br>

### 误区五：delete

在日常开发中，我们时常需要从数组中移除一个元素，我们期待的是某个下标上的元素被移除后，后面的元素会往前补齐，但是如果你使用 `delete` 关键字来操作时，可能并不是这样：

```javascript
const nums = [0, 1, 2, 3, 4];
delete nums[2];
console.log(nums); // [0, 1, undefined, 3, 4]
```

由此可见，`delete` 并不能达到我们的预期。那如何做到这一点呢？如果你不嫌麻烦，完全可以手动来实现这个功能。当然，JavaScript 也有对应的方法，这个方法就是 `splice`，这个方法有 2 到多个参数，第一个参数是需要删除元素的下标，第二个参数是需要删除的个数，后面的参数是新增元素，表示在该位置删除指定个数的元素后，再将新增的这些元素从该位置插入，如果没有的话，那么就仅仅是删除而不做插入。最后，这个方法会返回一个新的数组，这个数组里面存放的是被删除的元素：

```javascript
const arr = [0, 1, 2, 3, 4];
// 在下标 2 处删除一个元素，然后插入 100，101，102
const deletedEle = arr.splice(2, 1, 100, 101, 102);
console.log(deleteEle); // [2]
console.log(arr); // [0, 1, 100, 101, 102, 3, 4]
```

对于 `splice` 这个方法，容易和另一个内置方法 `slice` 弄混淆，这两个方法的名字太接近了，但是是两个完全不一样的方法。`slice` 表示的是浅拷贝数组的一部分，意思就是通过区间截取子数组。用的多了就清楚了。

<br>

## 总结

关于 JavaScript 的数组就说这么多。这里特别重要的一点是 **数组是包装后的对象**。我们上面也提到了很多使用上的误区，相信在你理解了基本性质后，这些误区也不难理解。在平时的开发中注意下就好。经常看到一些帖子吐槽 JavaScript 中的一些设计，的确如此，因为历史的原因，JavaScript 这门语言确实存在很多设计上的不足。但是如果对它的一些机制稍加了解，相信你在开发的时候心里就更有底，会去下意识的避开那些常见坑。说实在的，语言这东西就是一个工具，每个工具都有适合的场景和用途，在我们了解了这些工具的基本属性之后，根据实际的场景来做对应的选择和切换即可。