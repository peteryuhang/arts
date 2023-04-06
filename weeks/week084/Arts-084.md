> ARTS是什么？<br>
> **Algorithm**：每周至少做一个leetcode的算法题；<br>
> **Review**：阅读并点评至少一篇英文技术文章；<br>
> **Tip**：学习至少一个技术技巧；<br>
> **Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 322. Coin Change](https://leetcode.com/problems/coin-change/) <br>

[LC 518. Coin Change 2](https://leetcode.com/problems/coin-change-2/)

**题目解析**：

两道类似的题目，首先来看第一道，给定一个数值以及硬币的面值，问组合成这个面值最少需要硬币的个数。找题目中的关键点，“**填充面值**”，“**最少个数**”，基本上可以确定是动态规划中背包一类的问题。背包一类问题的特殊的地方就是用值来当作 DP 的状态，我们可以根据题目中的条件关系来确定状态 `dp[i][j]` 表示的意义是 “**用面值为 `coins[0]...coins[i] ` 的货币填充数值 j 最少需要的个数**”，确定完状态，接下来就是找到状态之间的联系，也就是写出递推关系式：

```
dp[i][j] = min(dp[i - 1][j], dp[i - 1][j - coins[i]] + 1, dp[i][j - coins[i]] + 1)
```

解释一下这个关系式是怎么写出来的。每当我们考虑一个状态的时候，比如 `dp[i][j]`，我们需要去思考一个问题，那就是得到`j` 用不用当前的面值进行填充。如果不用，那么就是前一个状态的顺延，也就是 `dp[i - 1][j]`。如果用，那就是 `dp[i - 1][j - coins[i]] + 1` 或者是 `dp[i][j - coins[i]] + 1`，这里之所以有两个状态是因为当前考虑的面值可以用无数次。那怎么确定到底是这几个状态中的哪一个呢？取最小值，因为题目要求的是最小。这里只是为了解释清楚，所以把所有可能的状态都列了出来，但是我们在计算 `dp[i][j - coins[i]] + 1` 的时候，已经考虑过 `dp[i - 1][j - coins[i]] + 1` 了，所以不需要重复考虑，因此可以把递推关系进一步简化：

```
dp[i][j] = min(dp[i - 1][j], dp[i][j - coins[i]] + 1)
```

写到这里，你发现了一个有趣的现象。虽然我们需要二维的 DP 数组来表示这么一个递推关系，但是实际我们并不需要二维数组。这也很好解释，`dp[i][j]` 与 `dp[i][j - coins[i]]` 同属一维，`dp[i][j]` 与 `dp[i - 1][j]` 看似需要用二维表示，但其实完全可以把它们放在同一个维度，并不会对后面的状态推导造成影响，处于节省空间的目的，我们还可以简化：

```
dp[j] = min(dp[j], dp[j - coins[i]] + 1)
```

知道了上面这些，基本上实现就没什么难的了，无非是考虑一些初始化和越界的问题。

第二道题目和第一道题目同属一类问题，只不过是第二道题求的是组合个数。状态定义上没有区别，不一样的只是状态之间的关系，也就是递推关系。你可以试着参照我上面的推导过程，看能不能推导出最优的关系式。

<br>

**参考代码 （LC 322）**：
```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];

    Arrays.fill(dp, Integer.MAX_VALUE);

    dp[0] = 0;

    for (int i = 0; i < coins.length; ++i) {
        for (int j = coins[i]; j <= amount; ++j) {
            if (dp[j - coins[i]] != Integer.MAX_VALUE) {
                dp[j] = Math.min(dp[j - coins[i]] + 1, dp[j]);
            }
        }
    }

    return dp[amount] == Integer.MAX_VALUE ? -1 : dp[amount];
}
```

**参考代码 （LC 518）**：
```java
public int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];

    dp[0] = 1;
    for (int i = 0; i < coins.length; ++i) {
        for (int j = coins[i]; j <= amount; ++j) {
            dp[j] += dp[j - coins[i]];
        }
    }

    return dp[amount];
}
```

<br>

## Review

[9 Hard Lessons I Struggled to Learn During My 18 Years as a Software Developer](https://betterprogramming.pub/9-hard-lessons-i-struggled-to-learn-during-my-18-years-as-a-software-developer-14f28512f647)

一个工作了 18 年的软件工程师给出的 9 个建议。这里面的有些建议之前都有提到过，但是有些之前没有注意到，现在想来确实挺有道理。

* **端正态度，积极合作**
    
    我们首先要摆正自己的态度，不能太以自我为中心。一个合格的工程师是需要多多参与讨论，多多听取别人的建议和反馈。所谓的 **刻意练习**，很重要的一点就是要有反馈，而我们自己只能找到一些很浅显的反馈。可能很多时候，很多想法或者习惯已经在我们脑海里根深蒂固了，如果没有他人的提醒，我们自己是很难察觉的。可见合作的重要性在这里不言而喻。

* **不要限于一门技术**
    
    工程师很多时候都有自己的偏好，比如使用哪种编程语言，用哪一个框架，甚至是使用哪一款 IDE。这些都没错，但我们不能仅仅只限于使用一种语言，一门技术，应该多多尝试不同的东西，这样能够让我们的视野更加开阔。在面对相同的问题的时候，我们就能够有更多的选择，也能很清楚地知道每个技术之间的关联，以及利弊。

* **学习解决问题的方法，而不是实现的方法**

    在开发中，我们更应该着眼的是**如何解决问题**，而不是熟练记住某个技术的用法或者实现细节。后者我们完全可以 Google 出来。在平日的学习中也应该如此。举个例子，比如我们现在想学习 **观察者模式**，这是一个面向对象的设计模式。我们更应该去花时间思考和理解的是：

    * 这个技术想解决什么样的特定问题？
    * 为什么一般的做法解决不了？相比一般的做法，它的优势在哪？
    * 它有哪些不足之处或者潜在的问题？怎么避免？

    而**不是**去思考：

    * 这个技术怎么实现？
    * 这个技术有几种实现方式？
    * 某种语言的类库是否支持这个技术的需求？

* **持续学习**

    如果你想要不断地成长，那么就需要不断地学习，更新自己的知识技能库。再加上计算机技术发展更新换代特别的迅猛，持续学习是很有必要的。那么我们该去学习些什么呢，主要是下面这些：

    * 新的计算机语言
    * 新的编程范式
    * 新的工作方式
    * 新的解决问题的方法
    * 新的和同事合作的方法
    * 新的审查代码和测试代码的方法

* **实现比优化更重要**

    在开发中，我们们常常习惯于在一个项目还没有成型的时候就开始花很多时间来思考如何优化的问题。其实正确的理解应该是，“**优化是建立在已经实现的项目之上**”。因为当一个项目还没有完成的时候，优化很可能只是局部的，优化并不能建立在整体之上，有些时候一些局部的优化放到一个大的层面来看真的可以说是无足轻重，过早的花时间去思考如何优化只会降低开发的效率。再者说来，很多时候我们都并不清楚我们的方案或者想法对不对，我们首先要做的只是去验证方案的可行性，而不是如何优化。另外，在想法还没有完全实现的时候，去假设其实现后可能存在的问题，这本身就不可靠。
    
* **一个项目的最后 10% 需要花费 90% 的时间**

    一个项目怎么才算是做完了？很多时候我们会简单地认为只要实现了对应的功能，做了相应的优化就算是完成了。这确实是这个项目的大部分内容。但是在这个时候并不能算作是完成，我们还需要测试、发现问题、写文档、教用户如何使用、以及做展示。甚至我们还需要对这个项目进行监控和后期的维护。这些东西大多都是一个项目的必要项。所以不要简单地认为你实现了代码就表明完成了项目，其实需要做的事情，需要花的时间还很多。
    
* **用副业提高能力是个可选项**

    很多工程师选择通过副业来提升自己的能力，这一点并没有错。但是这并不是说，仅仅从工作中学习是不够的。你需要思考的是，你的副业究竟能不能弥补你的能力上的缺失。你可以选择那些你在工作中不曾见到过的项目来提升自己。比如说，你工作的大部分时间都是在写测试，那么你就可以考虑在工作之外从 0 到 1 实现一个项目。再比如说，在工作中，你主要单独负责一个项目，那么在工作以外你可以尝试着去给一个现存的项目贡献代码。总的来说，我们更应该思考副业能够给我们带来的价值，而不是盲目地认为自己的副业越多，对自己就越好。


以上就是这篇文章想要告诉我们的主要内容。多读读前辈们的建议，才能少走没有必要的弯路。说多了还是一个态度的问题。不认真再怎么也找不到前进的方向和道路，认真并坚持下去你会发现不管怎么走都能到达目标。



<br>

## Tip

关于 JavaScript 中的内部槽，因为这个东西只能内部实现有关，可能有很多人不了解。这有一篇文章解释说明了这个东西：

[JS Tip of the Day: Internal Slots](https://forum.kirupa.com/t/js-tip-of-the-day-internal-slots/643171)

具体说来，内部槽（**Internal slot**）其实是 JS 对象的隐藏属性。用户不能直接访问或者操作，但是在运行时内部槽会被系统用来管理对象数据和对象的行为。内部槽的表示形式为 `[[<name>]]`，常见的内部槽有 `[[Prototype]]`、`[[Constructor]]` 等等。

虽然我们不能直接操作内部槽，但是我们可以通过一些 API 进行间接的操作，比如 `Object.getPrototypeOf()`。

文章解释了内部槽的一个性质，那就是不可以被继承，这会导致之前父类的属性或者方法无法传递到子类，举个例子：

```javascript
let map = new Map(); // map.[[Prototype]] = Map.prototype
let proto = Object.getPrototypeOf(map); // returns map.[[Prototype]]
console.log(proto === Map.prototype); // true

let fakeMap = Object.create(map); // new fakeMap object inherits from map
let fakeProto = Object.getPrototypeOf(fakeMap); // returns fakeMap.[[Prototype]]
console.log(fakeProto === Map.prototype); // false
console.log(fakeMap.get('Home')); // TypeError: incompatible receiver
```

这其实也是 ES6 之前的一个坑。为了弥补这个坑，ES6 引进了 class，然后通过 class 继承来解决之前存在的这个问题：

```javascript
class FakeMap extends Map {
    constructor (entries) {
        super(entries); // initializes this instance with Map internal slots
    }
}

let fakeMap = new FakeMap([['Home', '37.7N-122.5W']]);
console.log(fakeMap.get('Home')); // 37.7N-122.5W (not so fake anymore)
```

class 中的 `super()` 函数让子类和父类建立起了连接，由此解决了 JS 之前在类函数上存在的问题，让 JS 更像一般的 OOP 语言了。

<br>

## Share

[深入理解ES6(一)](./深入理解ES6(一).md)