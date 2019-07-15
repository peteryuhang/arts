## Algorithm

**题目**：<br>

给定二维平面上一堆点，快速找出距离最短的点对

**解答**：<br>

如果这个题按暴力方法解，针对每个点遍历所有的点，两层循环，时间复杂度 $O(n^2)$，那有没有快一点的方法呢？如果把这里的二维改成一维，你可能很快想到要用分治法，简单的思路就是将排序好的数组分治下去计算，这样每个子问题的时间为 $O(n)$，合并的时候只需要比较两个子问题的解，加上比较跨区域相邻两点距离即可，时间复杂度 $O(n\log n)$。但是这里是二维的情况，难就难在处理合并的问题上，现在考虑一个问题就是，合并时每个点最多可以和多少个跨区域的点产生小于当前两个子问题的解，画个图你可能就明白了

![](https://user-gold-cdn.xitu.io/2019/3/22/169a2a045fb53865?w=332&h=430&f=png&s=62818)

假设两个子问题的解的共同最短距离为 $\delta=min \lbrace minLeft,minRight \rbrace$，对于合并这两个子问题时，我们可以针对其中一个子问题的点考虑另外一个子问题中的点，但是这里另外一个子问题中的点不是全部考虑，我们只需考虑如上图所示范围内的点，但是你别忘了，对于另外一个子问题的解也是小于 $\delta$ 的，由 [鸽槽原理](https://zh.wikipedia.org/wiki/%E9%B4%BF%E5%B7%A2%E5%8E%9F%E7%90%86) 结合上图可知我们对于一个点只需要最多考虑六个点（一维情况只需考虑一个点），由此看来时间复杂度可以保持在$O(n\log n)$。可以参考这道题的 [详细解析](https://blog.csdn.net/lishuhuakai/article/details/9133961)，不懂的话看完你基本就能自己直接实现了。

**实现参考代码**：
```java
private double minDistance = Double.MAX_VALUE;
private int[][] pair = new int[2][2];

private int[][] find(int[][] points) {
    if (points.length < 2) {
        return new int[0][0];
    }

    Arrays.sort(points, new Comparator<int[]>() {
        @Override
        public int compare(int[] a, int[] b) {
            return a[0] - b[0];
        }
    });

    helper(points, 0, points.length - 1);

    return pair;
}

private void helper(int[][] points, int left, int right) {
    if (left + 1 > right) {
        return;
    }

    if (left + 1 == right) {
        double currentDistance = calculateDistance(points[left], points[right]);
        if (currentDistance < minDistance) {
            pair[0][0] = points[left][0];
            pair[0][1] = points[left][1];
            pair[1][0] = points[right][0];
            pair[1][1] = points[right][1];

            minDistance = currentDistance;
        }

        return;
    }

    int mid = (left + right) / 2;
    helper(points, left, mid);
    helper(points, mid + 1, right);

    List<int[]> validPoints = new ArrayList<>();

    for (int i = left; i <= right; ++i) {
        if (Math.abs(points[i][0] - points[mid][0]) < minDistance) {
            validPoints.add(points[i]);
        }
    }

    Collections.sort(validPoints, new Comparator<int[]>() {
        @Override
        public int compare(int[] a, int[] b) {
            return a[1] - b[1];
        }
    });
    
    int size = validPoints.size();
    for (int i = 0; i < size - 1; ++i) {
        for (int j = i + 1; j < size; ++j) {
            if (validPoints.get(j)[1] - validPoints.get(i)[1] >= minDistance) {
                break;
            }

            double distance = calculateDistance(validPoints.get(i), validPoints.get(j));

            if (distance < minDistance) {
                minDistance = distance;

                pair[0][0] = validPoints.get(i)[0];
                pair[0][1] = validPoints.get(i)[1];
                pair[1][0] = validPoints.get(j)[0];
                pair[1][1] = validPoints.get(j)[1];
            }
        }
    }
}

private double calculateDistance(int[] a, int[] b) {
    return Math.sqrt((a[0] - b[0]) * (a[0] - b[0]) + (a[1] - b[1]) * (a[1] - b[1]));
}

```

**代码解析**：<br>
这里偷了个懒，在合并的时候直接对 y 做了排序操作，其实整体的时间应该是 $O(n*\log n *\log n)$，更好的方法是在分治的刚开始就将数组针对 y 排好序，在合并的时候用一个 set 去把符合 x 条件的点先集合起来，然后去先前根据 y 排序好的数组遍历一遍，找出集合中存在的元素，这样挨个找出来的结果就是根据 y 排序好的 list，接下来进行一样的操作即可。这样下来没有排序的操作时间复杂度就是 $O(n\log n)$

<br>

## Review
这周拜读了 [Beginner Programmers' Mistakes](https://jscomplete.com/learn/pro-programmer/beginner-programmers-mistakes)，这篇文章更多的是从整体上告诉你如何做好一个程序员，列出了 25 个经验尚浅的程序员容易犯的错误，从写代码到做事策略，再到方法态度，给出了很多中肯之言，里面的有些话醍醐灌顶，也引用了很多代码界的名人名言，这里我分享几个我觉得特别容易犯的错误：

* **直接不加考虑地选择第一个想到的 solution** <br>
  
  对于一个问题，永远不会只有一种思路和解法；尝试去想不同的方法，想一想某个问题有没有不一样的解法，为什么这样做可以，那样做却不行；根据需求和条件选择最合适的方法才是恰当的，这样也是有利于自己成长发展的。
  > While the first solution might be tempting, the good solutions are usually discovered once you start questioning all the solutions that you find. If you cannot think of multiple solutions to a problem, that is probably a sign that you do not completely understand the problem.
  
* **为变化的事情做计划**
  
  简单一句话就是对不确定的或者变化的需求，最好是先不写代码
  >Growth for the sake of growth is the ideology of the cancer cell.<br>
    — Edward Abbey
* **不质疑写好的代码，不对其进行重构**
  
  随着项目的迭代，之前的代码肯定会有所影响，不断去思考，去更新才能写出优质的代码
  > What is worse is that if the bad code uses bad practices, the beginner might be tempted to repeat that bad practice elsewhere in the code base because they learned it from what they thought was good code.
  
* **始终优先考虑运行效率，即使有些优化没有必要**
  
  简单的东西往往是不容易出错的，[KISS 原则](https://zh.wikipedia.org/wiki/KISS%E5%8E%9F%E5%88%99)；如果首先上来直接考虑最好的优化，最优的算法，不考虑实现以及时间成本，还有代码的维护成本，往往结果本末倒置。
  >There are two ways of constructing a software design. One way is to make it so simple that there are obviously no deficiencies, and the other way is to make it so complicated that there are no obvious deficiencies. <br>
  — C.A.R. Hoare

* **不写 unit test** <br>
  这个很好解释，如果你敢说自己写 LeetCode，刷完所有的题从来都是一次通过（自己写的），bug free, 那你可以考虑不写 test


<br>

## Tip

之前在用 ssh 命令登陆 AWS 服务器，或者是用 scp 命令传文件到 AWS 服务器的时候，都会附上自己的密匙文件 *.pem，以下的几步操作可以帮你免密使用针对该服务器的 ssh，scp 命令：

1. 去到 `~/.ssh` 目录下，这里应该会有两个文件 `id_rsa` 和 `id_rsa.pub`，前者是私有密匙，后者是公有密匙
2. vim 打开 `id_rsa.pub` 后复制文件中的所有内容
3. 第一次登陆服务器使用 *.pem，也是去到 `~/.ssh` 目录，如果是 AWS Linux 服务器的话这里会有一个 `authorized_keys` 文件
4. vim 打开 `authorized_keys` 进去 append（注意不是覆盖） 你刚刚复制的 `id_rsa.pub` 里面的内容
5. exit 退出服务器，使用无密登陆 `ssh ubuntu@ip_address` 试试看

<br>

## Share
今天分享一遍自己写的 JavaScript 中的常识性问题，当然第一次看到还是蛮新奇的，感觉和其他自己目前所知道的语言比较，还是略微不同，现在还不是特别清楚这些不一样会带来什么样的优势和不足，但相信随着自己积累的增多，会有一些和之前不一样的感悟.
<br>

[关于 JavaScript 函数传参的那点事](./关于JavaScript函数传参的那点事)
