> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 450. Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/)

**题目解析**：

在二叉搜索树上删除一个节点，这道题目有一个隐含的条件，就是树上节点的值不重复。另外题目还要求时间复杂度需要保证 `O(h)` 这里的 h 表示的是二叉树的高度。

**其实这个题目是分成两个步骤的，第一个是找到对应的节点，第二个是删除节点**。因为是二叉搜索树，对于树上每个节点来说，其右子树的节点都要大于其左子树的节点，那么要找对应节点，我们可以从根节点开始，一路比较，大的话就去右边找，小的话就去左边找，这样每次我们都往下，可以保证时间复杂度是 `O(h)`。当我们找到了要删除的节点，在删除这一步就会有很多的细节，主要是因为我们需要调整余下的结构，以维持二叉搜索树的性质。

针对这个问题，我们可以分情况讨论：
```
        5
       / \
      3   6
     / \   \
    2   4   7
   /         \
  1           8
 
情况 1：当删除的节点没有左右子树，比如上图的 4、8、1
       这时直接删除即可，树依旧可以保持二叉搜索树的性质
 
情况 2：当删除的节点有左子树没有右子树，比如上图的 2
       这时我们只需要将整个左子树移到当前位置即可
       也就是将左子树的根节点放到删除节点的位置，其余不变

情况 3：当删除的节点没有左子树有右子树，比如上图的 6、7
       这时我们只需要将整个右子树移到当前位置即可
       也就是将右子树的根节点放到删除节点的位置，其余不变

情况 4：当删除的节点既有左子树又有右子树，比如上图的 5、3
       这时就有两种方法供选择：
           去到左子树中，找到值最大节点，将右子树全部移到这个节点下
           去到右子树中，找到值最小节点，将左子树全部移到这个节点下
```

通过上面的讨论分析，我们有了大致的思路。在实现方面，我们可以借助递归来巧妙地达到删除对应节点的目的。

<br>

**参考代码**：

```java

public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) {
        return null;
    }
    
    // 当前遍历到的节点大于要找的节点，去左边继续找
    if (root.val > key) {
        root.left = deleteNode(root.left, key);
    }
    // 当前遍历到的节点小于要找的节点，去右边继续找
    else if (root.val < key) {
        root.right = deleteNode(root.right, key);
    } 
    // 找到要删除的节点，进行删除操作
    else {
        // 情况 1 & 2
        if (root.right == null) {
            return root.left;
        } 
        
        // 情况 3
        if (root.left == null) {
            return root.right;
        }
        
        // 去到删除节点的右子树，找到值最小的节点
        TreeNode rightSmallest = root.right;
        while (rightSmallest.left != null) {
            rightSmallest = rightSmallest.left;
        }
        
        // 将删除节点的左子树全部移到这个节点下
        rightSmallest.left = root.left;
        
        // 返回右子树的根节点，放到当前删除节点的位置
        return root.right;
    }
    
    return root;
}
```

<br>

## Review

