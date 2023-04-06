## Algorithm

[LC 32. Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/)

**题目解析**：

输入是一个字符串，找出包含有效括号组合的最长子串，并且输入字符串中只含有一种括号。括号相关的问题首先可以尝试使用 **栈** 这个数据结构去解决，至于原因，想一想应该不难理解，如果进来一个右括号，也就是 ')'，它会和之前 **最后一次遍历到的左括号** 匹配，栈的 **先进后出** 的特性保证了这一要求。对于这道题目，因为我们要求的是子串的长度，因此我们可以考虑在栈中保存 index，这样子我们不仅可以通过 index 找到对应的括号，还可以借此来求长度，我们的思路可以分为下面几步：
* 从左到右遍历输入的字符串
* 如果遇到的是 '('，意味着这并不能和前面遍历过的部分组成合法答案，此时我们只需要把当前 index 入栈即可
* 如果遇到的是 ')'，这时我们就要看栈顶保存的元素了，这里就会有几种情况：
  * 栈顶保存的是 '('，表示当前元素和栈顶元素可以配对，这个时候我们需要把栈顶元素弹出栈，记录答案则记录当前 index 和弹出配对元素后的新栈顶 index 之间的距离，这个地方是重点，如果不理解，你可以思考下面两个例子：
    ```
    "((()()"
    "((())"
    ```
  * 栈顶保存的是 ')'，如果是这种情况，表示前面没有可配对的 '('，我们此时还是需要把当前 index 入栈，原因是 **我们确定距离需要知道边界**，如果不理解，还是有两个例子供你参考：
    ```
    "))(())"
    "())()()"
    ```
  * 栈是空的，当然在第一种情况中，你弹出栈顶元素后也会使得栈变空，为了避免这种情况，我们可以在最开始的时候推一个 -1 入栈，这样可以节省我们的判断次数，并且当栈中的没有元素的时候，我们也可以用这个 -1 来计算当前子串的长度，你可以参考下面这两个例子：
    ```
    "()"
    "()(())"
    ```

如果用栈来解决的话，这道题思路差不多就是这样，这里还有一个动态规划的解法，这其实是一个 **序列型动态规划**，难点在于问题并不是特别好拆解，我们可以定义 **dp[i] 表示的是 str[0...i] 的答案**，思路其实和前面很类似：
* 从左到右遍历输入的字符串
* 如果遇到的是 '('，意味着这并不能和前面遍历过的部分组成合法答案，因为 dp 状态数组中记录的是答案，这个时候说明 dp[i] = 0，也就是不用做任何记录
* 如果遇到的是 ')'，这时我们还是需要往前看：
  * 如果 str[i - 1] 是 '('，那么 dp[i] = dp[i - 2] + 2
  * 如果 str[i - 1] 是 ')'，这表示 str[i - 1] 已经配对了，因此我们还要继续往前看，**从当前位置往左**，看第一个没有被配对的 '('，怎么找这个位置呢，这里我们就可以利用 dp[i - 1] 这个信息，dp[i - 1] 表示的是之前匹配的长度，那么：
    * **i - dp[i - 1] - 1** 表示的就是从当前位置往左，第一个没有被配对的位置
    * 如果位置 i 和 位置 i - dp[i - 1] - 1 配对后，我们可以看看当前的序列是否可以和之前匹配的序列链接起来，也就是加上 dp[i - dp[i - 1] - 2]

两种方法时空复杂度都是 O(n)，解法也有相似之处，只是说切题点不一样

<br>

**参考代码（栈）**：
```java
public int longestValidParentheses(String s) {
    if (s == null || s.length() == 0) {
        return 0;
    }
    
    int n = s.length();
    
    char[] sArr = s.toCharArray();
    Stack<Integer> stack = new Stack<>();

    int result = 0;

    // -1 入栈用于处理边界条件
    stack.push(-1);

    for (int i = 0; i < n; ++i) {
        // stack.size() > 1 表示栈不为空，而且我们必须保证栈顶元素是 '('
        if (sArr[i] == ')' && stack.size() > 1 && sArr[stack.peek()] == '(') {
            // 配对的 '(' 出栈
            stack.pop();
            
            // 记录长度
            result = Math.max(result, i - stack.peek());
        } else { // 其他情况，直接将当前位置入栈
            stack.push(i);
        }
    }
    
    return result;
}
```
<br>

