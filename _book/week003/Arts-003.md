## Algorithm

**题目**：<br>

[LC 239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)

**解答**：<br>

滑动窗口问题是面试的常考题；老样子一步步来，先考虑最暴力的解法，就是循环遍历寻找，这样的话时间复杂度需要 $O(k * n)$，其中 n 是数组的长度，k 窗子的长度。然后考虑能不能优化，很容易想到的是 sorted_map，也就是堆，这种数据结构对其中的元素的删除跟增添操作都是 $O(\log n)$，查找最值的操作时间复杂度是 $O(1)$, 如果用这种数据结构维护窗子里面的数的话，那么可以将整体的时间复杂度降至 $O(n\log k)$。进一步再想，能不能继续优化呢？这里其实有一个细节就是，在窗口滑动过程中，新进来的数如果是最大值，那么之前的数永远不可能是最大值，可以将之前的数都踢走，如果之前有比新进来的数大的数，那就按普通队列存储就行，下面的生命周期永远是最短的，这样子的话我们可以考虑双端队列这么一个数据结构，就是两头可出可进，数组里面的所有元素最多被加入一次，被删除一次，这样看的话时间复杂度是  $O(n)$，空间复杂度 $O(k)$, 从时间复杂度上看，毫无疑问这是最优的解法。代码实现部分贴了两种解法，一种使用
Java 里面的 TreeMap，另外一种是使用 ArrayDeque（从 LeetCode 上面显示的运行时间比较，比用 LinkedList 实现的 Deque 快不少）。

**实现参考代码**：

TreeMap 版本，时间复杂度：$O(n \log k)$，空间复杂度：$O(k)$
```java
public int[] maxSlidingWindow(int[] nums, int k) {
        if (nums == null || nums.length == 0 || nums.length < k) {
            return new int[0];
        }
        
        int[] result = new int[nums.length - k + 1];
        
        TreeMap<Integer, Integer> tm = new TreeMap<>(new Comparator<Integer>() {
            @Override
            public int compare(Integer a, Integer b) {
                return b - a;
            }
        });
        
        for (int i = 0; i < nums.length; ++i) {
            if (i >= k) {
                if (tm.get(nums[i - k]) == 1) {
                    tm.remove(nums[i - k]);
                } else {
                    tm.put(nums[i - k], tm.get(nums[i - k]) - 1);
                }
            }
            
            if (tm.containsKey(nums[i])) {
                tm.put(nums[i], tm.get(nums[i]) + 1);
            } else {
                tm.put(nums[i], 1);
            }
            
            if (i + 1 >= k) {
                result[i + 1 - k] = tm.firstKey();
            }
        }
        
        return result;
    }
```

<br>

Deque 版本，时间复杂度：$O(n)$，空间复杂度：$O(k)$
```java
    public int[] maxSlidingWindow(int[] nums, int k) {
        if (nums == null || nums.length == 0 || nums.length < k) {
            return new int[0];
        }
        
        ArrayDeque<Integer> dq = new ArrayDeque<>();
        
        int[] result = new int[nums.length - k + 1];
        
        for (int i = 0; i < nums.length; ++i) {
            if ((i >= k) && (dq.peekLast() == i - k)) {
                dq.pollLast();
            }
            
            addIntoDeque(i, dq, nums);
            
            if (i + 1 >= k) {
                result[i + 1 - k] = nums[dq.peekLast()];
            }
        }
        
        return result;
    }
    
    private void addIntoDeque(int index, ArrayDeque<Integer> dq, int[] nums) {
        if (dq.isEmpty()) {
            dq.offerFirst(index);
            return;
        }
        
        while ((!dq.isEmpty()) && (nums[dq.peekFirst()] <= nums[index])) {
            dq.pollFirst();
        }
        
        dq.offerFirst(index);
    }
```

<br>

