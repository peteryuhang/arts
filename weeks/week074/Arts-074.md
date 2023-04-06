> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 528. Random Pick with Weight](https://leetcode.com/problems/random-pick-with-weight/)

**题目解析**：

题目给定一个权重数组，然后让你根据权重的大小随机返回数组中的一个元素，权重大的元素被返回的概率就大，一个元素被返回的概率等同于 `该元素的权重 / 总权重`。

这其实是一道非常经典的题目，还记得自己之前在面试的时候还遇到过这类题。它考察的点并不是复杂的逻辑思维，而是看你的编程以及思考问题的基本功，这类问题在实际的工作中也会时不时的出现，因此把它作为面试题是再正常不过的了。**在我看来，一道好的面试题能够测试出面试者可不可以把一个简单的问题，或者说是实际工作中的问题的方方面面给想清楚，解释清楚，并且实现出来没有严重的 bug，这就足够了**。扯远了，回到这道题目。如果想要根据权重来随机筛选元素，那么我们肯定是要把每个元素的权重和总的权重进行比较。最暴力的方法是根据权重，把所有元素被选中的概率全部都计算出来，然后再想办法去根据概率来筛选。这里有一个细节，概率的结果肯定不是整数，而是浮点数，有时除不尽还会有循环小数，当然了，你会说，用分数表示不就好了。不管怎么说，这样下去代码实现上面是比较困难的，而且搞不好求出来的还是近似解。

这里需要一个思维上的转变，我们可以尝试着去思考之前我们针对数组经常用的一个技巧，那就是**前缀和操作**。把前缀和操作运用到权重数组上，就会形成下面的一个新的数组：

```
[前 1 个数组的前缀和, 前 2 个数组的前缀和, 前 3 个数组的前缀和 ...]
```

这有什么用呢？由于元素的权重是不小于零的，因此这个数组其实就是一个正向排序好的数组。举个例子，如果输入的权重是 `[1,3]`，那么前缀和权重数组就是 `[1,4]`，如果我们在 \[0, 4) 上随机选取一个整数，那么 0 被选到的概率就是 0.25，选到 0 时，我们返回下标 0，1～3 被选到的概率就会是 0.75，选到 1～3，我们就返回下标 1。这不就是我们想要的吗？剩下的最后一个问题就是，**构建好前缀和数组后，如何确定要返回的下标**？这个问题其实可以看成是，去到排序好的数组里面寻找一个目标，或者是最靠近该目标的值，这里就可以完美的使用二分查找。

最后，可以看到，初始构建我们用了 `O(n)` 的时间，每一次的查找上我们用到 `O(logn)` 的时间。基于这道题目还有一些变种，比如，如果是需要返回的是元素而不是下标怎么办？输入的权重是浮点数怎么办？当然，如果说懂了基本的思想，这些也不是问题。


<br>

**参考代码**：
```golang
import (
    "math/rand"
)

type Solution struct {
    totalWeight int
    rangeArr []int
}

func Constructor(w []int) Solution {
    rangeArr, totalWeight := make([]int, len(w)), 0

    for i, e := range w {
        totalWeight += e
        rangeArr[i] = totalWeight
    }

    return Solution{totalWeight, rangeArr}
}


func (this *Solution) PickIndex() int {
    randValue := rand.Intn(this.totalWeight)

    start, end := 0, len(this.rangeArr) - 1

    for start + 1 < end {
        mid := start + (end - start) / 2
        if this.rangeArr[mid] <= randValue {
            start = mid
        } else {
            end = mid
        }
    }

    if this.rangeArr[start] > randValue {
        return start
    }

    return end
}
```

<br>

## Review

[Pro Git CH4](https://git-scm.com/book/en/v2/Git-on-the-Server-The-Protocols)

这一章主要讲的是如何搭建一个 Git 的 server。之前虽然说是一直在使用 Git，但是提到如何搭建一个类似 GitHub 的服务器，自己会觉得非常的复杂。书中提到了 Git 的几个依赖的远程协议，一起来看看：

* **Local**
  
  git 的本地协议，你可以在你的本机运行类似 `git clone /srv/git/project` 的指令，这个指令可以帮助你 clone 一个本地的仓库，另外，`git remote add local_proj /srv/git/project.git` 是把一个本地的仓库加到一个项目中去。这些操作都是在本地实现，不需要联网，当然如果只用这一种协议也就大大失去了 git 的意义。

* **HTTP**

  这也是一个比较常用的协议之一，依赖于 HTTP，用户可以通过用户名和密码进行权限的验证。但是优点既是缺点，每次都需要权限验证确实挺烦的。可以借助类似 `ssh-agent` 这样的工具来解决每次更新代码都要输入密码的困扰
  
* **SSH**

  这可以说是 Git 运用最广泛的协议，因为 SSH 基本上是大多数系统的标配，每个系统都会自带这个协议。这个协议也是加密的，因此会比较的安全。但是这个的缺点是不支持匿名访问，什么意思。想要拥有访问 git server 的权限，必须先告诉 git server 你的 public key，SSH 并不支持类似 HTTP 一样的用户名和密码验证。所以前期需要多做一些事情。
  
说完了上面这些个跟 Git 相关的协议后，我们来看看搭建 Git 服务器的几个步骤，不要把它想的太复杂，其实原理很简单：

1. 运行下面的 git 指令来克隆一个 git 仓库

   ```
   git clone --bare my_project my_project.git
   ```
   
   上面的这个指令其实就是把 my_project/.git 目录给复制到了 my_project.git 中去

2. 把上面生成的 git 文件上传到服务器：

  ```
  scp -r my_project.git user@git.example.com:/srv/git
  ```

基本上有了上面这两步，一个基本的 git 服务器就已经搭建好了，剩下的只是配置 ssh 文件，配置权限。对于有权限的用户，只需要下面的指令就可以克隆之前我们上传的 git 仓库

  ```
  git clone user@git.example.com:/srv/git
  ```

如果你仔细观察的话，对比 Github 或者是 Bitbucket 上面的 clone URL，你会发现格式就是上面我们列出的格式。当然像 GitHub 这种第三方的 Git 平台还做了许多的 UI 界面，用起来更加的方便，按一下按钮就可以达到效果。但其底层的原理是一样的。

<br>

## Tip

之前提到过有些时候我们需要把传送的二进制流用 base64 进行编码。那么这个操作用 JavaScript 如何实现。原本以为是用一个函数就可以解决的，但是浏览器端的 JavaScript 和 服务器端的 JavaScript（Node.js） 有点不一样。我们一一来看看

浏览器端的 JS，较为简单，只需要使用 `btoa` 用来编码 base64，用 `atob` 来进行解码。

而在 Node.js 中貌似没有内置 `btoa` 或者是 `atob` 这样的函数，一编码为例，我们需要先把字符串转化成二进制流，然后利用 toString 方法来进行转化：

```javascript
const base64String = Buffer.from(asciiString, 'binary').toString('base64');
```


<br>

## Share

最近由于工作需要，在研究 solr（一个类似于 elastic search 的搜索引擎）。本来想记录一下学习到的东西，但是对整体架构还不是特别了解，里面的重难点分的不是特别清楚，这一期的 Share 就先略过吧，等我有话题或者准备好了在来讲。

