## Algorithm

[LC 128. Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)

**题目解析**：

题目直接明了，给你一个未排序的数组，让你从中找出一些元素，使这些元素能够组成最长 **连续的递增序列**，输出这个序列的长度，元素的先后没有关系，比如：
```
[100, 4, 200, 1, 3, 2]

可以找出 4, 1, 3, 2 组成连续递增序列 1, 2, 3, 4

输出这个序列长度 4
```
很直接的想法是把数组排序一下，然后遍历一遍就可以找到答案，但是这道题目的难点在于它限制时间复杂度为 O(n)，这样一来，排序这条路走不通。

这道题目其实有一个特征，就是这道题目隐含着 **连通性** 这个性质在里面，怎么讲？我们还是拿上面那个例子来举例：
```
[100, 4, 200, 1, 3, 2]

我们从左向右枚举数组里面的元素，你可以认为枚举过的元素是有效的：
................100    枚举第一个元素，此时有 1 个连通区域
...4............100    枚举第二个元素，第二个元素和前面的元素互不相连，此时有 2 个连通区域
...4............100............200   枚举第三个元素，三个元素互不相连，此时有 3 个连通区域
1..4............100............200   枚举第四个元素，四个元素互不相连，此时有 4 个连通区域
1.34............100............200   枚举第五个元素，这个元素和之前第二个连通区域相连，连通区域维持在 4 个
1234............100............200   枚举第六个元素，这个元素和两个连通区域相连，连通区域变成 3 个

最后包含元素最多的那个连通区域所包含的元素个数就是我们要的答案
```
知道了这些东西对我们解题有什么帮助呢？关于连通性的问题，首先要想到的一个数据结构就是 **并查集**，这个数据结构的设计初衷就是为了解决连通性的问题，而且它的两个操作，查找以及合并的时间复杂度可以近似看成是 O(1)，因此用来解决这道题目再适合不过了。

如果你能想到并查集，那么这道题目其实就没有更多的难点，但我想说的是，这道题目其实还有一个比较有趣的解法，是利用 HashMap 来记录边界点所涵盖的连通区块长度，还是跟着例子走一遍：
```
[100, 4, 200, 1, 3, 2]

我们还是从左向右枚举数组里面的元素，每次遍历都去看这个元素的左右是否存在，并更新 HashMap：
100   此时 99 以及 101 都没有任何区块，Map {100=1}，表示 100 这个区块大小为 1
4     此时 3 以及 5 都没有任何区块，Map {100=1, 4=1}
200   此时 199 以及 201 都没有任何区块，Map {100=1, 4=1, 200=1}
1     此时 0 以及 2 都没有任何区块，Map {100=1, 4=1, 200=1, 1=1}
3     发现 4 是存在的，4，3 形成一个新的区块，
      这个区块的左边界是 3，右边界是 4，区块大小是 2，
      Map 中更新边界元素所代表的区块大小，Map {100=1, 4=2, 200=1, 1=1, 3=2}
2     发现左右边界同时存在，1, 2, 3, 4 形成一个新的区块
      这个区块的左边界是 1，右边界是 4，区块大小是 4，
      Map 中更新边界元素所代表的区块大小，并记录当前元素避免重复访问，
      Map {100=1, 4=4, 200=1, 1=4, 3=2, 2=4}
```
可以看到，每次记录的时候，我们只需要保证区块的边界元素所表示的区块大小是正确的即可，至于区块中间的元素其实无所谓，因为这些元素并不会被再次访问到

这个方法其实挺巧妙的，通过利用哈希表的元素向左右延伸来确定区块的大小。

<br>

**参考代码（并查集）**：
```java
class Solution {
    // roots 用来记录一个连通区域的代表元素
    private Map<Integer, Integer> roots = new HashMap<>();
    
    // counts 用来记录一个连通区域的元素个数
    private Map<Integer, Integer> counts = new HashMap<>();
    
    private int find(int a) {
        if (roots.get(a) == a)  {
            return a;
        }
        
        int root = find(roots.get(a));
        
        // 路径压缩
        roots.put(a, root);
        
        return root;
    }
    
    private void union(int a, int b) {
        int rootA = find(a);
        int rootB = find(b);
        
        if (rootA != rootB) {
            roots.put(rootA, rootB);
            
            // 两个连通区域合并，更新整个区域的元素个数
            counts.put(rootB, counts.get(rootA) + counts.get(rootB));
        }
    }
    
    public int longestConsecutive(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        
        for (int i = 0; i < nums.length; ++i) {
            if (roots.containsKey(nums[i])) {
                continue;
            }
            
            roots.put(nums[i], nums[i]);
            counts.put(nums[i], 1);
            
            // 查看相邻元素是否存在连通区块
            if (roots.containsKey(nums[i] - 1) && roots.containsKey(nums[i] + 1)) {
                int root = find(roots.get(nums[i] - 1));
                
                // 左右都存在连通区域，合并这三个区域
                union(nums[i], root);
                union(root, roots.get(nums[i] + 1));
            } else if (roots.containsKey(nums[i] - 1)) {
                int root = find(roots.get(nums[i] - 1));
                
                // 左边存在连通区域，合并这这两个区域
                union(nums[i], root);
            } else if (roots.containsKey(nums[i] + 1)) {
                int root = find(roots.get(nums[i] + 1));
                
                // 右边存在连通区域，合并这这两个区域
                union(nums[i], root);
            }
        }
        
        int result = 1;
        
        // 遍历所有连通区块，找到包含元素最多的区块
        for (int i : counts.keySet()) {
            result = Math.max(result, counts.get(i));
        }
        
        return result;
    }
}
```
<br>

