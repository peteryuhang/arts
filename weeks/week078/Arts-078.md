> ARTS是什么？<br>
>**Algorithm**：每周至少做一个leetcode的算法题；<br>
>**Review**：阅读并点评至少一篇英文技术文章；<br>
>**Tip**：学习至少一个技术技巧；<br>
>**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 316. Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/)

**题目解析**：

题意简单明了，给定一个输入字符串，字符串中有些字符出现了多次，让你删除一些字符，使得每个字符在字符串中都只出现一次。但是这里有一个限定条件，**输出的字符串必须是所有可能的结果中按字符串排序最小的**，什么意思？比如输入字符串是 `bcabc`，根据删除的位置不同，结果可能有多种情况，比如，`bca`，`bac`，`abc`，这里我们需要返回 `abc`，因为这个结果是所有情况中按字符串排序最小的。

个人觉得这道题目还是比较有意思的，有很多细节值得思考。这道题目的难点主要在于你如何找到那个最小的情况。如果是暴力求解的话，那么我们需要找到所有的情况，然后一一对比，选出最后的答案。这样的解法对于短的输入字符串是可以适用，但是一旦字符串长度变长，你会发现你的时间复杂度和空间复杂度都呈指数增长。

那么有么有好的方式来解决这个问题呢？我们需要往细了去思考，你可以思考一下，如果我们以正常的方式遍历输入字符串（从左到右），我们每到一个字符就去判断到底这个字符要不要删。这里的字符删不删的条件是什么？我们可以把这些条件列出来：

1. 如果字符串中，当前字符不重复，或是重复的部分已经被删了，那么这个字符不能删
2. 当前遍历到的字符比前面遍历到并保留下来的字符小，那么我们可以考虑去删除前面的字符（注意这里只是可以尝试去删前面的字符，并不代表前面的字符就一定就能被删除，具体还要看条件 1）
3. 当前遍历到的字符在前面出现过，当前的字符可以被删除。如果一个字符在之前出现过，并且被保留了下来，证明这个字符存在在前面可以使得整个字符串 “变小”，因此我们可以把存在在后面的，也就是当前遍历到的字符给删除，使得每个字符仅出现一次

那么我们如何保存我们选择的字符呢，并且我们能够很容易地回过头去把前面遍历过，并保存下来的字符删掉。这里很自然地想到了栈这个数据结构，另外我们可以用哈希表来存储每个字符的总数量，以及字符是否存在于栈中（是否选择）。到这里，基本的思路就算是有了，剩下的就是实现了，主要就是如何进行条件判断，只要把之前列出来的几个条件想清楚，实现部分也会引刃而解。

通过分析细节条件，可以说这道题就这么解决了。最后还有几个实现上面优化的小技巧，第一是**使用数组代替哈希表**，因为字符串中出现的都是英文字母，我们完全可以使用字符的 ASCII 码来充当数组的下标，这样做虽然整体的时间和空间复杂度并不会改变，但是减少了哈希的内部操作，会在一定程度上降低时间的消耗。

<br>

**参考代码**：
```java
public String removeDuplicateLetters(String s) {
    if (s == null || s.length() == 0) {
        return s;
    }

    Stack<Character> stack = new Stack<>();
    int[] counts = new int[256];
    int[] hash = new int[256];

    char[] sArr = s.toCharArray();
    for (char c : sArr) {
        counts[c]++;
    }

    for (char c : sArr) {
        if (hash[c] == 0) {
            if (stack.isEmpty() || stack.peek() < c) {
                stack.push(c);
            } else {
                while (!stack.isEmpty() && stack.peek() > c && counts[stack.peek()] > 1) {
                    char prev = stack.pop();
                    counts[prev]--;
                    hash[prev]--;
                }
                stack.push(c);
            }

            hash[c]++;
        } else {
            counts[c]--;
        }
    }

    StringBuilder sb = new StringBuilder();

    while(!stack.isEmpty()){
        sb.insert(0,stack.pop());
    }

    return sb.toString();
  }
```

<br>

## Review

[Pro Git CH7](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection)

继续 Pro Git 第七章，这一章介绍了很多 Git 中的一些工具。这里主要展示 3 个我个人觉得非常有用的工具：

1. `--ours` `--theirs`
	
    在平时，我们 merge 或者是 rebase 更新过后的代码库难免会产生冲突。有些时候，冲突的地方非常的多，你不想挨个去解决这些冲突，这样耗时耗力。一个简单的解决冲突的办法就是完全使用某一方的代码，只需要加上 `--ours` 或者 `--theirs` 这样的参数即可。`--ours` 表示的是采用当前分支上的内容，相反，`--theirs` 则是采用 merge 进来的分支上的内容。

