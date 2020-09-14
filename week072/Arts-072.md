> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 1286. Iterator for Combination](https://leetcode.com/problems/iterator-for-combination/)

**题目解析**：

题意是让你实现一个支持按序输出定长的字符串序列的迭代器。比如给定的字符串是 `abc`，序列长度是 2，那么依次的输出就应该是 `ab`，`ac`，`bc`。

看了网上的一些解法，绝大部分题解都对这道题目有误解。误解主要出在迭代器上，迭代器的一个很重要的性质就是 **即用即取**。这个好处是 **可以很大程度上节省空间和时间**。举个例子，有 100 万个产品信息，这些产品信息会被实时按序读取，但不一定全部读完，产品信息是通过系统内部计算和整合得到。我们可以一开始的时候把这些产品信息全部计算出来，这样做是可行的，但势必会耗费巨大的时间去得到所有的产品信息，即使后面的有些产品的信息可能不需要，你也需要把它计算出来，因为你并不知道有多少产品信息会被读到。而且为了存储这些产品信息，空间的消耗也是少不了的。但在这个场景下，如果利用迭代器的思想就不一样了，需要读取的时候，就去实时计算得到。把时间的消耗均摊到每一次读取上，也不需要保存计算出来的结果，不会有不必要的资源浪费。

回到这道题目上，**如果要实现的是一个迭代器，那么就必须要保证这是一个即用即取的数据结构**。要把数据读取的时间均分到每一次请求上。而网络上的大部分题解都是想方设法把所有的数据提前计算好存起来。这就是对题目本身的误解。

说了这么多，如果要实现迭代器的话，该如何来做呢？题目其实给了两个前提信息，第一是输入的字符串本身是排好序的，第二是字符串里面的字符不会有重复。第一个信息保证了我们可以从头到尾遍历来保证输出序列的有序性，第二个信息可以让我们通过字符来找对应的索引，但即使我们没有这个信息，依然可以解题。思路比较直接，但是细节较多。假设被选定的字符下面都有一个指针，然后去到下一个结果的时候，我们需要试着把最右边的指针向右移动，如果发现最右边的指针移出字符串了，那么就试着移动第二靠右的指针，依次类推，比如按照题目给的例子，就会是下面的场景：

```
abc
||

abc
| |

abc
 ||
```

那么如何存储这些指针呢？这里比较合理的做法是用栈，在脑海里跟着例子走一遍就知道了。到这里，剩下的就是实现了，实现上细节的描述都放在了参考代码中，还是结合代码来看会比较好理解些。

<br>

**参考代码**：
```java
class CombinationIterator {
    private Stack<Integer> stack;	// 存放在当前结果中的字符的索引
    private StringBuilder sb;		// 当前的结果
    private char[] chars;			// 输入字符串
    private int len;				// 序列的长度

    public CombinationIterator(String characters, int combinationLength) {
    	// 初始化
        this.stack = new Stack<>();
        this.chars = characters.toCharArray();
        this.sb = new StringBuilder();
        this.len = combinationLength;

		// 把最开始的几个字符放入栈和结果中，这样我们就有了第一个准备输出的答案
        for (int i = 0; i < combinationLength; ++i) {
            stack.push(i);
            sb.append(chars[i]);
        }
    }

    public String next() {
        String currentResult = sb.toString();
        int index = chars.length - 1;

		// 将最右边的字符向右移动
        // 如果最右边的字符已经在最右边，将其弹出，考虑移动第二右的字符。。。以此类推
        // 因为序列长度固定，移动时需要保证右边有足够的空位
        while (!stack.isEmpty() && stack.peek() == index) {
            index--;
            stack.pop();
            sb.deleteCharAt(sb.length() - 1);
        }

		// 栈为空，表明移到最后，直接返回答案
        if (stack.isEmpty()) {
            return currentResult;
        }

		// 弹出当前的索引
        index = stack.pop();
        sb.deleteCharAt(sb.length() - 1);

		// 下面的代码主要是将上面刚弹出的索引往右再移动一位
        // 然后补全右边缺失的部分
        while (stack.size() != len) {
            Character temp = chars[++index];
            sb.append(temp);
            stack.push(index);
        }

        return currentResult;
    }
    
    public boolean hasNext() {
        return !stack.isEmpty();
    }
}
```

<br>

## Review

[Pro Git CH2](https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository)

最近想系统学学 Git，Git 平时一直在用，但都是一些常规操作，想进一步了解进阶操作，让自己的工作更为高效，如果能够了解一些 Git 底层实现就更好了。Pro Git 是讲解 Git 较为全面的一本书，计划一周一章，读的是英文版，这里做个摘要

Pro Git 的第二章讲的是基本的一些操作，也是我们平时最常用到的那些，但是覆盖面很广

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f82c110422454042906599aba5caaed7~tplv-k3u1fbpfcp-zoom-1.image)

