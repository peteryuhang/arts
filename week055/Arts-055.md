> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 406. Queue Reconstruction by Height](https://leetcode.com/problems/queue-reconstruction-by-height/)（续）

**题目解析**：

回到这道题目，之前我们说的解法是排序后直接插入进动态数组，如果你还不熟悉之前的解法，可以点击 [这里](https://juejin.im/post/5e8e95f1518825738745b8e9)。之前的解法中我们将数组按照身高从大到小排好序，身高一样的话，人数少的排在前面，这么做的好处在于，后插入的元素不会影响之前插入的元素。但是我们也说了，由于插入的时候的随机性，最差情况下，将元素插入到动态数组头部导致整个数组后移，一次插入时间复杂度就是 `O(n)`，这么做会导致整个时间复杂度变成 `O(n^2)`。于是问题就变成了，时间方面是否可以继续优化呢？

由于插入到动态数组这一部分是导致时间变慢的原因，于是我们可以从这个地方入手。我们能否不用动态数组，或者说是直接用固定长度的静态数组？如果使用静态数组，那么有一点我们需要保证，那就是每次都需要将元素放置在本该属于它的位置，放置后就不会再移动。首先，如果要这么做，之前的排序规则肯定是不适用了，不理解的话你可以这么样来思考这个问题，**如果先放个子高的人，你并不清楚它前面到底需要留几个空位**。但是反过来看，如果先放矮的话，放完矮的，后面需要放置的都是比它高的，你只需要保证当前放置的元素的前面留有足够的空位即可，那到底前面该留多少空位？这就要看这个人前面有多少人比它高或者和他一样高，这个信息已经给我们了，直接拿来用就好。

上面我们知道了，这一次我们需要从矮到高放置元素，如果是一样高的话，先放前面人数多的，这个也很好解释，如果身高相同，先放人数多的可以为后面同高度的人留下空位，反过来的话就不行。排序的问题解决了，但是现在的问题是，如何插入？来看一个例子，`[[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]`，按我们先前讨论的那样排好序后就成了 `[[4,4],[5,2],[5,0],[6,1],[7,1],[7,0]]`。然后我们把静态数组开好，试着一个个往里放：

```
[[],[],[],[],[],[]]
[[],[],[],[],[4,4],[],[]]         // 最先放置最矮的人，因为前面比它高的人有 4 个，留出 4 个空位
[[],[],[5,2],[],[4,4],[],[]]      // 和先前一样，继续放置
[[5,0],[],[5,2],[],[4,4],[],[]]   // 和先前一样，继续放置
[[5,0],[?],[5,2],[?],[4,4],[],[]] // 放置 [6,1] 的时候，你会发现问题，这里的这个 1 其实
                                  // 指的是前面的空位个数，但是前面都放置了元素，元素之间有空位
                                  // 你无法在 O(1) 的时间确定要放到哪个位置
```

你可以看到，在一个个放置元素的时候，我们还是遇到了问题，为了确保前面的空位个数符合预期，看样子只能从头到位扫描一遍。这样的话，整体的时间复杂度还是 O(n^2)。这不就和之前一样吗？当然不是我们想要的。如果仔细看，你会发现我们现在需要解决的是一个区间上的问题，我们要确定某个区间的空位个数，而且这个空位个数还在不断更新（减少）。一般到这个时候，其实就要往更为高阶的数据结构上面去想。

这道题目我使用的做法是利用**线段树**，如果对线段树不了解，你可以看看之前的 [介绍](https://juejin.im/post/5e8e97ab51882573c508d363)。我们首先利用总人数构建一个线段树，线段树的每个节点表示的是一个区间，每个节点中我们存放该节点表示的区间上空位的个数。构建好树之后，我们再按之前讲的顺序一个个插入元素，因为树节点中存有区间的空位个数，因此我们可以利用这一点找到我们要插入的地方。线段树的平衡性保证了查找的时间复杂度是 `O(logn)`，这样一来我们就可以把整体的时间复杂度降至 `O(nlogn)`。

<br>

**参考代码**：

```java
// 线段树节点定义
private class SegmentTreeNode {
    // start，end 表示的是节点表示的区间
    // sum 表示的是该区间中空位的个数
    int start, end, sum;
    SegmentTreeNode left, right;
    SegmentTreeNode(int start, int end, int sum) {
        this.start = start;
        this.end = end;
        this.sum = sum;
        this.left = null;
        this.right = null;
    }
}

public int[][] reconstructQueue(int[][] people) {
    Arrays.sort(people, new Comparator<int[]>() {
        @Override
        public int compare(int[] a, int[] b) {
            if (a[0] != b[0]) {
                return a[0] - b[0];
            }
            return b[1] - a[1];
        }
    });
    
    // 首先利用人数来构建线段树
    SegmentTreeNode root = build(0, people.length - 1);
	
    int[][] result = new int[people.length][2];
    for (int[] p : people) {
        // 利用线段树的性质，逐个插入
        insert(root, p[1], result, p);
    }

    return result;
}

public SegmentTreeNode build(int start, int end) {
    if (start > end) {
        return null;
    }

    SegmentTreeNode root = new SegmentTreeNode(start, end, 0);
    
    // 到了具体的位置（叶子节点）
    // 一开始，每个位置就是一个空位，这里 sum 置为 1
    if (start == end) {
        root.sum = 1;
        return root;
    }

    // 自上而下而下递归分裂
    int mid = (start + end) / 2;
    root.left = build(start, mid);
    root.right = build(mid + 1, end);
    
    // 自下而上回溯更新
    root.sum = root.left.sum + root.right.sum;

    return root;
}

public void insert(SegmentTreeNode root, int k, int[][] result, int[] p) {
    // 找到了元素该放置的位置
    // 将元素放置到结果中去
    // 将 sum 置为 0，表示该位置不再是空位
    if (root.start == root.end) {
        result[root.start] = p;
        root.sum = 0;
        return;
    }

    // 我们数余留空位是从左向右数的
    // 如果左边区间的空位足够多，就去左边
    if (k < root.left.sum) {
        insert(root.left, k, result, p);
    } else { // 左边没有足够的空位，去右边，
             // 但是左边的空位也得算上
             // 原本要找的空位个数 - 左边的空位个数 就是我们需要再右边找的空位个数
        insert(root.right, k - root.left.sum, result, p);
    }
	
    // 每次我们都会插入一个元素，对应的空间的空位个数进行更新
    root.sum--;
}
```

<br>

## Review

[5 Secret features of JSON.stringify](https://medium.com/javascript-in-plain-english/5-secret-features-of-json-stringify-c699340f9f27)

这篇短文主要是将 `JSON.stringify` 的一些用法，有些用法确实可以让代码变得更加简洁。

`JSON.stringify` 其实目的是将 JavaScript 中的 object 转换为字符串，在 API 交互的时候经常会被用到，它的使用方式也特别的简单：

```javascript
const user = {
    name: 'pyh',
    age: 26
}
const userString = JSON.stringify(user); // {"name":"pyh","age":26}
```

当然这个函数它不止一个参数，你可以通过它的第二个参数指定你需要序列化的内容：

```javascript
console.log(JSON.stringify(user, ["name"])); // {"name":"pyh"}
```

你还可以定义你需要的方法来进行过滤：

```javascript
console.log(JSON.stringify(user, (key, value) => {
    if (typeof value === "string") {
        return undefined;
    }
    return value;
})); // {"age":26}
```

同时，想要达成这样的效果，你也可以选择在对象中写上 `toJSON()` 方法：

```javascript
const user = {
    name: "pyh", 
    age: 26, 
    hobby: "coding", 
    toJSON() { 
        return { 
            nameAge: `${this.name}  ${this.age}` 
        }; 
    },
};
JSON.stringify(user); // {"nameAge":"pyh  26"}
```

`JSON.stringify` 还有很多很有趣的用法，可以参考 MDN 的[链接](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

把常见的方法理解透彻，运用熟练自如，也是可以提高平时工作或是解决问题的效率

<br>

## Tip

这周学着用 KeyNote 做动画，比想象中的顺利。感觉用 KeyNote 做动画要比 PPT 简单许多。这里提一下 KeyNote 中的两个重要功能

* 每一张页面到下一张页面的变化过程可以加上设定让其展现动画效果
* 每张页面中的元素可以增加特效

其实认识到了这两点后就可以做出像样的动画来。看到网上很多教程讲述了很多 KeyNote 的高阶技巧，慢慢来吧，一点点学。

<br>

## Share

旷了一周，这周继续记录 Clean Code

[Clean Code 之旅 - 类与迭进](./CleanCode之旅-类与迭进.md)