**参考代码（哈希表）**：
```java
public int longestConsecutive(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    
    Map<Integer, Integer> distances = new HashMap<>();
    
    int result = 1;
    
    for (int num : nums) {
        if (distances.containsKey(num)) {
            continue;
        }
        
        // 查找向左能够延伸的最长距离
        int left = distances.getOrDefault(num - 1, 0);
        
        // 查找向左能够延伸的最长距离
        int right = distances.getOrDefault(num + 1, 0);
        
        // 更新此时的左右边界所表示的区块大小
        distances.put(num - left, left + right + 1);
        distances.put(num + right, left + right + 1);
        
        // 数组中可能存在重复元素，记录当前元素，避免再次访问
        distances.put(num, left + right + 1);

        result = Math.max(result, left + right + 1);
    }
    
    return result;
}
```

<br>

## Review
[How to understand any programming task](https://www.freecodecamp.org/news/how-to-understand-any-programming-task-aea41eabe66e/)

这篇文章的主要目的是讲述如何理解需求并制定解决问题的计划，作者在文中列出了三个比较重要的步骤：
* 分析需求
  * 将需求归类
    * 这是分析需求的首个步骤，你需要明白你需要做的是 debug、或是设计新模块、亦或是提高软件运行性能等等，明白了这个之后，你才会有个大致方向
  * 总结需求
    * 用一到两句简洁的语句去说明这个需求，主要是让自己清楚这个需求到底是什么，简单来说就是我们有什么资源，需要产出什么样的产品
  * 列出需求之下的任务
    * 列出如果需要完成需求，我们需要做哪些方面的工作，这里不需要太细节，但是大致的轮廓还是要有
  * 思考实现这个需求的目的是什么
    * 往往作为程序员我们只在乎怎么做，如何做，不太习惯去思考为什么要做，但是想想为什么有这个需求，这个需求能解决什么样的问题，这其实有助于我们提升自我的认知维度，也能够帮助我们更好地理解需求
* 制定方案
  * 了解团队当中的 “术语”
    * 比如 DAO、IC、CQRS 等等，特别是大公司，有着非常多的技术代名词，首先你需要弄清楚这些名词所代表的意思，这样才能更好地去制定方案，或是和别人交流想法
  * 定义清楚如何才算完成
    * 怎么样才算做好了？每个人对这个定义都不一样，有些人认为做好意味着 test case，logger 都弄好了，有些人则认为程序能够解决问题就好了，如果这个地方不去弄清楚，也会存在信息不对称的情况
  * 验证方案是否可以解决问题
    * 经过前面的一些步骤，目前你应该有了比较清晰的方案，但是这里需要倒回去再看看你的这些方案能不能满足需求，会不会存在一些问题等等
* 学会辩证地思考
  * 明白何时发表反对观点
    * 在你弄清楚整个问题以及他人的观点之前，尽量不要发表不同的意见，因为这个时候你其实对整个问题，或是框架并不是特别清楚，发表反对观点只会把问题弄得更糟。
  * 明白如何发表反对观点
    * 查找别人方案的漏洞可以从方案是否符合逻辑、方案是否会造成信息缺失、以及方案是否和需求一致来查看，可以委婉地提出自己的反对意见，比如说 “I’m not disagreeing with you, I just don’t understand...”  **让别人从你诚恳的问题中得知自己的错误其实是一个非常好的提出反对观点的策略**。

<br>

## Tip

这周记录一下 npm 上面的两个在 Node JS 开发中经常使用的 libs：

[winston](https://www.npmjs.com/package/winston)

这是 Node 当中的一个配置 log 的套件，只需要通过简单的配置，log 就会被写到制定的 file 中，通常它还可以搭配 [winston-daily-rotate-file](https://www.npmjs.com/package/winston-daily-rotate-file) 以达到定期删除过期的 log file 的目的，文件有些误导，我这里自己试验了一下，是可以 work 的:
```javascript
const winston = require("winston");
const DailyRotateFile = require("winston-daily-rotate-file");
const { format } = winston;
const appRoot = require("app-root-path");

const {
  combine, timestamp, label, printf,
} = format;

const myFormat = printf(({
  level, message, label, timestamp,
}) => `${timestamp} [${label}] ${level}: ${message}`);

winston.createLogger({
  format: combine(
    label({ label: "send notification" }),
    timestamp(),
    myFormat,
  ),
  transports: [
    new winston.transports.Console({level: "info"}),
    new DailyRotateFile({
      filename: `${appRoot}/logs/apiLog_%DATE%.log`,
      datePattern: "YYYY-MM-DD",
      zippedArchive: true,
      maxSize: "20m",
      maxFiles: "7d",
    }),
  ],
});
```

还有一个是 Mysql ORM 相关的 lib：

[mysql](https://www.npmjs.com/package/mysql)

[mysql-backbone](https://www.npmjs.com/package/mysql-backbone)

前者主要用来做数据库 URL 的定义和链接，后者主要用来在 Node 当中定义数据库的 schema，以及在程序中对数据库进行 CRUD，这个和之前使用过的 mongoose 不太一样，update 操作步骤相对于 create 和 delete 来说要更多写，这种东西多尝试基本上就没有问题。

<br>

## Share
还是继续动态规划，上次写了序列型动态规划，这次在这个基础之上再递进

[动态规划之字符匹配类动规](./动态规划之字符匹配类动规.md)