上面这张图是这一章的重点，展示的是一个文件在 git 中的各个状态：

* **Untracked**
  
  新创建的 文件/目录，或者是刚移到当前 git 目录下的 文件/目录 都会处于这个状态。Git 不会对这个状态下的 文件/目录 进行任何的版本控制。通过 `git add` 文件后，文件会进入到 staged 状态，这之后 git 就会对文件进行比较控制，也就是这时的文件是被 Git tracked 的。

* **Unmodified**
	
  unmodified 是工作目录相对于 staged 状态或者是最新的 commit 来说的。只要有改动，unmodified 就会变成 modified 状态。同时，当我们 commit，也就是提交 staged 状态下的文件，因为最新的 commit 变成了 staged 状态下的改动，这之中的文件也就从 staged 变成了 unmodified。

* **Modified**

 当我们改动一个文件，文件就会从 unmodified 变成 modified

* **Staged**

 `git add` 后文件就会到这个状态，这个状态可以看成是待提交状态。

<br>

接下来，我们重点剖析一下 git 中的常见命令

* **git add**

  只要是你用过 git，这个指令你肯定知道。但如果要解释一下这个指令的作用，可能会得到五花八门的答案，比如，“让 git 追踪新文件”，“提交新改动的地方” 等等。这些答案不能说不对，但是更好一点的解释是 **精确添加下一次 commit 的内容**。这里的精确的意思是 `git add` 可以精确到某个文件，不需要添加所有的改动，`git add` 的最终目的都是为了下一次 commit，可以把 `git add` 看成是一个筛选的动作。

* **git status**

  这个指令用于查看 git 目录状态，它有一个非常有用的 flag，也就是 `-s/--short`，这个指令在改动较多的时候比较有用，一行通过前缀字母显示一个文件的状态，常见前缀如下：
  ```
  ?? untracked
  A  staged
   M modified
  M  modified & staged
  MM modified & staged, then modified again
  ```

* **git diff**

  如果是单纯这个指令本身，不加任何的 flag，表示的是比较工作目录和 staged 状态下的差别。如果加上 `--staged/--cached` 比较 staged 状态下和最近一次的 commit 的差别。当然我们还可以用 `git difftool` 调出操作系统默认的比较指令进行文件比较，通常都是 `vimdiff`。

* **git commit**

  关于 `git commit` 也有两个比较有用的 flag，一个是 `-a`，加上了这个 flag 等同于在 `git commit` 之前加上了 `git add -A`，也就是跳过 `git add` 直接 commit。另一个是 `-m`，后面接 message，这个比较方便，写 commit message 不需要打开编辑器。
  
  另外一个比较有用的 flag 是 `--amend`，表示修改最近一次的 commit，如果当前 staged 状态下有改动，那么就新添加这些改动，如果没有的话，那么就仅仅是改动 commit message。

* **git rm**

  这个命令就是让 文件/目录 进入 untracked 状态。后面可以接正则，比如 `git rm log/\*.log` 表示的是移除 `log` 下的所有 `*.log` 文件；`git rm \*~` 表示的是移除当前目录下，后缀是 `~` 的所有文件。注意，直接使用这个指令，会让 文件/目录 从工作区中消失，这个删除动作是会被存在 staged 状态下的。另外一个比较有用的 flag 是 `--cache`，表示忽略 文件/目录，作用等同于 `.gitignore`。

* **git mv**

  这个命令应该知道的人比较少，它是 git 下的重命名操作，其实：
  ```
  git mv <file> <newfile> == mv <file> <newfile>; git rm <file>; git add <new file>
  ```
  可以看到这一个指令等同于三个指令的功效

* **git log**

  查看 git 的日志，这个指令的 flag 较多，但都是涉及显示的格式和内容
  * -p(--patch) 显示 diff，可以加上 -(num) 来限定显示的 commit 数量
  * --stat 显示文件的删改情况，比如增加了多少行，删除了多少行
  * --pretty= 后接一些参数，比如 oneline，format 来控制 commit 的格式
  * --graph 图形显示
  * --since, --until 通过时间来控制 commit 的显示，比如 `git log --since=2.weeks` 显示的是最近两周的所有 commit 的 log
  * --author 后接名字，表示显示某一作者的所有 commit
  * --grep 这个指令后面接字符串，git 会在 commit message 里面匹配，然后将匹配上的 commit 显示出来
  * -S 后面接字符串，表示仅仅显示该字符串被改动的地方
  * -- 后面接 文件/目录 路径，表示仅仅显示和该 文件/目录 相关的 commits
  * --no-merge 加上这个 flag 后，表示忽略掉 merge 相关的 commit 

* **git reset HEAD <file\>**

  把文件移出 staged 状态。

* **git checkout -- <file\>**

  移除某个文件或者目录的所有修改，或者说是，用最近一次 commit 上的文件内容替换当前指定的文件。这个指令慎用，一般还没有 commit 的改动，直接这样移除是没有办法恢复的。

