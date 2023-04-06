**导言**：
> 广度优先搜索是搜索算法当中的一种，本文会介绍广度优先搜索解决的一些算法问题，这里会将这些问题分类，并总结这个算法的优缺点等等

<br>

## 初识广度优先搜索
在讲解广度优先搜索之前，我们来看看几个常见的数据结构，**链表、树、图**。先来看看其中比较简单的数据结构 - 链表，它和数组类似，也是一个线性的结构，简单来说就是一条路径，你从头开始遍历，最终会将链表上面的节点都访问到，到达终点。**相比数组来说，链表在内存中的存储可以不是一段连续的区域**。链表节点中会有一个变量用来指明其下一个节点，将链表的表示用代码写出来，就会是下面这样：
```java
class ListNode {
    int val;
    ListNode next;
}
```
其中 val 表示的是这个节点上面值，next 表示的是这个节点的下一个节点。

讲完链表，我们来看看另外一个数据结构 - 二叉树，**它其实是链表的一个延伸**，这里，一个节点的下一个节点不再只有一个节点了，可能会有两个节点，如果把树的结构用代码表示出来，就会是：
```java
class TreeNode {
    int val;
    TreeNode left, right;
}
```
你可能会问，多叉树如何表示呢？这个也简单，一个树节点会有多个子节点：
```java
class TreeNode {
    int val;
    List<TreeNode> children;
}
```
**不管是两个节点还是多个节点，树当中是有层级关系的**，就是 parent 和 children 的关系，对于一般的树结构来说，一个节点内只会保存其 children 的信息，不会保存其 parent 的信息，给你一个树节点，你只会往其 children 的方向走，也就是说，**树的遍历其实是有方向性的**。

最后我们来看看图，图的话分为有向图和无向图，树其实是算有向图当中的一种，有向无向，主要是看边，如果两个节点连在一起，它们之间是互通的话就是无向图，如果只能从一个节点到另一个节点，反之可能不行，那就是有向图，不管是有向图还是无向图，在代码中，我们都可以表示下面这样：
```java
class GraphNode {
    int val;
    List<Integer> neighbors;
}
```
这不就是前面多叉树的表示方法吗？没错，但是图中的关系不再是 parent 和 children 的关系了，而是邻居的关系，**这里也没有层级结构了，每个节点都是平等的**。

讲完了这几个数据结构，我们再回过头来看看广度优先搜索这个算法，这个算法经常被用在树上和图上，我们来思考一下这个问题，**如果给你一个连通图上的一个节点，如何才能得到图上所有的节点呢？** 这个思路其实很简单，首先我们知道，我们可以把给定节点和其邻居加入到答案中，但是邻居还有邻居，因此我们还是得继续这个过程，直到把所有的点都找到，这之中我们可能会遇到一种情况就是，我们访问到了之前找到过的点，因此，这里我们还需要一个判重的机制。这里有一点是，每个点只可能找到其邻居，也就是说只会往其周围的点找，一次只向外扩散一格，**解决广度优先搜索问题，我们会使用队列这么一个 FIFO 的数据结构**，这不难理解，先找到的点我们先考虑其邻居。

<br>

## 问题分类

上面我们简单介绍了一下广度优先搜索这个算法，那么它可以用来解决什么样的问题呢？
* 层级遍历
* 由点到面遍历图
* 拓扑排序
* 求最短路径

我们一个一个来讲解，首先是层级遍历，前面讲过，每个节点只会找到其周围的节点，你可以想象成当前层的节点只可能找到下一层的节点（前一层遍历过不考虑），因此我们可以把一层找到的东西放在一起，这也就是用层这个概念对找到的所有节点进行归类。

第二点是**遍历图**，其实就是上面中的例子“给定连通图上面的一个节点，需要找到这个图中的所有节点”，你可能会问，遍历整个图有什么用呢？如果我们知道来所有节点的数量，**其实通过遍历整个图我们还可以判断一个图的连通性**，如果从一个点出发找不到某些点，那么说明其实这不是一个连通的图，有些节点不在图上，被分开了。

