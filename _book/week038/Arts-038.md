## Algorithm

[LC 410. Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/)

**题目解析**：

题目意思很简单，让你把输入数组分成 m 个子数组，并且求出 m 个子数组中的最大子数组和，问如何分才能使得这个值最小，输出这个值。

一般像这种求最值的题目都可以往动规的方向去思考，一开始是想着往区间类动态规划的方向去拆解问题，但最后发现 m 这个信息没有办法在状态中表示，于是试着重新定义状态，`dp[i][j]` 表示把 `array[i...n]` 拆成 j 个子数组，子数组当中的最小值，于是可以画出搜索树：
```
                    array[0...n] 分成 m 份
                    /           ...       \
            array[0] +                  array[0...n-m] +              
    array[1...n] 分成 m - 1 份 ...   array[n-m+1, n] 分成 m - 1 份
            ...                             ...
```
搜索树一出来，基本上就可以用记忆化搜索来解，但是这里还有一个小技巧，就是我们在求子数组和的时候我们可以借助前缀和数组来求解，这样会比较节省时间。这样下来时间复杂度是 O(m*n^2)，也很好解释，我们有 m * n 个状态，每个状态我们要试着去分 n 次。

这道题目还有一个二分法的解法，不看答案或者提示的话很难想到，因为这种二分是按值来二分，我们要求解的答案肯定是在 `min(array[0]...array[n])` 和 `sum(array)` 之间，我们不断尝试这个区间里面的值，看能否满足要求就好。二分的复杂度比较直观，`O(n * log(sum(array)))`，从 leetcode 的运行时间结果对比来看，二分要优于动态规划的解法，再一次体会了 log 级别的时间复杂度和线性时间复杂度相比，要快很多。

<br>

**参考代码（动态规划）**：
```java
public int splitArray(int[] nums, int m) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    
    int n = nums.length;
    
    int[] sumArray = new int[n + 1];
    
    // 求解前缀和数组
    sumArray[0] = 0;
    for (int i = 1; i <= n; ++i) {
        sumArray[i] = sumArray[i - 1] + nums[i - 1];
    }
    
    return divide(sumArray, m, new int[n][m + 1], 0);
}

private int divide(int[] sumArray,
                   int subArrayCount,
                   int[][] memo,
                   int startIndex) {        
    if (subArrayCount == 1) {
        return sumArray[sumArray.length - 1] - sumArray[startIndex];
    }
    
    if (memo[startIndex][subArrayCount] != 0) {
        return memo[startIndex][subArrayCount];
    }
    
    int result = Integer.MAX_VALUE;
    for (int i = startIndex + 1; i < sumArray.length - 1; ++i) {
        // array[startIndex]...array[i - 1] 为一个子数组 
        int left = sumArray[i] - sumArray[startIndex];
        
        // array[i]...array[n] 继续拆分
        int right = divide(sumArray, subArrayCount - 1, memo, i);
        
        result = Math.min(result, Math.max(left, right));
    }
    
    memo[startIndex][subArrayCount] = result;
    
    return result;
}
```

**参考代码（二分）**：
```java
public int splitArray(int[] nums, int m) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    
    int n = nums.length;
    
    int totalSum = 0;
    for (int num : nums) {
        totalSum += num;
    }
    
    int left = 0, right = totalSum;
    int result = Integer.MAX_VALUE;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int count = 1;
        int curSum = 0;
        int maxSum = 0;
        for (int i = 0; i < n; ++i) {
            if (curSum + nums[i] > mid) {
                count++;
                curSum = nums[i];
            } else {
                curSum += nums[i];
            }
            maxSum = Math.max(maxSum, curSum);
        }
        
        // 如果按当前的值来分，恰好可以分成 m 份，记录答案
        // 这里我们不能直接输出，要继续缩小答案在进行二分
        if (count == m) {
            result = Math.min(maxSum, result);
            right = mid - 1;
        } 
        // 当前子数组的数量小于 m，说明尝试的 “答案” 大了
        else if (count < m) {
            right = mid - 1;
        } 
        // 当前子数组的数量大于 m，说明尝试的 “答案” 小了
        else {
            left = mid + 1;
        }
    }
    
    return result;
}
```
<br>

