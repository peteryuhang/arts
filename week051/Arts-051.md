> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 1381. Design a Stack With Increment Operation](https://leetcode.com/problems/design-a-stack-with-increment-operation/submissions/)

**题目解析**：

题目让你设计一个类似栈的数据结构，这个栈除了基本的 `push` 和 `pop` 功能之外，还需要额外支持一个 “从栈底开始的 k 个元素自增 value” 的功能。

想要 AC 这道题目不难，你可以用一个数组来实现栈，对于 “元素自增” 功能，我们就遍历数组的前 k 个值，让其自增即可。但是仔细想想，这样做下来，“元素自增” 功能的时间复杂度就会是 `O(min(k, n))` 其中 n 是栈中当前元素的总数。我们知道，栈其实是属线性结构，它的两个原始操作都是 `O(1)` 的时间复杂度，如果我们能够让这个额外功能也具有 `O(1)` 的时间复杂度那就再好不过了。

那该如何去思考这个问题呢？如果不能一次性自增对应的元素，那我们唯一能做的就是保存这个 “元素自增” 操作对应的数据，以备后用。这里哪些数据是我们必须看重的？其实就是两个，k 和 value，我们需要知道当前自增操作覆盖栈中的哪些元素。这里有一点特别重要，**就是我们只需要记录最高位置，因为最高位置往下其实都生效**。到了这个位置，以后每次 stack 做 `pop` 操作都需要加上 value。当然，如果在这个时候，往 stack 中进行 `push` 操作，之前的 value 是不会对 `push` 进来的新值生效的。于是基于这两点，我们可以创建一个数组，这个数组专门记录奏效的最高位置，然后这个位置过了之后，就将对应的值清零，并将值移加到下一个位置。在这之中，有一个重点就是，生效的元素在任何时候都不能大于栈中现存的元素。

这样下来，我们可以把时间复杂度降到 `O(1)`，符合我们的预期。

<br>

**参考代码**：

```java
class CustomStack {
    private Stack<Integer> stack;
    private int[] inc;
    private int maxSize;

    public CustomStack(int maxSize) {
        this.maxSize = maxSize;
        this.inc = new int[maxSize];
        this.stack = new Stack<>();
    }

    public void push(int x) {
        if (stack.size() == maxSize) {
            return;
        }

        stack.add(x);
    }

    public int pop() {
        int index = stack.size() - 1;
        if (index < 0) {
            return -1;
        }

        int result = inc[index] + stack.pop();

        // 对高位生效的 value，也同样对低位生效
        // 将高位的值移加到低位
        if (index > 0) {
            inc[index - 1] += inc[index];
        }

        // 高位的值清零
        inc[index] = 0;

        return result;
    }

    public void increment(int k, int val) {
        // 栈为空，自增操作不会对任何元素生效，退出
        if (stack.isEmpty()) {
            return;
        }

        // 必须保证生效的值不超过当前存在的元素的个数范围
        int index = Math.min(k, stack.size()) - 1;

        // 记录最高值
        inc[index] += val;
    }
}
```

<br>

## Review

[Vim isn’t that scary. Here are 5 free resources you can use to learn it.](https://medium.com/free-code-camp/vim-isnt-that-scary-here-are-5-free-resources-you-can-use-to-learn-it-ab78f5726f8d)

这篇文章主要是想介绍 Vim 的，首先它说了诸多的学习并使用 Vim 的好处，在这个地方我就不多说了。好处就是能够让你脱离鼠标，灵活地在服务器上编写文件和脚本，而且 Vim 非常的轻量级，在大多的 Linux 系统中都是预装的。

这里重点是，文章给了一些学习 Vim 的资源，我们可以通过这些资源更好更快地接触并学习 Vim，让你逐渐习惯 Vim，并把它应用到你的工作生活中去。我们一起来看看：

1. VimTutor

    Mac 环境下，直接在命令行输入 `vimtutor` 就可以看到这个教程。这个是 vim 自带的教程，不长，基本上花个半小时就可以大致浏览完，主要就是告诉你 vim 中的一些基本指令，让你对 vim 有个大致的了解，对初学者会有帮助。
    
2. [OpenVim](https://www.openvim.com/tutorial.html)

    一个学习 vim 个虚拟环境，每一步都会有细致的讲解，也是对初学者非常有帮助的一个资源
    
3. [Vim Adventures](https://vim-adventures.com/)

    一款基于 vim 的游戏，玩过前三关，感觉非常有趣，但是后面的关卡要收费，关卡、情节都是基于一些基本的 vim 指令，旨在帮助你熟悉 vim 当中的指令，让你通过游戏能够对每个指令有个更为深刻的印象
    
4. [The basics of Vim](https://vimeo.com/showcase/2838732) 

    Vim 的一款视频教程，老师真人讲解，会引用一些真实的场景，让你并不单单只是记住几个指令，更重要的是灵活使用。当然了，全英文教学。
    
5. [Vim Cheat Sheet](https://vim.rtorr.com/)

    Vim 的 Cheat Sheet，记录这 Vim 当中的很常见的 command，你可以把它打印下来贴在自己的办公桌上，忘记的时候可以方便查阅。


<br>

## Tip

记录几个跟 git 相关的东西：

* 如果你要改动了一个文件，需要将文件还原（仅仅是该文件）
   ```
   >$ git checkout <filename> 
   ```
* Github 的 URL 其实是由几部分组成的：
   ```
   https://github.com/<owner>/<repo_name>/...
   ```
   
   知道这个有什么用？比如你知道了一个 commit 的 hash，就可以使用下面的 URL 快速通过浏览器定位到改动文件（commit）
   
   ```
   https://github.com/<owner>/<repo_name>/commit/<commit_hash>
   ```

<br>

使用 screen 指令时的一些认识：

* 在 screen 中是不能通过光标滑动 terminal 的，但是如果要回看之前的记录怎么办，直接用快捷键 `CTRL + A + ESC`退出 screen 模式，当你浏览完之前的记录后，按 `ESC` 又可以回到 screen 模式
* 常常会有这样的情况，就是 local 连到 remote，在一个 screen 中，但是突然断线了，重新登陆发现之前的 screen 还处在 attach 模式，无法使用 `screen -r` 进入。这时可以使用 `screen -d ...` 让 screen 的状态变成 detach，这样就可以使用 `screen -r` 再次进入 screen 了。

<br>

Go 当中的一些个值得注意的地方：

* 一个 package 中的大写的成员会被自动 export 出去
* 用 `:=` 定义变量只能在函数中使用，全局的话需要使用关键词 `var`
* 仅支持显示转换，所有的显示转换格式
  ```
  T(v)      // 将变量 v 转换为 T 类型
  ```
 * defer 关键字是基于函数栈实现的，如果一个一个函数中有多处使用了 defer，注意运行这些 defer 的原则是 **last-in-first-out**
 * slice 是不存储数据的，只是用来描述潜在静态数组的一部分
 * method 是加了 receiver 的 function，receiver 的 type 必须和该 method 在同一个 package 中。考虑到效率问题，建议使用 pointer 的形式作为 receiver
 * 只要是任何的 value 实现了 一个 interface 里面所有的函数，这个 value 就可以作为这个 interface 的 type。空 interface 能够接收任何 type 的 value
 * channel 是 go 中特有的一个结构，分为两种，一种是没有 buffer 的，接收和发送必须同时在线，如果不是的话，其中的一方先到的话就要等待。另外一种是有 buffer 的，只要 buffer 里面有值，接收一方就不需要等待，只要 buffer 不满，发送的一方就不需要等待

<br>

## Share

继续 《Clean Code》，这次我们来看看 注释、代码风格还有对象与数据结构。废话就不多说了，直接进入主题

[Clean Code 之旅 - 注释、代码风格、对象与数据结构](./CleanCode之旅-注释、代码风格、对象与数据结构.md)