以上便是 Pro Git 第二章个人认为比较重要的一些指令和用法，感觉这些东西不用记，工作中用的多了自然就记住了。重要的是理解一个指令的意义，这样才能融会贯通，而不是照猫画虎。

<br>

## Tip

这次记录几个 VIM 中比较有用的插件：

1. [Fugitive](https://github.com/tpope/vim-fugitive)

  这是一个全功能的 git 支持插件。使用方面也很简单，只需要在命令行模式下输入正常的 git 命令即可，但是命令的首字母需要大写，也就是 git 会变成 Git。除此之外，这个指令还提供了一些便捷的命令比如 cc 就相当于 `git commit`。

2. [GitGutter](https://github.com/airblade/vim-gitgutter)

  也是一个 git 支持插件，这个插件不提供完整的 git 命令，但是这个插件有些便捷的指令，可以让你很方便地对比自己当前的改动和上次提交 (commit) 的差别。

3. [Airline](https://github.com/vim-airline/vim-airline)

  让 VIM 界面的显示更加美观，安装后就可以使用。Airline 会显示当前的文件，如果有 Git 支持的话，还会显示当前所属的分支，以及修改、增加、删除的行数，当然我们还可以让其显示打开文件缓冲区的 tab，方便我们切换文件。

4. [NERDCommenter](https://github.com/preservim/nerdcommenter)

  用 VIM 进行编程的一大苦恼就是注释代码，特别是对于 VIM 新人来说。这个插件提供了便捷的注释方法。常用的如下：
   * \cc 把一行代码或者在 visual 模式下选中的地方注释掉
   * \cu 把注释恢复成正常的代码

<br>

## Share

分享一下最近在做的一个智能设备的项目，大致的框架是：设备定时通过 SIM 卡（4G网络）将数据流传送到 IOT 平台，IOT 平台会将设备传送上来的数据流进行转换发送给我这边的第三方服务器，服务器将传送过来的数据进行存储并在其 Web 页面进行展示。另外服务器这边也可以对设备进行命令下发，流程和之前类似。服务器这边会把数据发送给 IOT 平台，IOT 平台会对服务器发送过来的数据进行转换，然后下发给对应设备。

这里面有一个很重要的点就是数据的格式与类型转换。其中因为设备是移动设备，考虑到耗电量和功率等因素，其传送和接受的数据均为二进制流。而服务器这边用的是 RESTful API，所以是 JSON 格式。IOT 平台会把设备上报的二进制流编码成字符串，这里涉及到一个编解码的问题。一开始考虑的是将二进制流的每个字节转换成一个对应的 ASCII 码，但是这里有一个问题，在 ASCII 编码区间 0 ~ 127 的字符是可以成功转换的，不管用的是 UTF-8，还是 UTF-16，亦或是其它的编解码方式，这里的字符与编码的对应关系都是一样的。问题在于 ASCII 编码区间 128 ～ 255 之间的数，这个区间属于 ASCII 的扩展区间，首先根据版本的不同，会存在多种的对应关系，另外有的编码方式，比如 UTF-8，128 ~ 255 并不是它的编码区间，也就是说这个区间上并没有字符。这里需要用到一个编码转换方式 **[base64](https://zh.wikipedia.org/wiki/Base64)**。它的作用在于把二进制流转换为可打印的字符，另外他还有着简单的加密功能。

那具体 base64 编码的原理是什么呢，其实很简单，就是将每 3 个字节转换为 4 个字节。具体操作是将这 3 个字节的 24 个 bits 位拆分成 4 个分组，这样每个分组就会有 6 个 bits 位，然后分别在这 4 个分组前面加上 00，这样之前的 3 个字节就会变成 4 个字节，而且最重要的一点是，每个字节所表示的数值的大小都是属于 0 ～ 63 这个区间的，其中每个数值都会对应一个字符，对应关系可以参照下表：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0724299558d4c9c91a90db8a94bd490~tplv-k3u1fbpfcp-zoom-1.image)

比如下面这个例子就是将之前的 3 个字节拆成了 4 个字节，因为每个字节都会对应一个字符，也可以看作是 3 个字符转换成了 4 个字符：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65720631713347c49f134d03cae7d9bc~tplv-k3u1fbpfcp-zoom-1.image)

在 base64 转换后的字符串中经常可以看到 = 作为结束符。这是因为有时我们需要转换的字节长度并不是 3 的整数倍，我们需要在原先的数据流后面补 0，让其字节数是 3 的整数倍，补的字节就用 = 来表示，例子如下：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58ac52f488af486486a7fb87e0e2aaf1~tplv-k3u1fbpfcp-zoom-1.image)

之前经常看到 base64，现在终于懂了为什么我们需要这种转换方式，其最终作用是为了字符和二进制流之间的转换，有时使用它也可以避免明文传输数据信息，也算是一个简单的加密吧。