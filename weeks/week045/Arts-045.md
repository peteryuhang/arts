> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 787. Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/)

**题目解析**：

个人觉得这是一道和现实生活结合紧密的题目。两地之间有通行航班，并且每个航班都有票价，题目给定所有航班信息（出发地、到达地和票价），还有起点以及终点，问从起点到终点，不大于 K 次中转的最便宜的花费。

这道题目有好多种解法，这里说说最常见的两种，这个问题很容易知道是一个图相关的问题，图上的节点代表着城市，边代表的是航班。题目要求的是最小票价，而且步数有限制，因此我们可以考虑使用广度优先搜索，**我们只看从起点走 K 步后所有的组合情况，在搜索进行中我们可以不断地记录每个组合到达终点的开销，取其中的最小值即可**。实现广度优先搜索没什么好说的，可以直接套模版，然后具体问题具体分析。

但是如果你用广度优先搜索去实现后会发现，虽然思路很直接，但是实现起来细节挺多的，比较繁琐，代码量也比较长。第二种方法是利用动态规划，实现上面会相对来说简洁些，思路也很简单，因为这道题目我们关系的东西有 3 个，分别是 **当前到达的城市**，**第几次中转**，以及 **当前总花费**，因此我们可以定义动态规划状态 `dp[i][j]` 表示的是**在城市 j 的第 i 次中转的总花费**。实现上面来说的话，在出发地，以及每次中转的时候我们需要遍历一遍航班信息去更新我们的状态数组，这样算下来，一共 K 次中转，因此时间复杂度是 **O(K * F)**，其中 F 表示的是航班总数，空间复杂度是 **O(K * n)**，其中 n 是城市的数量。

<br>

**参考代码（广度优先搜索）**：
```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int K) {
    Map<Integer, Map<Integer, Integer>> graph = new HashMap<>();

    // 构建图
    for (int[] flight : flights) {
        if (!graph.containsKey(flight[0])) {
            graph.put(flight[0], new HashMap<>());
        }

        graph.get(flight[0]).put(flight[1], flight[2]);
    }

    Queue<int[]> queue = new LinkedList<>();
    
    // 队列中保存两个信息，分别是 当前所在城市 以及 当前总花费
    queue.offer(new int[]{src, 0});
    
    // 记录在除终点外所有的城市的最小开销
    Map<Integer, Integer> costAtNode = new HashMap<>();
    
    costAtNode.put(src, 0);
    
    // 最后要输出的答案，记录的是在终点的最小开销
    int totalCost = Integer.MAX_VALUE;
    
    // 记录当前考虑的是第几次中转
    int step = 0;

    // 广度优先搜索
    while (!queue.isEmpty()) {
        // 考虑完 <= K 次中转的所有情况，不需要继续考虑
        // 清空队列，记录答案
        if (step > K) {
            while (!queue.isEmpty()) {
                int[] lastStep = queue.poll();
                if (lastStep[0] == dst) {
                    totalCost = Math.min(totalCost, lastStep[1]);
                }
            }

            break;
        }

        int size = queue.size();

        for (int i = 0; i < size; ++i) {
            int[] curInfo = queue.poll();
            // 当前城市是终点，记录答案
            if (curInfo[0] == dst) {
                totalCost = Math.min(totalCost, curInfo[1]);
                continue;
            }
            
            // 如果当前城市不是终点
            // 遍历当前城市的所有航班
            if (graph.containsKey(curInfo[0])) {
                for (int neighbor : graph.get(curInfo[0]).keySet()) {
                    // 下一个城市之前记录过并且总票价便宜，则直接跳过
                    if (costAtNode.containsKey(neighbor) 
                            && costAtNode.get(neighbor) <= graph.get(curInfo[0]).get(neighbor) + curInfo[1]) {
                        continue;
                    }

                    costAtNode.put(neighbor, curInfo[1] + graph.get(curInfo[0]).get(neighbor));
                    queue.offer(new int[]{neighbor, 
                                          curInfo[1] + graph.get(curInfo[0]).get(neighbor)});
                }
                
            }
        }
        
        step++;
    }
    
    return totalCost == Integer.MAX_VALUE ? -1 : totalCost;
}
```

<br>

**参考代码（动态规划）**：
```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int K) {
    // dp[i][j] 表示的是在城市 j 进行第 i 次中转时的最小开销
    int[][] dp = new int[K + 2][n];

    // dp 状态数组初始化
    for (int[] d : dp) {
        Arrays.fill(d, Integer.MAX_VALUE);
        d[src] = 0;
    }

    int result = Integer.MAX_VALUE;

    for (int i = 1; i <= K + 1; ++i) {
        for (int[] flight : flights) {
            // 考虑第 i 次中转时
            // 如果第 i - 1 次中转后有到达过当前考虑的航班的出发地，说明当前航班有效
            if (dp[i - 1][flight[0]] != Integer.MAX_VALUE) {
                // 更新到达地的状态
                dp[i][flight[1]] = Math.min(dp[i - 1][flight[0]] + flight[2],
                                            dp[i][flight[1]]);
                // 如果到达地是最终的终点，更新答案
                if (flight[1] == dst) {
                    result = Math.min(dp[i][flight[1]], result);
                }
            }
        }
    }

    return result == Integer.MAX_VALUE ? -1 : result;
}
```
<br>

## Review
[The 10 Best Books I Read This Year](https://medium.com/better-marketing/the-10-best-books-i-read-this-year-1fee6d3c4a1b)

也是一篇介绍并鼓励阅读的文章。作者推荐了自己看过觉得不错的书，基本上都是和认知相关的，而且覆盖面特别广，有人文，历史，管理，方法论等等。看完文章后，回过头看看自己，有时我在思考，像计算机软件行业到底是理论重要些还是实践重要些，当然了，我知道二者都很重要，可能从事的工作不同各有偏重。那么，程序员这个行业中该如何通过阅读来增进自己的认知呢？之前我会认为需要多读些技术相关的书籍，但是现在想想，相比于自我认知和方法论一类的知识，或许技术并不是最迫切需要的，因为前者可以决定你前进的方向以及看待问题的方式，所以说技术人还不能仅仅只知道技术。

<br>

## Tip

这周学习了 MongoDB 里面的一些之前不知道的用法

* MongoDB 支持对数组中的元素进行搜索
  ```
  // 找到 color 中包含 red 的所有文档
  db.fruit.find({color: "red"})
  
  // 找到 color 中包含 red 或者 blue 的所有文档
  db.fruit.find({$or: [{color: "red"}, {color: "blue"}]})
  
  // locations 是数组，也可以通过 filed.sub_filed 的形式查找
  db.movies.find({"locations.city": "Rome"})
  ```
* mongoDB 的 updateOne/updateMany 方法要求更新条件必须具有以下之一，否则会报错：
  * `$set/$unset`: **增加/删除** 一个字段
  * `$push/$pushAll/$pop`: 从数组底部 **增加一个/增加多个/删除一个** 对象
  * `$pull/$pullAll`: 如果匹配 **指定/任意** 的值，从数组中删除相应的对象
  * `$addToSet`: 如果不存在则增加一个值到数组
* MongoDB 的聚合查询格式
  ```
  pipline = [$stage1, $stage2, ...$stageN];
  db.<COLLECTION>.aggregate(
    pipeline,
    { options }
  )
  ```

<br>

## Share

最近看了一本关于职场的书籍，在这里总结一下，作为程序员需要知道一些程序之外的东西，这些东西有时比技术本身更重要

[职场中必须了解的十个认知](./职场中必须了解的十个认知.md)

