**导言：**
> LeetCode 上面的二叉树问题一般可以看成是简单的深度优先搜索问题，一般的实现方式是使用递归，也会有非递归的实现方法，这边文章主要介绍一下解决二叉树问题的几个常规方法和思路，然后会给一个从递归转换到非递归的小技巧

<br>

## 两个通用方法和思路
拿到一道二叉树的问题，多半是需要你遍历这个树，只不过是在遍历的过程中，不同的题目要求你做的计算不一样。这里有两个遍历方法，**自顶向下的递归遍历**，以及**自底向上的分治**。两种方法都用到了递归，在代码实现上面，差别不是特别大，但是思路却截然相反，我们拿树的中序遍历这道题目来作为示例：

**自顶向下的递归遍历**：
```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) {
        return result;
    }
    
    helper(root, result);
    
    return result;
}

private void helper(TreeNode root, List<Integer> result) {
    if (root == null) {
        return;
    }
    
    helper(root.left, result);
    result.add(root.val);
    helper(root.right, result);
}
```
代码非常的简短，上面代码的重心全放在了 helper 函数上，**这个函数没有返回值**，它做的事情也非常的简单，就是去到对应的树节点，然后把节点的值加到 result 中，只是说这里我们要求的是树的 [中序遍历](https://en.wikipedia.org/wiki/Tree_traversal)，因此，我们要先去到当前树节点的左边，把左边所有的节点的值放到 result 中，才会继续放当前树节点，放完当前树节点的值后，我们会去到右边进行同样的操作。对于这种实现方法其实有点类似于循环遍历，只不过循环遍历只作用于数组还有链表这样的线性结构，对于树的话，这里我们采用了递归的方式去遍历，**你可以想象成一个小人在一棵树上面游走，他的目的是按顺序去记录树的节点值。**

**自底向上的分治**：
```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    
    if (root == null) {
        return result;
    }
    
    result.addAll(inorderTraversal(root.left));
    result.add(root.val);
    result.addAll(inorderTraversal(root.right));
    
    return result;
}
```
同一个问题，再来看看我们之前提到的另外一种思路实现。对于这种思路，你可以看我之前写的一篇文章，[从简单二叉树问题重新来看深度优先搜索](https://juejin.im/post/5cdef79ce51d45107d7cb843)， 这里我们也使用了递归，但是**这次的递归函数是有返回值**的，而且你也可以看到的是，我们没有将保存结果的 list 传入函数。正因为是自底向上，所以对于一个树节点来说，它这里会知道它子节点的返回值，也就是子节点的记录结果，在它这里会把左右子节点的结果，和它自己本身的结果汇总，然后将汇总的结果返回给上一层节点。和之前的递归遍历的思路相比的话，代码实现上面的区别可能就是，是将 result list 放在参数中，还是放在返回值中，但是思考方向是截然相反的。这里就不好用之前的 “**小人游走**” 来解释了。更好的解释方式是**公司职位**，这里你可以想象每个树节点就是企业里面的一个人，或者说是一个职位，最上面的 root 节点就是企业的 CEO，往下是副总裁，然后再往下是总监，总监往下是经理，。。。在副总裁这里，他会向他下面的两个总监要结果，在他这里把两个总监返回的结果汇总，然后上报给 CEO，他不会去关注经理的结果，底层员工的结果，在他这里，只关心下一层和上一层。

这两种方法没有好坏之分，有的题目使用**自顶向下递归遍历**的方式会比较直接一点，比如求最大最小值，有些题目则使用**自底向上分治**的方式会比较好一些，比如说 subtree 的问题。对于不同的题目，我们需要选择不同的方法，但是思考方式可以考虑从这两个方向去思考。

一般来说，二叉树问题的时间复杂度都是 O(n) ，这个时间复杂的怎么理解呢？你可以看成是在每个树节点上的操作的时间复杂度是 O(1)，但是要遍历所有的节点，因此 O(1) * n = O(n)。

<br>

## 递归转非递归
对于树的问题，我们也可以使用非递归的方式求解，其实**任何一个递归的解法，都可以转换为非递归**，而且就性能和稳定性来说的话，非递归的方式要比递归来的好。但是问题是，非递归代码不是那么好写，通常来说递归的解法会简洁不少，因此在面试中，面试官通常会让写递归。但是，有些时候，面对一个简单的问题，有些面试官会让你写出非递归的解法，比如上面的中序遍历的例子，相比于递归的简洁一致，非递归的实现可能每个人写出来都会略有不同，我记得我第一次试着把递归写成非递归是非常痛苦的，完全没有头绪。我这里给出了一个递归转非递归的通用方法，**不仅仅适用于树的问题，对于任何的递归问题都适用**，这个方法也是我在 LeetCode 的 discuss 中看到的，还是拿上面中序遍历作为例子，之前我们的代码实现：
```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) {
        return result;
    }
    
    helper(root, result);
    
    return result;
}

private void helper(TreeNode root, List<Integer> result) {
    if (root == null) {
        return;
    }
    
    helper(root.left, result); // line 0
    result.add(root.val);      // line 1
    helper(root.right, result);// line 2
}
```
你可以看到，我在 helper 函数最后 3 行代码标上了序号，后面的非递归实现的程序中会用到，这里我们主要是要实现 helper 函数中的东西。

非递归代码如下：
```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) {
        return result;
    }
    
    Stack<Integer> systemStack = new Stack<>(); // 表示函数进度
    Stack<TreeNode> paramStack = new Stack<>(); // 表示函数输入参数
    
    systemStack.push(0);
    paramStack.push(root);
    
    while (!systemStack.isEmpty()) {
        int curLine = systemStack.pop();
        TreeNode curParam = paramStack.peek();
        
        // 提前将当前进度后移，因为后面可能会有其他函数入栈
        systemStack.push(curLine + 1);
        
        if (curLine == 0) {
            if (curParam.left != null) {
                systemStack.push(0);
                paramStack.push(curParam.left);                    
            }
        } else if (curLine == 1) {
            result.add(curParam.val);
        } else if (curLine == 2) {
            if (curParam.right != null) {
                systemStack.push(0);
                paramStack.push(curParam.right);
            }
        } else {
            systemStack.pop();
            paramStack.pop();
        }
    }
    
    return result;
}
```
一般来说，**用非递归写递归，都需要用到一个数据结构-栈**，这个好解释，**递归的解法是利用了系统中提供的函数栈，非递归我们需要手动创建这么一个数据结构**，但是你可能会问的是，这里为什么要用到两个栈？你可以这样认为，**一个栈用来表示函数的运行进度，里面的元素表示此时该函数运行到了第几行代码，另一个栈用来记录函数的传入参数**，当然你也可以把这两个栈合成一个栈，里面装的是封装好的对象，这里为了解释起来清晰直观，就用两个栈。首先，根据之前的递归解法，我们最开始是把 root 传入 helper 函数，因此这时我们也把 root 加入函数栈，另外一个表示函数进度的栈往里面添加 0，表示当前函数运行到了第 0 行，然后就是 while 循环里面的东西，while 循环一开始我们就获取当前函数的输入参数和进度，然后根据函数的进度去看需要执行哪一段代码，因为有的代码会继续往栈里面添加函数，因此，我们需要提前把函数进度往后移动，你可以对应之前的递归的代码和我标的序号，你可以看到，整个 if-else if...else 部分就表示了之前的递归函数中的代码，只不过这里我们需要根据函数的进度去判断它要执行哪一行。

使用这种方法后，递归转非递归只需要往上套就行，不需要单独分析。

<br>

## 常见的二叉树题目
这里列了几道我觉得还不错的题，当然二叉树的问题远不止这些，要记住的是，**把题目做对永远不是目的，目的是学到一些普适的技巧，甚至是思维方式**，这样的话，能力才会有明显提升，不然，刷题只是在练习熟练度和背诵。

<br>

[LeetCode 145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)

**题目分析**：

树的后序遍历，这道题试着用非递归的方式实现，你可以试着用我上面讲的方式将递归转化为非递归，看看难度还是不是 hard？

**参考代码：**
```java
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    
    if (root == null) {
        return result;
    }
    
    Stack<Integer> systemStack = new Stack<>();
    Stack<TreeNode> paramStack = new Stack<>();
    
    systemStack.push(0);
    paramStack.push(root);
    
    while (!systemStack.isEmpty()) {
        int curSystemLine = systemStack.pop();
        TreeNode curParams = paramStack.peek();
        
        systemStack.push(curSystemLine + 1);
        
        if (curSystemLine == 0) {
            if (curParams.left != null) {
                systemStack.push(0);
                paramStack.push(curParams.left);                    
            }
        } else if (curSystemLine == 1) {
            if (curParams.right != null) {
                systemStack.push(0);
                paramStack.push(curParams.right);
            }
        } else if (curSystemLine == 2) {
            result.add(curParams.val);
        } else {
            paramStack.pop();
            systemStack.pop();
        }
    }
    
    return result;
}
```

<br>

[LeetCode 104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)

**题目分析**：

树的常规题型，求树的最大深度，前面讲的两种思路其实都可以，用递归遍历可能更直观，设一个全局变量，每去到一个节点就拿这个节点的深度和全局变量做比较，把所有的节点走完，最大的深度就选出来了，其实和遍历数组选出最大值是一个道理。

**参考代码：**
```java
private int max = -1;
public int maxDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    
    helper(root, 1);
    
    return max;
}

private void helper(TreeNode root, int currentDepth) {
    if (root == null) {
        return;
    }
    
    max = Math.max(max, currentDepth);
    
    helper(root.left, currentDepth + 1);
    helper(root.right, currentDepth + 1);
}
```


<br>

[LeetCode 508. Most Frequent Subtree Sum](https://leetcode.com/problems/most-frequent-subtree-sum/)

**题目分析**：

subtree 问题，一般来说使用自底向上的分治方法会更加的符合思维逻辑些，对于当前节点来说，如果知道了其左右 subtree 的频率，那么以当前节点为根节点的 subtree 的频率就出来了，我们可以使用一个 map 去记录每个值出现的次数，然后递归完后一起统计。

**参考代码：**
```java
public int[] findFrequentTreeSum(TreeNode root) {
    Map<Integer, Integer> freq = new HashMap<>();
    
    helper(root, freq);
    
    int maxFreq = 0;
    
    List<Integer> res = new ArrayList<>();
    
    for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
        if (maxFreq < entry.getValue()) {
            maxFreq = entry.getValue();
            res.clear();
        }
        
        if (maxFreq <= entry.getValue()) {
            res.add(entry.getKey());
        }
    }
    
    return res.stream().mapToInt(i->i).toArray();
}

private int helper(TreeNode root, Map<Integer, Integer> map) {
    if (root == null) {
        return 0;
    }
    
    int left = helper(root.left, map);
    int right = helper(root.right, map);
    
    int sum = root.val + left + right;
    
    map.put(sum, map.getOrDefault(sum, 0) + 1);
    
    return sum;
}
```

<br>

[LeetCode 549. Binary Tree Longest Consecutive Sequence II](https://leetcode.com/problems/binary-tree-longest-consecutive-sequence-ii/)

**题目分析**：

在一颗树上面求最长路径，这里的路径是指树上任何两个节点之间的通路，方向可以是 parent-child，也可以是 child-parent，还可以是 child-parent-child，但是必须是连续的单调序列，比如 [1,2,3,4,5]，或者是 [4,3,2,1]。不能是[1,3,4,5]，或者是 [1,2,1]。

拿到一道树的题目，具体是使用自底向上还是自顶向下的方式取决于，当前的节点需不需要其子节点或者是子树的结果，这道题来说，当前节点的答案会受到子树的答案的影响，但你可能会说，这样子考虑的情况会不会太多，其实对于一个节点来说，它和子树的路径只会是单调递增或者是单调递减的情况中的一种，另外它最多只有两个子树，如果想要把两个子树连在一起，则一个和它是递增的关系，另一个需要是递减的关系，因此我们只需要考虑当前子树递增的最大长度和递减的最大长度即可。

另外强调一点是，这里我们自底向上需要返回值，返回值中有多个变量，可以考虑返回一个对象，相比返回一个等长的数组来说，class 中可以定义每个返回变量的名称，会更有意义，也更清晰一些。

**参考代码：**
```java
private class ResultType {
    int inc, dec, max;
    ResultType(int inc, int dec, int max) {
        this.inc = inc;
        this.dec = dec;
        this.max = max;
    }
}

public int longestConsecutive(TreeNode root) {
    return helper(root).max;
}

private ResultType helper(TreeNode root) {
    if (root == null) {
        return new ResultType(0, 0, 0);
    }
    
    ResultType left = helper(root.left);
    ResultType right = helper(root.right);
    
    ResultType result = new ResultType(0, 0, 1);
    
    if (root.left != null && root.val + 1 == root.left.val) {
        result.inc = left.inc;
    }
    
    if (root.left != null && root.val - 1 == root.left.val) {
        result.dec = left.dec;
    }
    
    if (root.right != null && root.val + 1 == root.right.val) {
        result.inc = Math.max(result.inc, right.inc);
    }
    
    if (root.right != null && root.val - 1 == root.right.val) {
        result.dec = Math.max(result.dec, right.dec);
    }
    
    result.inc++;
    result.dec++;
    result.max = Math.max(result.inc + result.dec + 1, Math.max(left.max, right.max));
    
    return result;
}
```

<br>

[LeetCode 437. Path Sum III](https://leetcode.com/problems/path-sum-iii/)

**题目分析**：

也是二叉树上面的路径相关的问题，求出路径和等于 target 的路径的个数，这道题其实是一道比较难想的题目，自底向上和自顶向下貌似都不好解，原因是这道题的路径是不固定的，虽然说必须是从上到下，也就是从 parent 节点到 child 节点，但是路径的起点和终点是不固定的，这无形之中给题目增添了难度。一个最直观的解法就是，我们确定好路径的起点后，然后去找符合条件的终点，任何节点都可以成为起点，起点固定后，终点就会是在以起点为根节点的子树中，或者终点就是起点，于是代码可以写成下面这样：
```java
public int pathSum(TreeNode root, int sum) {
    if (root == null) {
        return 0;
    }
    
    return helper(root, sum) + pathSum(root.left, sum) + pathSum(root.right, sum);
}

private int helper(TreeNode root, int sum) {
    if (root == null) {
        return 0;
    }
    
    int count = 0;
    if (sum - root.val == 0) {
        count++;
    }
    
    count += helper(root.left, sum - root.val);
    count += helper(root.right, sum - root.val);
    
    return count;
}
```
上面的代码运行的时间复杂度是 O(n^2) 的，有没有更快的方法呢？这里就不得不说 TwoSum 题目里面的一个思想，在 [TwoSum](https://leetcode.com/problems/two-sum/) 这道题里，我们把 target - currentValue 存进 HashMap，下次遇到的值在 HashMap 里面出现了，就表明这个值其实有配对。把这个思想用到这道题目上面就是，parent-child 的路径上，root 节点到 parent 的路径和是 sumParent，root 节点到 child 的路径和是 sumChild，如果 parent-child 的路径是符合条件的路径，那么，**sumChild - sumParent = target**，也就是 **sumParent = sumChild - target**，在经过 parent 节点的时候，我们在 HashMap 中记录 root 到 parent 节点的路径和，以及该值出现的次数，到了 child 节点，我们就去 HashMap 中找看有没有 **sumChild - target**，如果有的话就表明配对成功，加上这个值的个数就好。当然这些情况只会发生在一条路径中，因此，这里我们还需要用到**回溯的思想**，退出某个节点时，我们需要将该节点的结果删除，保证一条路径上的结果不会影响其他的路径，比如 root 节点的左右子树就不能放在一块讨论，在这道题上，左右子树不是一条路径。

**参考代码：**
```java
public int pathSum(TreeNode root, int sum) {
    Map<Integer, Integer> count = new HashMap<>();
    
    // 覆盖节点上的值等于 sum（target）的情况
    count.put(0, 1);

    return helper(root, 0, sum, count);
}

private int helper(TreeNode root, int cur, int target, Map<Integer, Integer> count) {
    if (root == null) {
        return 0;
    }

    cur += root.val;

    int result = count.getOrDefault(cur - target, 0);

    count.put(cur, count.getOrDefault(cur, 0) + 1);

    result += helper(root.left, cur, target, count);
    result += helper(root.right, cur, target, count);

    count.put(cur, count.getOrDefault(cur, 0) - 1);

    return result;
}
```

<br>

[LeetCode 863. All Nodes Distance K in Binary Tree](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/)

**题目分析**：

这道题想说明的一个点就是，**树其实是图的一种**，树中的节点关系一般都会是 parent 到 child 的关系，在图中属于有向图，这道题的一个直接思路就是把树变成无向图，然后我们可以用一般的搜索（广度优先搜索）来解决该问题。

**参考代码：**
```java
public List<Integer> distanceK(TreeNode root, TreeNode target, int K) {
    Map<TreeNode, List<TreeNode>> graph = new HashMap<>();
    
    buildGraph(graph, root);
    
    Set<TreeNode> visited = new HashSet<>();
    Queue<TreeNode> queue = new LinkedList<>();
    
    queue.offer(target);
    visited.add(target);
    
    int step = 0;
    
    List<Integer> results = new ArrayList<>();
    
    while (!queue.isEmpty()) {
        int size = queue.size();
        
        if (step == K) {
            while (!queue.isEmpty()) {
                results.add(queue.poll().val);
            }
            
            break;
        }
        
        for (int i = 0; i < size; ++i) {
            TreeNode cur = queue.poll();
            
            for (TreeNode n : graph.get(cur)) {
                if (visited.add(n)) {
                    queue.offer(n);
                }
            }
        }
        
        step++;
    }
    
    return results;
}

private void buildGraph(Map<TreeNode, List<TreeNode>> graph, TreeNode root) {
    if (root == null) {
        return;
    }
    
    if (!graph.containsKey(root)) {
        graph.put(root, new ArrayList<TreeNode>());
    }
    
    if (root.left != null) {
        graph.get(root).add(root.left);
        
        if (!graph.containsKey(root.left)) {
            graph.put(root.left, new ArrayList<TreeNode>());
        }
        
        graph.get(root.left).add(root);
    }
    
    if (root.right != null) {
        graph.get(root).add(root.right);
        
        if (!graph.containsKey(root.right)) {
            graph.put(root.right, new ArrayList<TreeNode>());
        }
        
        graph.get(root.right).add(root);
    }
    
    buildGraph(graph, root.left);
    buildGraph(graph, root.right);
}
```

<br>

[LeetCode 366. Find Leaves of Binary Tree](https://leetcode.com/problems/find-leaves-of-binary-tree/)

**题目分析**：

找出所有的叶子节点，找到就删除，然后继续找，直到把树给删光，这里的删不是真的删，你可以思考如何不改变树的结构，而可以找出这些节点的值。很有意思的一点是，这道题其实是在求一个逆深度的问题，如果这道题改成遍历树，同一深度的值放在一起，应该会很简单，但其实我们可以把叶子节点的深度当作是 0，叶子结点往上一层的深度就是 1，同一深度的值放在一起，最后就会是答案。自底向上的方法我们提供了这样的便捷，于是这道题就转换成了简单难度的树的深度的问题。

**参考代码：**
```java
public List<List<Integer>> findLeaves(TreeNode root) {
    List<List<Integer>> results = new ArrayList<>();
    helper(root, results);
    
    return results;
}

private int helper(TreeNode root, List<List<Integer>> results) {
    if (root == null) {
        return -1;
    }
    
    int level = 1 + Math.max(helper(root.left, results), helper(root.right, results));
    
    if (results.size() <= level) {
        results.add(new ArrayList<Integer>());
    }
    
    results.get(level).add(root.val);
    
    return level;
}
```

<br>

[LeetCode 701. Insert into a Binary Search Tree](https://leetcode.com/problems/insert-into-a-binary-search-tree/)

**题目分析**：

二叉搜索树问题，因为二叉搜索树天然的性质，我们往往可以把时间复杂的降为 O(H)，这里的 H 表示的是树的深度，如果该树是平衡的话，那其实就是 O(lgn)。我们可以根据节点的值来判断该节点是在左边还是在右边，有点类似二分法，这样就不需要遍历整个树，插入节点的话，题目已经说明，只要是符合条件的结果都可以，因此我们只需要将节点插入到符合条件的叶子结点后面就好。

**参考代码：**
```java
public TreeNode insertIntoBST(TreeNode root, int val) {
    if (root == null) {
        return root;
    }
    
    TreeNode pointer = root;
    TreeNode prev = null;
    while (pointer != null) {
        prev = pointer;
        
        if (pointer.val > val) {
            pointer = pointer.left;
        } else {
            pointer = pointer.right;
        }
    }
    
    if (prev.val < val) {
        prev.right = new TreeNode(val);
    } else {
        prev.left = new TreeNode(val);
    }
    
    return root;
}
```

<br>

[LeetCode 450. Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/)

**题目分析**：

相比插入，删除会较为麻烦一点，原因是，删除一个节点，该节点后面的节点需要适当的调整，使得删除后的树还是个二叉搜索树。找到要删除的节点还是比较简单的，就是通过比较值来决定去到当前节点的左边还是右边，问题在找到了之后，该节点的子树该怎么处理，这里会有 4 种可能性：
* 左右子树都为空
* 左子树存在，右子树不存在
* 右子树存在，左子树不存在
* 左右子树都存在

第一种情况最简单，直接删除即可，第二种情况，我们只需把左子树替换成当前子树即可，第三种情况，和上面类似，我们只需把右子树替换成当前子树即可，最麻烦的是最后一点，这里有两种思路，从右子树中去选最小值节点，然后把左子树添加到该节点的左边；还有就是去左子树中选最大值节点，然后把右子树添加到该节点的右边，最后返回左子树或者右子树根节点作为当前节点即可。

**参考代码：**
```java
public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) {
        return null;
    }
    
    if (root.val > key) {
        root.left = deleteNode(root.left, key);
    } else if (root.val < key) {
        root.right = deleteNode(root.right, key);
    } else {
        if (root.right == null) {
            return root.left;
        } 
        
        if (root.left == null) {
            return root.right;
        }
        
        TreeNode leftLargest = root.left;
        
        while (leftLargest.right != null) {
            leftLargest = leftLargest.right;
        }
        
        leftLargest.right = root.right;
        
        return root.left;
    }
    
    return root;
}
```

<br>

[LeetCode 106. Construct Binary Tree from Inorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

**题目分析**：

给中序遍历和后序遍历的结果，让重新构建二叉树。这道题中，我们还是需要用到一个分治的思想，就是给定一个数组的范围，然后去建立一个子树，可能子树下面还有子树，那就一直递归下去，因为是二叉树问题，一个节点只会有左右两个子树，所以你会发现有点类似归并排序，快速排序，但是这里我们做的事情更简单，只是新建树节点，不需要比较元素。这里有一个细节需要注意的是，给出一个根节点，在中序遍历的结果中我们就可以知道，这个值往前就会是它的左子树的内容，往后就会是它的右子树的内容，至于怎么知道根节点是那个，其实就是后序遍历结果中的最后一个元素。

**参考代码：**
```java
public TreeNode buildTree(int[] inorder, int[] postorder) {
    if (inorder == null || inorder.length == 0) {
        return null;
    }
    
    Map<Integer, Integer> inorderIndex = new HashMap<>();
    
    int n = inorder.length;
    
    for (int i = 0; i < n; ++i) {
        inorderIndex.put(inorder[i], i);
    }
    
    return helper(inorder, postorder, 0, n - 1, 0, n - 1, inorderIndex);
}

private TreeNode helper(int[] inorder, 
                        int[] postorder, 
                        int si, 
                        int ei, 
                        int sp, 
                        int ep, 
                        Map<Integer, Integer> inorderIndex) {
    if (sp > ep || si > ei) {
        return null;
    }
    
    if (sp == ep) {
        return new TreeNode(postorder[sp]);
    }
    
    int curIndex = inorderIndex.get(postorder[ep]);
    
    TreeNode root = new TreeNode(inorder[curIndex]);
    
    root.left = helper(inorder, postorder, si, curIndex - 1,
                                           sp, sp + curIndex - si - 1, inorderIndex);
    root.right = helper(inorder, postorder, curIndex + 1, ei,
                                           sp + curIndex - si, ep - 1, inorderIndex);
    
    return root;
}
```

<br>

## 总结
树的问题就说到这里，还是那句话，总结普适的技巧和思想是关键，刷题的目的也只是为了巩固这些思想和技巧。