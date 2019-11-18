## Algorithm

[LC 679. 24 Game](https://leetcode.com/problems/24-game/)

<br>

**题目解析**：

题意是给你四张卡片，每张卡片上面是 1 - 9 当中的一个数字，题目问的是，这四张卡片之间通过加减乘除，以及括号构成表达式，是否有表达式能得到 24 这个结果。因为这道题的数据规模只有四个数字，因此我们可以先考虑一下最暴力的解法，也就是四个数字，四种运算，把所有的情况都罗列一遍，但是这里的问题是 **括号怎么处理**，需不需要把括号也当作是运算符？其实仔细想想，也很好办，因为加减乘除都是双目运算符，也就是说这个运算符需要两个数字，结果是一个数字，我们完全可以枚举元素之间的计算顺序来达到目的，其实括号也就是改变来元素之间的计算顺序。

因此题目的大致思路就是，从数组中选择两个数字，然后选择一个运算符号计算，将计算结果扔进数组，再次进行相同的步骤，直到我们发现数组中只剩下一个元素，然后这个元素等于 24，我们就可以停止枚举，因为我们已经找到答案，否则就继续枚举，直到我们枚举了所有的情况。我们通过回溯来达到枚举所有可能性这一需求

<br>

**参考代码**：
```java
public boolean judgePoint24(int[] nums) {
    if (nums == null || nums.length != 4) {
        return false;
    }
    
    List<Double> tempResult = new ArrayList<>();
    
    // 因为有除法，我们得到的可能是浮点数
    // 这里把输入数组的元素统一改为 Double 类型，方便计算
    for (int i = 0; i < nums.length; ++i) {
        tempResult.add((double)nums[i]);
    }
    
    char[] oper = {'*', '/', '+', '-'};
    
    return solve(tempResult, oper);
}

private boolean solve(List<Double> nums, char[] oper) {
    // 浮点数之间无法直接比较是否相等
    // 这里用精度代替
    if (nums.size() == 1 && Math.abs(nums.get(0) - 24) < 1e-8) {
        return true;
    }
    
    if (nums.size() == 1) {
        return false;
    }

    for (int i = 0; i < nums.size(); ++i) {
        double num1 = nums.remove(i);
        for (int j = 0; j < nums.size(); ++j) {
            double num2 = nums.remove(j);
            for (int o = 0; o < oper.length; ++o) {
                if (oper[o] == '*') {
                    nums.add(num1 * num2);
                } else if (oper[o] == '+') {
                    nums.add(num1 + num2);
                } else if (oper[o] == '/') {
                    nums.add(num1 / num2);
                } else {
                    nums.add(num1 - num2);
                }
                
                if (solve(nums, oper)) {
                    return true;
                }
                
                nums.remove(nums.size() - 1);
            }
            
            nums.add(j, num2);
        }

        nums.add(i, num1);
    }
    
    return false;
}
```

<br>

## Review

[The Solution to Feeling like Life Sucks in a 3-Minute Read](https://psiloveyou.xyz/the-solution-to-feeling-like-life-sucks-in-a-3-minute-read-dfe04e467d64)

算不上一篇技术文章，但是读完之后还是会有一些感触。

平时我们的关注点大多都在别人、或者周围的环境上，比如，领导对自己的付出是否满意，如何更好地去让别人理解自己的观点，如何让别人认同自己。因为注意力都放在别人的身上，我们就会去拿自己去和别人比较，发现自己不如别人的时候，就会觉得自己的生活特别的糟糕，何不花些时间来看看自己呢？今天的自己比昨天的自己有进步，这就够了，相信自己能够变得更好，这才是自信

<br>

## Tip

这次推荐一个压力测试的软件，[JMeter](https://jmeter.apache.org/download_jmeter.cgi)，可以看到这是一个 apache 的开源项目，也可以看看这个 [使用教程](https://www.tutorialspoint.com/jmeter/index.htm)，我仅仅是使用它来进行 API 的压力测试，可能还有很多有用的功能，等待开发。

<br>

## Share
接着来聊动态规划，这次是序列型动态规划，和之前的矩阵类动态规划有些许类似，但又有些不同。

[动态规划之序列类动规](./动态规划之序列类动规.md)