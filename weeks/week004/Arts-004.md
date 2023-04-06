## Algorithm

**题目**：<br>

[LC 37. Sudoku Solver ](https://leetcode.com/problems/sudoku-solver/)

**解答**：<br>

与 N 皇后问题一样，数独问题也是考察回溯与深度优先搜索的经典问题，这样的问题难点在于优化，也就是如何进行剪枝，何时进行剪枝。根据题目描述，我们需要保证横、竖以及 3X3 的方格内的所有数字不重复，这样我们可以通过用 boolean 标记数组来实现对横、竖、方块的验证，只要之前填的值有问题，就立刻返回 false，不然就填入符合要求的值，然后继续递归，直到所有的空位都被填满，这里有个点就是方块的定位可以通过当前行列下标除以 3 来定位，例如 [2/3][1/3] 表示的就是左上角那块空格。你也可以不使用标记数组，这样在验证的时候就需要多写一层循环来逐一判断，当然代码会比较简洁，但是在时间上面并不是特别的高效。

**实现参考代码**：

```java
public void solveSudoku(char[][] board) {
    boolean[][] rows = new boolean[9][9];
    boolean[][] cols = new boolean[9][9];
    boolean[][][] squares = new boolean[3][3][9];
    
    for (int r = 0; r < board.length; ++r) {
        for (int c = 0; c < board[0].length; ++c) {
            if (board[r][c] != '.') {
                rows[r][board[r][c] - '1'] = true;
                cols[c][board[r][c] - '1'] = true;
                squares[r/3][c/3][board[r][c] - '1'] = true;
            }
        }
    }
    
    helper(board, rows, cols, squares);
}

private boolean helper(char[][] board,
                       boolean[][] rows,
                       boolean[][] cols,
                       boolean[][][] squares) {
    for (int r = 0; r < board.length; ++r) {
        for (int c = 0; c < board[0].length; ++c) {
            if (board[r][c] == '.') {
                for (char i = '1'; i <= '9'; ++i) {
                    if (isValid(board, rows[r], cols[c], squares[r/3][c/3], i)) {
                        board[r][c] = i;
                        
                        rows[r][i - '1'] = true;
                        cols[c][i - '1'] = true;
                        squares[r/3][c/3][i - '1'] = true;
                        
                        if (helper(board, rows, cols, squares)) {
                            return true;
                        } else {
                            board[r][c] = '.';
                            rows[r][i - '1'] = false;
                            cols[c][i - '1'] = false;
                            squares[r/3][c/3][i - '1'] = false;
                        }
                    }                        
                }
                
                return false;
            }
        }
    }
    
    return true;
}

private boolean isValid(char[][] board, 
                        boolean[] row, 
                        boolean[] col, 
                        boolean[] square, 
                        char c) {
    if (row[c - '1'] || col[c - '1'] || square[c - '1']) {
        return false;
    }
    
    return true;
}
```

<br>

## Review
在 medium 上看到的一篇讲关于如何 debug 的文章:<br>

[How Debugging Can Make You a Better Developer](https://medium.com/swlh/how-debugging-can-make-you-a-better-developer-93db511be8cb) 

一提到 debug，没有人会喜欢，很有可能的是，你花费了数个小时，但是依然毫无收获，即使你发现了问题，大多数情况下也是一些不起眼的小问题，例如参数传错，变量忘记赋值之类的，如果不花大块的时间进行全面的深度总结的话，很难说能从中学到什么。文章中给出了 debug 的一些建议，是一些方向性的建议，如果能够形成意识，相信会大大缩短 debug 的时间，提升开发效率：

* 首先了解框架系统，以及全局的设计
  <br>
  
  debug 难点不在于解决问题，而是在于寻找问题，定位问题，问题可能就是出在一个点上，而往往这个点并不在我们直觉所认为的那个地方上，不知道你有没有过这样的经历，就是把自己写的函数反复看了好几遍，改了又改，但是依然没找到问题，最后发现其实是调用函数的地方出现了问题，告诉自己下次一定要注意，但是往往下次还是会犯类似的问题。这样低效的原因其实就是我们没有从全局来看待问题，一直盯着一个地方看，并不是一个好的寻找策略，搜索一般都是从大到小，而且常常思考每个部分在整体当中扮演的角色以及功能，也是有助于我们理解整个系统的。只有当我们了解了这些全局的东西，才能够更清晰地定位问题。

  > Always remember, you need a working knowledge of what the system is supposed to do, how it’s designed, and, in some cases, why it was designed that way. If you don’t understand some part of the system, that always seems to be where the problem is.
  
* 分离并缩小问题出现的范围
  <br>

  上面说到如何定位问题，但是有时候是知道问题大概是什么，但是整个系统很庞大，类似的问题可能在多个地方出现，这时的策略就是分治，这里的分治是指分开来检查，其实就是排除法了，主要的目的是缩小定位问题的范围，当然这一步是建立在上一步的基础之上的，只有当你了解了全局系统之后，才能更好的做分离

  > Always remember It’s hard for a bug to keep hiding when its hiding place keeps getting cut in half.

* 每次只改变一个地方
  <br>
  
  在 debug 的时候我们会有一些猜想，例如这几个地方可能是出错点，但是并不确定，需要验证，这时我们最好是同时只改变一个地方，看问题有没有得到解决，不然的话，多个地方同时被更改，问题解决了，你也不知道是哪出错了，问题还是没有找到，更糟糕的是你把之前好的代码更改了，往后可能会有更多的问题。

  > Always remember you can always tell exactly which parameter had the effect if you make one change at a time. And if a change doesn’t seem to have an effect, back it out right away!

* 验证你是不是真的解决了问题
  <br>

  有些时候我们定位到了问题，但是有可能并不只是这一个地方出错，很有可能这个问题是多处出错的结果；我们需要做的是测试在不同场景下的修复结果，如果把修复的代码拿掉，就会产生问题，把修复的代码加上，在任何的场景下，都不会有问题，这时我们才能说我们找到了并修补了 bug。
  
  > Always remember, If the system fails as it used to when you remove only the fix, only then you can be pretty sure the fix is indeed working. 
  
* 对于解决的问题，记录下来
  <br>

  我们总是在借鉴前辈的经验，来节省自己的尝试成本，前辈们犯过的错，我们总结了，思考了，下次很大几率不会犯同样的错，大大节省做事效率。同样的，我们也要知道我们自己干过的事情，更细致一点就是是什么问题？什么时候犯的？在什么情况下犯的？如何解决的？作者给了几个记录点，可以参考，根据实际的项目情况增添或者删减：
   * Issue #
   * Initiator (who logged the call)
   * Initiator Extension or Phone #
   * Date/Time Opened
   * Summary Description
   * Impact/Importance
   * Type (of fault)
   * Owner (of system)
   * Current Status (open, in process, closed)
   * Next Step
   * Next Step Date
   * Completion Date
   * Resolution, development request # or link to vendor support request
 
<br>

> “When debugging, novices insert corrective code; experts remove defective code.” - <i>Richard Pattis</i>

最后需要记住的是，debug 的过程其实是一个学习的机会，是一个让自己成为更好的程序员的机会。成长的过程往往都不是很轻松，都会有痛苦相伴，但是值得庆幸的是，你可以从中学到不少你从未接触，或者了解的比较少的东西，例如语言的特性，系统的设计理念，以及有用的开发工具等等，相信这些东西往后都会成为你职业技能上的宝贵财富。

<br>

## Tip
看极客时间 <strong>趣谈Linux操作系统</strong> 专栏，学到了几个 Linux 命令，之前没怎么了解，但是了解之后感觉是提升工作效率的利器：

* export 命令仅在当前命令行的会话中生效，如果需要永久生效，要在 .bashrc 文件里面进行设定
* ubuntu 系统下的软件管家 <br>
  安装 -> \$> apt-get install ... <br>
  搜索 -> \$> apt-cache search ... <br>
  删除 -> \$> apt-get purge ... <br>
  配置文件 ->  /etc/apt/sources.list 

* 后台运行命令 nohup <br>
  $> nohup command >out.file 2>&1 &
  <br>
  
  这里 “1” 表示文件描述符 1，表示标准输出，“2” 表示文件描述符 2，意思是标准错误输出，“2>&1” 表示标准输出和错误输出合并，合并到 out.file 里面

* 查看运行进程的命令 ps -ef <br>
  $> ps -ef | grep 关键字 | awk ‘{print $2}’ | xargs kill -9
  <br>

  ps -ef 可以单独运行，表示列出当前运行的程序，awk 可以很灵活地对文本进行处理，awk ‘{print $2}’ 是指第二列的内容，是运行程序的ID。我们可以通过 xargs 传递给 kill -9，也就是发给这个运行的程序一个信号，让它关闭。

* 服务运行命令 systemctl <br> 
  \$> systemctl start mysql <br>
  \$> systemctl enable mysql <br>
  \$> systemctl stop mysql
  <br>

  之所以成为服务并且能够开机启动，是因为在 /lib/systemd/system 目录下，创建了一个 XXX.service 的配置文件，从而成为一个服务

<br>

## Share
看了极客时间上面的 **10X程序员工作法** 专栏，结合自己现在的工作情况，有些小感悟，分享在这里，也是希望得到大神们的指点，也希望和志同道合的朋友们相互认识，相互成长。
<br>

[写项目代码之前必须要做的事](./写项目代码之前必须要做的事.md)