## Algorithm
这次说说这周 LeetCode Contest 中两道比较有价值的题目：

[LeetCode 1106. Parsing A Boolean Expression](https://leetcode.com/problems/parsing-a-boolean-expression/)

[LeetCode 1104. Path In Zigzag Labelled Binary Tree](https://leetcode.com/problems/path-in-zigzag-labelled-binary-tree/)

**思路分析（题目一）**<br>
给定一个字符串逻辑表达式，里面用字符 t 表示 true，f 表示 false，另外有三种逻辑符号 &、|、!，分别表示与或非，问最后表达式输出的结果，刚看到题目的时候我觉得这道题特别简单，用栈应该很好解决，但是写着写着问题就来了，发现要保存的东西不仅仅是 boolean 字符，还有逻辑符号，不然的话会造成信息缺失。于是我就想到了用两个栈来保存信息，一个栈存逻辑符号，另外一个栈存 boolean 字符，当我们遇到字符 ')' 的时候就向前计算计算到非 boolean 字符为止

<br>

**参考代码（题目一）**
```java
public boolean parseBoolExpr(String expression) {
    Stack<Character> operation = new Stack<>();
    Stack<Character> tf = new Stack<>();
    
    char[] exprArr = expression.toCharArray();
    
    int index = 0;
    
    while (index < exprArr.length) {
        if (exprArr[index] == '!' || exprArr[index] == '|' || exprArr[index] == '&') {
            operation.push(exprArr[index]);
            tf.push(exprArr[index]);
        }
        
        if (exprArr[index] == 't' || exprArr[index] == 'f') {
            tf.push(exprArr[index]);
        }
        
        if (exprArr[index] == ')') {
            boolean cur = tf.pop() == 't' ? true : false;
            char oper = operation.pop();
            while (tf.peek() != oper) {
                if (oper == '|') {
                    cur |= tf.pop() == 't' ? true : false;
                } else if (oper == '&') {
                    cur &= tf.pop() == 't' ? true : false;
                }
            }
            
            if (tf.pop() == '!') {
                cur = !cur;
            }
            
            char res = cur ? 't' : 'f';
            tf.push(res);
        }
        
        index++;
    }
    
    return tf.peek() == 't' ? true : false;
}
```

<br>

**思路分析（题目二）**<br>
这道题纠结了很久，看似二叉树问题，其实本身并没有考传统的二叉树递归遍历之类的东西，而是考的偏数学方面的知识。**一般的满二叉树，如果对节点从上到下，从左到右编号，从编号上看，你会发现左节点和父节点的关系是左节点的编号乘上 2 就是父节点的编号，右节点编号减一乘上 2 就是父节点的编号，那反过来说子节点的编号除以 2 都会是父节点的编号（利用程序中的整除性质）**，如果这道题不反转，就会很简单，直接一个递归或者循环就出来了，但是反转之后，一时半会很难发现规律，这里我们其实可以通过每一行的最大值和最小值来判断，每一行中的节点的理想父节点（在顺序编号下）在上一行中都是位于对称位置，这样我们就可以通过当前节点距中心节点的距离来获知当前行，当前节点的对称节点，然后对称节点除以 2 就是我们当前节点的父节点，以此类推，一个循环就出来了，代码实现一点也不复杂，但是规律难找

<br>

**参考代码（题目二）**
```java
public List<Integer> pathInZigZagTree(int label) {
    List<Integer> result = new ArrayList<>();
    
    int curNode = label;
    
    int curRowMax = 1;
    while (curRowMax < curNode) {
        curRowMax = curRowMax * 2 + 1;
    }
    
    int curRowMin = curRowMax / 2 + 1;
    
    while (curNode > 0) {
        result.add(curNode);
        curNode = (curRowMax - curNode + curRowMin) / 2;
        curRowMax = curRowMin - 1;
        curRowMin = curRowMax / 2 + 1;
    }
    
    Collections.reverse(result);
    
    return result;
}
```

<br>


## Review
这周没有特别读英文文章，主要时间都花在反复读 **Professional: JavaScript for Web Developers** 的第三版原版第六章<br>

[Professional: JavaScript for Web Developers](http://www.myedocs.com/onlinefiles/ebooks/Professional%20JavaScript%20for%20Web%20Developers,%203rd%20Edition.pdf)

感悟和总结都写在了这一期和前一期的 ARTS 的 Share 部分，这里就不做赘述。

<br>


## Tip
这周在调试前端的时候发现一个问题，就是 CROS 跨域访问的问题，具体没太了解，但是发现了一篇讲述这种问题解决方案的文章
[ajax跨域，这应该是最全的解决方案了](https://dailc.github.io/2017/03/22/ajaxCrossDomainSolution.html)，按照文章中的指示，我在后台 NodeJS 中 app.js 中添加了如下代码：

```javascript
app.all('*', function(req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "*");
    res.header("Access-Control-Allow-Methods", "*");
    res.header("X-Powered-By", ' *')
    //这段仅仅为了方便返回json而已
    res.header("Content-Type", "application/json;charset=utf-8");
    if(req.method == 'OPTIONS') {
        //让options请求快速返回
        res.sendStatus(200); 
    } else { 
        next(); 
    }
});
```
全部打星号，感觉虽然解决了问题，但是还有点暴力，以后再做优化吧，至少清楚了解决类似问题的方法，自己还是需要去了解 CROS 的具体原因，以及浏览器为什么要这么设计，这件事已提到日程上了。


<br>

## Share
JavaScript 中的对象，这周讲讲继承和多态的实现

[JavaScript 中的对象（二）- 继承与多态](./JavaScript 中的对象(二)-继承与多态.md)
