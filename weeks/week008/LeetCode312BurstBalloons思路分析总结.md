[LC 312. Burst Balloons](https://leetcode.com/problems/burst-balloons/)

### 题目描述
Given n balloons, indexed from 0 to n-1. Each balloon is painted with a number on it represented by array nums. You are asked to burst all the balloons. If the you burst balloon i you will get nums[left] * nums[i] * nums[right] coins. Here left and right are adjacent indices of i. After the burst, the left and right then becomes adjacent.

Find the maximum coins you can collect by bursting the balloons wisely.

Note:

You may imagine nums[-1] = nums[n] = 1. They are not real therefore you can not burst them.
0 ≤ n ≤ 500, 0 ≤ nums[i] ≤ 100

<br>

### 思路分析与代码
拿到这道题的时候，可以从题目描述看出最后要求解的问题是存在子问题的，而且最后的求解是 “最大值”，可以想到用动态规划来求解，但是难点在于状态怎么定义？递推方程是什么？数组里面的数的数目是在动态变化的，如果直接从动态规划的思路想，很难看出当前问题和前面的子问题的关系是什么。因此，首先可以考虑**暴力的深度优先搜索**，但是这里有两个思路：
* 当前考虑的气球是最**先**扎爆的气球
* 当前考虑的气球是最**后**扎爆的气球

其实第一种思路是最好理解的，但是写代码的时候你会发现很多问题，例如如何知道当前数的左右相邻的数是什么？你可以通过移除数组中的数来获得，但是这会导致时间的增多，并不是一个高效的做法。在看看第二种思路，如果最后打爆的气球编号是 i，那么说明 [0,i-1] 和 [i+1,n-1] 两个区间上面的气球已经被打爆，这里的答案就是 `1*i*1 + [0,i-1]的解 + [i+1,n-1]的解`，这样一个问题被拆分成两个子问题，子问题还可以继续往下拆分
```
               i
        [0,i-1] [i+1,n-1]
          ...     ...

ans(0,n-1) = max(
    nums[-1]*nums[0]*nums[1]+ans(1,n-1),            // 最后打 0 号气球
    ans(0,0)+nums[0]*nums[1]*nums[2]+ans(2,n-1),    // 最后打 1 号气球
    ans(0,1)+nums[1]*nums[2]*nums[3]+ans(3,n-1),    // 最后打 2 号气球
                      ...
    ans(0,n-3)+nums[n-3]*nums[n-2]*nums[n-1]+ans(n-1,n-1),  // 最后打 n-2 号气球
    ans(0,n-2)+nums[n-2]*nums[n-1]*nums[n],         //最后打 n-1 号气球
); where nums[-1] == nums[n] == 1
```

<br>

这里可以看出，这样的思路和分治很像，的确如此，这道题目特殊的地方也在于此，它其实是动态规划和分治的结合，我们稍后再详细说明，根据上面的思路，我们可以转化为代码：

```java
public int maxCoins(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    
    return helper(nums, 0, nums.length - 1);
}

private int helper(int[] nums, int l, int r) {
    if (l > r) {
        return 0;
    }
    
    int max = nums[l];
    for (int i = l; i <= r; ++i) {
        int cur = helper(nums, l, i - 1)
                    + get(nums, l - 1) * nums[i] * get(nums, r + 1)
                    + helper(nums, i + 1, r);
        
        max = Math.max(max, cur);
    }
    
    return max;
}

private int get(int[] nums, int i) {
    if (i < 0 || i >= nums.length) {
        return 1;
    }
    
    return nums[i];
}
```

<br>

上面的代码非常的简洁，核心代码就是 for 循环里面的递归，但是注意这只是暴力的解法，之所以是暴力是因为它做了很多之前做过、没有必要的重复操作，你可以从之前讲过的递推公式可以看到，或者可以画一个递归树状图来看。这里只需要加上一个数组帮助记录之前做过的事情，就可以大大提高效率

```java
public int maxCoins(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    
    int[][] dp = new int[nums.length][nums.length];
    
    return helper(nums, dp, 0, nums.length - 1);
}

private int helper(int[] nums, int[][] dp, int l, int r) {
    if (l > r) {
        return 0;
    }
    
    if (dp[l][r] != 0) {
        return dp[l][r];
    }
    
    int max = nums[l];
    for (int i = l; i <= r; ++i) {
        int cur = helper(nums, dp, l, i - 1)
                    + get(nums, l - 1) * nums[i] * get(nums, r + 1)
                    + helper(nums, dp, i + 1, r);
        
        max = Math.max(max, cur);
    }
    
    dp[l][r] = max;
    
    return max;
}

private int get(int[] nums, int i) {
    if (i < 0 || i >= nums.length) {
        return 1;
    }
    
    return nums[i];
}
```

<br>

其实上面的代码实现已经用到了动态规划了，但是是使用了递归的实现方式，这时候我们再回过头去看看动态规划里面的 “状态” 和 “递推公式” 就一目了然：

```
dp[i][j] -> 输入数组 [i,j] 区间上的最大值

dp[0,n-1] = max(
    nums[-1]*nums[0]*nums[1]+dp[1,n-1],         
    dp[0,0]+nums[0]*nums[1]*nums[2]+dp[2,n-1],
    dp[0,1]+nums[1]*nums[2]*nums[3]+dp[3,n-1],
                      ...
    dp[0,n-3]+nums[n-3]*nums[n-2]*nums[n-1]+dp[n-1,n-1],
    dp[0,n-2]+nums[n-2]*nums[n-1]*nums[n],
);
```

有了状态的定义和递推公式，我们就可以直接上手动态规划了，但是注意边界条件：
```
public int maxCoins(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    
    int[] newNums = new int[nums.length + 2];
    
    newNums[0] = newNums[nums.length + 1] = 1;
    for (int i = 0; i < nums.length; ++i) {
        newNums[i + 1] = nums[i];
    }
    
    int n = newNums.length;
    
    int[][] dp = new int[n][n];

    for (int i = 2; i < n; ++i) {
        for (int l = 0; l < n - i; ++l) {
            int r = l + i;
            for (int j = l + 1; j < r; ++j) {
                dp[l][r] = Math.max(dp[l][r], 
                             newNums[l] * newNums[j] * newNums[r] 
                                    + dp[l][j] + dp[j][r]);
            }
        }
    }
    
    return dp[0][n - 1];
}
```

这是一个二维的动态规划，如果在纸上画表格来看 DP 数组的记录轨迹的话，你会发现这里记录只用到二维矩阵的上三角，也就是以对角线为轴的一半；记录顺序也并不是一行一行下来的，而是以对角线进行的。

<br>

### 总结
这里可以看到我们解这道题的一个过程是
1. 思考并实现暴力求解
2. 画树状图或者思考是否有重复的子问题
3. 在暴力求解的基础上，看能不能增加记忆化，记录之前解过的子问题的解
4. 通过状态和递推公式，试着用动态规划求解

往往遇到不能一下子得到最优算法，或者没有什么思路的题，都可以按这个步骤试试。往往动态规划能够解决的问题，暴力搜索都可以解，但是反过来就不一定了。只是说暴力搜索它并不是我们的终点，但它却可以为我们提供一个不错的突破口，我们在此基础之上再来思考如何进一步地优化，得到我们最终想要看到的算法。不断地去熟练这么一个过程，相信思维能力和直觉能力会不由自主地提高。

另外就是这道题的一个非常巧妙的地方就是把分治的思想加了进来，分治算法与动态规划算法不同的地方，或者说是截然相反的地方是，分治是不存在重复子问题的，不理解的话可以想想**快速排序**，一个区间被一分为二，被分开的两个区间不存在任何交集，它们各自在各自的空间内做事情；正是因为这一点，在思考暴力求解的时候，按照分治的思想，选择的方向则是从结果导向，倒着去想，**先分再合**，分到不能分为止，再去合并，对于这道题来说，合并是非常简单的，就是相加；但是如果我们按照一般动态规划的思路顺着去想，那么打爆一个气球后，这个气球的左右区间将会合在一起，这无法将一个问题化成更小的问题去解决。虽然这样的思路打破了我们通常的思维方式，但是还是那句话，多积累，现在不会以后会，见得多了就不怕了。