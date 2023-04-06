# Algorithm
[LeetCode 51. N-Queens](https://leetcode.com/problems/n-queens/)<br>
[LeetCode 52. N-Queens II](https://leetcode.com/problems/n-queens-ii/)

**解答**：

经典的不能再经典的 N 皇后问题，这道题主要考察的是深度优先搜索和回溯的思想，要注意的是皇后可以竖着走、横着走、斜着走，如果我们从一个方向开始计算，比如从上往下，在一行之中，我们仅仅选择一个当前合适的位子，这里有两种思路：

- 当前位置影响下面的位置
- 当前位置被上面的位置影响

如果是第一种思路，那么得用一个二维数组记录所有位置是否有效，走到某一位置，先看当前的位置是不是有效位置，不是的话直接跳过，是的话，记得更新这个记录位置有效性的二维数组，主要是对没有遍历过的其他行的位置进行标记。<br>
如果是第二种思路，那么走到某一位置就往上看去判断当前位置有没有被上面影响，没有的话当前就是有效位置。<br>
下面给出的代码是基于第二种思路，主要是因为第一种思路涉及到回溯中，位置有效性数组的更新还有复原，还有就是位置可能被多重影响，必须使用整形数组，比较繁琐；其实两道题是一样的，只不过一个是输出所有的解，另一个输出解的个数。
<br>

**实现参考代码**：

```java
class Solution {
    private List<List<String>> results = new ArrayList<>();
    
    public List<List<String>> solveNQueens(int n) {
        if (n <= 0) {
            return new ArrayList<>();
        }
        
        char[][] chessboard = new char[n][n];
        
        for (int i = 0; i < n; ++i) {
            Arrays.fill(chessboard[i], '.');
        }
        
        dfs(n, chessboard, 0);
        
        return results;
    }
    
    private void dfs(int puzzleSize,
                     char[][] chessboard,
                     int currentLine) {
        if (currentLine >= puzzleSize) {
            List<String> path = new ArrayList<>();
            
            for (int i = 0; i < puzzleSize; ++i) {
                path.add(String.valueOf(chessboard[i]));
            }
            
            results.add(path);
            
            return;
        }
        
        for (int i = 0; i < puzzleSize; ++i) {
            if (validate(chessboard, currentLine, i)) {
                chessboard[currentLine][i] = 'Q';
                
                dfs(puzzleSize, chessboard, currentLine + 1);
                
                chessboard[currentLine][i] = '.';
            }
        }
    }
    
    private boolean validate(char[][] chessboard, int row, int col) {
        int n = chessboard.length;
        
        // check for the up
        for (int r = row; r >= 0; --r) {
            if (chessboard[r][col] == 'Q') {
                return false;
            }
        }
        
        // check for the up-right
        for (int r = row, c = col; r >= 0 && c < n; --r, ++c) {
            if (chessboard[r][c] == 'Q') {
                return false;
            }
        }
        
        // check for the up-left
        for (int r = row, c = col; r >= 0 && c >= 0; --r, --c) {
            if (chessboard[r][c] == 'Q') {
                return false;
            }
        }
        
        return true;
    }
}
```

# Review
分享一篇关于计算机语言的文章：  [Why should you learn Go?](https://medium.com/@kevalpatel2106/why-should-you-learn-go-f607681fad65) 

Go 语言作为一门新语言发展及其迅猛，这篇文章从各个角度给出了 Go 语言对比其他语言的优势：

- Go 让并发的开发更加的便捷也相对来说更高效，并发代码也更加简洁、直接、易懂
- 类似 C/C++，Go 也是一门编译型语言，而且相比 Java 来说，不需要解释器，编译过程中少了中间码，运行高效
- Go 没有类似类、继承、泛型、构造器、注解，这使得代码中不会含有复杂的逻辑关系，使得代码的阅读、维护、扩展在某种程度上更便捷
- Go 的背后是 Google，Go 就是由 Google 设计出来支持其全世界最庞大的云架构系统

看完了之后，一是觉得有学这门语言的欲望，二是很认同作者文中的一个观点就是，当下，如何高效写出一个能运行在绝大多数硬件之上的稳定且可扩展的软件才是值得我们深思的问题。

# Tip

Google 搜索技巧总结：

- 在某个词或句上加双引号，就表明搜索到的内容必须一个字符不漏地涵盖双引号里面的东西
- “-” 减号表示排除某些内容
- “～” 搜索和某个东西有关联的内容  
- "..." 前后加东西可以表示范围
- “@” 在社交媒体中寻找内容返回
- “site: ...” 在某个网站里搜索
- “related: ...” 找相似特质的网站
- “OR” 前后两个东西一个发生即可，这里注意 OR 要大写

举些例子：

在亚马逊和淘宝上搜索价值区间在 1000～1500 美元的手提电脑
```
site:amazon.com OR site:taobao.com laptop $1000...$1500
```

在 juejin.im 网站里搜索编程相关且必须含有 Java （大小写不区分）这个关键词的文章，但是必须排除含有 jvm 的文章
```
site:juejin.im ~programming "java" -jvm
```

搜索 Tucson 100～200 美元的住宿，排除 airbnb，要求 “靠近公交站”
```
Tucson ~budget ~accommodation $100...$200 -airbnb "nearby bus station"
```

以上列出来的只是部分技巧，其他的有待以后多总结多积累，一开始可能会觉得怪怪的，但是用的久了，可以很明显感觉这样的搜索更高效，搜出来的东西更符合自己的期待


# Share
[位运算中异或的常见用法总结](./位运算中异或的常见用法总结.md)

位运算这种东西我们可能平时不怎么会去关注，也很难有一个针对位运算系统全面的思维方式，但如果你经常阅读计算机语言源码，你会发现里面处处可见位运算，位运算相对来说更高效，个人觉得还是靠平时的思考和点滴的积累，这里我总结了位运算中相对灵活，应用面广的异或运算符，欢迎你的评论