**参考代码（动规）**：
```java
public int longestValidParentheses(String s) {
    if (s == null || s.length() == 0) {
        return 0;
    }
    
    int n = s.length();
    
    char[] sArr = s.toCharArray();
    
    int[] dp = new int[n];
    
    int result = 0;
    for (int i = 1; i < n; ++i) {
        if (sArr[i] == ')') {
            // 如果前一个位置是 '('，直接配对
            if (sArr[i - 1] == '(') {
                dp[i] = (i >= 2 ? dp[i - 2] : 0) + 2;
            } 
            // 前一个位置是 ')'
            // 我们从当前位置往左看，如果第一个没有被匹配的位置是 '('
            // 表明当前位置是可以被匹配的
            else if (i - dp[i - 1] - 1 >= 0 && sArr[i - dp[i - 1] - 1] == '(') {
                // 这里其实是 dp[i] = i - (i - dp[i - 1] - 1) + 1 = dp[i - 1] + 2
                // 但是我们还需要考虑之前的答案，也就是 dp[i - dp[i - 1] - 2]
                // 首先判断 i - dp[i - 1] - 2 是否越界
                // 如果没有越界就将其加上
                dp[i] = dp[i - 1] + 2;
                
                if (i - dp[i - 1] >= 2) {
                    dp[i] += dp[i - dp[i - 1] - 2];
                }
            }
            
            result = Math.max(result, dp[i]);
        }
    }
    
    return result;
}
```

<br>

## Review & Tip
[How to Publish Your First npm Package](https://medium.com/@bretcameron/how-to-publish-your-first-npm-package-b224296fc57b)

这次 review 的内容挺多，而且里面讲了挺多的技巧类的东西，于是考虑将 tip 和 review 合并在一起。

如何在 npm 上发布一个 package，之前总是用别人在 npm 上发布的 package，如果喜欢开源，不妨试着拿 npm 入手，试着自己造造轮子看看，说不定很好玩。文章一步步教你如何正确地在 npm 上发布 package，这里列一下重点：
1. 首先为你所要发布的 package 起个名字，你可以在 [npmjs.com](https://www.npmjs.com/) 上面注册账号并查看名字是否被用过，没有被用的话证明可以使用
2. 初始化你的项目，也就是你的 package 了，主要是创建一个 package.json 文件，在里面按照一定的格式填入 name、version、key word、repo URL 等等
3. 为你的 package 创建 repo 也就是代码仓库，这里推荐使用 GitHub，这样别人看你的开源代码的时候，文档中的页面链接可以直 link 到 GitHub 的 repo
4. (Tip) 作者在这个地方提到，建议使用 `__dirname` 而不是 `.`，**原因是 `dirname` 始终表示的是当前 script 所在的目录，而 `.` 则表示的是你运行 `node script` 时所在的目录**，这有时就会 confuse 用户。这边有个例外，就是 require() 函数可以使用 `.` 而不受影响。同时我们也可以使用 javascript 的 built-in 的 path 来帮助我们：
    ```
    const path = require('path');
    const my_path = path.join(__dirname, 'my-dir', 'my-file.js');
    ```
5. (Tip) 我们可以使用 module.exports 来导出函数、对象、甚至变量等等，还可以借助传入参数的 default 性质来根据用户的输入来控制行为
6. 如果你有看过 npm 上面的 package 的话，你应该也会知道，一般的 package 都会有一个 README.md 类似的网页文档供用户阅读，这里有一个 [模版](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2) 供参考，你也可以看看一些特别火的 package 的 README.md 的格式，一般说来会参照下面的格式：
    1. **Installation/Getting Started**
       
        显示如何 install package，用户如何 import package
    2. **Example Code/Quick Start**
        
        通过一个简单的例子来教用户如何快速上手使用
    3. **Contact US/Author(s)**
    
        你可以让用户知道你是谁，也可以加上 CONTRIBUTING.md，这里有个 [范例](https://www.contributor-covenant.org/version/1/4/code-of-conduct.html)
    4. **License**
    
        最后一个，也就是证书，GitHub 上面也有一些 [模版](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/licensing-a-repository)

7. 测试你的 package，作者建议测试目录的命名最后不要命名成 test，这样做是为了避免和一些框架的测试目录重名，还有就是你需要在 .gitignore 文件中添加测试目录。作者给了两个测试方法:
    * 去到测试目录下，运行:
      ```
      npm i "absolute/path/to/my/package"
      ```
    * 先去到 package 的根目录，运行 `npm link`，然后去到测试目录下运行：
      ```
      npm link my-package-name
      ```
    总的说来第二种方法更好些，特别当一个 package 中包含其他 package 的时候，但是第一种方法更贴近实际用户的使用习惯
8. 到这里，你可以发布你的 package 了，发布的步骤也相当的简单：
    1. 你可以创建 npm 的账号，你可以放上头像和自己的 GitHub 的链接来让增加些人气
    2. 在 terminal 中输入 `npm login` 登陆你的 npm 账号
    3. 去到你想要发布的 package 的根目录下，输入 `npm publish` 即可
9. 更新你的 package，push code 到代码仓库不能使你的 package 更新，你也不能覆盖之前发布过的版本，npm 上面使用 [semantic versioning](https://docs.npmjs.com/about-semantic-versioning) 作为更新的版本控制机制，npm 也有对应的版本控制指令：
    ```
    npm version patch // 1.0.1
    npm version minor // 1.1.0
    npm version major // 2.0.0
    ```
10. 在文档上面添加图标，最快捷省事的方法是使用类似 [shields.io](https://shields.io/) 第三方套件，它会为你的 markdown 文件自动生成图标




<br>

## Share
继续动态规划

[动态规划之区间类动规](./动态规划之区间类动规.md)