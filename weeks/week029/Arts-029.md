## Algorithm

[LeetCode 145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)

**题目解析**：

题目要求我们进行 **树的后序遍历**，如果你还对树的各种遍历没有任何概念，或许 Wiki 上面的 [这篇文章](https://zh.wikipedia.org/wiki/%E6%A0%91%E7%9A%84%E9%81%8D%E5%8E%86) 可以帮到你，**对于后序遍历，就是把左右子树遍历完成后，才会遍历根节点**，因为根节点是最后遍历到的，所以称之为后序遍历。

这道题的难点并不在于如何实现后序遍历，而是在于**如何使用非递归的形式去实现后序遍历**，如果你习惯了使用递归去解决一些二叉树或者是深度优先搜索的题目，看到这个问题，你其实会很不自在，我第一次做这道题的时候，脑子处于空转的模式，最后没办法，只能看答案，第二次做这道题，在第一次的基础之上，知道要使用 **栈** 这个数据结构，但是里面实现细节和逻辑关联太多，因纠缠不清而迷失了方向，第三次再做这道题，算是基本了解了，结合 leetcode discuss 里面的几个帖子，总结了这一类问题的一个思考方向。你可以先不看我下面的讲解，自己思考一下，如果给你这道题，你会如何去做。

在讲解思路之前，想说说递归转非递归的意义何在？作为一个程序员应该对 stackOverflow 这个词并不感到陌生，大部分的 stackOverflow 问题都是由于递归所造成的，**总体来说，非递归的程序相比于递归的程序，稳定性会更强**，当然，递归可以让代码变得非常简洁，这也是它的一个优势。我相信对于树这个结构，在稍微有一点复杂性的项目中会经常遇到，而树的遍历又是最最基础的一个概念，如果你知道了如何使用非递归去遍历树，那何尝不是一个好的备选方案呢？另外说来，使用递归的时候其实系统为我们做了很多事情，使用非递归就需要我们自己去考虑一些底层的逻辑，这何尝不是一个了解系统运作方式的大好机会？

有一个观念是 “**解题不能仅仅是解决题目本身，我们应该思考的是，如何将一个方法或者思想推广到类似的问题上**”，对于这道题来说，我们需要将问题拓展开来，也就是将如何非递归求解二叉树后序遍历这个问题拓展到 **对于递归的实现，如何将其转化为非递归的实现**。对于递归，我们使用了系统提供给我们的函数栈，我们需要做的事情仅仅是调用函数即可。而要用非递归去实现，意味着我们必须模拟并手动创建一个函数栈，那我们就来看看当我们递归进入下一层的时候，系统为我们做了那些事情，仔细想想，其实也不多，就两件事情是我们需要考虑的：
* 保存当前层函数的运行位置
* 保存当前层函数内的可访问变量

有了这两个信息，当回到当前层函数的时候，我们就可以继续去执行我们需要执行的代码，而不受当前层函数先前被中断的影响，我们结合代码一起来分析下

<br>

**参考代码（一）**：

递归代码
```java
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<Integer>();
    
    if (root == null) {
        return result;
    }
    
    helper(root, result);
    
    return result;
}

private void helper(TreeNode root, List<Integer> result) {
    if (root == null) {             // 0
        return;
    }
    
    helper(root.left, result);      //  1
    helper(root.right, result);     //  2
    result.add(root.val);           //  3
                                    //  4 (end)
}
```

这里重点是在 helper 这个递归函数上，你可以看到，我将这个函数分成了几小块，分割原则在于 **当前函数是否被中断**，没有被中断的部分的代码是连在一起执行的，中断意味着去到其他函数执行，那这个地方就需要进行新函数的入栈操作，有了这些该如何转非递归呢？带着问题接着看

<br>

**参考代码（二）**：

非递归代码
```java
public List<Integer> postorderTraversal(TreeNode root) {
    // 主函数预处理部分
    List<Integer> result = new ArrayList<>();
    
    if (root == null) {
        return result;
    }
    
    // 用于存放函数的运行位置
    Stack<Integer> positionStack = new Stack<>();
    
    // 用于存放函数可以访问到的变量，result 变量可以直接访问到，这里只有 root 这个变量是需要传递的
    // 因此就只存放这一个变量
    Stack<TreeNode> paramStack = new Stack<>();
    
    // 相当于递归实现中, 主函数 postorderTraversal 调用递归函数 helper
    // helper(root);
    positionStack.push(0);
    paramStack.push(root);
        
    // 递归函数部分
    while (!positionStack.isEmpty()) {  // 函数栈检查是否有需要执行的函数
        int curPosition = positionStack.pop();  // 获取当前层函数的位置
        TreeNode curParams = paramStack.peek(); // 获取当前层函数的参数
        
        // 当前层函数位置移动到下一个代码块
        // 你也可以把这一步操作放在每个代码块最后结束的时候进行
        // 但是不能把这句代码放在 while 循环的最后
        // 因为有的代码块涉及新函数入栈，这个操作是对当前层函数来说的，需要在新函数入栈之前执行
        positionStack.push(curPosition + 1);
        
        if (curPosition == 0) {         // 执行代码块 0 
            if (curParams == null) {    // 如果当前参数为空，函数出栈，否则，当前层函数继续执行
                paramStack.pop();
                positionStack.pop();
            }
        } else if (curPosition == 1) {  // 执行代码块 1，新函数入栈
            positionStack.push(0);
            paramStack.push(curParams.left);
        } else if (curPosition == 2) {  // 执行代码块 2，新函数入栈
            positionStack.push(0);
            paramStack.push(curParams.right);
        } else if (curPosition == 3) {  // 执行代码块 3，记录答案，当前层函数继续执行
            result.add(curParams.val);
        } else {                        // 当前层函数运行完毕，函数出栈
            paramStack.pop();
            positionStack.pop();
        }
    }
    
    return result;
}
```

结合上面的注释，相信你不难理解代码本身，当然这里为了清晰地展示，我使用了两个栈，你也可以将它们合并成一个栈。不知道你有没有注意到，这里 **我们思考这个问题的出发点并不是树的节点和节点之间的联系以及逻辑关系，我们的重心全都放在了系统函数栈实现的层面上**，我们的目的是如何去把递归代码变成非递归代码，有了递归代码，我们甚至都不需要知道什么是树的后序遍历。用这种方法，你可以试着用非递归去实现树的中序遍历和前序遍历：
* [LC 94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)
* [LC 144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)

这些都太简单了？你也可以思考下，如果递归函数有返回值，那么该如何处理？也可以试试复杂一点的递归实现，比如排列，组合，甚至说用非递归的方式去实现一个快排：），相信我，走一遍，你会对递归的理解更加地深刻且透彻。

<br>

## Review
[How to escape async/await hell](https://medium.com/free-code-camp/avoiding-the-async-await-hell-c77a0fb71c4c)

也是一篇关于 JavaScript 中的 async/await 的文章，之前看到的关于 async/await 的文章都是讲这个东西的优点，而这篇文章的侧重点在于如何更高效地使用它。之前看专栏也了解了 async/await 其实只是 promise 的一个语法糖，**主要为了解决异步代码风格的问题**，使用 async/await 可以让程序员用写同步代码的风格来书写异步代码。但是 JavaScript 本身就是一个异步执行的语言，都用同步的方式来写的话，对性能会造成影响，文章中给出了两个 async/await 的使用的技巧，使用这两个技巧，我们可以在某些情况下提升代码性能，这两个技巧是：
* 不带 await 运行 async 函数其实会返回一个 promise 对象，我们可以延后用 await 执行这个对象
* 使用 Promise.all() 可以并行执行多个 async 函数

我们提升代码运行效率的逻辑是，将相互存在依赖关系的异步调用放在一个函数中，这样，整体上看，外围函数之间不存在依赖关系，因此可以并行执行。另外我们也可以将 promise 对象，也就是被依赖函数作为参数传入依赖函数中，这样这个依赖函数需要被依赖函数产生的结果的时候再 await 运行 promise 即可，对于循环中的 await，我们也可以利用 promise.all() 来解决，如下
```javascript
async function orderItems() {
  const items = await getCartItems()    // async call
  const promises = items.map((item) => sendRequest(item))
  await Promise.all(promises)    // async call
}
```
感觉比自己之前写的 for 循环更加地简洁了。

<br>

## Tip
这周学习了 MVC 框架的相关概念

MVC 架构有两种常见的模型：
* 第一种：controller 控制 view 和 model 的交互，流程就是 controller 接收请求，然后根据请求访问 model，最后根据 model 返回的结果生成响应让 view 去处理
* 第二种：controller 和 view 都可以访问 model，controller 负责更新 model，view 去读取 model，非常适合需要 **CQRS**(Command Query Responsibility Segregation)，即**命令查询职责分离**的场景

对于 MVC 框架，还有一些其他的变种：
* **MVP**：P 就是 presenter，它代替了 controller，但是这个模式下，请求和响应都是由 view 这边接收和发出的，presenter 可以看作是一个桥梁，连接着 view 和 model，当然，presenter 也会做一些处理和整合操作
* **MVVM**：MVP 中 P 在这里变成了 viewModel，view 和 viewModel 实现了双向绑定，形成命运共同体，也就是一方的变化会马上反馈到另一方，用于自动同步状态
    
<br>

## Share
这次还是来介绍一个数据结构 - 字典树

[字典树概念与题型解析](./字典树概念与题型解析.md)