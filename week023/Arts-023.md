## Algorithm

[LeetCode 297. Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

**题目解析**：

**序列化** 和 **反序列化** 一个二叉树，在做这道题之前，你需要了解一下什么是序列化，**序列化就是把内存里面的东西变成字符串的一个过程**，反序列化，顾名思义就是序列化的逆过程。序列化有什么作用？在代码当中往往我们会用到很多数据结构去表示数据，比如数组、链表，堆，树，图等等，每当我们需要把这些数据结构从本地的 server 发送给其他的 server 的时候，我们会遇到一个问题就是，一个数据结构是在内存中存储和表示的，我们不可能把一段内存发给对方，而且有时这个内存还是不连续的，那需要怎么让对方知晓这个数据结构以及其内部的数据呢？序列化在这里就起到了作用，发送方把内存中的对象或者说是数据按照一定形式和规则编码成字符串，接收方根据这个字符串，以及发送方那边的编码规则去进行对应的解码，将这个字符串转换为具体的数据结构。

我们先来看看数组和链表怎样做序列化，其实非常简单，比如数组 `[1,3,5,6,7]` 序列化后就会变成 `"[1,3,5,6,7]"`，当然你也可以不需要括号，只是说你的编解码器要达成共识。像数组和链表这样的线性结构比较好序列化，如果对应到二叉树上，该如何序列化？当然这里有很多的表示方式，但是有一个前提就是，**你的字符串必须和你要表示的二叉树相互唯一对应**，不然就会出现一个结构编码或者解码出多个可能的情况，这并不是我们想要的。

我这里说一下我的表示方式，其实和题目中给的方式是一样的，举几个例子你就知道了：
```
       5
      / \
     3   4              =>   {5,3,4,null,null,null,6}
          \
           6
           
       3
      /                
     1                  =>   {3,1,null,null,2}
      \
       2
       
      2
     / \                =>    {2,1,3}
    1   3
```
其实就是从上到下，从左到右，一层一层去表示，遇到左节点或者右节点是空的话，就用 null 表示，然后需要保证最后面的元素不是 null 就行。由于涉及到层级遍历，这里用到了广度优先搜索。广度优先搜索这个算法理解和实现都不是太难，这次的 share 部分我会详细去讲。

<br>

**参考代码**：
```java
public String serialize(TreeNode root) {
    if (root == null) {
        return "{}";
    }

    Queue<TreeNode> queue = new LinkedList<>();
    List<String> content = new ArrayList<>();

    queue.offer(root);
    content.add(String.valueOf(root.val));

    while (!queue.isEmpty()) {
        TreeNode cur = queue.poll();

        if (cur.left != null) {
            content.add(String.valueOf(cur.left.val));
            queue.offer(cur.left);
        } else {
            content.add("null");
        }

        if (cur.right != null) {
            content.add(String.valueOf(cur.right.val));
            queue.offer(cur.right);
        } else {
            content.add("null");
        }
    }

    while (content.get(content.size() - 1).equals("null")) {
        content.remove(content.size() - 1);
    }

    StringBuilder sb = new StringBuilder();

    sb.append("{"); sb.append(content.get(0));

    for (int i = 1; i < content.size(); ++i) {
        sb.append(",");
        sb.append(content.get(i));
    }

    sb.append("}");

    return sb.toString();
}

// Decodes your encoded data to tree.
public TreeNode deserialize(String data) {
    if (data.equals("{}")) {
        return null;
    }

    String[] tree = data.substring(1, data.length() - 1).split(",");
    
    TreeNode root = new TreeNode(Integer.parseInt(tree[0]));
    
    Queue<TreeNode> queue = new LinkedList<>();

    boolean isLeft = true;
    TreeNode cur = root;
    
    for (int i = 1; i < tree.length; ++i) {
        if (isLeft) {
            if (!tree[i].equals("null")) {
                cur.left = new TreeNode(Integer.parseInt(tree[i]));
                queue.offer(cur.left);
            } else {
                cur.left = null;
            }
        } else {
            if (!tree[i].equals("null")) {
                cur.right = new TreeNode(Integer.parseInt(tree[i]));
                queue.offer(cur.right);
            } else {
                cur.right = null;
            }
        }
        
        if (!isLeft) {
            cur = queue.poll();
        }
        
        isLeft = !isLeft;
    }
    
    return root;
}
```
<br>

## Review
[10 Small Design Mistakes We Still Make](https://uxplanet.org/10-small-design-mistakes-we-still-make-1cd5f60bc708)

medium 上面的一篇关于设计产品的原则的文章。产品最终是要给用户去使用的，因此，产品的设计是要以用户为基准，相比于技术的日益增进，用户，或者说是人的思维和习惯是变化缓慢的，因此，学习人们的行为方式有助于实现产品的功能。另外这里有一个非常重要的原则理念就是 “**Don't make me think**”，这句话是以用户的角度出发的，面对一个网页或者应用，用户最想要的是快速解决当前的问题，或者说是迅速找到他们需要的答案或者资源，他们不希望去花时间思考你这个产品的结构，理念等等，因此让用户短时间内能够有所收获，有所帮助是实现产品的一个原则，文章给出了几点建议：
* 网页布局上面必须有条理，直观，层次分明
* 不要告诉用户产品的实现细节和理念，和产品具体怎么运作
* 尽量用图等直观的方式告诉用户具体流程
* 减少应用上面模糊不清或者有歧义的内容，不要让用户去猜测

其实做开发，设计产品，归根结底还是在和用户打交道，因此理解用户的痛点才是关键所在。


<br>

## Tip
这周学到了 JavaScript 中的一个知识点 - **变量提升**，举个例子：
```js
func();
console.log(variable);
var variable = 5;
function func() {
    console.log("this is a function");
}
```
这里第一行和第二行的代码会正确执行，其中第一行的代码会执行下面声明的 func 函数，第二行的代码输出 undefined。其实 JavaScript 代码**在执行之前会有一个编译的操作**，这个操作会将代码中声明的部分放到一个上下文对象中，这个过程也是顺序的，也就是说，如果有重复声明，后面的声明会取代前面的声明，另外就是如果函数声明和变量声明重复，优先考虑函数的声明，在执行代码的阶段，遇到一个变量，程序就会去到上下文对象中搜索。这也就解释了上面的代码中出现的情况。

<br>

## Share
关于广度优先搜索的一些思考和总结

[广度优先搜索（BFS）总结分析](./广度优先搜索（BFS）总结分析)