2. `git blame`
	
    这个指令相信大多数熟悉 Git 的人不会陌生，它可以把每一行代码所对应的最近的一次删改的 commit 给打印出来。这样对我们挑错和排查问题非常有帮助。另外，带上 `-L` 参数可以让它进行范围查找，对比较长的文件比较有用。还有一个非常有用的参数是 `-C`，它可以查找当前行的代码最初来自哪个 commit，甚至说当前行的代码之前是在其它文件中引入，git 也会将对应的文件和 commit 给找出来。

3. `git bisect`

	这也是一个帮助我们排查错误的小工具。常见的场景是，引入新的版本后发现了错误，这时我们需要找到对应的错误，换句话说就是找到最初引入错误的那个 commit。这里有个前提条件是，在某个 commit 之前，这个错误不会发生，但是在该 commit 之后，这个 commit 会一直发生。这样的场景我们完全可以借助二分搜索来辅助我们排查错误。这个 Git 指令使用起来也比较方便和直观，一开始你需要指定最早的一个你确定是没有问题的 commit，下面是一个使用范例：
    ```
	$ git bisect start		# 开始二分排查错误
    $ git bisect bad		# 当前的 commit 是有错误的
    $ git bisect good v1.0 	# tag v1.0 处的 commit 是不含有该错误的
    
    # 说明错误很有可能发生在 v1.0 - 当前 commit 之间
    ```
    启动二分查找后，git 地跳到你指定的 commit 和当前 commit 之间的某个 commit，然后需要判断这个 commit 是好是坏。如果是好的话，输入 `git bisect good`，坏的话输入 `git bisect bad`。以此往复下去，直到最终找到最初引入错误的那个 commit。
    
    这个指令在很大的程度上可以帮助我们排查错误，但是这也不绝对。因为这里假设某个错误出现后就会一直出现，有可能的一种情况是，这个错误被中间的某个 commit 掩盖（无意间修复）了，导致这个错误在某个 commit 后不再发生，但是新的 commit 又把它暴露了，这时用这个方法就很难找到根问题。当然，没有万能的方法，我们还是需要结合实际情况来做选择。
  
<br>

## Tip

这次来记录一个之前听说，但不怎么了解的概念 - **尾递归**。

这个又被常常称为 **“尾递归优化”**。我们先来看看普通的递归，还是拿一个例子来讲会比较清楚 - 求阶乘:

```java
int factorial(int n) {
    if (n <= 1) {
    	return n;
    }
    return n * factorial(n - 1);
}
```

递归会使代码变得非常的简洁，但是它的问题是会造成栈溢出。比如说我们要求 10000 的阶乘，那么函数在递归调用自身的过程中会在栈中不断压进函数，最大会压入 10000 个函数，这会严重影响栈的内存空间，如果超过了栈的空间的限定范围，这就是栈溢出。

那么怎么缓解递归所造成的这个问题呢？首先，我们要清楚我们为什么要把函数压入函数栈？答案是，因为我们需要保存函数中的局部变量，比如上面的例子，在递归计算 `factorial(n - 1)` 的时候，我们需要保存此时的 `n`，以便 `factorial(n - 1)` 完成后我们可以统计答案。但如果把上面的函数写成下面的形式，我们是否还需要把函数压入函数栈呢？

```java
int factorial(int n, int total) {
    if (n <= 1) {
    	return total;
    }
    return factorial(n - 1, total * n);
}
```

很明显，在递归调用发生时，当前函数中并不存在需要被保存的局部变量，所有需要的信息已经通过参数的形式递归传递下去。那么这时再把函数压入栈中就没有任何的意义。这种把递归调用写在函数的尾部并且使其没有依赖项的做法就是 **“尾递归优化”**。

这种优化并不仅用于递归，对于普通的函数调用（比如函数 A 调用函数 B）也适用，其主要的目的就是减少栈空间的不必要的消耗。只是说将这个优化用于递归效果会更加明显，毕竟栈溢出一直都是递归的最大痛点。

<br>

## Share

这段时间看了 [《JavaScript 语言精粹》](https://book.douban.com/subject/3590768/) ，学到了很多之前不知道的东西，这里做个分享，做个记录。

[揭秘 JavaScript 中的常见坑 - 数组](./揭秘JavaScript中的常见坑-数组.md)