## Algorithm

**题目一**：<br>

[LC 146. LRU Cache ](https://leetcode.com/problems/lru-cache/)

**解答**：<br>

设计一个 LRU 缓存，由于缓存容量有限，因此存在末尾淘汰，容量满了，最不常用的（Least Recently Used）元素会被移出缓存。要保证查询和增添元素的时间复杂度都是 $O(1)$。这道题看了一下网上的解法，貌似用数组也是可以实现的，但是数组里面还要存一个变量来表示该元素的新旧程度，当然时间复杂度的话就不是 $O(1)$ 的了。常规的实现方法是双向链表加上散列，当然对于 Java 来说还可以使用 LinkedHashMap 这样的数据结构，就会让实现变得非常简单，其实 LinkedHashMap 内部已经帮你实现好了很多东西，你不用在手动实现，这里还是用最原始的方法来一步一步实现：

**实现参考代码**：

```java
private class Node {
    Node next, prev;
    int key, value;
    Node(int key, int value) {
        this.key = key;
        this.value = value;
    }
}

private Node head;
private Node tail;
private int capacity;
private Map<Integer, Node> map;

public LRUCache(int capacity) {
    head = new Node(-1, -1);
    tail = new Node(-1, -1);
    tail.prev = head;
    head.next = tail;
    
    this.capacity = capacity;
    
    map = new HashMap<Integer, Node>();
}

public int get(int key) {
    if (!map.containsKey(key)) {
        return -1;
    }
    
    Node cur = map.get(key);
    
    deleteNode(cur);
    add2Head(cur);
    
    return cur.value;
}

public void put(int key, int value) {
    if (map.containsKey(key)) {
        Node cur = map.get(key);
        cur.value = value;
        deleteNode(cur);
        add2Head(cur);
    } else {
        if (map.size() == capacity) {
            map.remove(tail.prev.key);
            deleteNode(tail.prev);
        }

        Node newNode = new Node(key, value);

        add2Head(newNode);

        map.put(key, newNode);            
    }
}

private void add2Head(Node node) {
    node.next = head.next;
    head.next.prev = node;
    node.prev = head;
    head.next = node;
}

private void deleteNode(Node node) {
    node.prev.next = node.next;
    node.next.prev = node.prev;
}
```

<br>

## Review
在 medium 上的一篇关于 Unit test 断言的文章:<br>

[Rethinking Unit Test Assertions](https://medium.com/javascript-scene/rethinking-unit-test-assertions-55f59358253f)

测试对于代码的重要性不言而喻，其中断言是一个测试的关键，作者在文章的开始就给出，对于每一个 unit test 我们都要去问这 5 个问题：

* 这个测试测试的是什么东西（函数、模块、类、等等）？
* 那这个东西的功能是什么？
* 实际的输出是什么？
* 期望的输出是什么？
* 怎样让失败重现？

文章中指出断言越简单越好，最好就只用 deep equal，其他的复杂的断言形式没有必要，而且会把 unit test 弄得过于复杂，后期根据需求变化更改 test 的话就会不太容易


## Tip
这次分享两个更改 Mac 工作环境的小设置：

1. 去到 Mac 系统设置的 keyboard 栏把 “Key Repeat” 和 “Delay Until Repeat” 两项都设置成最快和响应时间最短，好处是打字和敲代码变得比之前流畅不少，而且用上下左右移动光标的时候更加的快捷和方便

2. 使用 iTerms + Oh-my-zsh 去更改 Terminal 的显示问题，Mac 自带的 Terminal 没法显示 git 的分支之类的信息，设置完成之后感觉比之前看的更加的美观和舒服，配置教程如下:

<br>

[iTerm2 + Oh My Zsh 打造舒适终端体验](https://segmentfault.com/a/1190000014992947)

效果图：
![](https://user-gold-cdn.xitu.io/2019/4/28/16a618cb6590d8fb?w=1281&h=587&f=png&s=680313)

<br>

## Share
股票买卖问题是 LeetCode 的一个系列问题，这里总结了一个动态规划的状态定义的框架，用来解决这一系列的问题
<br>

[股票问题汇总](./股票问题汇总)