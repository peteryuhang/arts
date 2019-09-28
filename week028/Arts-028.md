## Algorithm

[LeetCode 287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

**题目解析**：

给定一个整形数组，数组的长度是 n + 1，数组里面存放的元素是区间 [1, n] 上的数，这个数组中有且仅有一个元素是重复的，题目让我们找出这个元素，并且这道题给了如下的限制：
* 不能改变数组
* 只能使用 O(1) 的空间
* 时间复杂度必须小于 O(n^2)
* 重复的元素可重复多次

首先**不能改变数组**导致无法排序，也无法用 index 和元素建立关系；**只能使用 O(1) 的空间**意味着使用哈希表去计数这条路也走不通；**时间复杂度必须小于 O(n^2)** 表示暴力求解也不行；**重复的元素可重复多次** 这一条加上后，本来可以通过累加求和然后做差`sum(array) - sum(1,2,...,n)`的方式也变得不可行。本来是非常简单的一道数组遍历的题目，加上了上面这四个条件后感觉有点无从下手。

我们说做题要借助算法，而不是空想，因此对于这道题也不例外，我们可以问自己这样一个问题，就是 **“什么样的算法可以不使用额外的空间解决数组上面的搜索问题？”**，你这时应该隐隐约约知道了，没错，就是 **二分查找**，我想你纠结的可能会是，“这个数组没有排序啊，二分怎么做？”。这里澄清一个误区就是，二分法的使用并不一定需要在排序好的数组上面进行，**不要让常见的例题限制了你的思路**，二分法还有一个比较高级的用法叫做 **按值二分**。

这道题目交代的信息很少，**我们只需要关注两个东西 - 数组，数组里的元素**，利用二分我们需要去思考的是，我们要找符合条件的元素作为答案，那么**比答案小的元素具有什么样的特质，比答案大的元素又具有什么样的特质？**，结合题目给我们的例子来看看：
```
例1:
[1,3,4,2,2]                         元素个数
<= 1 的元素：1                          1
<= 2 的元素：1, 2, 2                    3
<= 3 的元素：1, 2, 2, 3                 4
<= 4 的元素：1, 2, 2, 3, 4              5

例2:
[3,1,3,4,2]
<= 1 的元素：1                          1
<= 2 的元素：1, 2                       2
<= 3 的元素：1, 2, 3, 3                 4
<= 4 的元素：1, 2, 3, 3, 4              5

极端一点的例子 (必须保证数组的长度是 n + 1, 并且元素都在区间[1,n] 上, 有且只有一个重复)
[3,3,3,3,4]
<= 1 的元素：                           0
<= 2 的元素：                           0
<= 3 的元素：3, 3, 3, 3                 4
<= 4 的元素：3, 3, 3, 3, 4              5
```
看完上面几个例子，相信你明白了一个事实就是，“如果选中的数**小于**我们要找的答案，那么整个数组中小于或等于该数的元素个数必然小于或等于该元素的值，如果选中的数**大于或等于**我们要找的答案，那么整个数组中小于或等于该数的元素个数必然**大于**该元素的值”，而且你可以看到，我们要找的答案其实就处于一个分界点的位置，寻找边界值，这又是二分的一个应用，而且题目已经告诉我们数组里面的值只可能在 [1, n] 之间，这么一来，思路就是在 [1, n] 区间上做二分，然后按我们之前提到的逻辑去做分割。整个解法的时间复杂度是 O(nlogn)，也是满足题目要求的。

上面的解法不是最优的，但是个人觉得是根据现有的知识比较容易想到的。另外一种 O(n) 的解法借鉴了 [Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) 的快慢指针找交点的思想，算法非常的巧妙，也非常的有趣，但不太容易想到，这里把代码放上。算法的思想不难，画画图就理解了。


<br>

**参考代码（一）**：

Binary Search
```java
public int findDuplicate(int[] nums) {
    int start = 1, end = nums.length - 1;
    
    while (start + 1 < end) {
        int mid = start + (end - start) / 2;
        if (isValid(nums, mid)) {
            start = mid;
        } else {
            end = mid;
        }
    }
    
    if (isValid(nums, start)) {
        return end;
    }
    
    return start;
}

public boolean isValid(int[] nums, int selectedElement) {
    int count = 0;
    for (int i : nums) {
        if (selectedElement >= i) {
            count++;
        }
    }
    
    return count <= selectedElement;
}
```

<br>

**参考代码（二）**：

快慢指针
```java
public int findDuplicate(int[] nums) {        
    int fast = nums[nums[0]];
    int slow = nums[0];
    
    while (fast != slow) {
        fast = nums[nums[fast]];
        slow = nums[slow];
    }
    
    slow = 0;
    while (fast != slow) {
        fast = nums[fast];
        slow = nums[slow];
    }
    
    return slow;
}
```
<br>

## Review
[Write to Express, Not to Impress](https://medium.com/swlh/write-to-express-not-to-impress-465d628f39fe)

这篇文章讲述的是英文写作的问题，作者在后面给出了很多很实际的建议，虽然主要是针对英文，但是个人觉得有些东西是互通的，中文写作也可以借鉴。文章的一个核心观点就是**写作中使用的词汇，句子结构，修辞手法等技巧类的东西越简单，越清晰就越好**。小时候学语文或者是英语，老师给我们讲述了很多很高大上的句式，描述事物的方法等等，我们也读了很多散文、诗歌，可能会有一个意识就是 “写作是一个高大上的东西，里面有很多技巧性的东西，并不是人人都能掌握的好”，但是其实现在看来，**写东西的第一要务是让读者能够一下子就明白你所要表达的意思**，特别是写技术类的东西的时候，其他的花哨的技巧无非是增添了读者理解文章的难度。一起来看看这些建议吧，看看如何能够把文章写的更加简洁，把文字描述地更加清晰

* 使用更恰当的动词去代替副词加动词
  * The child cried loudly.
  * The child screamed.    更恰当
* 尽量避免使用被动
  * I was given a raise by my boss.
  * My boss gave me a raise.  更恰当
* 尽量省略从句中的 that
  * I hope that my colleagues enjoy my presentation.
  * I hope my colleagues enjoy my presentation. 更恰当
* 使用程度性的描述词之前可以想想有没有更恰当的形容词
  * It’s extremely cold outside.
  * It’s freezing outside. 更恰当
* 如果有两个形容词描述名词的时候，尽量不要使用连接词
  * The long and crowded flight exhausted the flight attendants.
  * The long, crowded flight exhausted the flight attendants.  更恰当
* 句子开头尽量不要使用 there，而是直接跟名词
  * There is a common thought among the students that school days should be shorter.
  * The students think school days should be shorter.   更恰当
* 使用清晰易懂的单词代替短语
  * I made a decision to exercise daily.  
  * I decided to exercise daily.  更恰当
* 如果用两个意思很接近的形容词来描述名词的时候，可以考虑使用一个程度更强烈的形容词
  * The customers are happy and excited about today’s product launch.
  * The customers are thrilled about today’s product launch.  更恰当
* 使用积极的描述代替消极的描述
  * The living room lacks sunlight.
  * The living room is dark.  更恰当
* 少用空洞的 be, to be
  * The parent and teenager are in a state of disagreement about the curfew.
  * The parent and teenager disagree on the curfew.   更恰当
* 使用简单，更容易理解的词汇来描述
  * My core competency relates to getting buy-in from all stakeholders.
  * I like to ensure that everyone agrees.   更恰当
* 直接描述，不需要对自己写的东西做定义
  * My emotions got the best of me. In other words, I was angry.
  * I was angry.   更恰当
* 句子中的动词最好是直接表达出动作
  * Calcium makes the bones stronger.
  * Calcium strengthens the bones.  更恰当

<br>

## Tip

学习全栈专栏，知道了三个概念 pull, poll 以及 push
* **pull** 指的是客户端或者是服务器端主动发起请求去获取数据
* **poll** 是在 pull 的基础之上，增加了周期性的概念
* **push** 指的是服务器端主动发送数据到客户端

pull 应用的更多是因为对于服务器端不需要维护状态，但是一般来说 pull 的实时性不高。

**学习全栈技术的建议**：通过学习一两个经典技术，然后去对比了解同一领域相同的技术，掌握这些技术背后的本质内容，这样才能举一反三，学得更快。

JavaScript 的 promise：主要是为了解决编码风格的问题。利用了 JavaScript 中的微任务机制。

JavaScript 中的 async/await：实现用同步的方式去写异步的代码，背后使用到了 **生成器** 和 **协程**。

    
<br>

## Share
这周继续算法，讲一个热度不高，但是非常实用的数据结构 - 并查集

[并查集概念及用法分析](./并查集概念及用法分析.md)