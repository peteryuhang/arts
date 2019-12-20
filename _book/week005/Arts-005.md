## Algorithm

**题目一**：<br>

[LC 212. Word Search II ](https://leetcode.com/problems/word-search-ii/)

**解答**：<br>

给定一个二维的字符数组和一个单词列表，从单词列表中找出在字符数组中存在路径的单词；一开始看到题有两种思路，第一种比较直接，就是暴力的深度优先搜索，对于每个单词都在字符数组中找一遍，时间复杂度是 $O(n*m*w)$，这里 $n$，$m$ 分别表示字符数组的行数和列数，$w$ 表示的是单词的数目，这个时间复杂度其实是一个大概的，并不准确，因为有些时候在字符数组的一个格子都要去从四个方向去找，这里的寻找复杂度其实是指数幂的，如果找到了的话可以不用继续遍历字符数组，但是每个单词都要遍历一边字符数组，这个代价确实太大。第二种是优化的方案，就是利用 Trie 树，即字典树，先将单词构建字典树，然后针对字符数组中的每个字符，结合字典树看看其周围路径能不能构成单词，这里就只需要遍历一遍字符数组，而且用字典树去构建单词的时间复杂度是分开来的，并不是直接垒乘的，效率有大大的提升。深度优先搜索涉及到去重，就是一个路径不能经过一个地方两次，传统做法是要用到 boolean 数组，这里利用一个小技巧，直接在字符数组上面修改，修改之前保存之前的值，为的是以后的复原。

**实现参考代码**：

```java
private class TrieNode {
    TrieNode[] children = new TrieNode[26];
    String word = null;
}

private TrieNode root = new TrieNode();

private void buildTrie(String[] words) {
    TrieNode pointer = root;
    
    for (String word : words) {
        pointer = root;
        
        for (char c : word.toCharArray()) {
            if (pointer.children[c - 'a'] == null) {
                pointer.children[c - 'a'] = new TrieNode();
            }
            
            pointer = pointer.children[c - 'a'];
        }
        
        pointer.word = word;
    }
}

public List<String> findWords(char[][] board, String[] words) {
    if (words == null || words.length == 0) {
        return new ArrayList<String>();
    }
    
    List<String> result = new ArrayList<>();
    
    buildTrie(words);

    int m = board.length;
    int n = board[0].length;
    
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            helper(board, result, i, j, root);
        }
    }
    
    return result;
}

private void helper(char[][] board, 
                    List<String> result, 
                    int i, 
                    int j, 
                    TrieNode cur) {
    if (i >= board.length || j >= board[0].length || i < 0 || j < 0) {
        return;
    }

    char c = board[i][j];

    if (c == '#' || cur.children[c - 'a'] == null) {
        return;
    }
    
    cur = cur.children[c - 'a'];
    
    if (cur.word != null) {
        result.add(cur.word);
        cur.word = null;
    }
    
    board[i][j] = '#';

    helper(board, result, i + 1, j, cur);
    helper(board, result, i - 1, j, cur);
    helper(board, result, i, j + 1, cur);
    helper(board, result, i, j - 1, cur);
    
    board[i][j] = c;
}
```

**题目二**

[LC 51. N-Queens ](https://leetcode.com/problems/n-queens/)

还是 N 皇后问题，但是这次把位运算利用进来，详细可见之前发布的一篇文章：
<br>

[利用位运算解决 N 皇后问题](./利用位运算解决N皇后问题.md)

<br>

## Review
在 medium 上看到的一篇有着接近 20w claps 的文章，讲的是解决问题的方法:<br>

[How to think like a programmer-lessons in problem solving](https://medium.freecodecamp.org/how-to-think-like-a-programmer-lessons-in-problem-solving-d1d8bf1de7d2) 

说到编程有什么用，程序员如果不敲代码是不是一无是处？这里的关键点在于，程序员能从编程当中学到什么东西，很多东西都是相通的，如果你能总结出高效的解决问题的思考模式和方法，相信绝大多数的问题都会有一个很好的切入点，让你做到遇事不慌，心中有对策

>“Everyone in this country should learn to program a computer, because it teaches you to think.” —Steve Jobs

编程其实主要是为了解决问题，而不是无谓地 “堆砌砖头”，文章中作者认为解决问题的能力是在于能有一个好的思考框架，还有就是在这个框架上面反复地练习。

首先是一个好的思考框架，文中给出了几个步骤，我们一个个来看：

1. 明白问题 
2. 做好计划
3. **拆分问题**
4. 寻找问题

首先，明白问题是前提，往往觉得问题困难、棘手，有可能仅仅是自己心里对问题的不恰当，或者是不全面的认识造成的，造成这种认识可能是道听途说，也或者是自己之前的类似的经历给了自己某种感觉。这里需要做的就是把问题用语言说出来，越直白，越简单越好，说给一个完全不了解你做的事情的人，像是你爸，妈，甚至十几岁的小孩，他能听懂，这才证明你理解了问题。往往在你描述的过程当中，自己就可以发现自己的想法其实漏洞百出。

做任何事情都要有计划，即使计划非常简单，也要做到心中有数。做一步看一步是非常危险的做法，特别是当你对你做的东西不是特别了解的时候，运气不好有可能会导致你之前做的东西全是徒劳。

拆分问题是这里的重点，一个大的问题是很难直接下手去做的，将大的问题拆分成小的可以一下看出解的问题，这其实是一种能力，一种解决问题不可或缺的能力，将问题拆的越细，证明你对问题理解的越透彻，解决问题的能力也相应的越强。这个一下子做到很难，但是心中有这么一个概念，时刻敦促自己朝着对的方向前进，这样至少来说自己会在一个正确的路上

最后一个遇到困难了，寻找问题的能力，也就之前讲是 debug 的能力，这里作者给出了几点建议，一是退一步全面地看，而不是仅仅盯着某个小部分，另外就是寻找问题是建立在问题拆分之上，没有问题拆分，仅仅在整个大的项目当中去寻找问题，是没有任何意义的。这里也可以看看 [前一篇 ARTS](https://juejin.im/post/5ca68d21f265da30c7042c34) 的 Review 部分。

最后，也是最最重要的，就是练习，知道了这些其实只是一个方向，能做成什么样子，还是靠练习。这里作者提到，练习不仅仅是写代码的时候才能做，任何涉及思考的东西都可以用上这个框架，像是下棋、做数学题，甚至是玩游戏，只要是带有目标性质的东西，都可以尝试着去使用这一套思考框架，发现其中的不足，慢慢调整，慢慢精进
>“Just when you think you’ve successfully navigated one obstacle, another emerges. But that’s what keeps life interesting. Life is a process of breaking through these impediments - a series of fortified lines that we must break through.
Each time, you’ll learn something.
Each time, you’ll develop strength, wisdom, and perspective.
Each time, a little more of the competition falls away. Until all that is left is you: the best version of you.”<br>
—— Ryan Holiday (The Obstacle is the Way)

## Tip
这次记录下 Linux 环境下设置环境变量的方法：

Linux 环境下，可执行文件是靠配置文件去读取路径的，设置环境变量有三种方法，第一种方法是直接使用 export 命令，但是这个命令只在当前的 session 有效，也就是重启系统就会失效；另外两种就是修改环境变量文件，有两个文件可供选择，"/etc/profile" 和 用户主目录下的 ".bash_profile"，前者是对所有用户有效，后者只对当前用户有效，注意 PATH 定义多个路径需要用 “:” 符号隔开，下面记录一下常见的一些变量和命令：

* 常见的环境变量 
    * $PATH: 决定了 shell 将到哪些目录中寻找命令或程序
    * $HOME: 当前用户主目录
    * $MAIL: 当前用户的邮件存放目录
    * $SHELL: 指当前用户用的是哪种Shell
    * $HITSIZE: 指保存历史命令记录的条数
    * $LOGNAME: 指当前用户的登录名
    * $LANG: 和语言相关的环境变量
    * $PS1: 基本提示符，可直接使用命令 PS1="你想要的提示符" 试试看
    * $PS2: 附属提示符
    * $0: 脚本的名字

* 定制环境变量
    * 使用命令 echo 显示环境变量
    * 使用 export 设置一个新的环境变量
    * 使用 env 命令显示所有的环境变量
    * 使用 set 命令显示所有本地定义的环境变量
    * 使用 unset 命令清除环境变量
    * 使用 readonly 命令来设置只读变量

<br>

## Share
今天我们来聊聊面试这个话题，之前同样是看极客时间的 “面试现场” 专栏，感觉里面有些东西讲的很好，在这里我把我的认知和看法分享出来，也同时欢迎你的点评和分享
<br>

[关于面试，我这样看](./关于面试，我这样看.md)