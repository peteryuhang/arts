
## Algorithm
[LeetCode 164. Maximum Gap](https://leetcode.com/problems/maximum-gap/)

**思路分析**<br>
题意不难理解，给定一个未排序的数组，找出在排序状态下的相邻元素的最大差值。如果不仔细看，也许你会觉得这道题简单，它最后说到要求是 **O(n)** 的时间复杂度，这也就意味着普通排序这条路是走不通的。刚拿到这道题也是不知道咋整，没有任何头绪，看了 tag，才发现是 **bucket sort**，桶排序其实是一个容易被遗忘的算法，因为受用场景非常有限。是不是知道用桶排序来做，这道题就非常简单了呢，其实也不是，实现的话由于要考虑种种的情况，程序也不是特别好写。

解题之前还是简单来讲讲桶排序这个算法吧，这里的桶代表的是一个区间范围，每个桶的区间长度一般都是一样的，比如说给定数组 `[1,5,7,10]`，这里如果我们分 10 个桶，那么每个桶的区间长度就是 1，等同于每个桶其实就对应一个数，如果这里我们分 1 个桶，那么这个桶的区间范围就是 1 ~ 10，当然这里我给的两个例子都是极端的例子，在实际应用上我们应该结合实际情况合理分配桶。但是基本来说**桶的范围和个数是由数组中最大值、最小值以及数组中的元素的个数来决定的**，这样可以保证使用最少的桶覆盖所有的可能性。

这个题目要求我们求数组排序好后，相邻数的最大差值，这里我们首先遍历一遍数组得到最大值、最小值，仔细想想的话，如果排序好的数组当中的元素都是等间隔的，类似 `[2,4,6,8,10]` ，在数组长度，最大最下值确定的情况下，**在这种等间隔的情况下，求得的相邻数的最大差值是最小的**，这很好理解，因为同等资源都被等量分配了，不存在分配多和少的结果。因此，**如果我们按这个等量分配的长度来定义桶的长度的话，我们其实并不需要考虑桶内元素的差值**，我们需要做的只是记录每个桶中所有元素的最大值和最小值，然后拿这两个值去和相邻的桶的最大值和最小值做差。这样下来可以保证时间复杂度是 O(n) 的。

<br>

**参考代码**
```java
private class Bucket {
    int min = Integer.MAX_VALUE;
    int max = Integer.MIN_VALUE;
}

public int maximumGap(int[] nums) {
    if (nums == null || nums.length < 2) {
        return 0;
    }
    
    int min = Integer.MAX_VALUE;
    int max = Integer.MIN_VALUE;
    for (int i : nums) {
        min = Math.min(min, i);
        max = Math.max(max, i);
    }
    
    // 分配桶的长度和个数是桶排序的关键
    // 在 n 个数下，形成的两两相邻区间是 n - 1 个，比如 [2,4,6,8] 这里
    // 有 4 个数，但是只有 3 个区间，[2,4], [4,6], [6,8]
    // 因此，桶长度 = 区间总长度 / 区间总个数 = (max - min) / (nums.length - 1)
    int bucketSize = Math.max(1, (max - min) / (nums.length - 1));
    
    // 上面得到了桶的长度，我们就可以以此来确定桶的个数
    // 桶个数 = 区间长度 / 桶长度
    // 这里考虑到实现的方便，多加了一个桶，为什么？
    // 还是举上面的例子，[2,4,6,8], 桶的长度 = (8 - 2) / (4 - 1) = 2
    //                           桶的个数 = (8 - 2) / 2 = 3
    // 已知一个元素，需要定位到桶的时候，一般是 (当前元素 - 最小值) / 桶长度
    // 这里其实利用了整数除不尽向下取整的性质
    // 但是上面的例子，如果当前元素是 8 的话 (8 - 2) / 2 = 3，对应到 3 号桶
    //              如果当前元素是 2 的话 (2 - 2) / 2 = 0，对应到 0 号桶
    // 你会发现我们有 0,1,2,3 号桶，实际用到的桶是 4 个，而不是 3 个
    // 透过例子应该很好理解，但是如果要说根本原因，其实是开闭区间的问题
    // 这里其实 0,1,2 号桶对应的区间是 [2,4),[4,6),[6,8)
    // 那 8 怎么办？多加一个桶呗，3 号桶对应区间 [8,10)
    Bucket[] buckets = new Bucket[(max - min) / bucketSize + 1];
    
    for (int i = 0; i < nums.length; ++i) {
        int loc = (nums[i] - min) / bucketSize;
        
        if (buckets[loc] == null) {
            buckets[loc] = new Bucket();
        }
        
        buckets[loc].min = Math.min(buckets[loc].min, nums[i]);
        buckets[loc].max = Math.max(buckets[loc].max, nums[i]);
    }
    
    int previousMax = Integer.MAX_VALUE; int maxGap = Integer.MIN_VALUE;
    for (int i = 0; i < buckets.length; ++i) {
        if (buckets[i] != null && previousMax != Integer.MAX_VALUE) {
            maxGap = Math.max(maxGap, buckets[i].min - previousMax);
        }
        
        if (buckets[i] != null) {
            previousMax = buckets[i].max;
            maxGap = Math.max(maxGap, buckets[i].max - buckets[i].min);
        }
    }
    
    return maxGap;
}
```

<br>

## Review
[How To Use Destructuring and Arrow Functions to Improve Your JavaScript Code](https://medium.com/better-programming/use-these-javascript-features-to-make-your-code-more-readable-ec3930827226)

[Consider Yourself a Developer? You Should Solve the Project Euler Problems](https://blog.usejournal.com/consider-yourself-a-developer-you-should-solve-the-project-euler-problems-ed8d13397c9c)

这次看了两篇文章，第一篇是讲述的是 JavaScript 中的一些 Syntax 上面的建议，总结如下：
* destructing(翻译过来应该是解析吧)，object destructing 和 array destructing 都可以使程序更容易阅读
* javaScript 中函数可以作为返回值，因此，函数可以多层调用，例如 `fun()()` 
* 另外箭头函数是个神奇的东西，**它其实是隐式带有 return 的**，所以程序当中不需要写明 return 关键字
* 箭头函数和一般的函数还有一个区别就是对于箭头函数是保留了 this 关键字的（这个需要深入）

第二篇文章中提到了一个刷题网站，就是 **[Project Euler](https://projecteuler.net)**，和 LeetCode 等网站不同的是，这个网站是以数学题为主，每个题都或多或少地和数学或者说是逻辑相关，但是这些题目其实并不枯燥无味，做着觉得挺有趣的，这个网站不提供在线的 code 平台，问题都只需要提交一个答案，比如“第 100001 个质数是多少？”，你可以在自己的 IDE 中写代码，然后跑 case 就行了，提交完答案通过后，可以看到其他人的解法，令我惊奇的是，这个网站在 2004 年就有了，有些人还展示汇编的代码。总之这是一个非常不错的训练 coding 并熟悉一门语言的方式，随着题目的难度加大，你还可以加上 test case，使用 TDD 的方式，也就是把其当成一个项目了，网站上面的题目难度是逐渐递增的，感觉像是闯关一样，这里同样还有成就，排行榜之类的东西，让做题更有趣味性。

<br>


## Tip
这周跟着极客时间上的专栏学了些基本的 SQL 知识
* 可以使用 profiling 去收集 SQL 执行中的资源（时间）使用情况
     ```sql
     select @@profiling   -- 看 profiling 是否打开，开启能让 MySQL 收集 SQL 执行所使用的资源情况
    set profiling = 1    -- 打开 profiling
    show profiles        -- 查看当前会话产生的所有 profiles
    show profile         -- 获取上一次的执行时间情况
    show profile for query 2 -- 查询指定的 query id 的执行时间情况
    select version()     -- 查看 MySQL 的版本情况
     ```
* SQL 中的关键字的顺序
    ```sql
    SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT
     ```
    这里的关键字有些可有可无，但是**顺序一定不能颠倒**
* SQL 中的执行顺序
    ```sql
    FROM > WHERE > GROUP BY > HAVING > SELECT 的字段 > DISTINCT > ORDER BY > LIMIT
    ```
    知道这个顺序有助于更加了解 SQL 的内部执行，SQL 的执行其实是有步骤的，每一步其实都会生成一个虚拟表，这个虚拟表会作为下一步的输入参数，直到产生最后的结果


<br>

## Share
在 Project Euler 上面做了几道题，发现质数这个东西很有意思，这次就写个关于质数的分享吧，你可能会觉得这不是小学就学过吗？那就看看咯，看看是不是你想的那样

[关于质数，你所应该知道的](./关于质数你所应该知道的.md)
