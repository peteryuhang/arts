## 概述
[TwoSum](https://leetcode.com/problems/two-sum/) 作为 LeetCode 的第一题存在，想必大家应该对其并不陌生。如果仅仅是看这道题目本身，并不难，思想也特别的简单，但是关键问题在于，由这个问题演变出来的题目和思路比较多，而且存在着不少的细节问题，**今天我们就借着具体的题目和思路来看看 TwoSum 还可以怎么玩？**

<br>

## 两种思路
对于 TwoSum 类问题，总的来说有两种大的方向，一种方向是借助 Hash 表，另外一种是借助排序，然后利用相向双指针来解决问题，我们分别来看看：

**首先是 Hash 表的做法**：
```
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> remainValues = new HashMap<>();
    
    for (int i = 0; i < nums.length; ++i) {
        if (remainValues.containsKey(target - nums[i])) {
            int anotherIndex = remainValues.get(target - nums[i]);
            
            return new int[] {i, anotherIndex};
        }
        
        remainValues.put(nums[i], i);
    }
    
    return new int[] {-1, -1};
}
```

思路很简单，遍历数组，每访问一个元素，先判断其配对的元素是否在 Hash 表中，如果在的话就说明我们找到了答案，将其输出即可，如果没有找到，就将当前的元素放入 Hash 表中，以方便后面的元素来配对，这里因为题目要求输出元素在数组中的位置，所以用 HashMap 来存储访问过的元素和其对应的 index。我们再来分析一下其时空复杂度，由于使用了 Hash 表，**空间复杂度是 O(n)** 的，另外就是最差的情况，数组中的每个元素都要遍历到，因此**时间复杂度也是 O(n)**。从时间上面来看，这个算法肯定是最优的，这很好理解，你要在数组中寻找配对的答案，数组当中的数肯定都需要看一遍。

**排序加双指针的思路**：
```
public int[] twoSum(int[] n, int t) {
    if (n == null || n.length == 0) {
        return new int[0];
    }
    
    Arrays.sort(n);
    
    int[] result = new int[2];
    
    int l = 0, r = n.length - 1;
    while (l < r) {
        if (n[l] + n[r] == t) {
            result[0] = n[l];
            result[1] = n[r];
            return result;
        } else if (n[l] + n[r] < t) {
            l++;
        } else {
            r--;
        }
    }
    
    return new int[0];
}
```
这种思路的前提是题目没有要求我们必须输出元素在数组中的位置，因为排序会改变元素在数组中的位置，这里，我们输出元素本身即可。这里的思路就是一头一尾两个指针，每次判断的时候，我们将左右两个指针指向的元素加起来的和与我们要找的 target 对比，如果比 target 小，也就是说明如果在左指针不变的情况下，左指针加上左右指针中间的任意一个元素都会比 target 小，这也说明左指针不可能是我们要找的答案，因此向右移动左指针。如果是加起来和我们要找的 target 对比，比 target 大，分析类似，这时需要将右指针向左移动。我们来看看这里的时间复杂度，因为做了排序这么一个操作，其时间复杂度就会是 O(nlgn)，对于空间复杂度来说，这里并没有使用额外的空间，因此空间复杂度是常数级的 O(1)。

两种方法都讲完了，如果你有一些编程经验的话，相信这些东西不难理解。你有没有想过这两种方法分别比较适合什么样的情况呢，基于 TwoSum 问题，思考下面的变化：
* 如果题目要求输出所有可能的答案，该怎么处理？
* 如果题目要求输出所有可能的答案，并且数组中有重复元素该怎么处理？
* 如果题目要求找到比 target 小/大 的配对该怎么处理？
* 如果要找出两数之差等于 target 的配对，该如何进行？
* TwoSum 的解题思路是否可以拓展到 TreeSum 或者更多的配对？

上面的问题可能有些你曾想过，有些没有，那么就让我们带着上述的问题来看看具体的例题，熟练地将上述两种方法应用到实际的题目中去。

<br>

## 变形题目分析

题目还是 TwoSum，但是这时需要你返回所有可能的情况（不重复），并且数组中允许重复元素的出现。

**思路分析：**

首先我们需要思考的是，使用之前提到的两种方法中的哪一种会比较好。是不是两种方法都可以，你分析一下会发现其实两种方法都是可行的，和之前的 TwoSum 不一样的是，**这时当找到答案后，需要继续寻找，而不是直接返回**，但是由于数组中存在重复元素，因此两种方法里面都需要考虑去重的机制，对于 Hash 表的方法来说，其实你并不清楚之前是否添加过相同的答案，因此我们考虑使用一个集合去存储答案，保证答案的不重复性；对于排序的方法来说，去重的方式有所不同，排完序后，相同的元素会挨在一起，对于之前考虑过的元素，我们只需要略过就行，指针的移动很好地保证了这一点。这里就展示排序实现的方法：

**代码实现：**
```java
public List<int[]> twoSum6(int[] nums, int target) {
    if (nums == null || nums.length < 2)
        return 0;

    Arrays.sort(nums);
    
    List<int[]> results = new ArrayList<>();
    
    int left = 0, right = nums.length - 1;
    
    while (left < right) {
        int v = nums[left] + nums[right];
        if (v == target) {
            int[] result = {nums[left], nums[right]};
            results.add(result);
            left++; right--;
            
            while (left < right && nums[right] == nums[right + 1]) {
                right--;
            }
            
            while (left < right && nums[left] == nums[left - 1]){
                left++;
            }
        } else if (v > target) {
            right--;
        } else {
            left++;
        }
    }
    
    return results;
}
```

<br>

[LeetCode 1099. Two Sum Less Than K](https://leetcode.com/problems/two-sum-less-than-k/)

**思路分析：**

传统的 TwoSum 都是要你找到等于 target 的配对，那么如果说要找到 大于/小于 target 的配对呢？这个时候 Hash 表的方法就很难 work 了，因为 Hash 表比较适合处理等于的情况。那么就需要考虑如何使用排序加双指针的方法来解决这个问题，**这里，题目是要求小于 target 的数量**，我们还是按照之前的分析思路来分析，如果说当前左右指针指向的元素的和大于或者等于 target，那么势必我们需要向左移动右指针，让两个元素的和尽可能地小，当前头尾指针指向的元素和小于 target 的时候，这时我们需要记录答案，虽然这道题目里面没提，如果说要记录配对数量的话，这时并不是记录一个答案，如果说当前左指针固定，除了当前的右指针指向的元素，在左指针和右指针之间的数都是满足要求的，我们只需要加上这个区间的数量即可，当然如果数组中存在重复元素，那么我们就需要按照之前的套路遍历去重了，当然对于这道题来说，我们选择满足条件的最大值即可。

**代码实现：**
```java
public int twoSumLessThanK(int[] A, int K) {
    if (A == null || A.length == 0) {
        return -1;
    }
    
    Arrays.sort(A);
    
    int l = 0, r = A.length - 1;
    int result = Integer.MIN_VALUE;
    
    while (l < r) {
        if (A[l] + A[r] >= K) {
            r--;
        } else {
            result = Math.max(result, A[l] + A[r]);
            l++;
        }
    }
    
    return result == Integer.MIN_VALUE ? -1 : result;
}
```

<br>

[LeetCode 170. Two Sum III - Data structure design](https://leetcode.com/problems/two-sum-iii-data-structure-design/)

**思路分析：**

让你实现一个数据结构，这个结构支持 “**添加元素**” 和 “**TwoSum**” 两个功能。这时你需要综合两种方法的优劣性来选择，首先是 Hash 表的方法，如果使用这个方法，我们不需要考虑太多的东西，元素来了直接扔进数组就行，也就是说 **添加元素** 操作只需要 O(1) 的时间复杂度就可以完成，但是 **TwoSum** 的完成需要额外 O(n) 的空间；再来看看排序的方法，因为这里插入元素我们需要保证元素有序，因此 **添加元素** 需要 O(n) 的时间，但是这里 **TwoSum** 操作并不需要额外空间，综合来考虑，因为 **添加元素** 和 **TwoSum** 操作都会比较频繁，因此 Hash 表的方法在时间上面更优。

**代码实现：**
``` java
private Map<Integer, Integer> elements;
private int MAX_VALUE = Integer.MIN_VALUE;
private int MIN_VALUE = Integer.MAX_VALUE;
/** Initialize your data structure here. */
public TwoSum() {
    elements = new HashMap<>();
}

/** Add the number to an internal data structure.. */
public void add(int number) {
    elements.put(number, elements.getOrDefault(number, 0) + 1);
    MAX_VALUE = Math.max(MAX_VALUE, number);
    MIN_VALUE = Math.min(MIN_VALUE, number);
}

/** Find if there exists any pair of numbers which sum is equal to the value. */
public boolean find(int value) {
    if (value < 2 * MIN_VALUE || value > 2 * MAX_VALUE) {
        return false;
    }
    
    for (int i : elements.keySet()) {
        if (i * 2 == value && elements.get(i) >= 2) {
            return true;
        } else if (i * 2 != value && elements.containsKey(value - i)) {
            return true;
        }
    }
    
    return false;
}
```

<br>

[LeetCode 15. 3Sum](https://leetcode.com/problems/3sum/)

**思路分析：**

三数之和，Hash 表以及排序的思路都是可行的，但是这里涉及到去重，这里比较推荐的做法是排序加双指针。思路其实比较直观，确定一个元素，然后去找另外两个元素，这么一来就把 3Sum 转变成了 2Sum 的问题，这里我两个方法都实现了一下，你可以参考

**代码实现：**
```
// sort + two pointers
public List<List<Integer>> threeSum(int[] nums) {
    if (nums == null || nums.length < 3) {
        return new ArrayList<>();
    }
    
    Arrays.sort(nums);
    
    List<List<Integer>> results = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; ++i) {
        if ((i != 0) && (nums[i] == nums[i - 1])) {
            continue;
        }
        
        List<Integer> result = new ArrayList<>();
        
        twoSum(results, nums, i + 1, -nums[i]);
        
        result = null;
    }
    
    return results;
}

private void twoSum(List<List<Integer>> results,
                    int[] nums,
                    int startIndex,
                    int target) {        
    int left = startIndex, right = nums.length - 1;
    while (left < right) {
        if (nums[left] + nums[right] > target) {
            right--;
        } else if (nums[left] + nums[right] < target) {
            left++;
        } else {
            List<Integer> result = new ArrayList<>();
            
            result.add(-target);
            result.add(nums[left]);
            result.add(nums[right]);
            
            results.add(result);
            
            left++; right--;
            
            while ((left < right) && (nums[left - 1] == nums[left])) {
                left++;
            }
            
            while ((right > left) && (nums[right + 1] == nums[right])) {
                right--;
            }
        }
    }
}
```

```java
// HashSet
public List<List<Integer>> threeSum(int[] nums) {
    if (nums == null || nums.length < 3) {
        return new ArrayList<>();
    }
    
    Arrays.sort(nums);
    
    Set<List<Integer>> resultsSet = new HashSet<>();
    List<List<Integer>> results = new ArrayList<>();
    
    for (int i = 0; i < nums.length - 2; ++i) {
        if ((i != 0) && (nums[i - 1] == nums[i])) {
            continue;
        }
        
        Set<Integer> existedValue = new HashSet<>();
        
        for (int j = i + 1; j < nums.length; ++j) {
            if (!existedValue.contains(nums[j])) {
                existedValue.add(-nums[j] - nums[i]);
            } else {
                List<Integer> result = new ArrayList<>();
                
                Collections.addAll(result, nums[i], -nums[j] - nums[i], nums[j]);
                
                resultsSet.add(result);
            }
        }
        
        existedValue = null;
    }
    
    results.addAll(resultsSet);
    
    return results;
}
```

<br>

给定一个整数数组，在该数组中，寻找三个数，分别代表三角形三条边的长度，问，可以寻找到多少组这样的三个数来组成三角形？

样例 1:

输入: [3, 4, 6, 7]
输出: 3
解释:
可以组成的是 (3, 4, 6), 
           (3, 6, 7),
           (4, 6, 7)
           
样例 2:

输入: [4, 4, 4, 4]
输出: 4
解释:
任何三个数都可以构成三角形
所以答案为 C(3, 4) = 4

**思路分析：**

需要选出三条边，使得这三条边能够构成三角形，咋眼看上去这道题貌似和 TwoSum 没啥关系，我们回顾一下中学时期学的东西，三边构成三角形的条件是 **任意两边之和大于第三边**，那是不是说我们需要把三条边都组合配对考虑一下？其实不用，我们可以得出下面的结论
```
a < b < c && a + b > c => 三角形
```
如果已知三条边的大小顺序，那么其实我们只需要比较一次即可。

你再看看这是不是我们熟悉的 TwoSum 变种问题 - **如果题目要求找到比 target 小/大 的配对该怎么处理？**，这个时候我们从右往左选定 c，然后使用 TwoSum 来找出 a, b 即可，由于题目只要求输出个数，那么就按照之前讲的思路，直接相加即可


**代码实现：**
``` java
public int triangleCount(int[] S) {
    if (S == null || S.length == 0) {
        return 0;
    }
    
    Arrays.sort(S);
    
    int result = 0;
    
    for (int i = S.length - 1; i >= 2; --i) {
        int l = 0, r = i - 1;
        while (l < r) {
            if (S[i] < S[l] + S[r]) {  // S[i] < S[l] + S[r] && S[i] > S[r] > S[l]
                result += r - l;  //  直接加上可能的个数
                r--;
            } else {
                l++;
            }
        }
    }
    
    return result;
}
```

<br>

[LeetCode 18. 4Sum](https://leetcode.com/problems/4sum/)

**思路分析：**

4Sum 的思路和 3Sum 的思路是一样的，只不过这时我们需要先将其转换成 3Sum 来处理，其实就是比 3Sum 多了一次选择

**代码实现：**
```java
public List<List<Integer>> fourSum(int[] nums, int target) {
    if (nums.length < 4) {
        return new ArrayList<>();
    }
    
    Arrays.sort(nums);
    
    List<List<Integer>> results = new ArrayList<>();
    
    for (int i = 0; i < nums.length - 3; ++i) {
        if (i != 0 && nums[i] == nums[i - 1]) {
            continue;
        }
        
        List<List<Integer>> subResults = threeSum(nums, i + 1, target - nums[i]);
        
        for (List<Integer> res : subResults) {
            res.add(nums[i]);
            results.add(res);
        }
    }
    
    return results;
}

private List<List<Integer>> threeSum(int[] nums, int start, int target) {
    List<List<Integer>> results = new ArrayList<>();
    
    for (int i = start; i < nums.length - 2; ++i) {
        if (i != start && nums[i] == nums[i - 1]) {
            continue;
        }
        
        List<List<Integer>> res = twoSum(nums, i + 1, target - nums[i]);
        
        for (List<Integer> r : res) {
            r.add(nums[i]);
            results.add(r);
        }
    }
    
    return results;
}

private List<List<Integer>> twoSum(int[] nums, int start, int target) {
    List<List<Integer>> results = new ArrayList<>();
    
    int l = start, r = nums.length - 1;
    while (l < r) {
        if (nums[l] + nums[r] < target) {
            l++;
        } else if (nums[l] + nums[r] > target) {
            r--;
        } else {
            List<Integer> result = new ArrayList<>();
            result.add(nums[l]); result.add(nums[r]);
            results.add(result);
            
            r--; l++;
            while (l < r && nums[r] == nums[r - 1]) {
                r--;
            }
            
            while (l < r && nums[l] == nums[l + 1]) {
                l++;
            }
        }
    }
    
    return results;
}
```

<br>

给定一个 target，求出两数之差等于 target 的情况

**思路分析：**

如果使用 Hash 表的做法，对于传统的 TwoSum，我们找到的答案是满足等式:
```
num1 + num2 = target
```
因此这个时候判断的时候，我们只需要判断 `target - nums` 在不在 Hash 表中即可，对于两数之差的话，我们找到的答案是满足等式：
```
num1 - num2 = target or num2 - num1 = target
```
在这种情况下，我们需要判断 `target + num` 以及 `num - target` 即可，也就是相比之前，判断条件不同且多了一个。

如果是使用排序加双指针的方法呢？这其实是个打破思路局限的很好例子，这个时候我们需要用到同向双指针了，两个指针均从左向右移动，一前一后，用右边的减去左边的差值来和 target 做比较，如果小了，移动右指针，大了，移动左指针，等于，输出答案。


**代码实现：**
```
public int[] twoDiff(int[] n, int t) {
    if (n == null || n.length == 0) {
        return new int[0];
    }
    
    Arrays.sort(n);
    
    int[] result = new int[2];
    
    int l = 0, r = 1;
    while (r < n.length) {
        if (l == r) {
            r++;
        }
        
        if (n[r] - n[l] == t) {
            result[0] = n[l];
            result[1] = n[r];
            return result;
        } else if (n[r] - n[l] < t) {
            r++;
        } else {
            l++;
        }
    }
    
    return new int[0];
}
```


<br>

## 总结
TwoSum 相关的问题就分析到这里，非常有趣的一点是，**TwoSum 不仅可以扩展成 3Sum，4Sum，它也可以扩展为 KSum，** 但是这里就需要用到动态规划的知识了，暂不在这里讨论。希望这次分享的内容对你有所帮助。