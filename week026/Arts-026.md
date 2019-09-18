## Algorithm

[LeetCode 815. Bus Routes](https://leetcode.com/problems/bus-routes/)

**题目解析**：

题目给你一堆公交路线，以及起点和终点，要你求出 **最少** 需要乘坐多少辆公交才能从起点到终点，这里每个公交都是环线。

普通的路径问题只有地点（节点）这么一个概念，但是这道题目有两个概念 “**公交车**” 和 “**公交站点**”，如果我们把这道题抽象成图，很明显，**公交站点** 是图上的节点，那 **公交车** 怎么处理，题目的问题可是 **“最少要搭多少辆公交车”**。对于这道题目，我们可以这样思考，每个公交站点都会存在一个或者多个公交，每个公交又会对应一个或者多个公交站点，这其实是个多对多的问题，如果把这道题抽象成图，从图上的起点去到图上的终点，在中途的任意一点我们其实都可以知道这一点存在哪些公交车，有了这些公交车，我们又可以求得其可以到达的下一个 **公交站点**，因此思路就出来了：
```
公交站 -> 对应的公交车 -> 下一个可以到达的公交站
```

这道题其实就是转了个弯，中间增加了一层映射关系，其本质还是图上的搜索问题，使用 **广度优先搜索** 或者 **深度优先搜索** 都可以很好地解决，但是考虑这道题求的是 **最短距离**，广度优先搜索更为恰当些。

<br>

**参考代码**：
```java
public int numBusesToDestination(int[][] routes, int S, int T) {
    if (routes == null || routes.length == 0) {
        return -1;
    }

    // 如果起点和终点是同一个位置，直接结束
    if (S == T) {
        return 0;
    }

    //  stop No.    bus No.
    Map<Integer, Set<Integer>> stop2Bus = new HashMap<>();

    // 建立 “公交站点” -> “公交车” 的映射关系
    for (int i = 0; i < routes.length; ++i) {
        for (int j = 0; j < routes[i].length; ++j) {
            if (!stop2Bus.containsKey(routes[i][j])) {
                stop2Bus.put(routes[i][j], new HashSet<Integer>());
            }

            stop2Bus.get(routes[i][j]).add(i);
        }
    }

    Queue<Integer> queue = new LinkedList<>();
    Set<Integer> visited = new HashSet<>();

    // 将起始位置的所有公交车都加入队列
    // 并用 Set 去重，同一个公交站点不可能访问两次
    for (int i : stop2Bus.get(S)) {
        visited.add(i);
        queue.offer(i);
    }

    // 广度优先搜索层级遍历
    int busNeedTake = 1;        // 所需要搭的公交车数量
    while (!queue.isEmpty()) {
        int size = queue.size();

        for (int i = 0; i < size; ++i) {
            int curBus = queue.poll();
            
            // 当前遍历到的公交车可以到达终点，返回答案
            if (stop2Bus.get(T).contains(curBus)) {
                return busNeedTake;
            }
            
            // 将当前公交车所经过的所有站点的所有公交车加入队列
            for (int j = 0; j < routes[curBus].length; ++j) {
                for (int bus : stop2Bus.get(routes[curBus][j])) {
                    if (visited.add(bus)) {
                        queue.offer(bus);
                    }
                }
            }
        }
        
        busNeedTake++;
    }
    
    return -1;
}
```
<br>

## Review
[How a Googler solves coding problems](https://blog.usejournal.com/how-a-googler-solves-coding-problems-ec5d59e73ec5)

文章的作者给出了解决编程类问题的步骤，这些问题可以是 LeetCode 上面的算法题，也可以是工作中的业务函数，**清楚、熟悉并习惯一个方法能够让我们平时的思考更系统化，更清楚自己应该先做什么，后做什么，该做什么以及不改做什么**，文章中给出了解决问题的 5 个步骤：
* **第一步：在图纸上设计逻辑算法**。通常在我们一开始遇到一个问题，我们需要知道问题是什么，我们有什么策略可以用来解决这个问题，这个时候我们可以在纸上用画图的方式帮助我们理解我们遇到的问题场景以及我们选用的逻辑算法，同时我们可以简单验证我们的思路和想法是不是能够 handle 所有场景，当然这些场景最好全面些，我们首先要保证策略在逻辑的大层面是说的通的，这样才有接下来实现代码的意义。
* **第二步：用文字来描述代码要做的事情**。在动手使用代码来实现之前，我们可以用我们熟悉的日常生活中的语言来描述我们要干的事情，也就是把我们上面想到的算法逻辑给表达出来，往往在输出的时候我们就能发现一些自己之前没有覆盖到的点，这一步也是帮助我们把想法给具体化的第一步。
* **第三步：写伪代码实现**。这里为什么先要写伪代码？**如果我们对我们使用的编程语言不是特别熟悉，往往我们会陷入实现细节的坑，而忽略了大层面上面的代码逻辑构建**，而先用伪代码去实现可以让我们清楚知道我们需要用到什么样的函数模块来完成实现，也清楚知道自己需要去了解自己所用到的编程语言的哪一部分，这样，自己所需要做的事情就更清晰了。
* **第四步：将伪代码转换成代码**。上一步有了伪代码作为铺垫，相信这一步应该不会太难，当然如果在转换过程中遇到自己不熟悉的部分，可以标明，方便后面 Google。
* **第五步：将自己不清楚的东西弄清楚**。很多程序员新手习惯猜测，但是不清楚的东西放在一边不管往往会产生 bug，而且当不清楚的东西越多，出错点就越多，出现了 bug 也比较难 debug。**不清楚的东西，越早弄明白越好**，比如当自己对一个内置函数模块不是特别了解的时候，应该 Google，然后检验这个内置函数模块在当前的场景是不是适用，**当程序当中的灰色地带越少，我们就对自己写的东西会越自信**。


<br>

## Tip
[JavaScript console is more than console.log()](https://medium.com/devgorilla/the-console-object-provides-access-to-the-browsers-debugging-console-354eda9d2d50)

上面的文章给出了 JavaScript 中的 console 的一些比较实用的方法，在这里总结一下：
* 当我们要输出多个对象的时候：
  ```
  console.log({ foo, bar })
  ```
* 可以像 logger 一样标记输出信息的种类
  ```
  console.warn()
  console.error()
  console.info()
  ```
* 当我们需要输入多行代码的时候，可以让输出更有层级结构
  ```
  console.group()
    console.log()
    console.log()
    ...
  console.groupEnd()
  ```
* 用 table 的形式更直观地去显示要输出的内容
  ```
  console.table()
  ```
* 追踪调用的 console 的具体路径，可以用于程序异常报错的具体定位：
  ```
  console.trace()
  ```
* 用于测试运行时间
  ```
  console.time()
  ...
  console.timeEnd()
  ```
* 打印出 JavaScript 对象中的所有属性
  ```
  console.dir()
  ```
* 清空 console
  ```
  console.clear()
  ```

    
<br>

## Share
写 ARTS 已经半年了，这次还是来聊聊算法吧。看看简单的算法题 - TwoSum 还可以怎么玩？

[TwoSum 相关问题思路总结](./TwoSum相关问题思路总结)

