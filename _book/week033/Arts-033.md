## Algorithm

[LC 218. The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/)

<br>

**题目解析**：

题目是说在二维坐标上面给你一些规则的长方形建筑，这些建筑可以使用 `[Li, Ri, Hi]` 来表示，Li 表示的是建筑的最左边对应的 x 轴坐标，Ri 表示的是建筑的最右边对应的 x 轴坐标，Hi 表示建筑的高，题目让你输出所有建筑放在一起的整体轮廓，轮廓也是用点来表示，用一条线上最左边的点表示。

拿到这道题首先感觉的是细节非常多，如果深入去想的话很容易陷到题目的细节中去，我们先分析一下大致的思路，思路清晰后再去考虑细节。题目给定的输入是按 x 轴排好序的，最后的输出也是需要按 x 轴排好序，这么来看，我们就需要从 x 轴的左边向右去分析和计算，**你可以想象成有一条垂直于 x 轴的直线从最左边一直缓慢地往右边移动，每移动到一个建筑的边界，我们就会去做判断**，判断无非两种情况，即把该点加入到答案中，或者是不加入。大致的思路其实就是这样，但是这里面有很多细节，我们一个个来讨论。

题目要求输出的点是水平线最左边的点，一个建筑有左右两个顶点，这两个点在考虑的时候会有所不同，因此我们需要标记一下，最左边的点就标记成建筑的起始点，最右边的点就标记成建筑的结束点，你可能会问为什么要标记？看看下面一些细节的分析你就懂了：
```
输入一：[[1, 4, 8], [1, 5, 7]]      // 起始边界重合
输出一：[[1, 8], [4, 7], [5, 0]]

输入二：[[1, 4, 8], [4, 5, 7]]      // 起始边界和终止边界重合
输出二：[[1, 8], [4, 7], [5, 0]]

输入二：[[1, 4, 8], [3, 4, 7]]      // 终止边界和终止边界重合
输出二：[[1, 8], [4, 0]]
```
大体上是这三种情况，**起始边界重合**我们仅考虑最高的那个建筑而忽略那些次高的建筑，**起始边界和终止边界重合**我们仅考虑起始边界忽略终止边界，**终止边界和终止边界重合**我们二者都不考虑。

你可以看到上面的分析过程其实是一个有条件的排序的过程，**我们会根据一个边界点的坐标 (x,y) 以及它是起始边界还是终止边界来排序**，说到排序就不得不提到堆这个数据结构，当然这道题的实现上有多种方式，我说说自己的思路，我首先会对输入的数组进行排序，当起始边界重合时，我会把高的建筑排在前面，这样判断的时候如果发现堆中有高的建筑存在，那么就不会考虑当前的点；起始边界和终止边界重合时，我会把低的建筑排在前面，在这种情况下我会把起始边界排在前面，就是先考虑起始边界点；终止边界和终止边界重合，会把低的建筑排在前面，这样可以让其先输出。

<br>

**参考代码**：
```java
private class Point {
    int x, y;
    boolean isEnd;
    
    Point(int x, int y, boolean isEnd) {
        this.x = x;
        this.y = y;
        this.isEnd = isEnd;
    }
}

public List<List<Integer>> getSkyline(int[][] buildings) {
    List<List<Integer>> results = new ArrayList<>();
    
    if (buildings == null || buildings.length == 0) {
        return results;
    }
    
    List<Point> points = new ArrayList<>();
    
    // 把一个建筑拆成两个点，起始点，终止点，加入到数组中
    for (int i = 0; i < buildings.length; ++i) {
        points.add(new Point(buildings[i][0], buildings[i][2], false));
        points.add(new Point(buildings[i][1], buildings[i][2], true));
    }
    
    Collections.sort(points, new Comparator<Point>(){
        @Override
        public int compare(Point a, Point b) {
            if (a.x != b.x) {
                return a.x - b.x;
            }
            
            // 起点重合，将高的建筑排在前面
            if (!a.isEnd && !b.isEnd) {
                return b.y - a.y;
            }
            
            // 终点重合，将低的建筑排在前面
            if (a.isEnd && b.isEnd) {
                return a.y - b.y;
            }
            
            // 终点和起点重合，将起点排在前面
            return a.isEnd ? 1 : -1;
        }
    });

    // 创建堆，堆中保存的是建筑的高，逆序排列
    // 因为这里同时会存在多个相同高度的建筑，因此使用 TreeMap 来实现堆
    TreeMap<Integer, Integer> tm = new TreeMap<>((Integer a, Integer b) -> (b - a));
    
    for (Point point : points) {
        if (!point.isEnd) { // 当前点是起始点
            // 当前建筑比目前出现过的所有的建筑都要高，将该点加入答案中
            if (tm.isEmpty() || tm.firstKey() < point.y) {
                results.add(new ArrayList<>(Arrays.asList(point.x, point.y)));
            }
            
            // 因为是起始点，需要将建筑加入堆中
            tm.put(point.y, tm.getOrDefault(point.y, 0) + 1);
        } else {    // 当前点是终止点
            
            // 终止点意味着要将建筑从堆中弹出
            if (tm.get(point.y) == 1) {
                tm.remove(point.y);
            } else {
                tm.put(point.y, tm.get(point.y) - 1);
            }
            
            // 如果堆为空，表示当前考虑的建筑是最后一个建筑，输出高度 0
            if (tm.isEmpty()) {
                results.add(new ArrayList<>(Arrays.asList(point.x, 0)));
            } // 如果堆不为空，看看之前的建筑是不是比当前建筑高
              // 如果之前建筑比当前建筑低，输出之前建筑的高度
            else if (tm.firstKey() < point.y) {
                results.add(new ArrayList<>(Arrays.asList(point.x, tm.firstKey())));
            }
        }
    }
    
    return results;
}
```

