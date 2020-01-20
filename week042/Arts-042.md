> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 1312. Minimum Insertion Steps to Make a String Palindrome](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/)


**题目解析**：

题目给定一个字符串，问最少插入多少个字符可以使得这个字符串变成回文串。拿到这个问题，我们可以先回顾一下，如何判断一个字符串是回文串？最直接的做法就是相向双指针，这道题我们也可以参考这样的思路，如果头尾的两个指针指向的字符相等，那我们就同时相向移动两个指针，**但是如果头尾的两个指针指向的字符不相等，我们就需要插入字符了**，这里有两个选择，在头指针的前方插入尾指针指向的字符，移动尾指针；还有就是在尾指针的后方插入头指针指向的字符，移动头指针。每当头尾指针指向的字符不相同的时候，我们都有这两种选择，不知道说到这个地方，你是否发现这道题目其实是一道区间类的动态规划题目，最终，我们要求使得区间 `[0, n]` 是回文的最小插入次数，我们可以定义，`dp[i][j]` 表示的是使得区间 `[i, j]` 是回文的最小插入次数，我们也可以得出递推方程：

```
s[i] == s[j]: dp[i][j] = dp[i + 1][j - 1]
s[i] != s[j]: dp[i][j] = Math.min(dp[i][j - 1], d[i + 1][j]) + 1
```
推荐使用记忆化搜索的实现方式，这样可以减少不必要的考虑。

<br>

**参考代码**：
```java
public int minInsertions(String s) {
    int[][] memo = new int[s.length()][s.length()];

    for (int i = 0; i < s.length(); ++i) {
        Arrays.fill(memo[i], Integer.MAX_VALUE);
    }

    return helper(s.toCharArray(), 0, s.length() - 1, memo);
}

private int helper(char[] sArr, int left, int right, int[][] memo) {
    if (left >= right) {
        return 0;
    }

    if (memo[left][right] != Integer.MAX_VALUE) {
        return memo[left][right];
    }

    if (sArr[left] == sArr[right]) {
        return memo[left][right] = helper(sArr, left + 1, right - 1, memo);
    }

    return memo[left][right] = Math.min(helper(sArr, left + 1, right, memo),
                                        helper(sArr, left, right - 1, memo)) + 1;
    
}
```

<br>

## Review
[97 Things Every Programmer Should Know](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/)

继续 97 件事

[The Boy Scout Rule](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_08/)

简单直白的文章只讲了一件事情，就是 **帮助别人也是帮助自己**，看到不合乎规范的代码，就尽自己的能力去改进，先不要管这是谁写的代码，能做一点是一点，如果一个团队，人人都有这样的意识，那么整个事情一定会朝着正确的方向大步前进。

[Check Your Code First before Looking to Blame Others](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_09/)

每当我们发现了非常棘手的 bug 的时候，我们很可能会下意识地去怀疑系统、编译器、网络等环境是否出错，我们会花大量的时间去根据自己的猜想去进行排查，但是到头来却发现还是自己的问题，这其中，我们花了大量的时间去证明自己的主观猜想，这其实是不可取的，更可靠的其实是从上到下，用排除法，排除那些最不可能出错的地方，剩下的或许就是答案。

[Choose Your Tools with Care](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_10/)

在当下的开发中，我们往往会选择使用一些免费的第三方库函数，这为我们的开发节省了大量的成本和时间，但是作者这里也提到，频繁地，不假思索地使用这些库函数同样也会产生不小的问题，比如如果耦合依赖过多，一个模块的更新升级有可能会导致整个系统的不可用，还有就是配置文件会变得错综复杂，作者强调 “**仅当有需要的时候再考虑使用第三方套件和外部工具**”，另外，尽量不要去使用一些低层次的外部工具，比如 socket，使用 middleware 进行替代，还有就是将外部工具的使用和业务逻辑耦合开来，这样对于更换工具以及错误的排查也会有帮助。

<br>

## Tip

这周分享 LeetCode 在线 IDE 上的两个小技巧，这两个技巧对于每周的 contest 可以节省一些时间：
* 函数中传入参数的名称可以更改（将长的名字更替为短的名字节省敲代码的时间）
* custom test case 可以输入多个，不同的 test case 之间，除了原有的回车之外不需要任何的间隔符

<br>

## Share

眼看就新的一年开始了，大家都开始制定新的一年的目标，在这里我也分享一下我的新年愿望和计划吧

[新年计划-展望2020](./新年计划-展望2020.md)