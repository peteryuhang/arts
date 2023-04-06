> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 78. Subsets](https://leetcode.com/problems/subsets/)

**题目解析**：

求出一个集合的所有子集，这属于搜索类题目的入门经典题，**一般像这种需要求出所有的方案的题目，都可以考虑使用深度优先搜索**，这道题目有意思的地方在于它不止一种实现方式，在讲具体的实现方式之前，我们先来看看给定一个集合，他有多少个子集，因为是集合，所以里面的元素不重复，对于集合中的一个元素，其可以在子集中，也可以不在，因此，如果给定一个大小是 n 的集合，它的子集则会是 2^n，因为题目要求把所有子集都列出来，所以时间复杂度至少也是 O(2^n) 级别的。

首先我们来看看常规的解法，利用递归进行深度优先搜索，我们可以把搜索树画出：
```
输入 [1,2,3]：

         []
      /   |   \
    [1]  [2]  [3]
   /   \  ...  ...
 [1,2] [1,3] ... ...
 ...
```
这里简单解释一下这个搜索树，**树的层级代表的是递归的深度**，你可以看到的是，这颗树上的每个节点我们都需要记录，没有重复的节点，另外一点需要考虑的是，每记录完一个节点以后，我们都需要考虑是否进入下一层递归，每往下一层，需要记录的子集就会多一个元素，因为子集里面没有重复，因此当前加入的元素不在下一层的考虑范围，有了这些认识后，可以很容易地写出相应的代码。因为这里只是简单地遍历树，树上有多少节点，时间复杂度就是多少，即 O(2^n)。

这里讲讲两种非递归的写法，第一种是使用 for 循环，我们还是根据之前的例子一起看看：
```
[1,2,3]
空集：[]
子问题 [1] 的情况：    [], [1]
子问题 [1,2] 的情况：  [], [1], [2], [1,2]
子问题 [1,2,3] 的情况：[], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]
```
这样可以看出，当前子问题仅仅是基于上一个子问题的结果进行 append 操作，也就是把上一个子问题的结果都加上当前考虑的元素，然后进行结果的累加即可。这种方法，每次的操作都在记录答案，并不存在 if-else 判断的情况，因此时间复杂度还是 O(2^n)。

另外一种实现方式是利用位运算，因为每一个元素只有两种情况，在集合中以及不在集合中，因此我们可以看相应的 bit 来决定是否加入元素，还是结合例子来看看：
```
[1,2,3]
0000 0000 (0): []
0000 0001 (1): [1]
0000 0010 (2): [2]
0000 0011 (3): [1,2]
0000 0100 (4): [3]
0000 0101 (5): [1,3]
0000 0110 (6): [1,2]
0000 0111 (7): [1,2,3]
```

首先这个方法基于的一个前提就是 n 不会太大，需要记录的答案并不会多于 Java 中 int 的最大值，我们首先可以根据 n 求出答案的个数，比如例子中 n = 3，因此需要记录的答案就是 2^n = 8，然后从 0 ~ 2^n-1 枚举即可，这种方法的时间复杂度相比之前并没有提升，因为我们需要考虑 2^n - 1 个答案，并且每个答案需要考虑每个元素，这样下来时间就是 O(n*2^n)。



<br>

**参考代码（递归）**：
```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    if (nums == null || nums.length == 0) {
        return result;
    }
    
    helper(nums, result, new ArrayList<>(), 0);
    
    return result;
}

private void helper(int[] nums, List<List<Integer>> result, List<Integer> path, int index) {
    result.add(new ArrayList<>(path));
    
    for (int i = index; i < nums.length; ++i) {
        path.add(nums[i]);
        helper(nums, result, path, i + 1);
        path.remove(path.size() - 1);
    }
}
```

**参考代码（非递归）**：
```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    
    if (nums == null || nums.length == 0) {
        return result;
    }
    
    result.add(new ArrayList<Integer>());
    
    for (int i = 0; i < nums.length; ++i) {
        int size = result.size();
        for (int j = 0; j < size; ++j) {
            List<Integer> newSet = new ArrayList<>(result.get(j));
            newSet.add(nums[i]);
            result.add(newSet);
        }
    }
    
    return result;
}
```

**参考代码（位运算）**：
```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    
    int n = nums.length;
    int totalPath = 1 << n;
    
    for (int i = 0; i < totalPath; ++i) {
        result.add(new ArrayList<>());
    }
    
    for (int i = 0; i < totalPath; ++i) {
        for (int j = 0; j < nums.length; ++j) {
            if ((i >> j & 1) == 1) {
                result.get(i).add(nums[j]);
            }
        }
    }
    
    return result;
}
```
<br>


## Review
[Which Git branching model should I select for my project?](https://medium.com/aventude/which-git-branching-model-should-i-select-73aafc503b5f)

这篇文章讲述的是 Git model 的选择，文章中给了几个 Git model，第一个 model 中包含 QA、Dev、Release 等等 branch，通常我们 push 一次 commit 到 master 需要层层把关，这在作者看来有点没有效率，而且 master 上面的 commit 相对来说较少，没有连续性，不利于 code review，于是作者提出了 Trunk Based Development (TBD)，在这种模式下，不同 feature 的 branch 可以直接 merge 到 master，省去了中间的一些过渡分支，这样极大的加快了开发的效率。

但是设计这种东西，并不是十全十美的，有好的地方也会存在缺陷，因为少了一些过渡分支，比如 QA、Dev 等，push 到 master 上面的代码的质量没法得到很好的保障，因此在这种模式下做开发，需要开发者有良好的素质，如果是一些没有经验的开发者使用这种开发模式，则需要在 merge 到 master 之前加大审核的力度。

作者最后提到，每个 Git Flow 都有它存在的价值，通常我们设计一个 Flow 需要根据 **项目复杂度**、**团队的人员能力经验**、以及 **业务的目标** 来决定，没有最好的，只有相对来说比较合适的，当然这些东西需要在项目之初就定好，这样大家才有适应的时间。

<br>

## Tip

这次来说一个 Git 的指令，`cherry-pick`

从功能上来看，`cherry-pick` 也是 merge 相关的指令，但是这个指令只会针对一个或多个 commit 进行 merge，而不是基于 branch merge，通常在我们进行一些热修复(Hot-fix) 的时候可以考虑使用，具体使用如下：
```git
$ git checkout master
$ git cherry-pick 99daed2
```
上面的操作是把 `99daed2` 这次 commit merge 到当前 master 上，如果有 conflict，那么就按常规的方法解决即可。

<br>

## Share
这次来聊一聊一类比较特殊的动态规划 - 背包类动态规划

[动态规划之背包类动规](./动态规划之背包类动规.md)