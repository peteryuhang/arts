> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 1187. Make Array Strictly Increasing](https://leetcode.com/problems/make-array-strictly-increasing/)

**题目解析**：

题意是给定两个数组，arr1 和 arr2，你可以将 arr2 中的值赋值给 arr1，比如 `arr1[i] = arr2[j]`，每进行这样一次赋值就算做一次操作，问最少进行多少次操作可以让 arr1 严格递增。

看到这道题目的时候我不知道你是否想到一道经典的动态规划问题 —— **[LIS](https://leetcode.com/problems/longest-increasing-subsequence/)**，在 LIS 中我们只需要求出严格递增的最长子序列长度，而这道题目则貌似需要你做一些额外的思考，但是不管怎样，我们可以尝试着用动态规划去思考该问题。在 LIS 问题中，动态规划的状态我们需要记录的仅仅是序列的终止位置，也就是序列的最右边元素的下标，因此，在 LIS 问题中，我们定义 `dp[i]` 表示的是 `arr[0...i]` 区间的答案。回到这道题目，你发现仅仅有个位置是不足以表达当前状态的，因为每遍历到 arr1 的一个位置的时候，你很难确定这个地方到底要不要进行操作，举个题目给的例子：
```
arr1 = [1,5,3,6,7], arr2 = [1,3,2,4]

和 LIS 的思考方式类似，我们从左到右遍历 arr1
当遍历的第二个元素，也就是 5 的时候，你不确定到底要不要进行赋值操作
因为此时，即使你不进行任何操作，区间 arr1[0,1] 也是符合条件的
```
所以，我们还要在动态规划状态中多加一个表示更换次数的状态，因此，定义 `dp[i][j]` 表示在区间 `arr[0...i]` 上做 `j` 次替换后，序列右边界（`arr1[i]`） 的最小值，你可能会问为什么是最小值，这个很好解释，**只有此时位置的上的值越小，后面才越有可能减少操作的次数。**

除了这种方法，还有借助二分法来确定右边界的，但是这些方法本质都是借助动态规划，这类方法的时间复杂度是 O(n^2 * logm)，其中 n 是 arr1 的长度，m 是 arr2 的长度。可能这道题目还有更优的方法，也欢迎补充。

<br>

**参考代码**：
```java
public int makeArrayIncreasing(int[] arr1, int[] arr2) {
    int n = arr1.length;
    
    // dp[i][j] 表示的是 arr1[0...i-1] 进行 j 次赋值操作后，arr1[i-1] 的最小值
    // 注意这里 dp[i][j] 并不是我们要求的答案，j 才是
    int[][] dp = new int[n + 1][n + 1];

    // 这里我们借助 TreeSet 来保存 arr2 中的元素
    // 这样我们可以快速获知 arr1 中的值是否能够被施行赋值操作
    TreeSet<Integer> ts = new TreeSet<>();

    for (int i : arr2) {
        ts.add(i);
    }

    for (int i = 1; i <= n; ++i) {
        Arrays.fill(dp[i], Integer.MAX_VALUE);
    }

    dp[0][0] = Integer.MIN_VALUE;

    for (int i = 1; i <= n; ++i) {
        for (int j = 0; j <= i; ++j) {
            // 如果当前 arr1[i - 1] 可以直接加到当前序列后
            // 直接设定当前 dp[i][j] 以 arr1[i - 1] 结尾
            if (arr1[i - 1] > dp[i - 1][j]) {
                dp[i][j] = arr1[i - 1];
            }

            // 如果可以进行赋值操作，进行赋值操作
            // 但是 dp[i][j] 记录最小的 arr1[i - 1]
            if (j > 0 && ts.higher(dp[i - 1][j - 1]) != null) {
                dp[i][j] = Math.min(dp[i][j], ts.higher(dp[i - 1][j - 1]));
            }

            // 当遍历完整个 arr1，如果序列有效，即时返回答案保证 j 最小
            if (i == n && dp[i][j] != Integer.MAX_VALUE) {
                return j;
            }
        }
    }
    
    // 无法使得 arr1 严格递增，返回 -1
    return -1;
}
```

<br>

## Review

[Four “clean code” Tips](https://engineering.videoblocks.com/these-four-clean-code-tips-will-dramatically-improve-your-engineering-teams-productivity-b5bd121dd150)

作者先是从自己的经验讲述了 **代码质量** 的重要性，并推荐了一波马丁的 《clean code》，这本书被广泛赞誉，是程序员必读书籍，作者给出了改善代码质量的 4 条重要建议，如下：
* 没有测试，就是失败的代码

  我觉得这一条其实就是一个意识问题，有些人习惯了不写测试，习惯了系统出现了问题，熬夜抢修，习惯一旦形成就很难纠正过来，但如果你心里有这个意识，至于怎么写测试，其实相对来说都不重要，总的来说，是测试越多越好。

* 学会命名

  不要小看命名，命名是计算机软件行业的千古难题，总体说来名字要起的有意义，能够让别人一看命名就知道这个名字下所代表的函数或者变量的用途，另外就是容易被 IDE 的查找功能检索到，个人认为每个人都有自己的命名习惯，不强求人人相同，但是我们必须清楚了解自己的命名习惯，每当你想起一个模块或者一个功能的时候，你可以很清楚地知道这部分的相关函数或者类你是如何命名的就好了。

* 类或者函数需要简洁并且遵从单一职责原则

  在叙述这一点上，作者给了个例子，相比于把所有逻辑都写在一个函数中，把逻辑拆分开来可以大大降低每个函数中的代码长度，这样做有什么好处呢？首先，函数的逻辑复杂度和耦合性降低，这将有助于我们测试，另外，这样更容易遵从单一职责原则，这会大大降低后期维护的难度：
  
  > The Single Responsibility Principle (SRP) states that a class or module should one, and only one, reason to change.   -- Robert Martin

  同时，在如何做到这一点上，作者也给出了自己的见解，首先，当你要实现一个模块或者复杂逻辑的时候，可以将实现的步骤一步步写下来，这里的一步可以看作是一个简单逻辑，拆分的越细，就越好实现，每一步可以用一个函数进行封装，这样做下来前期需要比较大投入，但是往长期看，这是值得的。

* 函数必须没有副作用

  一个函数必须名副其实，比如说你创建了一个 getUserNameAndPassword 的函数，这里面就不应该有让用户登陆的作用，一个有副作用的函数有很多危害，比如误导代码维护者、难测试、难 debug，以及对代码设计和结构会造成混乱等等。

总之，这些都是非常好的建议，遵守起来也不需要特别高深的思维复杂度，只需要知道，并有这方面的意识就好，意识层面的东西往往就是这样，它在一定程度上决定了你前进的方向，努力的成效，是至关重要的。

<br>

## Tip

积累并学习了一些 Python 强大而灵活的内置函数：

* **sorted()**

  Python 的排序非常灵活，这里只是给出一些例子：
  
  ```
  print(sorted(Arr, key = lambda x : len(x)))
  print(sorted(students, key = lambda student : student[2]))
  print(sorted(students, cmp=lambda x,y : cmp(x[0], y[0])))
  ```
* **zip()**

  接受任意多个序列作为参数，返回一个 tuple 列表：
  
  ```python
  arr1 = [1,2,3,4,5]
  arr2 = ['a','b',c','d',e']
  
  print(list(zip(arr1, arr2))) # [(1,'a'),(2,'b'),(3,'c'),(4,'d'),(5,'e')]
  ```
  
  zip 过后的 list 的长度是参数中序列的最小值

* **enumerate()**

  可以很方便地取出索引

* **all()**

  里面元素都是 **True** 的话返回 True，否则返回 False
  
* **any()**

  里面任何一个元素是 **True** 的话返回 True，如果是全部是 False 或者空，返回 False
  
* **cmp()**

  里面有两个参数 `cmp(x,y)`，`x < y` 返回负数，`x == y` 返回 0，`x > y` 返回正数。

* **dict()**
  
  转换成字典表示，用起来很灵活

  ```
  listOfTuple = [(1,'a'),(2,'b'),(3,'c')]
  dict(listOfTuple) # {1: 'a', 2: 'b', 3: 'c'}
  ```

* **pow(x,y [,z])**

  返回 x 的 y 次幂(如果 z 存在的话则以 z 为模)。
  
  等同于 `res = (x ** y) % z`

Python 的这些函数很方便，而且他们内部有优化，运行起来非常的快速，所以一定要学起来，尝试着去使用

<br>

## Share

这次来聊另外一个有趣的话题 —— **逆商**

[我们该如何看待逆境](./我们该如何看待逆境.md)