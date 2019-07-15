## Algorithm

[LeetCode 140. Word Break II](https://leetcode.com/problems/word-break-ii/)

[LeetCode 472. Concatenated Words](https://leetcode.com/problems/concatenated-words/)

**题目一：思路分析**<br>

也是一道比较经典的回溯问题，在搜索的方式上，有两种思路，第一种思路是基于字符串的字符，另外一种是根据词表里面的单词，显然是第二种更优，代码也会更加简洁，但是这一题的难点在如何把暴力搜索变成记忆化搜索，也就是我们通常说的动态规划，如果你画出递归树的话，你可以明显看到这里是有重复子问题的，这样，问题就变成了原问题和子问题的关系，以及子问题的解如何表示？因为考虑到输入字符串或者说输入字符串的子串是允许多个拆分可能性的，因此这里我们考虑用一个 Map 去存储当前子串对应的所有可能的拆分情况，然后当前考虑的单词直接加到这些子串之前，就是当前的解，利用递归回溯更新，最后返回的就是答案，但是这里有一点需要注意的是，每当传入的字符串是空的时候，表示之前的单词是最后一个单词，这时需要在返回的 list 中添加一个空串占位，表示递归前面的函数传进来的 word 是可行的。

**题目一：参考代码**
```java
public List<String> wordBreak(String s, List<String> wordDict) {
    if (s == null || s.length() == 0) {
        return new ArrayList<String>();
    }
    
    Map<String, List<String>> hash = new HashMap<>();
    
    List<String> path = helper(s, wordDict, hash);
    
    return path;
}

private List<String> helper(String remain,
                            List<String> wordDict,
                            Map<String, List<String>> hash) {
    if (remain.equals("")) {
        List<String> l = new ArrayList<String>();
        l.add("");
        return l;
    }
    
    if (hash.containsKey(remain)) {
        return hash.get(remain);
    }
    
    List<String> path = new ArrayList<>();
    for (String word : wordDict) {
        boolean isStartsWith = remain.startsWith(word);
        
        if (isStartsWith) {
            List<String> subPath = helper(remain.substring(word.length()), wordDict, hash);

            for (String sub : subPath) {
                if (sub.equals("")) {
                    path.add(word);
                } else {
                    path.add(word + " " + sub);
                }
            }
        }
    }
    
    hash.put(remain, path);
    
    return path;
}
```
<br>

**题目二：思路分析**<br>

这道题和前面那道题非常的类似，区别就是输入参数上，这里给的就是一个词表，字符串和词表合二为一了，需要查找的字符串在词表里面。完全可以按照上面那道题的解法，就是以记忆化搜索的形式去做动态规划，但是这一题想想其实没有必要像之前那样使用非常多的字符串操作，字符串本身是不可变的，过多的字符串操作会对时间效率产生影响，使用 **StringBuilder** 这样的函数又会使代码复杂，题目要求我们仅仅是找出符合的单词，于是这里考虑使用**字典树**，首先根据单词列表构建字典树这里就不用说了，这里唯一需要注意的一点是，需要考虑词表中出现空串的情况。然后后面就是每个单词去字典树中走一遍，找到了符合要求的单词就让字典树从头开始，count + 1，但是注意传入参数和递归出口的关系，也就是字典树节点和当前字符的关系，如果像我的代码一样，字典树节点和当前字符滞后一位，那么函数出口应该考虑，如果 index 出了单词，就考虑当前节点是不是一个有效的节点，如果有效，并且 count >= 1，那么这个单词就是符合要求的。


**题目二：参考代码**
```java
private class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isWord = false;
}

private void buildTree(String[] words) {
    TrieNode pointer = root;
    
    for (String word : words) {
        if (word.equals("")) {
            continue;
        }
        
        pointer = root;
        
        for (char c : word.toCharArray()) {
            if (pointer.children[c - 'a'] == null) {
                pointer.children[c - 'a'] = new TrieNode();
            }
            
            pointer = pointer.children[c - 'a'];
        }
        
        pointer.isWord = true;
    }
}

private TrieNode root = new TrieNode();

public List<String> findAllConcatenatedWordsInADict(String[] words) {
    if (words == null || words.length == 0) {
        return new ArrayList<String>();
    }
    
    buildTree(words);
    
    List<String> results = new ArrayList<>();
    for (int i = 0; i < words.length; ++i) {
        if (helper(root, words[i].toCharArray(), 0, 0)) {
            results.add(words[i]);
        }
    }
    
    return results;
}

private boolean helper(TrieNode cur, char[] word, int count, int index) {
    if (cur == null) {
        return false;
    }
    
    if (index == word.length) {
        if (count >= 1 && cur.isWord) {
            return true;
        } else {
            return false;
        }
    }
    
    if (cur.isWord
           && helper(root.children[word[index] - 'a'], word, count + 1, index + 1)) {
        return true;
    }
    
    return helper(cur.children[word[index] - 'a'], word, count, index + 1);
}
```

<br>

## Review
一篇关于学习编程的建议性文章:<br>

[The main pillars of learning programming — and why beginners should master them](https://medium.freecodecamp.org/the-main-pillars-of-learning-programming-and-why-beginners-should-master-them-e04245c17c56)

* **TDD（Test-Driven-Developement）测试驱动开发**<br>
  新手往往并不知道测试的重要性，他们可能会觉得测试有时很烦人，不写测试能够大大地节省开发效率，但是测试，特别是单元测试是对代码安全的保障，防止代码在日后带来严重的问题。如果在一开始保持写代码就必须有测试的意识，是对后面的学习新知识很有帮助的，养成良好的习惯真的很重要。

* **基础优先**<br>
  函数、变量、条件判断以及循环是所有程序的基础，必须把这些基础性的，特别是很多认知性的东西弄清楚，并且赋予足够的、全面的练习后，再去考虑一些应用层面的学习；在这里必须求稳，而不是求快。

* **学习使用库和框架**<br>
  对于新手来说，一开始肯定是需要使用别人写的代码，能够基本理解这个库的功能和用途，使用正确的框架做正确的事情，这样能够提高新手的学习以及开发效率，不然新手就会对编程失去兴趣，觉得太难，独自造轮子的事情是对于经验丰富的人来说的

* **找一个好老师**<br>
  有时候别人的一句话等于自己想好几天，有一个好的老师带领，确实对学习很有帮助，也会让你开始的路好走很多

* **挑战和动机**<br>
  继续学习，持续学习的动力是源于自己内心的动机，以及外界的挑战，这两个因素都会督促你去想方设法去提高自己的能力和认知，我们要时刻保持上进的精神，同时我们也要明白自己当前是不是在舒适区内，勇于走出舒适区，主动去寻找自己的下一个挑战

<br>

## Tip
初识 Chrome 浏览器的 Network 面板

分成五大部分：
 * **控制器** -> 用于控制
    * 控制抓包（开始、停止、清除请求）
    * Preserve log（跨页面加载保存请求）
    * Capture Screenshots （屏幕截图）
    * 停用浏览器缓存
    * Clear browser cache（手动清除浏览器缓存）
    * Clear Browser Cookies（手动清除浏览器 Cookies）
    * Offline（离线模式）
    * Network Throttling（可自定义网速）
 * **过滤器** -> 有一个搜索栏，外加一些按钮，帮助过滤搜索请求文件信息
    * domain:... 只显示来自指定定义域的请求
    * has-response-header:... 显示包含指定 HTTP 响应标头的资源
    * is:... is:running -> 查找 WebSocket 资源，is:from-cache -> 查找缓存读出资源
    * larger-than:... 显示大于指定大小的资源
    * method:... 显示通过指定 HTTP 方法类型检索的资源
    * mime-type:... 显示指定 MIME 类型的资源
    * schema:... schema:http
    * status-code:... 仅显示指定状态编码的资源
    * set-cookie-value
    * set-cookie-name
    * set-cookie-domain
* **概览** -> 显示 HTTP 请求，响应的时间轴
* **请求列表** -> 请求的文件资源，默认时间排序
    * Name: 资源的名称
    * Status: HTTP 状态码
    * Type: 请求的资源的 MIME 类型
    * initiator: 发起请求的对象或者进程
        * Parser（解析器）: Chrome 的 HTML 解析器发起的请求
        * Redirect（重定向）: HTTP 重定向启动了请求
        * Script（脚本）: 脚本启动了请求
        * Other（其他）: 用户点击、输入，等等
    * size: 服务器返回的响应大小
    * time: 总持续时间，从请求开始到接收响应的最后一个字节
    * waterfall: 各请求相关活动的直观分析图
* **概要** -> 请求总数、总数据量、总花费时间

这里有个小技巧就是，我们可以按住 shift 键，然后鼠标悬停在某个资源上，可以显示这个资源的上下游请求（下游请求是由上游请求发起的）

另外补充一下**浏览器的加载流程**：
* 解析 HTML 结构
* 加载外部脚本和样式表文件
* 解析并执行脚本代码（部分脚本会阻塞页面的加载）
* DOM 树构建完成（DOMContentLoaded 事件）
* 加载图片等外部文件
* 页面加载完毕（load 事件）

<br>

## Share
这已经是第十次 ARTS 了，这次做一个阶段性地总结吧，谈谈收获，谈谈计划，再谈谈理想。

[ARTS 阶段性总结](./ARTS阶段性总结)