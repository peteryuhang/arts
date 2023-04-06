> ARTS是什么？<br>
> **Algorithm**：每周至少做一个leetcode的算法题；<br>
> **Review**：阅读并点评至少一篇英文技术文章；<br>
> **Tip**：学习至少一个技术技巧；<br>
> **Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 334. Increasing Triplet Subsequence](https://leetcode.com/problems/increasing-triplet-subsequence/)

**题目解析**：

给定一个整型数组，问能否在这个数组中找到 3 个元素，这 3 个元素的索引是递增的，并且元素的值也是递增的。比如给定的数组是 `nums`，如果能在之中找到三个索引 `i`，`j`，`k`，这 3 个索引的关系是 `i < j < k`，并且 `nums[i] < nums[j] < nums[k]`，那么就返回 `true`，否则返回 `false`。

如果是仅仅考虑解题不考虑算法的效率问题，我们可以先尝试着去找可行解。不论如何，我们只需要去找到 3 个元素。从左到右遍历，如果我们选中了一个元素作为 `i`，那么我们只需要继续向右去找 `j`，`k` 即可。这样一来，只需要 3 层循环就可以覆盖所有的情况：

```java
public boolean increasingTriplet(int[] nums) {
    if (nums == null || nums.length < 3) {
        return false;
    }

    for (int i = 0; i < nums.length; ++i) {
        for (int j = i + 1; j < nums.length; ++j) {
            if (nums[j] > nums[i]) {
                for (int k = j + 1; k < nums.length; ++k) {
                    if (nums[j] < nums[k]) {
                        return true;
                    }
                }
            }
        }
    }

    return false;
}
```

上面的代码是没有问题的，在 Leetcode 上也可以提交通过。但是时间复杂度有点高，会是 `O(n^3)`。

想要提升算法的效率，我们就必须借助更加高级的算法思想。重新来看这道题目，你会发现，我们无非是要在数组中找一个递增序列。至于这个递增序列具体是什么我们并不关心，我们只关心序列的长度。我们之前就做过这么一道题目，[LC 300. Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)，具体的讲解可以点[这里](https://juejin.cn/post/6844904000022642695)。这道题目我们想求的是数组中最长递增序列的长度。那么，是不是可以说，我们只要知道最长序列的长度，然后拿这个长度与 3 进行比较即可得到答案呢？的确如此，之前我们用的是动态规划，那么套在这道题目，只需要加一个返回条件即可：

```java
public boolean increasingTriplet(int[] nums) {
    if (nums == null || nums.length < 3) {
        return false;
    }

    int[] dp = new int[nums.length];
    Arrays.fill(dp, 1);

    for (int i = 0; i < nums.length; ++i) {
        for (int j = 0; j < i; ++j) {
            if (nums[i] > nums[j]) {
                dp[i] = Math.max(dp[j] + 1, dp[i]);
            }

            // 增加的返回条件，只要长度大于 3，直接返回 true
            if (dp[i] >= 3) {
                return true;
            }
        }
    }

    return false;
}
```

那么是不是这样就是最优解了呢？对于求最长递增子序列这道题目，我们之前也说过，动态规划的解法并不是最优的，最优的解法是二分法，其中的重点是对找到的序列结果进行二分，在这里我就不详细展开了，有兴趣的可以去看看，理解这个解法对于你理解这道题目非常有帮助。

但是回到这道题目，因为我们需要找的元素仅仅是 3 个，而不是 N 个。并且，当我们找到第 3 个元素的时候，我们就可以直接返回 `true` 了，也就是说，我们需要记录的仅仅是 2 个元素的值，那么用变量来表示就足够了。

大致的逻辑就是，如果我们发现当前遍历的元素没有第一个元素大，那么就更新第一个元素的值，否则就和第二个元素进行比较，如果没有第二个元素大，就更新第二个元素，否则就说明当前遍历到的元素是第三个元素，这时就可以直接返回 `true` 了。一个总体的思路就是，让记录的元素尽可能的小，这样会给后面遍历到的元素留有被记录的可能，也就是我们更加可能找到第三个元素。

虽然说，代码就那么几行，但是这里面涉及的逻辑还是非常精妙的，需要细细回味。

<br>

**参考代码**：
```java
public boolean increasingTriplet(int[] nums) {
    if (nums == null || nums.length < 3) {
        return false;
    }

    int smaller = Integer.MAX_VALUE, bigger = Integer.MAX_VALUE;

    for (int i = 0; i < nums.length; ++i) {
        if (nums[i] <= smaller) {
            smaller = nums[i];
        } else if (nums[i] <= bigger) {
            bigger = nums[i];
        } else {
            return true;
        }
    }

    return false;
}
```

<br>

## Review

[Solid Relevance](http://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html)

分享一篇 Bob 大叔关于 SOLID 原则的讨论，更准确的说是对 SOLID 质疑的抨击。有些人认为，随着技术的发展，SOLID 中的一些原则已经过时，甚至觉得 SOLID 中的一些观点存在着错误，限制了软件技术的发展。Bob 大叔对每一个原则都做了说明，并对相应的质疑作出了强有力的反击：

* **Single Responsibility Principle**
    
  简单解释就是，**只能有一个原因使得一个单元（模块/组件）发生改变。如果有多个原因可以使某单元发生变化，那么需要将此单元拆分成更小的单元**。如果不遵循这个原则，你很难想象我们写出来的软件产品会是什么样的。比如说，我们需要将前端界面、数据库、以及业务 API 的实现分开。这样做可以降低整体的复杂性，也可以方便我们协同合作。另外，一个单元的错误并不会直接影响其他单元，这也有助于我们维护和排查错误。

* **Open-Closed Principle**

  **一个模块应该允许扩展，禁止直接修改**。不知道你有没有这样的体会，特别是在一些小的软件公司。代码库中的某个，或者某些函数中充斥着大量的 `if...else` 语句，并且还布满着大量的变量和业务逻辑。每当我们需要添加新的业务逻辑，或者是对原有的业务逻辑进行改变的时候。我们需要浏览大量的不相关的代码，才能找到我们需要修改的地方。这里存在两个严重的问题，一是这样的代码严重影响了开发的效率。另外一点就是，这种样子的代码是很难被测试的，特别容易出错。
  
  究其原因，这样的代码，在代码架构设计的初期，可能并没有考虑到这个原则。导致后期的维护和堆叠代码异常吃力。

* **Liskov Substitution Principle**

  这个原则的名字可能你没怎么听说过。它表示的意思是，**子类型具体的实现应该基于基类型的定义**。举个 Java 中的例子，一个 interface 会被很多不同的 class 实现。但是不管 class 具体如何实现，我们都可以通过 interface 的本身的定义，准确地知道任意一个实现它的 class 中的定义是怎么样的。这样的好处是我们把底层的一些具体，庞大且复杂的东西抽象出来了，让我们从一个更高的层面来看待并管理模块，而不至于陷入细节的泥潭。

* **Interface Segregation Principle**

  接口分离原则。这个说的是，**我们需要让每个接口尽可能的小，这样用户才不会过度依赖于那些他们用不上的部分**。就好比你设计了一个巨大的借口，这个借口中包含着许许多多的方法。就使用的角度，我可能仅仅需要这众多方法中的其中一、两个。但是最终我还是需要实现那些我完全用不上的方法，这就很影响开发的效率了。另外一点，运行时的依赖和模块之间的大量链接也是会影响程序的执行效率的。如果很好的考虑并遵守了这项原则，相信代码的实现和执行的效率都会得到不小的提升。

* **Dependency Inversion Principle**

  用一句话解释就是，**高层逻辑的展现不应该依赖于低层的细节**。注意，我这里用的是 **展现**，而不是 **实现**。就比如说，在业务代码层面，我们不希望看到一些 SQL 语句，这些 SQL 语句应该在更低的代码层面，而它们理应被类似 ORM 这样的工具封装起来。这样，在业务代码层面我们就只需要关注业务代码层面需要关注的事情。这里的目的和之前的 Liskov Substitution Principle 很类似，就是为了把低层复杂的东西抽象出来，让我们站在一个较高的层面更好，更方便地管理它们。

很多人都在说，我们需要尽可能地写简单的代码。问题是写简单的代码并不是说到就能做到的，特别是当业务慢慢变得复杂的情况下。而上面列的这些原则就是为了写出简单的代码而给出的方式，方法，换句话说，就是告诉你如何才能写出简单的代码。**这些原则并没有过时，它们就好像是计算机领域的根基，伴随着计算机发展的这几十年，一直都没怎么变过，但却一直产生着潜移默化的深远影响**。


<br>

## Tip

这次来说说 generator 这个东西，以及我们该在什么情况下使用它。

首先来看看下面这段 python 代码：

```python
def read_file(filepath):
    content = []
    with open(filepath) as f:
        for line in f:
            content.append(line)
    return content
```

上面这个函数会从输入的文件路径中读取文件每行的内容，并将读出的内容放到一个 list 中，最后返回。

这段代码本身并没有问题，但现在考虑下面这两个需求：<br>
1. 文件特别大，怎么样才能尽可能减少内存的消耗
2. 这个文件会实时增加新的内容

基于上面两点要求，我们就可以使用 generator（生成器）。它的特点就是 **即用即取**。可以把代码改成下面这样：

```python
def read_file(filepath):
    with open(filepath) as f:
        for line in f:
            yield line
```

这样，你每需要获取一行内容时，就可以运行上面的这个函数。每运行一次，这个函数只会返回一行的内容，然后停在那里。当你再次运行它的时候，它就紧接着会返回下一行的内容。

使用生成器需要注意的一点是，它针对每个元素仅仅返回一次，也就是生成器是无法回头的。**在处理一些实时的动态情景中可以考虑使用**。


<br>

## Share

感觉好久没有写文章。争取再把之前养成的良好习惯给捡回来

[Effective Python 读书笔记（一）](./Effective%20Python%20读书笔记（一）.md)

