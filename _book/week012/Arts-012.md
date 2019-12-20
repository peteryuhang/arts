[LeetCode 115. Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/)

**思路分析**<br>

题目的意思是求出一个字符串（子串）在另一个字符串（主串）的子序列中出现的次数，再换个说法，主串中有多少子序列是等于子串的。一个字符串匹配问题，我们当然可以考虑到暴力求解，思路大概就是，我们假设主串长度是 n，子串长度是 m，我们在主串中选出所有的长度为 m 的子序列，看这些子序列有多少是等于子串的，这样的话时间复杂度是很高的，其实就是组合问题的时间复杂度，是指数级别的。于是要考虑动态规划，基本上和字符串匹配相关的题目都可以考虑试着画画表格，这道题拿到手我一开始想到要用动归，状态的定义也不是太难，就是 `dp[i][j] -> 子问题 s[0...i], t[0...j] 的答案`，但是这个递推方程想了挺久的，因为想不出来还不停地试着重新定义状态，最后画表格总结规律，慢慢地就出来了；一般这样的字符串匹配问题，对于当前的问题，也就是 `dp[i][j]`，我们可以去观察相邻的子问题 `dp[i - 1][j - 1], dp[i][j - 1], dp[i - 1][j]`，字符串匹配问题，我们是一个个字符对比考虑，因此这里也只会有两种情况，当前比较的字符相等和当前比较的字符不相等，如果说相等，`s[i] == t[j]`，这时我们需要考虑的子问题是 `s[0...i-1], t[0...j-1]`，对于这个子问题的答案我们已经计算过了，是 `dp[i - 1][j - 1]`，但是这还没有完，这只能算是一部分答案，还需要考虑的子问题是 `s[0...i-1], t[0...j]`，也就是去掉当前 s 中的字符 `s[i]`，答案也已经计算过了，就是 `dp[i][j - 1]`，很多人可能会问，这么考虑会不会重复？其实不会，因为所有的当前问题不会去考虑子问题 `s[0...i], t[0...j-1]`，也就会保证考虑的子问题不会重复，这么说可能不好理解，画个表格就会更好理解。如果说不相等，`s[i] != t[j]`，这时我们需要考虑的子问题就是 `s[0...i-1], t[0,...j]`，直接点说就是，去掉主串当前考虑范围中的当前字符的解。

<br>

**参考代码**
```java
public int numDistinct(String s, String t) {
    if (s == null || t == null || s.length() < t.length()) {
        return 0;
    }
    
    if (s.length() == 0 || t.length() == 0) {
        return 1;
    }
    
    char[] sArr = s.toCharArray();
    char[] tArr = t.toCharArray();
    
    // dp[i][j] -> answer for subproblem: s[0...i] and t[0...j]
    
    // if s[i] == t[j] -> dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j]
    // dp[i - 1][j - 1] means the count not consider s[i], which is s[0...i-1] match t[0...j-1]
    // dp[i][j - 1] means the count consider s[i], which is s[0...i] match t[0...j-1]
    
    // if t[i] != s[j] -> dp[i][j] = dp[i][j - 1]
    int[][] dp = new int[sArr.length][tArr.length];
    
    dp[0][0] = (sArr[0] == tArr[0] ? 1 : 0);
    for (int i = 1; i < sArr.length; ++i) {
        for (int j = 0; j < Math.min(i + 1, tArr.length); ++j) {
            if (sArr[i] == tArr[j]) {
                dp[i][j] = (j == 0 ? dp[i - 1][j] + 1 : dp[i - 1][j] + dp[i - 1][j - 1]);
            } else {
                dp[i][j] = dp[i - 1][j];
            }
        }
    }
    
    return dp[sArr.length - 1][tArr.length - 1];
}
```

<br>


## Review
两篇出于同一作者，观点鲜明，卓有见地的文章，讲的是编程当中的误区:<br>

