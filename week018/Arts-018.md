## Algorithm
[PE 24. Lexicographic Permutations](https://projecteuler.net/problem=24)

[LeetCode 1124. Longest Well-Performing Interval](https://leetcode.com/problems/longest-well-performing-interval/)

**思路分析（一）**<br>
这是一道 Project Euler 上面的题目，之前介绍过，这个平台上面的题目偏数学一点，但是大部分还是需要写程序解决。这道题是和排列相关的题目，**如果给你数字 0，1，2，那么所有排列从小到大就会是 `012, 021, 102, 120, 201, 210`，然后题目问如果给你 0，1，2，3，4，5，6，7，8，9 让你去排，那么从小到大排在第一百万的数是多少？** 写普通算法程序题写多了，见到排列首当其冲想到用深度优先搜索，最后发现如果把所有的排列可能都给列出来，然后排序，这样搞下来程序运行的时间复杂度真的是没眼看 **O(n!)**，比指数的时间复杂度还要高。于是试着用数学上面的方法来解，其实思路很简单，就是一个数一个数地选。首先第一个位置（最高位）有十种可能性，我们可以利用 10! / 10 来计算出每一个数字打头有多少种排列，然后用 **一百万 / (10! / 10)** 来决定我们要选排在第几的数，选中一个数后，剩下的数就只有 9 个，将 10 换成 9，重复上面的操作，当然，一百万这个值也得根据我们找的数来做更新，比如你第一个数选择了 2，那么 0 和 1 打头的 **2 * 10! / 10** 种组合都可以减掉，剩下的我们需要继续在后面的 9 个数的排列中找

<br>

**参考代码（一）**
```java
public static void main(String[] args) {
    System.out.println(lexicographicPermu(10, 1000000L));
}

private static String lexicographicPermu(int numberOfDigit, long target) {
    int copyOfNumerOfDigit = numberOfDigit;
    long copyOfTarget = target;

    // list 里面存放的是所有的可以选择的情况，并从大到小排列
    // 这里是 0，1，2，3，4，5，6，7，8，9
    List<Integer> candidate = new ArrayList<>();
    for (int i = 0; i < copyOfNumerOfDigit; ++i) {
        candidate.add(i);
    }

    String result = "";

    // 只要没有选完，就继续选择
    while (copyOfNumerOfDigit != 0) {
        long totalPermutation = 1;
        int n = copyOfNumerOfDigit;
        
        // 计算 n!
        // 例如，选择第一个数，那 totalPermutation 就是 10!
        while (n != 1) {
            totalPermutation *= n;
            n--;
        }
        
        // 计算选择当前情况下的第几个数
        // 这里 copyOfTarget - 1 是为了防止整除的情况
        // 举 0，1，2 这个例子，当我们要选择排在第 2 的那个数，也就是 021
        // 选第一个数的时候， totalPermutation = 3 * 2 * 1 = 6, copyOfNumberOfDigit = 3
        // 2 / (6 / 3) = 1，显然不符合要求，(2 - 1) / (6 / 3) = 0 才是选中了 0
        int selection = (int)((copyOfTarget - 1) / (totalPermutation / copyOfNumerOfDigit));
        result += candidate.remove(selection);
        
        // 更新下一次要选择第几个数
        copyOfTarget -= (totalPermutation / copyOfNumerOfDigit) * selection;
        
        // 选了一个数，可选项相应减少一
        copyOfNumerOfDigit--;
        copyOfTarget = copyOfTarget <= 0 ? 0 : copyOfTarget;
    }

    return result;
}
```

<br>

**思路分析（二）**<br>
LeetCode 这次竞赛的一道题，感觉是一道比较诡异的题。题目描述很简单，**给定一个数组，这个数组里面的每个值表示的是一个人的一天的工作时间，当这个人的某天工作时间大于 8 的话，那么这一天就可以算作是积极的，不然就是消极的，问最长的积极区间**。我起初知道这个肯定不能用滑动窗口来解，然后试着 O(n^2) 的暴力求解法，暴力法应该很好理解，就是一个区间会有起始和终止，我确定其中一个去找另外一个就行，但是很遗憾，直接 TLE。最后的解法是看答案的，感觉很难想到，其利用了数组的前缀和的性质，什么是前缀和数组？就比如一个数组是 `[1,2,3,4,5,6]`，我们可以得到其前缀和数组是`[0,1,3,6,10,15,21]`，这个有什么用？有了这个数组你可以求出任意一段区间的和 `array[i]+...+array[j] = sum[j] - sum[i]`，比如这个例子中 `array[2]+...+array[5] = sum[5]-sum[2] = 18`。这道题中，我们可以将大于 8 的数看成 1，小于等于 8 的看成 -1，这道题有了前缀和数组还不够，对于相同的前缀和我们只考虑最小的那个，也就是说 `array[0]+array[1]=-1`, `array[0]+array[1]+...+array[10]=-1`，我们只考虑第一种情况，原因是我们是要选择一个最长的区间，当我们遍历的时候我们假设当前遍历的点是区间的终止，这时我们去找区间的开始的地方，希望这个开始的 index 越小越好，这也是这道题最诡异的地方。

<br>

**参考代码（二）**
```java
public int longestWPI(int[] hours) {
    Map<Integer, Integer> map = new HashMap<>();
    
    // curTiringDay 记录区间 [0, i] 的积极值
    int curTiringDay = 0, result = 0;
    for (int i = 0; i < hours.length; ++i) {
        if (hours[i] > 8) {
            curTiringDay++;
        } else {
            curTiringDay--;
        }
        
        // curTiringDay > 0 表示 [0,i] 是积极区间，直接记录即可
        if (curTiringDay > 0) {
            result = Math.max(result, i + 1);
        } 
        // curTiringDay <= 0, [0,i] 是消极区间
        // 这时我们要找 curTiringDay - 1 的 index
        // 这样 curTiringDay - (curTiringDay - 1) > 0 刚好满足，区间 [index, i] 就是积极的
        // 保证 [index, i] 是最长的需要在更新的时候保证 map 当中存在就不更新
        else {
            map.putIfAbsent(curTiringDay, i);
            if (map.containsKey(curTiringDay - 1)) {
                result = Math.max(result, i - map.get(curTiringDay - 1));
            }
        }
    }
    
    return result;
}
```

<br>

## Review
[Understanding JSON Web Token Authentication](https://blog.bitsrc.io/understanding-json-web-token-authentication-a1febf0e15)

文章中介绍了 JWT 的一些性质，并且给出了相应的实现。
* JWT 保证了身份验证的安全性，JWT 其实是一个长长的字符串，有三个部分用 '.' 进行分隔，这三个部分分别是 Header、Payload 以及 Signature。
    * Header 表示的是这个 token 的类型以及所使用的加密算法
    * Payload 是我们要发送的数据
    * Signature 是用来验证这个 token 是否被改变，通常要结合 private key
* npm 上的 **jsonwebtoken** 可以帮助我们去创建 JWT
* npm 上的 **express-jwt** 可以帮助我们来进行传入 JWT 的验证

<br>

## Tip
这周看狼叔的 ["如何正确的学习 Node.js"](https://i5ting.github.io/How-to-learn-node-correctly/#10308) 一篇文章中提到了一个学习 Node 的一个方法就是每天都去看一些 Node 的开源模块，并且这里给了一个 [repo](https://github.com/parro-it/awesome-micro-npm-packages)，里面有很多的 NPM 的模块，每天都去看看别人是怎么写开源模组的，然后结合自己在工作中写代码的方式和方法，试着去发现其中的不同，取其精华，去其糟粕，相信慢慢地对 JS 的代码理解能力会增强。慢慢来，比较快

<br>

## Share
看了高程 3 中函数表达式一章，深刻体会了函数作为 JavaScript 的一等公民的实际用处，在这里写写总结，希望以后回过头来看，体会会更深。

[JavaScript 中的闭包函数](./JavaScript中的闭包函数.md)

