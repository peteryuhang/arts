> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 403. Frog Jump](https://leetcode.com/problems/frog-jump/)

**题目解析**：

一只青蛙过河，河里面有石头，青蛙只能跳到石头上确保不掉入水中。输入是一个递增的数组，里面的元素表示的是石头的位置，如果青蛙前一次跳了 k 步，那么当前只能跳 k - 1，k 或是 k + 1 步，青蛙不能往回跳，问青蛙能不能到达终点，也就是数组的最后一个位置。

我一开始是直接使用了常规的动态规划去解题，毕竟这里题目的特征可谓是相当明显。`dp[i][j]` 表示的是是否可以从 `stones[i]` 位置跳到 `stones[j]` 位置。因为当前的跳跃的步数和之前的跳跃的步数有关，所以我们需要考虑两次跳跃。这样一来时间复杂度就变成了 `O(n^3)`，会超时报错。

这道题目如果从普通深度优先搜索的方向去思考，其实相对来说会简单不少，其实就是在每一个位置上都有三个选择，k - 1, k, k + 1。每个选择都去递归的试一下，只是单纯的暴力搜索肯定是不够的，因此我们需要增加记忆化的 cache，当前的位置和选择要跳的步数联合起来可以作为记忆化的 key 值来提升效率，这种方法的具体时间复杂度很难计算，主要跟输入数组中的值有关，但是从 LeetCode 的结果可以看到，相比于暴力的动态规划，还是快了不少。


<br>

**参考代码（暴力动归 TLE）**：

```java
public boolean canCross(int[] stones) {
    if (stones == null || stones.length == 0) {
        return false;
    }

    if (stones[1] != 1) {
        return false;
    }

    int n = stones.length;
    boolean[][] dp = new boolean[n][n];

    dp[0][1] = true;

    for (int i = 1; i < n; ++i) {
        for (int j = 0; j < i; ++j) {
            for (int k = 0; k < j; ++k) {
                if (dp[k][j]) {
                    if (stones[i] - stones[j] == stones[j] - stones[k]
                        || stones[i] - stones[j] == stones[j] - stones[k] + 1
                            || stones[i] - stones[j] == stones[j] - stones[k] - 1) {
                        dp[j][i] = true;
                    }
                }
            }
        }
    }

    for (int i = 0; i < n; ++i) {
        if (dp[i][n - 1]) {
            return true;
        }
    }

    return false;
}
```

**参考代码（记忆化搜索）**：
```java
public boolean canCross(int[] stones) {
    if (stones == null || stones.length == 0) {
        return false;
    }

    Set<Integer> posSet = new HashSet<>();

    for (int stone : stones) {
        posSet.add(stone);
    }

    return helper(stones, 0, 0, new HashMap<String, Boolean>(), posSet);
}

private boolean helper(int[] stones,
                       int k,
                       int pos,
                       Map<String, Boolean> memo,
                       Set<Integer> posSet) {
    // 如果超出范围，直接返回 false
    if (pos > stones[stones.length - 1]) {
        return false;
    }

    String key = "p" + pos + "k" + k;
    // cache
    if (memo.containsKey(key)) {
        return memo.get(key);
    }

    // 依次遍历三种情况，k - 1, k, k + 1
    for (int i = k - 1; i <= k + 1; ++i) {
        // 青蛙不能后退和原地不动
        if (i <= 0) {
            continue;
        }

        // 如果青蛙下一个跳到的位置不在数组中，继续遍历
        if (!posSet.contains(pos + i)) {
            continue;
        }

        // 去到下一个位置，递归到下一层
        if (helper(stones, i, pos + i, memo, posSet)) {
            return true;
        }
    }

    // 记录答案，如果青蛙此时在最后一个位置，则表示青蛙过河成功
    // 否则该方案下过河失败
    memo.put(key, pos == stones[stones.length - 1]);

    return memo.get(key);
}
```

<br>

## Review

[What software engineers don’t tell you](https://medium.com/swlh/what-software-engineers-dont-tell-you-5e9660b87fe6)

当下，互联网蓬勃发展。很多人都朝着 “码农” 这条路上去，有的转专业，有的换职业，网络上的各种培训机构比比皆是，很多人都像打了鸡血似的，拼命地往这条路上挤。但我们是否想过一个问题，就是 “**为的是什么？现实真如我们想的那么美好吗？**”，我们总会对媒体，公众夸大的事实信以为真。作者在这篇文章中讲述了一些程序员这个职业上的一些鲜为人知的事情，或许你可能会被泼上一头冷水，但是或许可以让你及时从自己的幻想中醒悟过来，抹去一些不切实际的期待。

需要知道的是，并不是所有的程序员都是一年拿着几百万的薪水，然后平日轻轻松松干自己喜欢的事情。进大公司的人是少数，拿高薪的也是少数。还有就是程序员完成的产品往往并不是出于自己的意愿或是需要。碰上不好的老板或是公司，那么加班加点便是常事。程序员的工作几乎都在写代码，工作中充满了细节和困难，很多时候没有更好的选择。

认清自己很重要，每个人都会有自己犯愁的事情，每个人也都有自己的目标，当你朝着自己认为对的目标前进的时候就不要看别人，别人或许有比你更高的目标，但这并不影响你什么，也不能说明他就比你开心，比你幸福。**失望往往源自于盲目的比较**。相信自己，朝着自己觉得对的方向去吧，那里阳光明媚，四季如春。

<br>

## Tip

最近开始重新使用 Vim，这个东西反复学，但是不练就是不会，这次下定决心要学好，这次先记录一些基础且常用的，知道的就不记了：

* 查找命令
```
/text       n -> 下一个，N -> 前一个
?text       反向查找
```
* 替换命令
```
:s/old/new
:s/old/new/g
:%s/old/new
:%s/old/new/g
```

* 文件操作
```
:e + filename     打开新文件
:r + filename     读取文件到当前文件
```

<br>

## Share

继续看耗叔的文章，有些文章回头看会有些不同的体会：

[高效学习：源头、原理和知识地图](https://time.geekbang.org/column/article/14321)

这一周刚换工作，去到一家有一定规模的公司，算是张了不少见识，里里外外忙得不可开交，有机会写个文章聊聊。