## Review

[The Super Simple Strategy For Greater Focus](https://medium.com/personal-growth/the-super-simple-strategy-for-greater-focus-6d467b9d66c8)

这篇文章讲述的是如何变得更加 “专注”，当今社会，丰富多彩，我们的时间可以用在各种各样的事情上面，我们很难长时间集中注意力在一件事情上面，很多时候，你的内心会不由自主地去关注身边很多琐碎的事情，比如有没有邮件通知，微信谁又发朋友圈了，今天晚上吃什么，之前的事情结果如何。。。

是我们真的不知道如何去 “专注” 吗？文章中有一句话说的我非常认同，原话是：

> Most people don’t have trouble with focusing. They have trouble with deciding.

难的其实并不是专注本身，而是如何选择，选择手头哪一件事情去专注，**往往我们想把手头所有的事情都做好，但是结果却是没有做好一件事。** 文章中提到，我们很难，甚至是不可能同时集中注意力在好几件事上，当然认知类的事情除外，比如说看电影，吃零食，做家务，一般像是编程、读书、写作这类需要深度思考的事情，同时去做多个只会适得其反，很没有效率。

说了这么多，那么到底该怎么去做到 “专注” 呢？文章中也给了一句话：

> We go where we look.

翻译过来就是 “**我们朝着我们眼中的方向前进**”，既然我们无法做到同时集中注意力做好多件事，那么我们就集中注意力做好一件事吧，这件事的选择也非常简单，就选择那些你觉得值得你做，能给你带来深远意义的事情，但是记住，一次只选一件事，做好这件事，甚至把它深入你的习惯中，然后再考虑其他的事。

<br>

## Tip

这周记录一下用 JavaScript 写 for 循环时发现的问题

最普通的 for 循环是：
```javascript
const array = ['1', '2', '3'];
for (let i = 0; i < array.length; i++) {
  // do something
}
```
但是这种写法有时会看着比较累赘，特别是我们需要对一个结构里面的元素进行遍历的时候，这个时候我们可以使用下面的写法
```javascript
for (const val in array) {
  // do something
}
```

但是上面的写法在某些情况下还是不理想，因此这个时候我们可以考虑使用 forEach
```javascript
const array = ['1', '2', '3'];
array.forEach(val => {
    // do something
})
```
这种写法和之前不同之处在于它和 callback 函数做了结合，和之前相比，这种写法的优点在于结构清晰明了，还可以借助 JavaScript 异步的优势，但是缺点是无法做到用简单的 break 终止循环。

这里有一个非常容易误导人的地方，总结一下：
```javascript
const array = ['1', '2', '3'];
array.forEach(val => {
    console.log(val);
})
console.log(100);

// 打印结果
// 1
// 2
// 3
// 100
```
100 是最后打印的，证明程序是顺序执行的，但是如果在循环里面添加一些 I/O 处理，比如下面的代码：
```javascript
let array = ['1', '2', '3'];
array.forEach(async (val) => {
    console.log(val);
    const a = await(1);
    console.log(a);
})
console.log(100);
// 打印结果
// 1
// 2
// 3
// 100
// 1
// 1
// 1
```
在这里我们用 async/await 模拟了一下 I/O，你可以看到最后的结果和之前不一样之处。总结下来就是如果 forEach 中有 I/O 请求的话，JavaScript 会利用非阻塞 I/O 的性质，callback 函数的非阻塞部分会先执行，阻塞部分会在整个主函数结束后进行，其他的简单代码逻辑会和顺序执行的时候一样，和 forEach 类似的还有内置函数 map

<br>

## Share

[别让自己“墙”了自己](https://coolshell.cn/articles/20276.html)

这周没时间写文章，分享皓叔的一篇文章，主要讲的还是选择的问题，很多时候，选择比努力更重要，没有选择就努力去创造机会让自己去拥有更多的选择，不要给自己的人生设限，这样人生才会有更多的可能性，也会见到更大的世界，认识更多地志同道合的人。