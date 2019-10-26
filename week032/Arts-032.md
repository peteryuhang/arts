## Algorithm

[LC 460. LFU Cache](https://leetcode.com/problems/lfu-cache/)

<br>

**题目解析**：

设计一个缓存，这个缓存的淘汰算法是基于里面元素的使用频率的，也就是说当新的元素进入缓存，如果缓存满了，那么该缓存中使用频率最低的元素会被淘汰，从而使得新的元素成功加入缓存。这里要求是 O(1) 时间的读取和写入。

如果接触过缓存算法的话，我想你应该对 [LRU Cache](https://leetcode.com/problems/lru-cache/) 不会特别陌生，我们可以利用一个双向链表加 HashMap 去实现，实现逻辑总结下来也非常的简单：
* 每次新加，或者读取的元素会被移到双向链表的头部
* 当缓存满了，新加元素的时候，处于双向链表末尾的元素会被淘汰

你可以看到，**如果实现一个 LRU，我们只需要考虑链表的头部和链表的尾部的操作，然后实时更新 HashMap 即可**，但是要实现一个 LFU 的话，仅仅有这些是不够的，我们还需考虑频率这个属性。我这里先把 LRU 搬出来讲主要是为了先有一个思路，如果实现一个 LFU 参照 LRU 的实现思想和结构，其实思考思路就会变成：
* 每次新加，或者读取元素，链表如何改变？
* 当缓存满了，怎么淘汰元素？

因为每次操作都要 O(1) 时间，这里的第二点，也就是淘汰元素，很容易想到，我们还是需要淘汰双向链表末尾的元素，**然后很自然就想到其实我们需要维护一个按频率从大到小排好序的双向链表**，重点来了，读取元素更新频率后如何改变链表？如果你仔细读题目的话，你会发现，题目说如果两个元素的频率相同，按 LRU 来处理，那么再一看就很明显了，其实 LFU 就是多个 LRU 拼接的产物：
```
    
    key:  1 <-> 2 <-> 3 <-> 4 <-> 5 <-> 6 <-> 7 <-> 8 <-> 9
    freq: 3     3     3     2     2     2     1     1     1
          |    LRU1      |       LRU2      |      LRU3     |
```
**这些 LRU 都有一个特点，在同一个 LRU 中的节点的频率是相同的**，我们还需要一个 HashMap 保存每个 LRU 的起始节点，因为 LRU 的添加方式是往头添加的。讲到这里其实题目思路已经说的差不多了，但是直接去实现的话还是会遇到问题，主要是情况并没有考虑完整
```
情况一： key:  i <-> target <-> j
        freq: 4       2        1        
        解释：get(target) 或者 put(target,value) 操作后，target 位置不需要改变，
             频率为 2 的 LRU 会被删除，频率为 3 的 LRU 会被创建

情况二： key:  i <-> target <-> j
        freq: 4       2        2        
        解释：get(target) 或者 put(target,value) 操作后，target 位置不需要改变，
             频率为 2 的 LRU 改变起始节点，频率为 3 的 LRU 会被创建

情况三：key:  i <-> target <-> j
       freq: 3       2        1        
       解释：get(target) 或者 put(target,value) 操作后，target 需要被移到频率为 3
            的 LRU 头部，并且删除 频率为 2 的 LRU，频率为 3 的 LRU 改变起始节点

情况四：key:  i <-> target <-> j
       freq: 3       2        2        
       解释：get(target) 或者 put(target,value) 操作后，target 需要被移到 freq 为 3 
            的 LRU 头部，频率为 3 的 LRU 改变起始节点，频率为 2 的 LRU 改变起始节点

情况五：key:  i <-> j <-> target
       freq: 3     2       2
       解释：get(target) 或者 put(target,value) 操作后，target 需要被移到 freq 为 3 
            的 LRU 头部，freq 为 3 的 LRU 需要改变起始节点
```
你可以看到，这里面的实现细节还是挺多的，但是有了**将 LFU 拆分成多个 LRU** 这样的思路和认识后，其实只用思考 **对当前频率的 LRU 怎么处理，对下一频率的 LRU 怎么处理** 即可，这些画画图应该不难。

<br>

**参考代码**：
```java
class LFUCache {
    private class Node {
        Node prev, next;
        int key, value, freq;
        Node(int key, int value, int freq) {
            this.key = key;
            this.value = value;
            this.freq = freq;
        }
    }

    private int capacity;

    private Node head, tail;

    private Map<Integer, Node> lfu;
    private Map<Integer, Node> freq;    // 记录 LFU 中每个 LRU 的起始节点

    public LFUCache(int capacity) {
        this.capacity = capacity;

        this.head = new Node(-1, -1, -1);
        this.tail = new Node(-1, -1, -1);
        
        head.next = tail;
        tail.prev = head;

        this.lfu = new HashMap<>();
        this.freq = new HashMap<>();
    }
    
    // 将 element 从当前位置移除
    private void delete(Node element) {
        element.next.prev = element.prev;
        element.prev.next = element.next;
    }
    
    // 将 element 插入到 next 节点的前面
    private void addFront(Node element, Node next) {
        element.next = next;
        element.prev = next.prev;
        next.prev.next = element;
        next.prev = element;
    }
    
    public int get(int key) {
        if (!lfu.containsKey(key)) {
            return -1;
        }
        
        Node element = lfu.get(key);
        
        int currentFreq = element.freq;
        int updatedFreq = element.freq + 1;
        
        // 对于当前 LRU，如果当前元素处于 LRU 的头，则要改变
        // 后面有相同频率的节点的话，就要将头替换
        // 没有的话，直接删除当前的 LRU
        if (freq.get(currentFreq) == element) {
            if (element.next.freq == currentFreq) {
                freq.put(currentFreq, element.next);
            } else {
                freq.remove(currentFreq);
            }
        }
        
        // 对于更新频率后的 LRU
        // 如果不存在，直接建立，建立后的 LRU 的头就是当前节点，并将这个 LRU 插入到之前的 LRU 前面
        // 如果存在，直接加到这个 LRU 头部
        if (!freq.containsKey(updatedFreq)) {
            if (freq.containsKey(currentFreq)) {
                delete(element);
                addFront(element, freq.get(currentFreq));
            }
        } else {
            delete(element);
            addFront(element, freq.get(updatedFreq));
        }
        
        element.freq += 1;
        
        freq.put(updatedFreq, element);
        
        return element.value;
    }
    
    public void put(int key, int value) {
        // 极端 case，如果 cache 容量为 0，则不能存放元素
        if (capacity == 0) {
            return;
        }
        
        // 如果元素存在，更新 value 还有频率
        if (lfu.containsKey(key)) {
            // 更新 value
            Node cur = lfu.get(key);
            cur.value = value;
            
            // 更新频率
            get(key);

            return;
        }
        
        Node cur = new Node(key, value, 1);
        
        // 如果 cache 满了，移除最后一个节点
        if (lfu.size() == capacity) {
            Node least = tail.prev;

            // 如果该节点是 LRU 的头部，因为是最后一个节点，也是尾部，直接移除这个 LRU
            if (freq.get(least.freq) == least) {
                freq.remove(least.freq);
            }
            
            lfu.remove(least.key);
            
            tail.prev = least.prev;
            least.prev.next = tail;
        }
        
        // 刚加入的元素的频率为 1
        // 如果有相同频率的元素，加入到对应 LRU 头部
        // 没有的话，直接从尾部加入
        if (freq.containsKey(1)) {
            Node next = freq.get(1);            
            addFront(cur, next);
        } else {
            addFront(cur, tail);
        }
        
        freq.put(1, cur);
        lfu.put(key, cur);
    }
}
```

<br>

## Review
[11 JavaScript Tricks You Won’t Find in Most Tutorials](https://medium.com/@bretcameron/12-javascript-tricks-you-wont-find-in-most-tutorials-a9c9331f169d)

文章给出了一些 JavaScript 中的小技巧，能够让代码变得更加清晰简洁：

* 去除数组中重复的元素：
``` javascript
const array = [1, 1, 2, 3, 5, 5, 1]
const uniqueArray = [...new Set(array)];
console.log(uniqueArray); // Result: [1, 2, 3, 5]
```
* 条件判断的妙用
``` javascript

// && 的返回值是第一个为 false 的元素，或是最后一个为 true 的元素
let one = 1, two = 2, three = 3;
console.log(one && two && three); // Result: 3
console.log(0 && null); // Result: 0

// || 的返回值是第一个为 true 的元素，或是最后一个为 false 的元素
let one = 1, two = 2, three = 3;
console.log(one || two || three); // Result: 1
console.log(0 || null); // Result: null

// 实际应用场景

return (foo || []).length;
return (this.state.data || 'Fetching Data');
```
* 添加 ! 符号可以将任意类型的变量转换为 Boolean 类型
``` javascript
const isTrue  = !0;
const isFalse = !1;
const alsoFalse = !!0;

console.log(isTrue); // Result: true
console.log(typeof true); // Result: "boolean"
```
* 转换成字符串：
``` javascript
const val = 1 + "";

console.log(val); // Result: "1"
console.log(typeof val); // Result: "string"
```
* 转换成数字
```
let int = "15";
int = +int;

console.log(int); // Result: 15
console.log(typeof int); Result: "number"

const int = ~~"15"
console.log(int); // Result: 15
console.log(typeof int); Result: "number"
```
* 指数
``` javascript
console.log(2 ** 3); // Result: 8
```
* 将浮点数快速转换成整数
``` javascript
console.log(23.9 | 0);  // Result: 23
console.log(-23.9 | 0); // Result: -23
```
* 使用箭头函数实现自动 binding
* 截取数组片段
``` javascript
let array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
array.length = 4;
console.log(array); // Result: [0, 1, 2, 3]

let array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
array = array.slice(0, 4);
console.log(array); // Result: [0, 1, 2, 3]
```
* 以数组的形式获取尾部的元素
``` javascript
let array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
console.log(array.slice(-1)); // Result: [9]
console.log(array.slice(-2)); // Result: [8, 9]
console.log(array.slice(-3)); // Result: [7, 8, 9]
```
* JSON.stringify() 的一些隐藏操作
``` javascript
console.log(JSON.stringify({ alpha: 'A', beta: 'B' }, null, '\t'));
// Result:
// '{
//     "alpha": A,
//     "beta": B
// }'
```

<br>

## Tip
这里推荐两款软件，**第一个是 [XMind](https://www.xmind.net)**，构建知识地图会很方便

![](https://user-gold-cdn.xitu.io/2019/10/26/16e051941cf60d9b?w=2078&h=1324&f=png&s=286124)

**第二个是 [Focus To Do](http://www.focustodo.cn)**，这是一款基于 “番茄计时法” 定制的软件，在上面，你可以很方便地进行计划的定制和专注时间的设定，而且之前的记录也会被保留，是提升专注力、安排计划的神器。

<br>

## Share
这次还是再聊一聊一个大多数人都关心的话题 - 面试

[谈谈 “面试”](./谈谈面试.md)