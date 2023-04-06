> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 967. Numbers With Same Consecutive Differences](https://leetcode.com/problems/numbers-with-same-consecutive-differences/)

**题目解析**：

这道题题目的意思没有特别的明确，需要仔细读题才能理解。题意是让你找符合条件的所有整数，这些整数的位数为 N，并且每一位与相邻位的绝对值为 K。举个例子，输入条件是 N = 3，K = 7，那么 `707` 就是一个答案，`818`，`181` 也算是答案，但是左边打头的一位不能是 0，比如 `070` 就不是答案。

理解题意之后，你能明显感觉到，每个答案需要我们一位一位地去构建。另外这个题还有一个特征，就是当你确定了最左边的那一位上的值后，后面的位就可以顺推。当然，你确定最右边位上的值也可以反向推，但是这里题目规定最左边位上不能为 0，因此从最左边开始构建，程序相对来说会比较好实现。确定了一位，推导下一位无非有两种情况，比当前位上的值大 K，还有就是比当前位上的值小 K，另外对位上的值也有限制，不能超过 9，也不能小于 0。知道了上面的这些后，剩下的就是去实现一个递归函数。

实现的时候只需要注意两点即可，当构建的整数的长度等于 N 的时候，我们就可以把其加入到答案中去，另外就是需要特殊考虑 N = 1 这样的特殊情况。


<br>

**参考代码**：
```java
public int[] numsSameConsecDiff(int N, int K) {
    Set<Integer> set = new HashSet<>();

    if (N == 1) {
        set.add(0);
    }

    for (int i = 1; i <= 9; i++) {
        helper(set, i, K, N, 1);
    }

    int[] result = new int[set.size()];
    int index = 0;
    for (int ele : set) {
        result[index++] = ele;
    }

    return result;
}

private void helper(Set<Integer> result, int cur, int K, int N, int len) {
    if (len == N) {
        result.add(cur);
        return;
    }

    int lastDigit = cur % 10;

    if (lastDigit + K < 10) {
        helper(result, cur * 10 + lastDigit + K, K, N, len + 1);
    }

    if (lastDigit - K >= 0) {
        helper(result, cur * 10 + lastDigit - K, K, N, len + 1);
    }
}
```

<br>

## Review

[Pro Git CH3](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)

这周继续 Pro Git，这一章主要讲的是分支。和其它的版本控制工具相比，分支可谓是 Git 的杀手锏，Git 支持很多其它版本控制工具所不能支持的分支特性。

### commit 结构

首先来看看 commit 的在实现上的一个大致结构：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a627f7f6bcd7484fb7834838aa625beb~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b39fbbb9f6d48e5bedb372c3176d32f~tplv-k3u1fbpfcp-zoom-1.image)

第一张图显示的是一个 commit 的结构，可以看到这个结构有点类似于树，首先开始的 commit 节点上存放的是一些元信息，比如 commit 的作者，commit message，还有 commit 创建的日期等等。这个节点指向的是一个快照节点，这个节点不直接存放代码信息，而是存放该 commit 对应的 blob，这里解释一下，blob 是最小储存单元，上面存放着代码片段，可能一个 commit 会由多个代码片段组合而成。

第二张图显示的是 commit 之间的关系，每一个 commit 指向的是之前的一个 commit。你可能会说，commit 之间的关系有点像链表，在这张图上看是没错，但是我们加上分支的话就不是了，它更像是一个树。

### 分支特性

了解了 commit 的结构后，我们现在来说说分支。**分支其实就是指向特定 commit 的一个指针**。Git 当中的分支特别的轻便，占用的空间仅仅是 41 个 Bytes。因此，我们可以很自由地删除，创建分支，但凡你对其它的版本控制工具有所了解的话，你就会知道其实这个其实并不容易。

说到分支，离不开的就是 **分支合并**，意思就是把两个分支上的内容整合在一起。这里有两种情况：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb41b4dbbb8c4be59495c79b077beb9b~tplv-k3u1fbpfcp-zoom-1.image)