## Review
在其他网友分享的 ARTS 中看到了一篇讲 Java HashMap 具体实现的文章，[How does a HashMap work in JAVA](http://coding-geek.com/how-does-a-hashmap-work-in-java/) 看完后弥补了之前认知上的不足，在这里把自己新的认知分享一下，当然还是推荐有时间的朋友可以看一看，或许你会有不一样的发现

* get()，remove()，还有 put() 均会返回 value，注意是 value，而不是 key
* Java 8 里面的 hash 函数如下：
  ```java
  // the "rehash" function in JAVA 8 that directly takes the key
  static final int hash(Object key) {
      int h;
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
  // the function that returns the index from the rehashed hash
  static int indexFor(int h, int length) {
      return h & (length-1);
  }
  ```
  这里还是要说一下的是，hash() 这个函数其实是干了三件事情
  
        1. 通过 key 自带的 hashCode() 函数得到 key 的哈希值
        2. rehashes，就是给 hashcode 从新定位，防止因为 collision 而导致的分布不均的问题
        3. rehashes 的结果作为 HashMap 里面数组的访问下标参考值(不是最终的下标)返回

   通过这两个简短的函数，有一个认识就是，HashMap 里面的内部数组大小最好是 2 的指数幂，否则会造成数组利用率太低的情况，例如 array 大小是 17 的话，那么 `length - 1` 就会是 `00010000`，进行 `h & (length - 1)` 操作后时只会生成两个不同的 index，一个是 `00010000`，另一个是 `00000000`，反过来，array 大小是 16 的话，`length - 1` 会是 `00001111`，`h & (length - 1)` 操作后会有几率生成 16 个不同的index。
  
* Java 提供了几个 HashMap 的构造函数，可参照 [HashMap Doc](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html)
    ```java
    public HashMap() {...}
    public HashMap(int initialCapacity) {...}
    public HashMap(int initialCapacity, float loadFactor) {...}
    public HashMap(Map<? extends K,? extends V> m) {...}
    ```
    使用者可以通过 initialCapacity 指定 HashMap 内部初始数组的大小，数组会自动扩容，条件由 loadFactor 来控制，阈值是 `(capacity of the inner array) * loadFactor`，超过这个值，数组会扩为之前的两倍，如果不指定这两个值，initialCapacity 的默认大小是 16，loadFactor 默认大小是 0.75；有一点需要注意的是，扩容其实就是把旧的数组中的所有的元素用之前的那两个函数(hash, indexFor)再过一遍，映射到新的数组中，之前在同一个 index 下的元素在映射过后位置可能会不同
* 多线程的情况下，最好是用 ConcurrentHashMap，而不是 HashTable，因为相比HashTable 整个操作都加锁，而 ConcurrentHashMap 锁的范围只会限制在数组的单个 index 上，就是只是在访问数组同一个 index 的情况下会被加锁，这样速度会比 HashTable 快很多
* Java 8 把数组每个 index 下的数据结构在特定的情况下变成了红黑树，对比之前的链表，查询效率要快一点，但是时间和空间往往是一个 trader-off 的关系，红黑树的实现空间要比链表大一些
    ``` java
    // in the way of linked list
    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
        …
    }
    
    // in the way of red-black tree
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        final int hash; // inherited from Node<K,V>
        final K key; // inherited from Node<K,V>
        V value; // inherited from Node<K,V>
        Node<K,V> next; // inherited from Node<K,V>
        Entry<K,V> before, after;// inherited from LinkedHashMap.Entry<K,V>
        TreeNode<K,V> parent;
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;
        boolean red;
        … 
    }
    ```
 
* HashMap 最核心的部分其实是 hash function，这个函数很大程度上决定了里面的元素能不能分布均匀，这当然得通过具体问题来进行设计，使用者在设计自己的 Object 是，可以实现里面自带的 hashCode() 函数来根据实际情况优化自己的设计

<br>

## Tip

Markdown 大家都很熟悉，最近写 ARTS 才开始用 Markdown，用好了确实很方便，包括 Markdown 其实是兼容 HTML 的，HTML 的一些 TAG 也可以根据需求去使用，这里列了些之前没有想到过，但是很实用的 Markdown 技巧：

* 嵌套引用<br>
  `>>`
* 嵌套列表
    ```
    1. 第一层
         + 1-1
         + 1-2
           + 1-2-1
           + 1-2-2
    ```
    无序列表和有序列表可以随意互相嵌套
 
* 意大利斜体 <br>
   `*content*`
* Markdown 不支持指定图片的显示大小，但是可以通过 `<img />` 标签来指定，例如 <br>
  `<img src="https://avatars2.githubusercontent.com/u/3265208?v=3&s=100" alt="GitHub" title="GitHub,Social Coding" width="50" height="50" />`
* 删除线 <br>
  `~~content~~`
* 表格，使用 | 来分隔不同的单元格，使用 - 来分隔表头和其它行：
  ```markdown
  name | age
  ---- | ---
  Yu   | 25
  che  | 23
  ```
  * :--- 表示左对齐
  * :--: 表示居中对其
  * ---: 代表右对齐 
 <br>

  另外就是斜体，加粗，链接之类的语法标记也可以在表格中使用
* 任务标记
    ```
    - [ ] task1
    - [x] task2    标记任务已完成
        - [ ] task2-1   子任务
        - [ ] task2-2
    - [ ] task3
    ```


<br>

## Share
这周分享之前写过的关于背包问题的算法题目，给出了一些个人的理解和题目的分析，欢迎一起讨论和分享
<br>

[常见背包问题解法分析](./常见背包问题解法分析.md)