第三点是**拓扑排序**，这里可以参考我之前写的一篇文章 [拓扑排序原理和习题分析](https://juejin.im/post/5d02ce39f265da1bbc6fd0b9)。

第四点，也是比较常用的就是**找出图上两点的最短路径**，当然这里是有条件的，就是这个图必须是简单的连通图，什么是简单图，就是边没有权重，或者说权重都为固定的值。从一个点出发，找到下一层的所有点，从下一层的点出发，找到下下层的所有点，每到一层就算走一步，当我们找到我们要找的点，此时的步数就是最后的答案。

对于广度优先搜索的时间和空间复杂度的分析也是比较简单，一般问题都需要遍历整个图，因此时间复杂度是 **O(N + M)**，空间复杂度是 **O(N)**，这里的 N 表示的是节点的总数量，M 表示的是边的数量，有些图中，比如说全连通图(M = N^2)，我们遍历的时候，会尝试去走所有的边，对于空间来说的话，一般只会记录访问过的节点和当前层的节点，不会去考虑边的情况，因此时间复杂度和空间复杂度在这里还是不太一样的。

<br>

## LeetCode 常见题目

[LeetCode 103. Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)

**题目分析**：

考察上面提到的第一点，**层级遍历**。这道题其实是树的层级遍历的变形，我们需要记录每一层的信息，但是记录的顺序有区分，第一层从左向右记录，第二层反过来，从右向左记录，第三层从左向右记录，。。。，你可以看到每一层的记录方向和上下层都不一样，是一种交错的形式，当然我们可以都从左向右记录，然后到特定的层就把记录的结果给反转一下，但是这里有一个小技巧就是，**我们都是从左向右记录，但是记录方式不一样，一种记录方式是从列表的尾部加入，另一种是从链表的头部加入**。在树上的遍历相对来说比较简单，因为树的遍历是有方向性的，这个方向性确保了我们不会访问到我们之前访问过的节点，因此，这里我们不需要使用 Set 或者 boolean 数组去帮助去重。

**参考代码**：
```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    
    if (root == null) {
        return result;
    }
    
    Queue<TreeNode> queue = new LinkedList<>();
    
    queue.offer(root);
    
    boolean isReverse = false;
    while (!queue.isEmpty()) {
        int size = queue.size();
                    
        List<Integer> curLevel = new ArrayList<>();
        
        for (int i = 0; i < size; ++i) {
            TreeNode cur = queue.poll();
            if (cur.left != null) {
                queue.offer(cur.left);
            }

            if (cur.right != null) {
                queue.offer(cur.right);
            }
            
            if (isReverse) {
                curLevel.add(0, cur.val);
            } else {
                curLevel.add(cur.val);
            }
        }
        
        isReverse = !isReverse;
        
        result.add(curLevel);
    }
    
    return result;
}
```

<br>

[LeetCode 261. Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/)

**题目分析**：

考察第二点，由点及面遍历图。首先需要知道的问题是，**如何判断一个树是不是有效的？** 这里有两点，**第一就是树上是没有环的**，怎样才能确定其无环呢？如果你画出一个树，你就会发现，不管怎么画，其节点数和边的数量都是 **n - 1 = edges** 的关系，那么有这一点是不是就够了，并不是，**我们还需要判断连通性**，也就是说从一个节点出发，可以遍历到所有的点，满足这两个条件的图才能算是树。

**参考代码**：
```java
public boolean validTree(int n, int[][] edges) {        
    if (n - 1 != edges.length) {
        return false;
    }
    
    Map<Integer, Set<Integer>> graph = new HashMap<>();
    
    buildGraph(graph, edges, n);
    
    Queue<Integer> queue = new LinkedList<>();
    Set<Integer> visited = new HashSet<>();
    
    queue.offer(0);
    
    while (!queue.isEmpty()) {
        int cur = queue.poll();
        
        if (!visited.add(cur)) {
            return false;
        }
        
        for (int nei : graph.get(cur)) {
            queue.offer(nei);
            
            graph.get(nei).remove(cur);
        }
    }
    
    return visited.size() == n;
}

private void buildGraph(Map<Integer, Set<Integer>> graph, int[][] edges, int n) {
    for (int i = 0; i < n; ++i) {
        graph.put(i, new HashSet<Integer>());
    }
    
    for (int[] edge : edges) {
        graph.get(edge[0]).add(edge[1]);
        graph.get(edge[1]).add(edge[0]);
    }
}
```

<br>

[LeetCode 317. Shortest Distance from All Buildings](https://leetcode.com/problems/shortest-distance-from-all-buildings/)

主要考察求最短距离，可以参考之前 [ARTS](https://juejin.im/post/5d0bef25f265da1b91639ade) 中的算法部分。这里有一点需要解释的是，**矩阵其实也可以看作是一个图**，矩阵中的空格可以看作是一个节点，每个节点可以连接到其相邻的上下左右 4 个节点，每两个节点之间形成一条边，对于一个 m * n 的矩阵，如果看成一个图的话，图的节点的数目就是 m * n，边的数目是 m * n / 2。

<br>

## 总结
广度优先搜索就说到这里，这里我没有列出很多的题目，**理解算法本身，及其适合解决的问题是关键**，广度优先搜索主要适合解决层级遍历、由点及面遍历图、拓扑排序以及在求在简单图上两点之间的最短距离，理解了这些原理性的东西后，再去刷一些题目去巩固这些知识点，最后才能对这个算法了然于心。