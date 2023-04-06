> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 332. Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/)

**题目解析**：

输入是一堆飞机票，飞机票上有起始地和目的地，这些飞机票都属于一个人的一次的旅行，让你重构这个人的乘机顺序，题目规定的一开始的起始地点。另外可能有多个答案，返回按字符串大小排序最小的那个答案即可。

一开始看这道题目，直觉反应就是一道图相关的题目，并且这里的对应概念很清晰，**每个地点代表的是图上的节点，每次飞行代表的是图上的边**。而且这些信息都可以从输入条件中获取。**我们需要做的是找到一条路线，把图上的所有点和边都连起来**，但是这里就会有一个顺序的问题，当你到一个节点上发现有两条路可以选，那到底选那条？根据题意，我们需要选择按字符串大小排序小的那个，但有可能那条路是走不通的，比如下面这个例子：

```
[["JFK","KUL"],["JFK","NRT"],["NRT","JFK"]]
```

首先构建出图：


![](https://user-gold-cdn.xitu.io/2020/7/6/17320736a304d0b6?w=327&h=253&f=png&s=6752)

在 JFK 你有两条路可走，分别是到 KUL，还有 NRT，如果是按字符串大小进行选择，这里会选择 KUL，最后你发现这条路径并不能把所有的节点连在一起。

对于这种情况，我们可以采取的策略就是回溯。也就是说我们还是按照字母顺序进行考虑，但是如果发现路走不通，就回到之前的状态继续考虑其它的情况，比如上面的例子，我们在 KUL 发现路不对，走不通，就需要退回到 JFK 那里，重新考虑其它的可能。我们可以用深度优先搜索来很好的实现这么一个想法，在深度优先搜索之前将每个节点的相邻节点按字符大小排序，保证在遍历的时候我们优先考虑字符排序小的相邻节点，只要一找到路径我们就返回。

上面的是最常见的解法，但是我在 Discuss 里面看到了一个比较有意思的解法。这个解法同样是用深度优先搜索，但是并没有考虑回溯。其思想是，我们还是将每个节点的邻居按字符大小排序，优先考虑 “小” 的邻居，每考虑一个邻居就将这个邻居移出该节点的邻居集合，每到一个节点，如果发现这个节点已经没有相邻的节点，那么就把这个节点加入到答案中，最后将答案集合反转就是我们要的最终答案。

这个解法之所以可行，在于我们不断地将边和节点移出图，也就是说这里的图是动态变化的，**如果一个节点已经没有邻居节点，则表示该节点在当前图上会是最后一个遍历到的节点。** 我们先加入答案的是最后遍历到的节点，那么最后我们就需要反转答案。可以结合例子来看会更清晰。

<br>

**参考代码**：
```go
func findItinerary(tickets [][]string) []string {
    m := make(map[string][]string, len(tickets)+1)

    // 构建有向图
    for _, t := range tickets {
        m[t[0]] = append(m[t[0]], t[1])
    }

    // 对每个节点的邻居排序，保证优先考虑字符排序小的节点
    for k := range m {
        sort.Strings(m[k])
    }

    ans := []string{}
    
    // 深度优先搜索遍历
    DFS("JFK", m, &ans)

    // 反转答案数组
    i, j := 0, len(ans) - 1
    for i < j {
    	ans[i], ans[j] = ans[j], ans[i]
    	i++
    	j--
    }

    return ans
}

func DFS(start string, m map[string][]string, ans *[]string) {
    // 如果当前节点还有邻居
    // 表明此时的遍历并不会在这里结束，继续遍历
    for len(m[start]) > 0 {
        cur := m[start][0]
        // 每选择一条路径，就将对应的边移出图
        // 因为如果图上包含环的话，这样做可以避免无限循环导致栈溢出
        m[start] = m[start][1:]
        DFS(cur, m, ans)
    }

    // 如果当前节点没有邻居
    // 表明此时的遍历结束，将当前节点加入答案
    *ans = append(*ans, start)
}
```

<br>

## Review

[Using DSH (Distributed Shell) to Run Linux Commands Across Multiple Machines](https://www.tecmint.com/using-dsh-distributed-shell-to-run-linux-commands-across-multiple-machines/#:~:text=DSH%20is%20short%20for%20%E2%80%9CDistributed,can%20obtain%20the%20source%20at.)

这篇文章主要是介绍一个 Linux 下的指令 `dsh`

dsh 的全称是 “Distributed Shell” 或是 “Dancer's Shell”。dsh 主要用于执行一些远程的 shell 命令，有些时候我们需要在多个 remote 机器上执行我们的 Linux 命令，当然你可以 ssh 进到每个机器下执行命令，但是这样及其不方便，特别是在分布式的场景下，我们有大量的机器需要执行脚本命令，`dsh` 在这种情形下就会发挥作用。

在 ubuntu 下安装 `dsh` ，只需要一个指令：

```
>$ sudo apt-get install dsh
```

`dsh` 的主要环境配置在 `/etc/dsh/dsh.conf` 中，我们需要确认在配置文件中 `remoteshell =ssh`，如果你的对应配置是 `remoteshell =rsh`，就需要改过来，因为 rsh 是一个不加密的协议。

另外，在 `/etc/dsh` 目录下你会找到一个 `machines.list` 的文件，该文件主要是用来指定机器的 IP 或者是 Hostname。

说了这么多，那么 dsh 具体怎么用呢？文中举了下面这个例子：

```
>$ dsh –aM –c uptime
```

`-a` 表示的意思是在 `machines.list` 上面所有的机器上执行命令。`-M` 指明机器名也要和命令的执行结果一起返回。`-c` 表示的是要执行的命令。`uptime` 是要在机器上面执行的命令。

除了这些，我们还可以指定机器分组。在 `/etc/dsh/groups/` 目录下，创建一个文件，这个文件的名称就是组的名称，你可以在这个文件中加入机器，就和前面的 `machines.list` 的格式一样，比如下面的指令就是在名为 `cluster` 的组的机器上运行 `w` 指令并返回机器名称：

```
>$ dsh –M –g cluster –c w
```

另外关于 `dsh` 的其它用法，你也可以参照这个 [网站](http://manpages.ubuntu.com/manpages/bionic/man1/dsh.1.html)

<br>

## Tip

这周学习了 CSS 的布局，首先分两个部分来说说：

* **CSS 布局有什么用？**

  我们知道 CSS 中每个 HTML 的元素都是一个方盒子，这个盒子有一些属性，比如 padding，border，margin，width，height 等等，CSS 主要目的就是设置这些属性的值。那么每个盒子具体出现在屏幕或者是浏览器的那个位置呢？还有盒子与盒子之间是一个什么样的间距和作用关系？这些都需要 CSS 布局来实现，CSS 布局主要用来定义 HTML 元素出现的位置，可以说是一个大致的展现结构，有了这个结构，我们就可以把具体的盒子放到结构中的某个位置。

* **CSS 布局的实现：**

    先把 HTML 和 CSS 的代码以及结果图贴上：
    
    HTML
    ```html
    <section class="grid-test">
        <div class="row">
            <div class="col-1-of-2">
                Col 1 of 2
            </div>
            <div class="col-1-of-2">
                Col 1 of 2
            </div>
        </div>
    
        <div class="row">
            <div class="col-1-of-3">
                Col 1 of 3
            </div>
            <div class="col-1-of-3">
                Col 1 of 3
            </div>
            <div class="col-1-of-3">
                Col 1 of 3
            </div>
        </div>
    
        <div class="row">
            <div class="col-1-of-3">
                Col 1 of 3
            </div>
            <div class="col-2-of-3">
                Col 2 of 3
            </div>
        </div>
    
        <div class="row">
            <div class="col-1-of-4">
                Col 1 of 4
            </div>
            <div class="col-1-of-4">
                Col 1 of 4
            </div>
            <div class="col-1-of-4">
                Col 1 of 4
            </div>
            <div class="col-1-of-4">
                Col 1 of 4
            </div>
        </div>
    
        <div class="row">
            <div class="col-1-of-4">
                Col 1 of 4
            </div>
            <div class="col-1-of-4">
                Col 1 of 4
            </div>
            <div class="col-2-of-4">
                Col 2 of 4
            </div>
        </div>
    
        <div class="row">
            <div class="col-1-of-4">
                Col 1 of 4
            </div>
            <div class="col-3-of-4">
                Col 3 of 4
            </div>
        </div>
    </section>
    ```
    
    <br>
    
    SCSS
    ```scss
    $grid-width: 114rem;
    $gutter-vertical: 8rem;
    $gutter-horizontal: 6rem;
    $color-primary: #55c57a;
    
    @mixin clearfix {
        &::after {
            content: "";
            display: table;
            clear: both;
        }
    }
    
    .row {
        max-width: $grid-width;
        margin: 0 auto;
        
        &:not(:last-child) {
            margin-bottom: $gutter-vertical;
        }
        
        @include clearfix;
        
        [class^="col-"] {
            float: left;
            background-color: $color-primary;
            &:not(:last-child) {
                margin-right: $gutter-horizontal;
            }
        }
        
        .col-1-of-2 {
            width: calc((100% - #{$gutter-horizontal}) / 2);
        }
    
        .col-1-of-3 {
            width: calc((100% - 2 * #{$gutter-horizontal}) / 3);
        }
    
        .col-2-of-3 {
            width: calc(2 * ((100% - 2 * #{$gutter-horizontal}) / 3) + #{$gutter-horizontal});
        }
    
        .col-1-of-4 {
            width: calc((100% - 3 * #{$gutter-horizontal}) / 4);
        }
    
        .col-2-of-4 {
            width: calc(2 * ((100% - 3 * #{$gutter-horizontal}) / 4) + #{$gutter-horizontal});
        }
    
        .col-3-of-4 {
            width: calc(3 * ((100% - 3 * #{$gutter-horizontal}) / 4) + 2 * #{$gutter-horizontal});
        }
    }
    ```

    <br>
    
    结果图展示：
    
    
    ![](https://user-gold-cdn.xitu.io/2020/7/6/17320c33e80b789b?w=1171&h=530&f=png&s=19267)
    
    上面的代码我们是可以在不同的前端项目中重复利用的，布局的原理也很简单，就是根据整个屏幕的宽度，还有盒子的宽度以及盒子间隔的宽度计算得到。这里的重点是 CSS 中的 `calc` 函数，还有我们定义的 `clearfix` 函数，`calc` 允许你在 CSS 中根据屏幕大小计算，`clearfix` 的目的在于消除 float 的影响，如果你在 CSS 中 float 了一个元素，那么这个元素很可能会失去父节点的一些属性，`clearfix` 可以让元素继续继承父节点的一些属性。

<br>

## Share

这次分享极客时间上面的一篇文章：

[职业规划 ：转管理是程序员的终极选择吗？](https://time.geekbang.org/column/article/249391)

在职场中，基本可以分为两中岗位，个人岗位和管理岗位。我们中的大多数可能都是从个人岗位干起的，毕竟相比于个人岗位，管理岗位只是少数群体。但是我们往往都会羡慕管理岗位上的人，这个不难理解，一般来说，管理岗位有更好的报酬，也有更大的权利，在管理岗的人，有更加灵活的自主决策权，不用完全听命于别人。这些都是管理岗位相比于个人岗位的好处，或者是优势所在。但你有没有想过管理岗位所要背负的责任呢？就拿一个部门经理举例，对内他要管理团队中的成员，特别是绩效管理，要做到公平，这样别人才会踏踏实实跟着你干，对外他要协调与各个部门的关系，为团队争取更多的资源，还要敢于做出决策和承诺。另外团队中的成员遇到的问题他都要进行考量和权衡，也就是说他必须对公司的业务和发展方向有明确的认识。说了这么多，管理岗位的最终目的是让别人，或者说是手下的人能够干的出色，这并不是说只要你有熟练的技术就足够的，另外管理岗位身上的责任也会很大，如果是个人岗位，我们需要做的可能就是把手头分发下来的任务做好，我们自己对自己负责即可。但是管理岗位的人不光要对自己负责，还要对团队中的成员负责，有些时候很多东西并不是自己能控制的了的。

到底要不要转管理，当然是要，毕竟不试一下怎么知道自己合不合适。但是还有一个问题是 “什么时候转？”，就我而言，我觉得把基本的技术学扎实了，有了自己的主攻方向，并在该方向有所建树。到了这个时候，如果我觉得技术已经掌握的差不多了，可能就会选择管理岗试试看。如果发现管理岗位不适合自己，全身而退也未尝不可，毕竟有扎实的技术功底在，不愁在这个行业中混不下去。与其说争着进管理岗位，不如安安稳稳花个 10 年甚至是更长的时间沉淀自己，多积累些知识，不止是技术上的。等时机到了，该有的自然会有。