> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 1353. Maximum Number of Events That Can Be Attended](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended/)

**题目解析**：

题目给定一个二维数组，数组表示的是每个事件的起始和终止时间（单位：天），每天你只能参加一个事件，问你最多可以参加多少事件。

这道题目和之前的一道 [meeting room II](https://juejin.im/post/5d83f94a5188255457503cf5) 的题目极其相似，但是直接硬套原来的解法又貌似不可行。原因在于 meeting room 求解的是最多有多少会议在时间上冲突，这样我们就可以安排最少的会议室来保证所有会议的正常进行。而 **这道题并不是问有多少事件有冲突，而是让你选择参加事件，保证能参加尽可能多的事件。** 虽然，两道题目上有这样的区别，但是我们依然可以借助 meeting room 的解法来解这道题目。

回顾一下，meeting room 问题中，我们首先是将会议按 **起始时间** 进行递增的排序，然后我们从头到尾遍历这个排序好的数组，在遍历的过程中，使用了一个 **优先队列（最小堆）** 存放会议的结束时间，当一个会议来了，我们首先去看队列中是否有会议结束，如果有，那么说明不需要增加额外的会议室，如果没有，那么我们就要为当前会议额外安排会议室。优先队列的有序性保证了我们每次取到的都是最快结束的那个会议，遍历到最后直接输出优先队列的大小就是答案。这里的重点是 **优先队列中保存的东西究竟是什么？** 如果是仅仅认为优先队列中存放的是 “会议的结束时间”，那么理解就仅仅停留在表面，我们需要更进一步的理解。我们可以理解为优先队列中存放的其实是**正在进行的会议**。我们可以试着用这个理念来解这道题。

回到这道题目，如果我们也使用同样的思路，排序数组，用一个优先队列来存放正在进行的事件，我们每天都可以去参加事件，这里有一个 **贪心的思想**，因为我们每天只能参加一个事件，**如果有多个事件可以参加，我们需要保证我们参加的是最快会结束的那个事件**，这个想想应该不难理解。我们每天都去看是不是有事件可以参加，在这一天有新的事件开始，我们也需要将其结束时间加入到队列中，同时如果有过期事件，我们也需要将其移出队列，队列中始终保存的是 “有效的事件”，这样下来，我们就可以得到最后的答案。**这里和之前最大的不同在于，我们遍历的是天数而不是事件**，除此之外，其余的地方都一样。这么下来，时间复杂度就是 `O(Dlogn)` 其中，D 是最大的天数（由最后结束的事件决定），n 是事件的个数。


<br>

**参考代码**：

```java
public int maxEvents(int[][] events) {
    int n = events.length;

    int lastDay = 0;

    // 获得 D
    for (int i = 0; i < n; ++i) {
        lastDay = Math.max(lastDay, events[i][1]);
    }

    PriorityQueue<Integer> pq = new PriorityQueue<>();
    
    Arrays.sort(events, (a, b) -> (a[0] - b[0]));

    int curEvent = 0;
    int res = 0;

    // 遍历天数
    for (int day = 1; day <= lastDay; ++day) {
        // 如果在当前天有新的事件开始
        // 将其结束时间加入队列，表示这是正在进行的事件
        while (curEvent < n && events[curEvent][0] == day) {
            pq.offer(events[curEvent++][1]);
        }

        // 将过期的事件移出队列
        while (!pq.isEmpty() && pq.peek() < day) {
            pq.poll();
        }

        // 参加事件
        if (!pq.isEmpty()) {
            pq.poll();
            res++;
        }
    }

    return res;
}
```

<br>

## Review

[How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)

这篇文章讲述如何写好一个 commit message，有些人可能会说，commit message？随便写写不就好了。但相信你看完了这篇文章，你的观点会有所改变。

我们很清楚地知道，`git log` 给我们展现的是一个代码仓库，说白了就是一个项目的开发历程，好的 commit message 能够清晰地告诉别人你的开发进度以及过程，别人不需要去读你的代码就能够大概知道项目的一个大致发展情况，好的 commit message 能够让你在短短的几小时，甚至几分钟就能概览完别人几个月甚至几年的编程历程，可见写好 commit message 能够大大提高我们的开发效率。另外来说，我们总是强调命名，代码风格，目的都是让人能够快速理解，加快合作开发的效率，写好 commit message 也不例外。

文章给了 7 点建议帮助你写出简洁、易懂、规范的 commit message，我们分别来看看
* Separate subjuect from body with a blank line
  
  一个 commit message 其实包含两部分的，一个是 subject，另一个是 body，你可以类比电子邮件。通常在 subject 中，我们只用简洁的一句话描述这个 commit 做的事情，这就好比邮件的标题，在 body 中我们会详细地讲解 commit 做的具体的事情。这两部分最好是用空行隔开，这样我们使用 `git log --oneline` 或是 `git shortlog` 等类似工具的时候，subject 部分会被清晰地展现出来，而省去 body 部分的内容，这就和你在邮箱首页浏览所有邮件一样，**展现标题，忽略细节**。

* Limit the subject line to 50 characters
  
  我们要尽可能地使 subject 简洁，这里的 50 个字符仅仅是一个提醒，并不是一个严格的规定，你可以把其定为最终的目标，但不用纠结于此。如果你不能用简短的一句话来填写 subject，那么就说明你的这一次的 commit 过于庞大，需要拆分，**尽量保证一个 commit 解决一个问题，这就像编程中的单一职责原则一样**。

* Capitalize the subject line

  subject 的首字母需要大写，这个其实可以看作是一个风格上面的约定，约定好了就遵守，坚持下来，这样可以减少不必要的误解和猜测。
  
* Do not end the subject line with a period
 
  subject 省略句号，本来字符数量就有限定，在这说来，类比邮件，一般标题都没有句号

* Use the imperative mood in the subject line

  使用命令的口吻去写 subject，这一点其实很重要，如果遵守了，commit message 看上去会比较的一致，而且我们也可以相对容易地理解 commit 所解决的问题，命令性的口吻就是不带任何人称的语句，你可以参考下面的例子：
  
  * clean the room
  * close the door
  * delete the code
  
  其实 git 本身也是这样做的，你可以看到一些常见的 git 指令的输出内容：
  ```
  Merge branch '...'                    # git merge
  Merge pull request #... from ...      # github pull request
  Revert "..."                          # git revert
  ...
  ```
  要想写出 imperative mood 的 subject，文章还给了一点建议，就是看 subject 的内容是否能填入句子 `If applied, this commit will <your subjuect here>`，如果填入后，读起来顺畅，正常，那么说明你的 subject 就是 imperative mood 的
  
* Wrap the body at 72 characters

  通常，我们如果要给 commit message 写 body 的话，需要用到编辑器。但是编辑器对于一行的字数往往并没有要求，在填写 body 的过程中，如果我们不强制换行，则写出来的 body 的 format 会严重影响到 message 的展示，这一点也是为了更好的展示
  
* Use the body to explain what and why vs. how

  body 用来解释这个 commit 做的是什么，为什么要做，以及是怎么做的。通常这三点的解释能够回答大多数的疑问，如果是改动的话，你可以指出之前是怎样的，存在什么样的问题导致你想去做这样的改动，改动之后，也就是现在是怎样的？写出了这些信息，相信别人阅读了你的代码后将不会来麻烦你，反倒是心底默默感谢并赞美你。

一个小小的 commit message 也有这么多的学问，看来一个东西的价值需要深究。

<br>

## Tip

项目当中要用到 golang, 这几天在恶补，记录一下：

首先是一些 Go 中和其他语言不同且容易出错的地方：
* Go 里不允许任何的隐式类型转换
* Go 有指针但是不支持指针运算
* string 在 Go 中是值类型，其默认的初始化值为空字符串，而不是 nil
* Go 语言没有前置的 `++` `--`
* 仅支持 `for` 循环关键字
* 条件语句支持变量赋值

数组的声明：
```go
var a [3]int     			    // 声明并初始化为默认零值
b := [3]int{1, 2, 3}		    // 声明同时初始化
c := [2][2]int{{1, 2}, {3, 4}}	// 多维数组初始化
```

Go 中的数组是静态的，数组之间是可以比较的。但是 Go 中还支持一个结构，叫做 **切片**，其实就是动态数组，它的定义和数组极其相似，区别在于没有给定 size：

```go
a := []int{1, 2, 3, 4}
b := []int{1, 2, 3, 4}
// if a == b { //切片只能和nil比较
// 	t.Log("equal")
// }
```

Map 的声明：
```go
m := map[string]int{"one": 1, "two": 2, "three": 3}
m1 := map[string]int{}
m1["one"] = 1
m2 := make(map[string]int, 10 /* Initial Capacity */)
```

Map 中的元素不存在也会返回 0 值，判断一个元素在 Map 中存在的方法：
```go
if v, ok := m1[3]; ok {
    t.Logf("Key 3's value is %d", v)
} else {
    t.Log("key 3 is not existing.")
}
```

Map 的遍历：
```go
m := map[string]int{"one": 1, "two": 2, "three": 3}
for k, v := range m {
    ...
}
```

函数的一些注意事项：
* 可以有多个返回值
* 所有参数都是值传递：slice、map、channel 会有传引用的错觉
* 函数可以作为变量的值
* 函数可以作为参数和返回值

可变参数：
```go
func sum(ops ...int) int {
    s := 0
    for _, op := range ops {
    	s += op
    }
    return s
}
```

defer 函数 - 类似 finally，在函数最后返回的时候执行：
```go
func TestDefer(t *testing.T) {
    defer func() {
    	t.Log("Clear resources")
    }()
    t.Log("Started")
    panic("fatal error")  // defer 仍然会执行
}
```

<br>

## Share

这期来说一说自己为什么要成为一个程序员，面临哪些挑战，怎么坚持下去

[一个码农的心路历程](./一个码农的心路历程.md)