比如这里我们需要将 master 分支与 hotfix 分支进行合并，这里只需要将 master 对应的指针移到 hotfix 对应的 commit 上即可，这种又称作 **fast-forward**。这种合并因为只涉及指针的移动，因此会比较简单，而且并不会出问题。

另外一种是正常的合并，如下图：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99fc57d8d3a64c1589322a5d53b23715~tplv-k3u1fbpfcp-zoom-1.image)

这里我们需要将 master 与 iss53 进行合并。合并的时候 Git 会去比较 3 个节点上的内容，一个是两个分支的公共 commit 节点（C2），另外的是两个分支上最新的 commit 节点。比较完这 3 个节点上的内容后，Git 会新生成一个 commit 节点，这个节点上保存的就是两个分支整合后的内容。当然，分支合并最常见的问题就是合并冲突，也就是说两个分支上的最新 commit 都修改了同一处的内容，这个时候就需要先去把冲突解决后，在进行合并。

另外关于分支合并，Git 除了支持 merge 之外，还支持 **rebase**。两者的区别在于，rebase 在合并后并不会保存合并前的分支信息：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7eafd7138a29405fb337f9912e88d259~tplv-k3u1fbpfcp-zoom-1.image)

至于哪个更好，该使用那种分支合并方式，这里存在着争论。rebase 的优点是让主分支显得更加的简洁，但是它毕竟移除了一些分支的历史记录。具体要使用哪一个还是看个人的习惯和感受吧。

关于 rebase，当 commit 已经存在于你的本地之外了，最好不要对这个 commit 使用 rebase 操作，否则会造成本地和远端不一致的情形，再多人协作的时候更是如此。

### 常见指令

关于分支的基本指令，这里就不说了，这里看几个比较有用的，但是平时不怎么了解的指令

* git log <branchName\>

  看一个分支下的 commit 日志

* git mergetool

  在分支合并的时候遇到冲突，这个指令可以生成一个视觉化的界面，带着你浏览一遍冲突的地方，在修改文件比较多，冲突比较多的时候比较有用

* git branch -v

  显示每个 branch 的最近的一次 commit 信息，如果要看远端的分支信息，可以使用 `git branch -vv`

* git --move <oldName\> <newName\> <br>
  git --set-upstream origin <newName\> 
  
  分支的重命名，一个是本地的，一个是远端的

* git push origin --delete <branch\>

  删除远端的分支

* git rebase --onto master <firstBranchName\> <secondBranchName\>

  要理解上面的指令的意义，可以先看下图：
  
  ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21dd1d96563e40ff89d12dead4158509~tplv-k3u1fbpfcp-zoom-1.image)
  
  这里我们需要将 client 分支 rebase 到 master 分支上去。因为这两个分支并没有公共节点，所以一般的 rebase 操作不适用，这里只需要加上一个 `--onto` flag 即可。完整的指令就会是：
  
  `git rebase --onto master server client`

* git pull --rebase

  等同于 `git fetch` + `git rebase`
  
  

  

<br>

## Tip

这次记录下，VIM 编辑器下和终端的交互方式。

在使用流行的 IDE 时，比如 VS Code 或是 Sublime。我们都习惯于在视窗的某个部分可以显示终端，这样在我们编写代码的同时也可以很方便地运行终端命令，并不需要切换界面。

其实 VIM 也是支持这一点的，**只需要在 VIM 的命令行模式下输入 terminal 或是 term 即可**。需要注意的是，当需要关闭打开的终端窗口时，我们需要通过指令 <C-W\>N 退出终端模式，这是终端会进入普通模式，我们可在命令行模式下使用 `!q` 将其关闭。

另外 VIM 下的终端是可以和编辑器交互的，这里大部分的命令都是以 <C-W\> 开头。比如下面这些：

* <C-W\>" 后面跟寄存器号，表示粘贴该寄存器的内容到终端里
* <C-W\> 后面跟 j, k 等可以跳转窗口
* <C-W\>: 相当于普通窗口中的 :，执行命令行模式的命令

只要 VIM 的基本指令运用熟练，VIM 也可以成为一个全面的 IDE，只是说需要一个熟悉的过程。

<br>

## Share

简单聊一下，程序员焦虑的话题

[程序员的焦虑](./程序员的焦虑.md)

