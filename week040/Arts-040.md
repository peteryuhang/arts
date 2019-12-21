> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 1292. Maximum Side Length of a Square with Sum Less than or Equal to Threshold](https://leetcode.com/problems/maximum-side-length-of-a-square-with-sum-less-than-or-equal-to-threshold/)

**题目解析**：

给定一个二维矩阵，需要在二维矩阵中找正方形，使得正方形内所有元素之和小于一个阈值，求正方形的最大边长。

一开始拿到这道题，感觉不难，于是用了一个暴力的方法求解，想法很简单，就是我们只需要知道正方形左上角那个点的位置，再加上边长我们就可以确定这个正方形，于是我们就可以去枚举点位和边长，首先枚举所有的点位，这一操作需要 m * n 的时间复杂度，然后我们还需要枚举边长，另外就是，确定了一个正方形，你需要把正方形之中的元素加起来，这么一下来，时间复杂度就会是 O(m * n * side^3)。实现完程序，直接超时。

有些时候，**一维上面 work 的解法可以把其衍生到二维上来**，比如这道题我们就可以利用前缀和数组。我们知道两个点可以确定一个矩形，我们先固定起始点 `[0][0]`，`prefixSum[i][j]` 表示的是 `[0..i][0...j]` 这个矩形内的所有元素和。前面说到，我们可以由一个点位加上边长确定一个正方形，这个时候我们把这个点选在右下角，这么一来我们就可以通过 prefixSum 来直接求解该正方形的所有元素的和，假如当前考虑的正方形的右下角的点是 `[i][j]`，边长是 side，那么元素和就是：
```
sumOfSquare = prefixSum[i][j] - prefixSum[i - side][j] - prefixSum[i][j - side] + prefixSum[i - side][j - side]
```
其实就是面积之间的加减替换，你画画图应该不难理解。

这里还有一个小优化，就是不需要每个点位都从 1 到 `Min(m,n)` 去枚举边长，在当前点位得到的最大边长可以用作下一个点位的起始考虑边长，这样一下来，时间复杂度会降至 O(m * n)。

**prefixSum 的这个思想可以算作是动态规划了，这里我们可以看到其实使用空间换时间，当前问题的求解可以由之前计算过的子问题的解快速推得**。


<br>

**参考代码**：
```java
public int maxSideLength(int[][] mat, int threshold) {
    int m = mat.length, n = mat[0].length;

    int[][] prefixSum = new int[m + 1][n + 1];

    for (int i = 1; i <= m; ++i) {
        for (int j = 1; j <= n; ++j) {
            prefixSum[i][j] = mat[i - 1][j - 1] + prefixSum[i - 1][j] 
                            + prefixSum[i][j - 1] - prefixSum[i - 1][j - 1];
        }
    }

    int len = 0, result = 0;

    for (int r = 1; r <= m; ++r) {
        for (int c = 1; c <= n; ++c) {
            len = result + 1;

            while (r - len >= 0 && c - len >= 0) {
                int cur = prefixSum[r][c] - prefixSum[r - len][c] - prefixSum[r][c - len]
                            + prefixSum[r - len][c - len];

                if (cur <= threshold) {
                    result = len;
                    len++;
                } else {
                    break;
                }
            }
        }
    }

    return result;
}
```

<br>

## Review
这段时间开始看 [97 Things Every Programmer Should Know](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/)

这周看了前 3 篇文章，依次总结下

[Act with Prudence](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_01/)

谨慎行事，在软件的开发过程中，很多时候摆在我们面前的是两个选择，那就是 “**做正确的事**” 和 “**快速完成**”，有些时候，时间比较赶，我们会选择后者，我们信誓旦旦地对自己说，有些东西目前看来可做可不做，为了赶进度，暂时先不做，以后有时间再回来做。但请记住，当你做这个决定的时候，你已经欠下了一笔 **技术债**，负债是会有利息的，这里也不例外，你此时已经意识到正确的做法是什么，但是不肯花时间去改，以后项目体量变大了，你需要花更多的时间去回顾，去思考，去更改。因此，**每欠下一笔债最好拿本子记下，尽可能快速地去还清，这才是一个明智之举**。

[Apply Functional Programming Principles](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_02/)

讲的是我们需要花些时间来学习函数式编程，作者在这个地方给出了原因。函数式编程暗示了，一个函数接受相同的输入，一定会产生相同的输出，这样一来代码的透明性增加，程序出 bug，我们可以很快速地进行定位。另外函数式编程的思考也可以很好地辅助我们学习其他的思想和原理。

[Ask "What Would the User Do?" (You Are not the User)](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_03/)

最终决定产品好不好的还是用户，但是问题在于程序员和用户的思维方式大相径庭，用户看重的往往是产品的使用的方便程度以及最终的效果，程序员则关注的是具体的实现细节，因此作为程序员，也需要时不时跳出固定思维圈，积极去思考什么才是用户真正想要的，作者在文章中给出了几点意见：
* 用户的行为都有共性可言，他们做事情往往都会按照一定的固定顺序
* 当用户遇到不清楚的问题的时候，他们关注的范围会缩小，因此，根据具体功能给建议往往会比产品使用手册要有效的多
* 当用户找到了一种有效的使用方式，只要这种方式不是太差，他们都会坚持这种使用方式，因此，只给一个方法会比给多种不同方法要有效的多

另外，文章的最后，作者还提到，往往 “用户说他/她想要的” 和 “他/她实际想要的” 并不一致，因此，**通过观察用户是如何使用产品来思考产品的设计方案会比直接询问用户意见要好的多**。

<br>

## Tip

今天推荐一个排版软件 gitbook，只需要简单的几步就可以把 repo 给变成一本书，具体可以看 [这里](https://www.npmjs.com/package/gitbook)

<br>

## Share

这次来个动态规划系列文章做个结尾，其中还会涉及一个空间优化的小技巧

[动态规划之空间优化与总结回顾](./动态规划之空间优化与总结回顾.md)