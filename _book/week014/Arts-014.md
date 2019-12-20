## Algorithm

[LeetCode 317. Shortest Distance from All Buildings](https://leetcode.com/problems/shortest-distance-from-all-buildings/)

**思路分析**<br>
一道 2 维矩阵上最短路径问题，建议类似的最短路径问题首先考虑**广度优先搜索**，这里说一下深度优先搜索为什么不行，我们求两个坐标距离最短，如果用深搜的话，一条路径搜下去你不能保证这条路径上经过的点都是当前最短，如果硬要求的话必须找出所有的可能的路径，这样时间复杂度特别高；广搜的话我们只需要一个数组去记录我们经过的坐标即可，保证每个点只访问一次，且这次访问得到的就是这个点到起点的最短距离。

这道题还有一个特别诡异的地方，第一次做一般是想不到的，题目描述是求一个空位(0)，到所有建筑(1)的距离最短，我们想当然会从空位开始去找建筑，这样每个空位去找一遍，最后在所有空位中选出最小的那个即可，但是你有没有想过一个问题，那就是**这个图上的建筑多还是空位多**？如果你想过，并且看过几个测试样例，你会发现，图上空位(0)的数量远大于建筑(1)的数量，每搜索一次相当于遍历所有的点，如果从空位作为起始点开始搜的话，那么搜索的次数远大于从建筑作为起点开始搜，因此这里需要一点逆向思维，从建筑开始搜，有一点不同的是我们需要对每个空位到建筑的距离进行累加，这个不难理解

<br>

**参考代码**
```java
private class Pair {
    int x, y;
    Pair(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

private int[] dirX = {0, 0, 1, -1};
private int[] dirY = {1, -1, 0, 0};

public int shortestDistance(int[][] grid) {
    if (grid.length == 0 || grid[0].length == 0) {
        return -1;
    }
    
    int m = grid.length;
    int n = grid[0].length;
    
    int[][] distance = new int[m][n];
    int[][] counts = new int[m][n];
    
    int count = 0;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (grid[i][j] == 1) {
                bfsMark(grid, distance, counts, i, j);
                count++;
            }
        }
    }
    
    int min = Integer.MAX_VALUE;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) { 
            if (distance[i][j] != 0 && counts[i][j] == count) {
                min = Math.min(min, distance[i][j]);
            }
        }
    }
    
    return min == Integer.MAX_VALUE ? -1 : min;
}

private void bfsMark(int[][] grid, int[][] distance, int[][] counts, int i, int j) {
    Queue<Pair> queue = new LinkedList<>();
    
    int m = grid.length;
    int n = grid[0].length;
    
    boolean[][] visited = new boolean[m][n];
    
    queue.offer(new Pair(i, j));
    visited[i][j] = true;
    
    int length = 1;
    while (!queue.isEmpty()) {            
        int size = queue.size();
        
        for (int s = 0; s < size; ++s) {
            Pair cur = queue.poll();
            
            for (int k = 0; k < 4; ++k) {
                int curX = cur.x + dirX[k];
                int curY = cur.y + dirY[k];

                if (curX < 0 || curY < 0 || curX >= m || curY >= n 
                        || visited[curX][curY] || grid[curX][curY] != 0) {
                    continue;
                }

                queue.offer(new Pair(curX, curY));
                distance[curX][curY] += length;
                visited[curX][curY] = true;
                counts[curX][curY]++;
            }                
        }
        
        length++;
    }
}
```

<br>


## Review
关于学习计算机语言的一些建议:<br>

[The One Programming Language to Rule Them All](https://medium.com/better-programming/the-one-programming-language-to-rule-them-all-620366df2805)

关于学习哪门语言这样的问题往往会吸引人们的眼球，在一般人看来学习一门上手快，见效快，最好还是应用广的语言再好不过。人们往往会急于求成，要花一年的时间来学的东西，如果能在一个月学完就最好，他们会想着如何去走捷径，选择那些他们觉得轻松的东西去学。久而久之，这种想法就会形成习惯，到时可能再看看自己，其实真正学到的东西很少。

作者在文章中提到，**最好的语言是逻辑**，这句话怎么理解，其实所有的语言到了计算机底层都会转为一条条指令，没任何区别，**我们想让计算机帮助我们解决问题，我们就要了解计算机的 “思考问题的逻辑”**。人与计算机沟通和人与人沟通的最大区别在于对于一个问题描述的细致程度，我们和正常人沟通的时候，我们其实不需要描述特别细致，特别具体，就好像告诉一个人怎么去某个地方，我们会说 “直走到XX路右转，然后搭XX公交，到XX站下就是了”，因为**人和人对世界或者事物外表状态的理解其本质是相同的**，我们大家都识字，也认得公交站牌，你说几个名词，我大多数情况下也见过，你不需要补充说明。但是计算机不是这样，刚开始它什么也不知道，它不识字，不知道什么是右转，也不知道什么是公交车，如果是第一次做这个事情，你必须非常细致、毫无遗漏地告诉它每个细节信息，当然你可以让它记忆，但是你必须告诉它，不能默认它知道。不常和计算机打交道，突然让它做事情，有时你会觉得茫然，但这都是正常的。**我们要学会用计算机的思考逻辑和计算机打交道，这个才是计算机专业要干的事情。和计算机打交道久了，明白了它的逻辑，你会发现这种方式让你很踏实，因为它说一就是一，它很诚实，只要按照某个步骤就一定能得到某个结果，而且它从不犯错，如果说你写的程序出现问题，那 99.999999% 是你的问题，你不需要去猜测是谁的问题，你要做的仅仅是去找到并解决这个问题**。

怎么学计算机的思考逻辑，还是基础啊，这是个说了成百上千遍都不会变的话题，从基础学起，学习那些不会变的算法和数据结构、设计思想、代码逻辑才能让你不被这个时代所抛弃，也同时能让你在这个时代里看清真相。

<br>


## Tip
vagrant 是一个快速构建虚拟环境的工具，其运行在 Oracle 的 VirtualBox 上，其配置主要写在 Vagrantfile 文件之中，这次记录下它的一些常用配置
```bash
Vagrant.configure("2") do |config|
  config.vm.hostname = "ubuntu"
  config.vm.box = "base"
  # 分配私有 IP，这样可以在 HOST 机上访问虚拟机的 localhost
  config.vm.network "private_network", ip: "192.168.50.4"
  # 共享文件
  config.vm.synced_folder "path in host...", "path in virtal machine"
  config.vm.provider "virtualbox" do |vb|
    vb.name = "pyh3326"
    vb.gui = false
    vb.memory = "1024"
  end
end
```

另外 vagrant 的初始化、登陆、重新加载、关机命令分别为为:
```bash
>$ vagrant up
>$ vagrant ssh
>$ vagrant reload
>$ vagrant halt
```
记得不用虚拟机要关闭

<br>

## Share
最近看到布隆过滤器(Bloom Filter)这个数据结构，面试当中也是被经常问到，这次写在分享里面吧

[布隆过滤器原理和应用分析](./布隆过滤器原理和应用分析.md)
