## Algorithm

[LeetCode 407. Trapping Rain Water II](https://leetcode.com/problems/trapping-rain-water-ii/submissions/)

**题目思路分析**<br>

在 1 个 2 维的矩阵中，每个格子都有其高度，问这个 2 维矩阵能够盛多少的水。首先我们分析，格子能够盛水的必要条件是其周围存在格子比当前格子高，这样水才能够被框得住，但是仔细一想，最外围的格子怎么办？它们是存不了水的，可以把最外围的格子想象成围栏，它们的作用就是保证里面格子的水不会流出来，所以我们就得先考虑这些格子，它们的高度直接决定了内部格子的蓄水量，但是这些格子也有局部性，一个格子的长短并不会影响矩阵当中所有的格子，但是它会影响与其相邻的格子，那么我们就需要有一个考虑的顺序，那就是优先考虑最外层最短的格子，由于每个格子都会影响到其周围的格子，内部格子也需要列入考虑范围，每次我们都考虑最短的格子，然后看其周围有没有没考虑过的比它还短的格子，于是就有了考虑的先后顺序：
1. 考虑最外层格子
2. 选出最外层最短的格子
3. 考虑该格子与其相邻的内部格子是否能盛水，并把这个内部格子也纳入考虑范围
4. 在考虑范围内的所有格子中选出最短的格子，重复步骤 3

这里需要注意的是，每次纳入考虑范围的格子是加了水之后的高度，而不是之前的高度，原因想一下应该不难理解。另外就是可以使用了 “堆” 这个数据结构来帮助实现寻找 “当前考虑范围内最短的格子” 这个操作步骤。

**参考代码**
```java
private class Pair {
    int x, y, h;
    Pair(int x, int y, int h) {
        this.x = x;
        this.y = y;
        this.h = h;
    }
}

private int[] dirX = {0, 0, -1, 1};
private int[] dirY = {-1, 1, 0, 0};

public int trapRainWater(int[][] heightMap) {
    if (heightMap.length == 0 || heightMap[0].length == 0) {
        return 0;
    }
    
    int m = heightMap.length;
    int n = heightMap[0].length;
    
    PriorityQueue<Pair> pq = new PriorityQueue<>(new Comparator<Pair>() {
        @Override
        public int compare(Pair a, Pair b) {
            return a.h - b.h;
        }
    });
    
    boolean[][] visited = new boolean[m][n];
    
    for (int i = 0; i < n; ++i) {
        pq.offer(new Pair(0, i, heightMap[0][i]));
        pq.offer(new Pair(m - 1, i, heightMap[m - 1][i]));
        
        visited[0][i] = true;
        visited[m - 1][i] = true;
    }
    
    for (int i = 1; i < m - 1; ++i) {
        pq.offer(new Pair(i, 0, heightMap[i][0]));
        pq.offer(new Pair(i, n - 1, heightMap[i][n - 1]));
        
        visited[i][0] = true;
        visited[i][n - 1] = true;
    }
    
    int result = 0;
    while (!pq.isEmpty()) {
        Pair cur = pq.poll();
        
        for (int k = 0; k < 4; ++k) {
            int curX = cur.x + dirX[k];
            int curY = cur.y + dirY[k];
            
            if (curX < 0 || curY < 0 || curX >= m || curY >= n || visited[curX][curY]) {
                continue;
            }
            
            if (heightMap[curX][curY] < cur.h) {
                result += cur.h - heightMap[curX][curY];
            }
            
            pq.offer(new Pair(curX, curY, 
                              Math.max(heightMap[curX][curY], cur.h)));
            visited[curX][curY] = true;
        }
    }
    
    return result;
}
```

<br>

## Review
一篇关于 Node.js 中项目代码结构的文章:<br>

[Bulletproof node.js project architecture](https://dev.to/santypk4/bulletproof-node-js-project-architecture-4epf?utm_source=mybridge&utm_medium=blog&utm_campaign=read_more)

作者的有几个观点我觉得还是很值得借鉴的：
* **使用三层结构**<br>
  controller、service、model 分别处理 REST 请求、逻辑处理、以及数据库操作，这样的好处是 controller 中不会有逻辑操作，每个函数的任务都很清晰、明确，代码会更加的简洁，而且程序出 bug，定位问题也会更高效
* **利用 JS 中的发送和监听机制来处理业务**<br>
  举个例子，当我们在服务器端新建一个 user 账号时，这个 “新建” 操作可能会涉及到查找记录、创建账号、初始化信息、发送邮件通知等等，如果这些东西都在一个模块中进行，难免会使这个模块中的代码逻辑变得很复杂。我们可以把这些东西分成一系列的小监听组件（XXX.on(...)），这样新建操作只需要 emit “新建” 这个事件，相关的组件都会被触发，这样在主代码逻辑中会更加地清晰，而且这些小的监听组件也可以被重复使用
* **使用注入依赖（Dependency Injection）的方式来组织代码**<br>
  node.js 中可以考虑使用 typedi 这个库来实现注入依赖
* **考虑环境变量和config文件结合的方式来存储私密资料**<br>
  把类似密码、数据库 IP 一类重要的信息直接写在功能代码中或者普通文件中，都不是一个安全的方式，Node.js 使用 process.env 设置环境变量的方式存储这些重要的私密信息，我们可以考虑使用 **dotenv** 这个库来帮助在 .env 隐藏文件中定义这些私密信息，但是这里有一点不好的是没有办法整合归类，于是考虑把这些环境变量在export 到一个 config 文件中整合归类，这样，在我们写代码的时候借助 IDE 的功能补全可以快速地找到对应的环境变量
* **不要把所有的逻辑都放在同一个函数或者文件中** <br>
  有一个清晰的设计理念就是，“一个组件只做一件事情”，这样的代码才是可测试的代码，代码的重用率才会提高，也是更便于他人理解的

作者同时在文章的开头给出了他觉得不错的一个文件结构：<br>
src<br>
&nbsp; &nbsp; app.js          # App entry point<br>
&nbsp; &nbsp; **api**             # Express route controllers for all the endpoints of the app<br>
&nbsp; &nbsp; **config**          # Environment variables and configuration related stuff<br>
&nbsp; &nbsp; **jobs**            # Jobs definitions for agenda.js<br>
&nbsp; &nbsp; **loaders**         # Split the startup process into modules<br>
&nbsp; &nbsp; **models**          # Database models<br>
&nbsp; &nbsp; **services**        # All the business logic is here<br>
&nbsp; &nbsp; **subscribers**     # Event handlers for async task<br>
&nbsp; &nbsp; **types**           # Type declaration files (d.ts) for Typescript
  

<br>

## Tip
最近在学 Python，总结一些列表、元组、集合、字典的使用注意事项：

**列表和元组**
* 列表是动态的，元组是静态的，列表可变，元组不可变
* 列表和元组都支持负数索引和切片操作
* 列表根据 **over allocate** 原则去进行扩充
* 同等大小的列表会比元组多 **16** 个字节，原因在于存储指针和记录最大容量的变量
* 新建元组会比新建列表更高效，原因在于 Python 的垃圾回收机制对于不用的、大小不是太大的元组会进行缓存，等到新建元组的时候就不需要再向操作系统请求开辟内存空间，直接使用这些缓存的空间即可
* 一些常用的内置函数
    * **count**
    * **index**
    * **sort**          # 原地排序
    * **reverse**       # 原地反转
    * **sorted**        # 排序后返回排序好的新列表/元组
    * **reversed**      # 反转后返回排序好的新列表/元组

**集合和字典**
* 集合和字典的初始化都可以使用 {}
* 可以直接使用 **==** 判断两个字典或者集合中内容是否完全相同
* 使用 **value in set/dict** 来判断一个 key 是否存在于集合或者字典中
* 可以考虑使用 **get(key, default)** 来获取对应值而不产生报错
* 集合使用 **remove** 删除元素，**add** 添加元素，字典使用 **pop** 删除元素，注意 pop   在集合中是删除最后一个元素，但是集合本来就是无序的，最好不要使用这个函数
* 对集合进行排序会返回一个列表，对字典进行排序会返回一个含有二元组的列表
* 字典和集合在底层实现中和 Java 的 HashMap 和 HashSet 类似，但是处理冲突的解决方法有所不同，遇到冲突时会继续寻找，直到找到为止，而不是使用链表或者树的结构
<br>

## Share

这次继续来积累算法知识，这次看看深度优先搜索，通过它，我们可以发现很多高级的算法，将学过的东西建立联系、融会贯通也是一件非常有意义的事情。

[从简单二叉树问题重新来看深度优先搜索](./从简单二叉树问题重新来看深度优先搜索.md)