<br>

## Review
[Developers: Here Are 8 Questions You Should Ask Your Employer Before Taking the Job](https://medium.com/better-programming/developers-here-are-8-questions-you-should-ask-your-employer-before-taking-the-job-3e3cc67a855d)

在面试当中我们不断地向公司展示我们的优势，但是反过来，我们也需要去考虑这个公司的前景，以及公司目前的内部状况，毕竟面试是双向的选择，我们需要保证接下来我们几年的工作是符合我们期待的，因此需要做些筛选。对于大公司来说，可能我们会从不同渠道获得很多相关的资讯，但是其实你对你要去的某个团队并不了解；有些小公司会比较封闭，你也不太好从一些公众的渠道去获知一些情况。这篇文章提到，在面试过程中，我们可以在 “向面试官提问” 的这个环节去了解一个公司的方方面面，文章给出了 8 个比较好的问题：
* **How Often Do You Work Evenings and Weekends?** 

  这个问题主要是了解工作压力，如果经常需要在周末，晚上加班，意味着你需要牺牲很多你的业余时间，你不能够拥有属于自己的时间，去陪家人、放松自己、或者是学学自己想学的知识，那么就需要慎重考虑了
 
* **What’s the Average Turnover Rate?**
 
  去餐馆吃饭，评价一个餐厅的生意好不好，我们看翻桌率，翻桌率越高证明餐厅生意越好，一个职位也是讲究更替率，当然这个评价标准是反过来的，如果一个位置上的人频繁更替，公司或者团队员工流动量大，经常有员工动不动就离职，这个就可以从侧面反应大部分人不太愿意长期留在公司，可能会有各种原因，这个就得看面试官解释。

* **What’s a Typical Day Like in This Position?**

  让面试官描述一下，一般他在公司是如何度过的，看看公司的文化是不是和自己期待的公司文化切合。
  
* **How Have You Supported Developer Professional Development in the Past?**

  公司有没有一些机制能够帮助新员工快速适应新环境，特别是对于那些刚步入社会不久的员工，如果公司有专门的培训，指导手册，工作上面专门安排资历更高的员工带领，那说明这个公司其实挺看重员工的成长，否则的话，要根据自身情况慎重考虑了。
* **Ask for a Tour Around the Office at the End**

  面试结束之后可以要求参观一些公司的环境，如果公司的环境比较苛刻，那么就需要思考下在这个地方待个几年是否可以接受。

* **What’s the Best and Worst Thing About Working Here?**

  每个地方都有它的优点和缺点，如果面试官只说优点不说缺点，那只能说明要不就是面试官并没有仔细考虑或者说是在说谎，也要慎重考虑。

* **Ask Them to Describe Their Software Development Lifecycle (How Often Do They Release Code?)**
  
  可以问问一个公司的的产品开发周期，问问他们一般一个产品的开发周期会是多久，中间会经历怎样的流程，如果时间允许，也可以讲讲细节，比如在开发设计之前会不会做市场调查，或者是调研，程序员是只管闷头做事，还是说有机会发表对产品的看法和建议，一般产品是在星期几发布，如果是在周五的话，产品线上出问题谁来维护？总之是要了解这个公司是不是有一个好的开发流程和策略。

* **Ask About the Company’s Long Term Vision (What Are Your Plans for the Next Five Years?)**

  询问公司的长远目标，如果公司的长远目标和自己的职业发展目标契合，那至少说明可以有个憧憬，如果一个公司没有具体的方向和目标，大概率来说公司的员工也并没有一个长远目标，可能每天连自己做的产品是什么都不清楚，这会很糟。

<br>

## Tip

这周学习并使用了另一个进程管理命令 pm2，相比之前提到的 screen，pm2 主要针对 node，还提供了一些额外的功能，比如**性能监控**、**自动重启**、**负载均衡** 等等，这里记录一下 pm2 中的常见命令：

全局安装：
```
npm install -g pm2
```

更新版本：
```
pm2 update
```

运行程序：
```
pm2 start app.js
pm2 start app.js --watch            # 当文件发生改变时，自动重启
pm2 start app.js --log <log_path>   # 将 log 写入指定的文件
pm2 start app.js -- arg1 arg2 ...   # 传入额外的参数
...
```

进程状态管理指令：
```
pm2 restart app_name
pm2 reload app_name
pm2 stop app_name
pm2 delete app_name
```

查看状态：
```
pm2 [list|ls|status]
```

查看 logs:
```
pm2 logs
pm2 logs --lines 200         # 查看之前的 log
```

监控面板：
```
pm2 monit
pm2 plus                # 基于 web 的面板
```



<br>

## Share
这次说说经常被提及的一个算法 - 动态规划

[动态规划之初识动规](./动态规划之初识动规.md)