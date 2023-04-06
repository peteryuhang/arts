## Algorithm

[LeetCode 25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

**题目解析**：

链表相关的题目，是一个反转链表题目的变形。题目要求在一个链表中以 k 个链表节点为单位进行反转，什么意思？你可以想象把一个很长的链表分成很多个小链表，每一份的长度都是 k (最后一份的长度如果小于 k 则不需要反转)，然后对每个小链表进行反转，最后将所有反转后的小链表按之前的顺序拼接在一起。其实**链表的题目并不需要特别强的逻辑推理，它主要强调细节实现，难也是难在细节实现上面**，虽然大致的方向知道，但是很可能写着写着就会乱。这个题目实现的时候要把握住几个要点，第一，在反转子链表的时候，上一个子链表的尾必须知道，第二，下一个子链表的头也必须知道，第三，当前反转的链表的头尾都必须知道，只要把握住了这三个要点，实现的时候耐心点应该问题不大

<br>

**参考代码**：
```java
public ListNode reverseKGroup(ListNode head, int k) {
    if (head == null || head.next == null || k <= 1) {
        return head;
    }
    
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode pointer = dummy;
    
    while (pointer != null) {
        // 记录上一个子链表的尾
        ListNode lastGroup = pointer;
        
        int i = 0;            
        for (; i < k; ++i) {
            pointer = pointer.next;
            if (pointer == null) {
                break;
            }
        }
        
        // 如果当前子链表的节点数满足 k, 就进行反转
        // 反之，说明程序到尾了，节点数不够，不用反转
        if (i == k) {
            // 记录下一个子链表的头
            ListNode nextGroup = pointer.next;
            
            // 反转当前子链表，reverse 函数返回反转后子链表的头
            ListNode reversedList = reverse(lastGroup.next, nextGroup);
            
            // lastGroup 是上一个子链表的尾，其 next 指向当前反转子链表的头
            // 但是因为当前链表已经被反转，所以它指向的是反转后的链表的尾
            pointer = lastGroup.next;
            
            // 将上一个链表的尾连向反转后链表的头
            lastGroup.next = reversedList;
            
            // 当前反转后的链表的尾连向下一个子链表的头
            pointer.next = nextGroup;
        }
    }
    
    return dummy.next;
}

private ListNode reverse(ListNode head, ListNode tail) {
    if (head == null || head.next == null) {
        return head;
    }
    
    ListNode prev = null, temp = null;
    while ((head != null) && (head != tail)) {
        temp = head.next;
        head.next = prev;
        prev = head;
        head = temp;
    }
    
    return prev;
}
```
<br>

## Review
[JavaScript ES6 — write less, do more](https://medium.com/free-code-camp/write-less-do-more-with-javascript-es6-5fd4a8e50ee2)

JavaScript 最大的一次变迁就是从 ES5 到 ES6，整个 JavaScript 的代码风格都发生了很大的变化，这篇文章给出了几个主要的变化，这里列出总结：
* **let 和 const**，早起的 JS 中，定义变量只会使用 var 这个关键字，用这个关键字定义的变量的作用域是函数作用域，顺便简单提及下，在 ES5 中，对于变量来说没有块作用域。**let 和 const 定义的变量拥有了块作用域，其中使用 const 声明的变量不能够再被重新赋值**。
* **箭头函数**，这也是 ES5 和 ES6 代码风格各异的一个主要原因，箭头函数使得函数的使用和定义变得更加的便捷，如果函数足够简单，连 return 关键字都不需要，如下：
    ```js
    const myFunc = name => `hello word ${name}`;
    ```
* **模版字符串**，就是上面的例子中输出的 String，我们可以使用两个 ` 符号，相比于引号，其更好地支持**变量名引入和换行**。
* **初始化参数**，其实这个在很多语言中都有，就是给参数一个默认值
* **数组和对象的拆解赋值**，Javascript 中的 object，也就是 Python 中的 dict，Java 中的 HashMap，一般将 object 中的东西提取出来，重新声明成变量的形式，需要反复地从对象中查找赋值，现在 ES6 的这个功能解决了这一困扰，访问 object 中的 key,value 对，我们不再需要 . 符号：
    ```js
    const { age, name, career, family, life } = person;
    ```
    和对象一样，数组同样支持这样的功能，只不过取出的元素是按顺序排列好的：
    ```js
    const array = ['1','2','3','4'];
    const { v1, v2, v3 } = array;
    console.log(v1); // '1'
    console.log(v1); // '2'
    console.log(v1); // '3'
    ```
* **import** 和 **export**，这也是非常强大的一个功能，import 就是将其他的 module 引入，export 是将某个 class 或者是函数导出，一个文件中可以 export 多个函数或者 class，以及变量，同时，在 import 的时候也可以使用之前提到的对象拆解的方式引入。

另外的一些比较耳熟能详，比如说是 promise，class，这里就不做过多的提及。

<br>

## Tip
这周学习了网络中的 IP 地址的几个相关知识，通常我们看到有些 IP 地址经常在不同的地方出现，比如 192.168.0.10。这里有一个 **保留网段** 的概念，就是国际标准组织划分了一些 IP 地址出来，这些 IP 地址不用做公网的 IP，而仅仅保留做内部网通信的时候使用。

192.168.2.12/24 这样的表示方法经常可以见到，这里 / 后面的 24 表示的是 **子网掩码**，其实这里的例子中的 192.168.2.12 并不是真实的 IP 地址，这个值可以拆解成 **主机地址** 和 **网络地址（IP 地址）**，子网掩码就是起到拆分的作用，24 表示 32 个 bits 中有 24 个 1，8 个零，表示成 10 进制就是 255.255.255.0，用这个值去和 192.168.2.12 去做与运算，得到的答案就是真实的网络地址是 192.168.2.0

<br>

## Share
这周把 LeetCode 上面的一些和二分查找相关的题目做了一下，还是挺多的，这里写了个总结

[二分查找模版、题型全面解析](./二分查找模版、题型全面解析.md)

