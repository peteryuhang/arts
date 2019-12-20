## Algorithm

[LeetCode 864. Shortest Path to Get All Keys](https://leetcode.com/problems/shortest-path-to-get-all-keys/)

**题目解析**：

非常有意思的一道搜索问题，在一个矩阵内，给定初始点，要你取得图中所有的钥匙，并输出取得所有钥匙所需要的 **最小步数**，门只有对应的钥匙才能开，另外图中还会有墙阻断路线。看到最小步数，脑袋里面马上反应是使用 **广度优先搜索**。之前说过，其实我们可以把矩阵看成是一个图，矩阵中的对应的位置就是图上的节点，每个位置和其上下左右四个位置相连，这样图上的边也就有了。

难点在于细节的把控上面，还有就是你怎么解决 “**对应钥匙开对应门**”，我们来看看下面这个例子:
```txt
. . . . . . . . . . B
. . . . # . . . . . .
. @ . A b # . . . . a
. . . . # . . . . . .
```

对于图上的遍历，不管是使用深度优先搜索，还是使用广度优先搜索，我们都会使用一个数据结构用来记录我们走过的点，根据具体的要求，这个数据结构可以是数组，也可以是 Set，目的是防止走之前的老路，如果没有这样一个数据结构，程序会无休止地运行下去。但是你看到上面的例子，我们必须去到远处拿到钥匙 a 之后，我们才可以拿钥匙 b，你会发现如果要遵循 “访问过的节点不能再继续访问” 这么一个要求，那么我们的实现思路在这里会遇到阻碍。一开始，遇到这个问题，我使用了一些数据结构去记录门还有点和点的距离，最后发现设计太复杂，程序没法写下去了。看到 discuss 里面有一个思路非常好，**我们判断一个点是不是被访问过，不仅仅看其二维坐标，还要看第三维的东西，这里的第三维的东西就是钥匙**，如果我们之前到一个位置上面只拿了两把钥匙，这时我们手里有三把钥匙，那么我们依然可以到这个位置上面去，钥匙在这里就好像是第三维的坐标一样。

因为题目里面说到最多只会出现 6 把钥匙，对于每把钥匙只有两种状态，获得和没有获得，这里还有一个技巧就是用一个整数去表示当前我们获得的钥匙，再次体会到了位运算的强大之处，**发现如果一类东西的可能的个数并不是特别大，并且每个东西只有两种状态的时候，可以考虑使用整形去表示，并用位运算进行处理**。

<br>

**参考代码**：
```java
private class Pair {
    int x, y, steps, keys;
    Pair(int x, int y, int steps, int keys) {
        this.x = x;
        this.y = y;
        this.steps = steps;
        this.keys = keys;
    }
    
    @Override
    public boolean equals(Object obj) {
        Pair o = (Pair)obj;
        if (this == o) {
            return true;
        }
        
        return (this.x == o.x && this.y == o.y && this.keys == o.keys);
    }
    
    @Override
    public int hashCode () {
        return x * 100 + y * 10 + keys;
    }
}

private int[] dirX = {0, 0, -1, 1};
private int[] dirY = {-1, 1, 0, 0};

public int shortestPathAllKeys(String[] grid) {
    if (grid == null || grid.length == 0 || grid[0].equals("")) {
        return -1;
    }
    
    int m = grid.length, n = grid[0].length();
    
    Queue<Pair> queue = new LinkedList<>();
    
    Set<Pair> visited = new HashSet<>();
    
    int totalKeysNum = 0;
    
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            char cur = grid[i].charAt(j);
            if (cur >= 'a' && cur <= 'f') {
                totalKeysNum++;
            }
            
            if (cur == '@') {
                Pair startPoint = new Pair(i, j, 0, 0);
                queue.offer(startPoint);
                visited.add(startPoint);
            }
        }
    }

    while (!queue.isEmpty()) {
        Pair cur = queue.poll();

        if (cur.keys == (1 << totalKeysNum) - 1) {
            return cur.steps;
        }
        
        for (int i = 0; i < 4; ++i) {
            int nextX = cur.x + dirX[i];
            int nextY = cur.y + dirY[i];
            
            if (nextX < 0 || nextY < 0 || nextX >= m || nextY >= n) {
                continue;
            }
            
            char c = grid[nextX].charAt(nextY);
            
            Pair nextInfo = new Pair(nextX, nextY, cur.steps + 1, cur.keys);

            if (c == '#' || (c >= 'A' && c <= 'F' && ((cur.keys >> c - 'A') & 1) == 0)) {
                continue;
            }
            
            if (visited.contains(nextInfo)) {
                continue;
            }
            
            if (c >= 'a' && c <= 'f') {
                nextInfo.keys |= (1 << c - 'a');
            }
            
            queue.offer(nextInfo);
            visited.add(nextInfo);
        }
    }
    
    return -1;
}
```
<br>

## Review
[7 Reasons Why JavaScript Async/Await Is Better Than Plain Promises (Tutorial)](https://dev.to/gafi/7-reasons-to-always-use-async-await-over-plain-promises-tutorial-4ej9)

经常使用 node 的人，应该对 **async/await** 不会陌生，虽然它只是 JavaScript 中的一个语法糖，背后的实现依然是 Promise，但是相比于 Promise，它还是有很多的过人之处。这其实就和 JSX 是一样的，在 react 中虽然你可以使用函数调用的方式去写 HTML，但是 JSX 的出现会让程序的可读性增加不少。这篇文章讲述了 async/await 的几点过人之处：
* **让代码更加的简洁**，使用过 Promise 的人都会知道，对于稍微复杂一点的逻辑，Promise 实现起来需要依靠 then 函数级联和嵌套传递变量，这么一来会让代码的可读性降低，但是 async/await 的出现改变了这一点，你不需要使用 then，只需要在异步运行的代码的前面加上 await 即可。
* 相对于 Promise，**async/await 的错误处理会更加的直接和简洁**，普通的 try...catch 包裹并不能检测 Promise 里面的异常。Promise 的异常处理需要使用 .catch() 函数，当级联和嵌套非常多的时候，出现错误我们也很难定位错误，而 async/await 的话，普通的 try...catch 依然有效，这其实最符合我们对程序代码的直观认识。
* async/await 不仅对异步代码有效，**对于同步执行的代码，使用 async/await 也不会受到任何的影响**，这个有什么用呢？现在 NPM 上面的开源库函数非常的多，有些库函数是异步执行的，有些则是同步，当你调用这些库函数的时候，你可能并不知道这个函数是异步执行的还是同步执行的，如果是异步执行的函数，它是会返回 promise，这时你可以对 promise 进行 .then() 之类的操作，但是如果是同步函数的话，你直接使用 .then() 是会报错的，你还必须将其转换成 promise 类型。async/await 完美解决了这个问题，不管是同步还是异步，async/await 都预先会将其转成 promise 的形式，并达到同步的效果，你需要做的事情其实就是在需要调用的函数前面加上 async/await 即可。

但是这里我想说的是 async/await 是很好用，但是 Promise 也不能被完全舍弃，在有些情况下我们还是得借助 Promise 里面的函数，比如需要执行一组不相干的函数，我们可以使用 **Promise.all()**，这样可以大大缩短程序运行的时间。遇到具体的问题依然要具体分析，**不能过分依赖某项技术，平时的思考以及好奇心才是兴趣和能力提升的关键**。

<br>

## Tip
这周依旧积累了不少关于 JavaScript 的知识，首先是关于 JavaScript 的上下文环境变量的：
* JavaScript 中只有 **全局上下文**、**函数上下文** 和 **eval 函数上下文**
* 上下文变量是存在栈当中，上下文变量在栈中的位置是由函数的位置决定
* 在执行上下文变量中有四个重要的东西
    * 词法环境 - 使用 const 和 let 定义的东西
    * 变量环境 - 使用 var 定义的东西
    * this 变量
    * 表示外部环境的变量
* 使用 const 和 let 的时候，如果使用在声明之前，会导致 **暂时性死区（Temporal dead zone）**，另外就是 暂时性死区会导致 typeof 不再是一个安全的操作，总之记住**先声明，再使用**
* JavaScript 是通过 **词法作用域链** 去寻找变量的，词法作用域链是由代码声明的位置决定的，它是静态作用域链，在代码阶段就决定好了，和函数怎么调用没有关系
* 根据词法作用域，内部函数可以访问外部函数声明的变量，当通过调用一个外部函数返回一个内部函数后，即使外部函数调用结束了，但是内部函数引用外部函数的变量依然保存在内存中，我们把这些变量的集合称之为**闭包**，JavaScript 引擎会依照 当前执行上下文 -> 闭包 -> 外部词法作用域 的顺序来查找变量
* 函数闭包如果作为全局变量，那么它会一直存在；可以考虑将函数闭包在局部内使用，这样可以避免不必要的内存泄漏

<br>

另外就是 JavaScript 中的 this 变量:
* 和执行上下文一样，this 也分为 3 种，全局中的 this，函数中的 this，以及 eval 函数中的 this
* 全局上下文中的 this 指向的是 window 变量，如果函数中的 this 不进行绑定的话，也指向的是 window 变量，有 3 种方式绑定函数中的 this：
    * 使用 call、apply、bind 函数进行绑定，这 3 个函数在使用上面稍有区别，可以参考下面的例子：
        ```js
        let obj = { number: 3 };
        let addTogether = function(a, b, c){
         return this.number + a + b + c;
        };
        
        // call
        console.log(addTogether.call(obj, 1,4,6));  // 14
        
        // apply
        console.log(addTogether.apply(obj, [1,4,6]); // 14
        
        // bind
        console.log(addTogether.bind(obj, 1,4,6)()); // 14
        // or
        console.log(addTogether.bind(obj)(1, 4, 6)); // 14
        ```
    * 使用对象调用其内部的一个方法，这个方法中的 this 会指向对象本身
    * 通过构造函数来设置
* 嵌套函数中的 this 并不会继承其外部函数中的 this，这其实是 JavaScript 中的一个设计缺陷，解决这个问题有两种方法：
    * 使用箭头函数，箭头函数区别于一般的函数，其并不会创建单独的执行上下文变量
    * 在外层函数中，将外层函数的 this 变量赋值给其他的变量，在嵌套函数中通过词法作用域链来访问获得
    


<br>

## Share
这周在 medium 上面发表了自己的第一篇英文文章，相对中文，感觉英文表达起来还是比较地生疏，希望自己以后多加练习

[“this” in JavaScript](https://medium.com/@peter.yuhangpeng/this-in-javascript-a8f0d9a2aaa2)