[The Greatest Developer Fallacy Or The Wisest Words You’ll Ever Hear?](https://skorks.com/2011/02/the-greatest-developer-fallacy-or-the-wisest-words-youll-ever-hear/)

[On The Value Of Fundamentals In Software Development](https://skorks.com/2010/04/on-the-value-of-fundamentals-in-software-development/)

第一篇文章讲的是程序员耳熟能详的一句话 “**我在需要的时候再去学习**”，这句话本是没有错，因为计算机软件这个领域新技术层出不穷，不可能都学完，而且为了学习去学习也没有意义。但是现在的问题是，**很多程序员把这句话当成了不去深入专研和挖掘知识的借口**，作者还给出了一个观点 “**你不可能去学习你完全不知道的东西**”，怎么理解？假如说你只知道 Java Spring 这一个 Web 的框架，当你需要构建 Web 服务器的时候，你只会去考虑用 Java Spring，但是实际上，除了这个，还有 Python 的 Django，JavaScript 的 Express，Ruby 的 Ruby On Rails，这些都是 Web 框架，但它们适用于的场景不同，上手难度不同，适合解决的问题也不一样，相对于 Java Spring，这些技术知识都是你不怎么了解，也不知道其优缺点，即使它们再好，对你来说也等同于无。作者提出我们应该成为一个 **T 型人才**，他同时也指出当你去**深入专研某个方向的时候，你会连带学到这个方向之外的很多东西**，所以问题的解决方案就是，对于你感兴趣的东西，你要学的更深入些，不要学在表面，当然**相对于知识的积累，更重要的是对这个行业的热情，绝大多数的学习到最后是否有成效和收获都会归结为态度问题**。

第二篇文章讲的是基础知识的学习，作者给出了基础的定义，一个是从**宏观**来看，另一个是从**微观**来看，从宏观上来看，比如**数据结构和算法，操作系统，网络协议的知识**，之所以是宏观上的，是因为这些东西往往是普适的，任何的技术都会基于此，举个例子，不管是 Java 中的 ArrayList，还是 C++ 中的 Vector，Python 中的 list，JavaScript 中的 list，这些东西本质都是动态数组，只是表示的方式不一样，你如果理解了数据结构和算法中的动态数组的相关知识和由来，这些你会学的很快，使用也会得心应手，其他相关知识也一样。微观之所以是微观，是因为其落实到具体技术，比如对于 Java 的 ArrayList 的学习，你需要去熟悉相关的 API 文档，了解并熟练掌握其各种的常见 API 函数和相关的操作，这些东西可以帮我们快速写代码，有时候不知道一个 API 或者内置函数，往往会大大影响我们开发的效率。所以总结下就是**宏观上的基础学习能够让你学的更快，微观上面的基础的学习会让你实现，写代码的效率更快，同时对比不同技术之间的差异，达到触类旁通的效果**。

<br>


## Tip
分享一个 Linux 上面挂载程序的指令，screen 命令，使用起来特别简单，一般步骤如下：
```
>$ screen  
按 enter 键
>$ node ...      (这里输入命令)
ctr + A          (挂载)
ctr + D          (退出)
```
<br>

这周看极客上面的 HTTP 专栏，学习了一些网络的概念，之前完全不知道，但又频繁见到的一些词汇现在有了概念了，算是扫盲吧，记录一下，巩固一下：
* **WWW 万维网**是互联网的一个子集，它基于 HTTP 协议，传输 HTML 超文本资源
* **CDN（Content Delivery Network）**，是客户端到服务器中间的一个部分，应用了 HTTP 中的缓存和代理技术，代替源站响应客户端的请求
* **RPC（Remote Procedure Call）**，远程过程调用，是一个计算机协议，允许一个计算机调用另一个计算机上的子程序，而程序员无需额外为这个操作编程
* **HTTPS** 是运行在 SSL/TLS 上的 HTTP，**TLS**（Transport Layer Security）是由 **SSL**（Secure Socket Layer）发展过来的，运用了很多加密算法
* **IP 协议**主要解决寻址和路由问题，**TCP 协议**是建立在 IP 之上，基于 IP 协议，提供可靠的字节流形式的通信
* **正向代理**是指客户端这边的代理，**反向代理**是指服务器端的代理

<br>


## Share
这周读了《清醒思考的艺术》这本书，书中给出了 52 个思维误区，我觉的有些误区对于我们这些程序员，或者说是想成就一番事业的人士非常有借鉴意义

[学习中的典型思维误区总结](./学习中的典型思维误区总结.md)
