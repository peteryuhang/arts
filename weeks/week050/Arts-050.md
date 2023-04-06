> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 162. Find Peak Element](https://leetcode.com/problems/find-peak-element/)

**题目解析**：

题目让你找出一个数组中的 `peak element`，数组中可能存在一个或者多个 `peak element`，但是你只需要找出一个就好。

这道题目最直接的办法就是直接遍历一遍数组，然后将每个元素与其左右相邻的元素进行比较，符合条件输出即可。显而易见，这么做时间复杂度是 `O(n)`，n 为数组中元素的个数。有没有更快的方法呢？比 `O(n)` 还要快的话，一般来说只会是 `O(lgn)` 和 `O(1)`，`O(1)` 显然是不可能的，那么就只剩下 `O(lgn)`。通过这个时间复杂度，我相信你应该知道用什么样的算法，没错就是**二分查找**。

那么用二分查找怎么做呢？题目描述中有一个细节是，我们可以认为 `arr[-1] == arr[n] == -Inf`，**也就是两头的元素只需要和它相邻的一个元素比较即可**。再进一步想，这里其实还隐藏了一个信息，就是我们**二分查找顺着递增的方向去找的话就一定能够找到峰值**。如果能够分析到这里，那么这道题基本上就算是解决了。

之前看到过这道题目的一个 follow up，就是进阶版。输入不再是一个数组，而是一个 2D 矩阵，让你在这之中寻找 `peak element`，这里的 `peak element` 必须比其上下左右的元素都要大，这要怎么解决呢？类似的，你可以假设边界元素要比边界外的元素来的大。一维的解法放在二维思路也是类似的，就是按列或者行去使用二分查找，然后对应的，行或者列进行普通遍历搜索，时间复杂度就会是 `O(nlogm)` 或者 `O(mlogn)`，但是这里注意的是，我们 **普通遍历搜索找的是最大值而不是极大值**，为什么，就拿按行普通比例来说，如果你找到了这一行中的最大值，那么这个最大值也是这一行中的极大值，但是这个还不够，我们还得向上，向下比较，如果我们发现这个元素还不是二维矩阵中的极大值，我们就要顺着较大的那个方向去做行的二分，这就和一维的情况相类似，朝着大的方向去找总会找到答案，我们就是用这个方法**不断逼近**我们想要的答案的。

但是上面那个还不是最优的解法，你可以想一下，当我们找到了一行中的最大值，发现这个元素的上方或者下面有元素比它大，按照二分的思想，这个时候我们需要缩小查找范围，如果是之前的解法，这个范围的缩小仅限于行。但是如果我们也同样的对列进行遍历查找，那是不是可以缩小列的范围，这样一来行与列都应用上二分的思想进行遍历，我们可以把时间复杂度降至 `O(m + n)`。时间复杂度的证明可以使用普通等比数列求和公式得出，这里就不过多赘述。


<br>

**参考代码**：

```java
public int findPeakElement(int[] nums) {
    if (nums == null || nums.length == 0) {
        return -1;
    }
    
    int start = 0, end = nums.length - 1;
    while (start + 1 < end) {
        int mid = start + (end - start) / 2;
        
        if (nums[mid] > nums[mid - 1] && nums[mid] > nums[mid + 1]) {
            return mid;
        } else if (nums[mid] > nums[mid - 1] && nums[mid] < nums[mid + 1]) {
            start = mid;
        } else {
            end = mid;
        }
    }
    
    if (nums[start] > nums[end]) {
        return start;
    }
    
    return end;
}
```

**参考代码（Follow Up - O(nlogm)）**：

```java
public List<Integer> findPeakElementFollowUp(int[][] nums) {
    int left = 1, right = nums.length - 2;
    List<Integer> result = new ArrayList<Integer>();
    while (left <= high) {
        int mid = (left + right) / 2;
        int col = find(mid, nums);

        if (nums[mid][col] < nums[mid - 1][col]) {
            right = mid - 1;
        } else if (nums[mid][col] < nums[mid + 1][col]) {
            left = mid + 1;
        } else {
            result.add(mid);
            result.add(col);
            break;
        }
    }

    return result;
}

public int find(int row, int[][] num) {
    int col = 0;
    for (int i = 0; i < num[row].length; i++) {
        if (num[row][i] > num[row][col]) {
            col = i;
        }
    }

    return col;
}
```

**参考代码（Follow Up - O(m + n)）**：

```java
public List<Integer> find(int x1, int x2, int y1, int y2, int[][] num) {
    int mid1 = x1 + (x2 - x1) / 2;
    int mid2 = y1 + (y2 - y1) / 2;

    int x = mid1, y = mid2;
    int max = num[mid1][mid2];
    for (int i = y1; i <= y2; ++i) {
        if (num[mid1][i] > max) {
            max = num[mid1][i];
            x = mid1;
            y = i;
        }
    }

    for (int i = x1; i <= x2; ++i) {
        if (num[i][mid2] > max) {
            max = num[i][mid2];
            x = i;
            y = mid2;
        }
    }

    boolean isPeak = true;
    if (num[x - 1][y] > num[x][y]) {
        isPeak = false;
        x -= 1;
    } else if (num[x + 1][y] > num[x][y]) {
        isPeak = false;
        x += 1;
    } else if (num[x][y - 1] > num[x][y]) {
        isPeak = false;
        y -= 1;
    } else if (num[x][y + 1] > num[x][y]) {
        isPeak = false;
        y += 1;
    }

    if (isPeak) {
        return new ArrayList<Integer>(Arrays.asList(x, y));
    }

    if (x >= x1 && x < mid1 && y >= y1 && y < mid2) {
        return find(x1, mid1 - 1, y1, mid2 - 1, num);
    }

    if (x >= 1 && x < mid1 && y > mid2 && y <= y2) {
        return find(x1, mid1 - 1, mid2 + 1, y2, num);
    }

    if (x > mid1 && x <= x2 && y >= y1 && y < mid2) {
        return find(mid1 + 1, x2, y1, mid2 - 1, num);
    }

    if (x >= mid1 && x <= x2 && y > mid2 && y <= y2) {
        return find(mid1 + 1, x2, mid2 + 1, y2, num);
    }

    return new ArrayList<Integer>(Arrays.asList(-1, -1));
}

public List<Integer> findPeakElement(int[][] num) {
    int n = num.length;
    int m = num[0].length;
    return find(1, n - 2, 1, m - 2, num);
}
```
<br>

## Review

[Know Your IDE](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_45/)

这篇文章讲述了现代 IDE 对程序员的影响。有正面的，也有负面的。正面的就是，IDE 为我们写代码提供了便利，大大节约了我们的开发时间，我们不需要去查找比对，也不需要花时间记忆一些没有必要的快捷键或是指令，让我们把精力都放在思考代码的逻辑上。

但是反过来说，这往往也是不好的，因为我们太过于依赖 IDE 给我们提供的遍历，而且现代的 IDE 其实并没有什么学习曲线，往往用个一两天就会了，这会让我们放弃去学习一些有用的东西，比如 Linux 中的一些类似 `find`, `sed`, `sort`, `uniq`, 或是 `grep` 的指令，这些个指令非常的强大，能够在命令行中方便快捷地进行搜索和编辑。因此，我们还是要意识到，前期花费一些时间来学习 `vim` 或是 `emacs` 之类的命令行编辑器还是很有用的，另外就是不要完全依赖于 IDE 的普通功能，这样会让你止步不前，大胆去探索这些 IDE 的进阶功能吧，说不定你能发现很多很好玩的东西，也能增强自己的编码能力。


<br>

## Tip

在操作系统之下，有一个东西叫做 **交换空间（swap space）**，你可以把其看作是和内存相类似的存储空间。但是这个交换空间对于操作系统来讲，属于一个备胎，主要用于将部分内存中的数据换下来，以腾出内存空间用于其他需求。在一个系统中，物理内存的容量是有限的，当物理内存快使用完时，操作系统会使用交换分区（如果有的话）。当系统内存使用紧张时，操作系统根据一定的算法规则，将一部分最近没使用的内存页面保存到交换分区，从而为需要内存的程序留出足够的内存空间；在SWAP中的内存页面被访问到时，系统会将其重新载入到物理内存中去运行。

有些时候，**内存中加载的东西过多，导致程序崩溃，那么可以考虑重新给交换空间分配较大的容量**，具体指令如下：
```bash
$> sudo swapoff -a        # 首先关掉交换空间的使用
$> sudo dd if=/dev/zero of=/swapfile bs=1G count=16  # 分 16G 的容量到交换空间
$> sudo mkswap /swapfile    # 分配
$> sudo swapon /swapfile    # 启动交换空间
$> grep SwapTotal /proc/meminfo # 查看当前交换空间
```

<br>

另一个技术技巧是关于 npm 和 node 的。对于一个大的项目，或者说是长期项目，其框架的版本都比较固定，一般来说不会改变，因为改变会造成系统的不稳定。关于 npm 的版本管理，我们可以使用 nvm，全称 node version manager，它可以用于管理 npm 以及 node 的版本（通常 npm 和 node 的版本是绑定的），nvm 的安装可以跑 script 进行，当然就是跑几个指令，可以参考 [链接](https://github.com/nvm-sh/nvm#install-script)：

用 nvm 安装 npm 和 node，可以参考
```bash
$> sudo -s
$> nvm install 12.13.1      # 根据 node version 安装，注意这里是 node，不是 npm
$> exit
$> nvm use 12.13.1
```


<br>

## Share

今天就开始我们的 clean code 旅程，这次鉴赏的是 1 - 3 章

[Clean Code 之旅 - 命名与函数](CleanCode之旅-命名与函数.md)
