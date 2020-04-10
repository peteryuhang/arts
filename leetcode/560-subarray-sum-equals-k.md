## LeetCode 图解 | 560. Subarray Sum Equals K

[LC 560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

### 题目解析

题意很简单，就是让你在一个数组中找出子数组和等于 K 的所有子数组的个数。首先这道题并没有特别要求时间复杂度，我们可以大胆地去思考可行解。子数组是由一个区间组成，一个区间是由两个边界，或者说是左右两个元素确定，那么我们把所有元素两两配对的情况都遍历一遍即可，于是我们可以得到下面的代码：

```java
public int subarraySum(int[] nums, int k) {
    if (nums == null || nums.length == 0) {
        return 0;
    }

    int n = nums.length;

    int result = 0;
    for (int i = 0; i < n; ++i) {
        int sum = 0;
        for (int j = i; j < n; ++j) {
            sum += nums[j];
            if (sum == k) {
                result++;
            }
        }
    }

    return result;
}
```

首先要说明一点，上面的代码在 LeetCode 平台上是可以提交通过的，而且意图直接了当。就是**找出所有的 `[i,j]` 区间及其区间和 `sum`，然后和 k 进行比较计数**。但是你也可以发现，这个代码的时间复杂度比较高，两个嵌套 for 循环，最后就是 `O(n^2)` 的最差时间复杂度。

但有了这个可行解，我们可以基于此进行优化。上面的解法把一个区间的两头都遍历了一下，那么我们就可以思考，**能不能只知道区间的一头，就可以确定符合条件的区间个数？** 如果我们从左向右遍历数组，当我们遍历到一个元素的时候，如果以当前遍历到的元素作为区间的右边界，那么我们必须从已经遍历过的元素中找一个作为左边界。很明显，这个元素左边的元素我们全部都已经遍历过了，问题来了，怎么选择？这里可以用到一个 **前缀和** 的知识。数组中一个元素的前缀和是指这个元素和其前面所有的元素的加和，知道了这个我们可以很容易的求出一个区间的和，如果我们用一个哈希表保存前面元素的前缀和，那么当遍历到一个元素的时候，就可以去哈希表中寻找有没有符合条件的前缀和：

```
假设一个数组：[...i,...j...]
前缀和：prefixSum_i = sum([0,...,i])
前缀和：prefixSum_j = sum([0,...,i,...,j])
区间和：intervalSum[i + 1, j] = prefixSum_j - prefixSum_i

当我们知道了 prefixSum_j，我们要找的符合条件的值 x 就是：
prefixSum_j - x = k 
x = k - prefixSum_j
这个 x 是前面任意元素的前缀和
```

你可以看到，通过前缀和加哈希表，我们就可以把这道题的时间复杂度给降到 `O(n)`。总得来说这道题目不难，在这里我们首先给出了暴力的可行解，再在暴力可行解的基础上进行优化。

这种题很容易在面试的时候遇到，如果在面试的时候遇到，切记一定要展现出自己的思考过程。因为面试是一个交流的过程，你要给面试官的感觉是你在和他合作探讨解决一个问题，而不是仅仅给出答案。**在交流之中，不断地通过面试官给的需求和条件推出未知解，遇到不懂的也大胆说出自己的想法，你要展现出的思考过程需要是渐进的，连贯的。** 就像你展现出的自己是由慢到快，从走到跑的这么一个过程，这就会给人一种很自然的状态。总之，在面试中做出题目永远不是目的，而是要表现出自己是一个懂得倾听，善于合作的伙伴。

<br>

### 动画描述

见 PPT

<br>

### 代码实现

```java
public int subarraySum(int[] nums, int k) {
    if (nums == null || nums.length == 0) {
        return 0;
    }

    int n = nums.length;

    // map 用于记录前面遍历过的元素的前缀和
    // 因为可能存在相同的前缀和
    // 这里用 key 表示前缀和，value 表示前缀和的个数
    Map<Integer, Integer> map = new HashMap<>();
    
    int result = 0, prefixSum = 0;

	// 初始化 map，如果整个前缀和数组就是答案，需要减去的值就是 0
    map.put(0, 1);
    for (int i = 0; i < n; ++i) {
        prefixSum += nums[i];

        // 如果需要区间的值是 k，那么要找的左边界元素的前缀和就是 minuend
        int minuend = prefixSum - k;

        // 如果要找的前缀和存在，就记录答案
        if (map.containsKey(minuend)) {
            result += map.get(minuend);
        }

        // 记录当前元素的前缀和
        map.put(prefixSum, map.getOrDefault(prefixSum, 0) + 1);
    }

    return result;
}
```