[The Powerful Differences Between Good and Great Programmers.](https://levelup.gitconnected.com/the-powerful-differences-between-good-and-great-programmers-276f6d5bed52)

这篇文章主要是在讲好的程序员和伟大的程序员之间的区别。通常来说，好的程序员的评价标准比较直观，比如说能够写出简洁优雅的代码、有扎实的代码以及逻辑功底、对编程有热情、乐于帮助周围的同事等等。但是这些只能让你成为一个好的程序员，它并不能让你成为一个伟大的程序员。

想要成为一个伟大的程序员，除了上面这些，你还需要一些额外的特质。

* 有极强的学习能力

  伟大的程序员会乐于去学习并了解一些新的技术。自学能力非常的强，懂得如何去问问题，问对的问题，问对的人。

* 能够很好地权衡实用与完美

  计算机这个行业中，很多问题都是需要权衡，需要取舍的。没有绝对的好，也没有绝对的不好，一个技术用在某个特定的场景下，很好，但是用在一些其他的场景就不如其他的技术。伟大的程序员能够在实用和完美之间做好取舍与权衡，理性分析问题，做对的事情，不走极端。

* 有很好的直觉

  往往看待一个问题，在有理性的分析之前，我们先会有一个直觉上的认知。这可以算是认识问题并解决问题的一个突破口。伟大的程序员通常在看到问题的蛛丝马迹后，就能够在脑袋里面对问题有个合理的猜测，并且心里也会针对不同的预期，准备相应的应对方案。这种直觉是超越编程之外的一种思维模式，习得这种能力不仅需要平时的积累，而且还跟人的一些特质有关。

* 有非常好的沟通能力

  仅仅是沉浸到代码中，写出好的代码并不能够让其他人认识到你做的东西的优势。想让别人，或者说是一般的人理解并认同你，你还需要有好的沟通能力。伟大的程序员往往言简意赅，能够用简洁而直白的语言让周围的人懂得一些不容易解释的东西。

你可以看到，上面列出来的这几项其实并不会与特定的技术，或者是成就挂钩。没错，一个程序员是否伟大，并不能仅仅通过他的薪水、职位高低、甚至是编码能力来决定，更看重的往往是态度方面的问题，**如果一个程序员能够积极主动去学习，提高自己，刻意地去通过自身或者外界的反馈来改变自己的认知，尝试着用自己学到的东西努力去解决一些问题，带动着周围的人也一起积极地看待问题，那么你也可以收获伟大**。


<br>

## Tip

[Git Learn Branching](https://learngitbranching.js.org/) 是一个学习 Git 的很好的平台，这里通过互动界面让你对 Git 当中的一些常见用法有更深入的认识。虽然说用 Git 已经很久了，但是跟着走了一遍后，还是觉得自己知道的太少，这里做个学习记录吧：

* 整个 Git 仓库就是一个树结构，Branch 其实就是指针，当你在 Git 中创建了一个 branch 其实就是创建了一个指向特定 Commit 的指针，最新产生的 Commit 就是这个树的一个叶子节点。认识到这一点很重要，因为这可以说是整个 Git 的基础
* **Rebase**：`git rebase` 作用是拷贝一些 commit 去到另外一个地方，相比于 `git merge`，它能够更好地维持 branch 的整洁，常见用法如下：
    ```
    # 将 branch2 的不同部分移到 branch1 上
    # branch1 的位置不发生改变，branch2 的位置发生改变
    # 如果当前位置已经在 branch2 上了，那么 branch2 可以省略
    >$ git rebase branch1 branch2  
    
    # 将当前的 branch rebase 到 origin/master 分支上（表示 remote 的 master 的分支）
    >$ git rebase origin/master
    
    # -i 可以让你任意选择之前 commits，也可以置换这些 commits 的前后顺序
    # 这里的 master 也可以用特定的 commit 来代替
    >$ git rebase -i master
    ```
* **detaching head**：经常在 Git 的提示中经常看到这个名词，它表示的意思是 HEAD 当前在一个特定的 Commit 上，而不在某个 branch 上。
* **relative refs**：两个便捷的符号 ^ 和 ~，使用它们可以让命令更简洁
    * ^ : 表示某个 commit 的父 commit
    * ~ : 也是表示某个 commit 的父 commit，但是后面可以接数量
    ```
    比如下面是一个仓库的 Git 记录:
             C0
             |
             C1
             |
             C2
             |
             C3 <- (master, HEAD)
    
    # 将 master 移动到 C0
    >$ git branch -f master HEAD~3 
    ```
* **cherry-pick**：复制一系列的 commits 到你当前（HEAD）的位置下，这可以算是 Git 当中非常灵活的一个指令，有些时候可以替代 rebase
    ```
    >$ git check-pick <c1> <c2> ...
    ```
* **tag & describe**: 这其实是两个指令，tag 就是针对某个 commit 做个标记，因为 commit hash 太难记了，为了突出某个 commit，我们可以给它起个醒目的名字。但是注意，我们并不能直接通过 checkout tag 来移动 HEAD。describe 是将离当前位置最近的 tag 信息给打印出来，打印出来的格式是 `<tag>_<numCommits>_g<hash>`，我们可以通过 describe 指令拿到最近的 tag 的 hash，然后就可以 checkout 过去了。
* **remote branch**：格式是 `<remote name>/<branch name>` 通常这里的 `<remote name>` 会是 `origin`，但是注意的是，在本地我们不能够直接 checkout remote branch，不然会 detaching HEAD
* **fetch**：
    * 关于 `git fetch` 的三点重要认知：
        * 从 remote 下载 local 缺失但是 remote 存在的 commits
        * 会更新 local 上 remote branch 的位置
        * 不会改变 local （branch）的状态
    * fetch 的高级用法
        * `git fetch <remote name> <remote branch name>` 下载 remote 上特定的 branch 上的信息
        * `git fetch <source>:<destination>` 下载 remote 上特定的 branch 上的信息到本地特定 branch 上去
* **push**: 同 `fetch` 相类似，`git push` 也有一些高级用法：
    * `git push <remote name> <remote branch name>` push 到 remote 特定的 branch 上去
    * `git push <source>:<destination>` push 本地特定的 branch 到 remote 特定的 branch 上去
* **diverged history**：每当你 push commit 之前，你需要确保 local 和 remote 保持同步，这里有四个常见操作，其中 1、3 等同（rebase），2、4 等同（merge）：
    1. `git fetch` + `git rebase origin/master` + `git push`
    2. `git fetch` + `git merge origin/master` + `git push`
    3. `git pull --rebase` + `git push`
    4. `git pull` + `git push`
* **remote tracking**：举个例子，local 上 `origin/master` branch 是用来表示 remote 上的 master。我们也可以创建其他的 branch 来和 `origin/master` 关联起来，这样 `fetch`、`pull`、`push` 操作可让相关联的 branch 自动移动。这里注意，我们前面也说过，`fetch` 操作是不会改变 local 的状态的。有两个指令可以帮我们关联 branch：
    * `git checkout -b branch <remote name>/<branch name>`
    * `git branch -u <remote name>/<branch name> <local branch>`

当然 Git 远不止这些，还有 `git stash` 之类的指令，但是这些是最常用到的一些指令

<br>

## Share

继续 Clean code 之旅

[Clean Code 之旅 - 异常、边界、单元测试](./CleanCode之旅-异常、边界、单元测试.md)
