**题目**：
<br>

[LeetCode 51. N-Queens](https://leetcode.com/problems/n-queens/)

**分析**：<br>

N 皇后问题是考查递归回溯的经典问题，深度优先搜索的难点在于如何剪枝，在这个问题里面的剪枝，我们需要利用额外的空间去记录当前行的有效空位，只要一行当中找不到有效空位，递归将不会继续下去。一般的解法是开一个 n*n 的数组，优化一点解法是利用三个数组，分别表示左上，上，右上对当前位置的影响情况，这里有一个点就是一个位置会对其左下，下，右下产生影响，其中，下最好记录，即当前空位的列，左下有一个规律是，`row + col` 的和是固定的，例如，(0,3),(1,2),(2,1), 它们的行号和列号加起来都是一个相同的值，右下的规律是`row - col`，或者 `col - row` 是固定的，看图也很好理解，往右下的话，行列都会递增，而且每次递增幅度都是 1，Java 代码如下：


```java
public List<List<String>> solveNQueens(int n) {
    if (n <= 0) {
        return new ArrayList<>();
    }
    
    boolean[] col = new boolean[n];     // 上
    boolean[] pie = new boolean[2 * n]; // 左上
    boolean[] na = new boolean[2 * n];  // 右上
    
    List<String> result = new ArrayList<>();
    List<List<String>> results = new ArrayList<>();
    
    helper(results, result, col, pie, na);
    
    return results;
}
    
private void helper(List<List<String>> results,
                    List<String> result,
                    boolean[] col,
                    boolean[] pie,
                    boolean[] na) {
    if (col.length == result.size()) {
        results.add(new ArrayList<String>(result));
        return;
    }
    
    int currentRow = result.size();
    for (int i = 0; i < col.length; ++i) {
        int pieId = currentRow + i, naId = currentRow - i + col.length;
        
        if (!col[i] && !pie[pieId] && !na[naId]) {
            char[] row = new char[col.length];
            Arrays.fill(row, '.');
            row[i] = 'Q';
            
            result.add(new String(row));
            col[i] = true; pie[pieId] = true; na[naId] = true;
            
            helper(results, result, col, pie, na);
            
            col[i] = false; pie[pieId] = false; na[naId] = false;
            result.remove(result.size() - 1);
        }
    }
}
```

**优化**：<br>

这里开始引入我们今天的重点，那就是能不能不用额外的空间进行记录；可以假想 N 其实没多大（LeetCode 的 testcases 也只有 9 个），这道题的深度优先搜索的时间复杂度其实可以看成指数幂，因此 N 不会特别大；这里有个非常巧妙的点就是利用位运算，是怎么想到的呢？首先要明确一点就是，一个空位只有两种状态，即有皇后和无皇后，我们之前是利用 boolean 数组，这个很直观很容易想到，但是用三个整数来代替上面的三个 boolean 数组是否可行？在 N 不是特别大的情况是完全可行的，以 Java 为例，我们可以用 int 中的 32 个 bits 来分别表示上面代码中的 boolean 的每个空位，代码如下，这里 0 表示有效，1 表示无效:

```java
private List<String> result = new ArrayList<>();
private List<List<String>> results = new ArrayList<>();

public List<List<String>> solveNQueens(int n) {
    if (n <= 0) {
        return new ArrayList<>();
    }
    
    dfs(n, 0, 0, 0);
    
    return results;
}

private void dfs(int n, int col, int pie, int na) {
    if (n == result.size()) {
        results.add(new ArrayList<String>(result));
        return;
    }
    
    // 获得所有的有效空位
    // (col | pie | na) 可以得到所有被占的空位，取反之后将有效空位置为 1
    // 与上 (1 << n) - 1，是设定考虑范围，比如 8 皇后，那么只用考虑低 8 位即可
    int bit = ((~(col | pie | na)) & ((1 << n) - 1));
    
    // bit > 0 表示有空位
    while (bit > 0) {
        // 选择最低位的一个空位
        int tmp = bit & (-bit);
        
        // 构建当前行的答案
        String str = constructString(tmp, n);
        result.add(str);
        
        // col | tmp 是将 col 中当前选择的这一列置为 1，也就是无效
        // (pie | tmp) << 1 是设置之前行和当前行对左下的影响
        // (na | tmp) >> 1 是设置之前行和当前行对右下的影响
        dfs(n, col | tmp, (pie | tmp) << 1, (na | tmp) >> 1);
        
        result.remove(result.size() - 1);
        
        // 将当前选择的这个最低位置为 0
        bit &= bit - 1;
    }
}

private String constructString(int i, int n) {
    char[] row = new char[n];
    
    Arrays.fill(row, '.');
    int tmp = 1, indx = 0;
    
    while (i != 0) {
        if ((tmp & i) != 0) {
            row[indx] = 'Q';
        }
        
        i >>= 1;
        indx++;
    }
    
    return new String(row);
}
```



这样不但是节省了空间，而且从时间上来看，它只考虑了有效空位，和之前代码的 for-loop 遍历一行当中所有位置来比较的话，也有提升