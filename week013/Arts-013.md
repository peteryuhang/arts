## Algorithm

[LeetCode 126. Word Ladder II](https://leetcode.com/problems/word-ladder-ii/)

**思路分析**<br>
这道题是一道经典的搜索问题，难点还是在于优化。题目要求找出所有可能的解，我们可以把这道题转化为一个**无向图**，每个单词表示的是图上的节点，保证每个单词的邻居单词跟这个单词只有一个字母的不同，如果用暴力的深度优先搜索，我们在图上选中起始节点和终止节点，考虑所有的从起点到终点的可能的路径，这么做会大大增加搜索所消耗的时间，那么怎么优化呢？和我们之前常常提到的增加记忆化不同的是，这里我们考虑使用标记法，因为在搜索的时候，我们要保证我们当前的搜索方向是朝着终点去的，什么意思，就是说，**下一个我们要去的节点到终点的距离要比到当前我们所在节点到终点的距离更近**，根据这么一个思路我们可以进行两次搜索，第一次仅仅是标记，第二次才是搜索可能的答案。这里标记使用的是广度优先搜索，为什么不使用深度优先搜索？**因为这个图中可能存在环，深度优先搜索没法保证标记的正确性**。搜索答案的时候我们可以使用深度优先搜索，也可以广度，这里我使用了深搜。

<br>

**参考代码**
```java
public List<List<String>> findLadders(String beginWord, String endWord, List<String> wordList) {
    if (beginWord.equals(endWord) || !wordList.contains(endWord)) {
        return new ArrayList<>();
    }
    
    List<List<String>> results = new ArrayList<>();
    
    // bfs mark
    Map<String, Integer> memo = new HashMap<>();
    for (String word : wordList) {
        memo.put(word, -1);
    }
    
    memo.put(beginWord, -1);
    memo.put(endWord, 0);
    
    bfsMark(endWord, memo);
    System.out.println(memo);
    
    // dfs find path
    List<String> path = new ArrayList<String>();
    path.add(beginWord);
    
    dfs(memo, beginWord, endWord, wordList, path, results, memo.get(beginWord));
    
    return results;
}

private void dfs(Map<String, Integer> memo, 
                 String curWord,
                 String endWord,
                 List<String> wordList,
                 List<String> path,
                 List<List<String>> results,
                 int step) {
    if (curWord.equals(endWord)) {
        results.add(new ArrayList<String>(path));
    }
    
    if (step <= 0) {
        return;
    }
    
    for (String word : wordList) {            
        if (memo.get(word) == step - 1 && checkDiff(word, curWord)) {
            path.add(word);
            
            dfs(memo, word, endWord, wordList, path, results, step - 1);
                
            path.remove(path.size() - 1);
        }
    }
}

private boolean checkDiff(String a, String b) {
    int count = 0;
    for (int i = 0; i < a.length(); ++i) {
        if (a.charAt(i) != b.charAt(i)) {
            count++;
        }
        
        if (count > 1) {
            return false;
        }
    }
    
    return count == 1;
}

private void bfsMark(String endWord, Map<String, Integer> memo) {
    Queue<String> queue = new LinkedList<>();
    queue.offer(endWord);
    
    int stepCount = 0;
    while (!queue.isEmpty()) {
        int size = queue.size();
        stepCount++;
        
        for (int i = 0; i < size; ++i) {
            String curWord = queue.poll();
            
            char[] curWordArr = curWord.toCharArray();
            for (char r = 'a'; r <= 'z'; ++r) {
                for (int j = 0; j < curWordArr.length; ++j) {
                    char tmp = curWordArr[j];
                    curWordArr[j] = r;
                    
                    String newWord = String.valueOf(curWordArr);
                    
                    if (memo.containsKey(newWord) && memo.get(newWord) == -1) {
                        queue.offer(newWord);
                        memo.put(newWord, stepCount);
                    }
                    
                    curWordArr[j] = tmp;
                }
            }
        }
    }
}
```

<br>


## Review
一篇关于如何问别人技术问题的文章:<br>

[How To Ask Questions The Smart Way](http://www.catb.org/~esr/faqs/smart-questions.html)

问问题也是一门学问，特别是在网上问问题，文章读起来不是太好懂，但还是理解了、学到了不少，总结一下重点内容：
* 在问别人问题前，一定要对你的问题有所了解、并且尝试着去解决，你可以 Google，或自己手动模拟测试可能的情况，再或者问你身边的在行的朋友，如果尝试了很多方法，但是依旧没有办法解决，在来问问题。自己发现问题并解决问题是对自己的成长帮助最大的，这样也可以保证你问的问题相对来说比较有质量，深思熟虑过后的问题更能够吸引高手的注意，相信没有哪个程序高手愿意回答类似 “Java 如何打印输出” 这样的问题。
* 在网络上 POST 问题的时候注意问题标题的**清晰、直接、易懂**。作者给出了一个 “**对象-错误表现**” 的模版，对象说的是什么东西，具体到哪个地方出问题，这里越具体越好，最好加上版本号，具体开源模组等等；错误表现是指这个东西发生了什么情况，这里要做的是言简意赅，直接了当说明症状。标题不能过长但又要说明大致的情况，保证别人阅读起来不会耗时耗力，但又能马上反应过来这个问题是什么
* 上面讲了 POST 问题的标题，再说说描述问题的正文部分，这里要注意的是**不要加上一些自己的猜测，别人需要花时间阅读这些猜测不说，还要去理解你为什么这么想，容易混淆视听，导致别人不愿意再看下去**，更好的做法是描述问题发生的环境，然后你是怎么做的导致了这个问题，还有一点就是你要写出一些东西表明你尝试着去解决这个问题，但是因为种种原因，失败了，这么做有两点原因：
    * 让别人明白这个问题对你来说是真的问题，你很希望自己得到帮助，别人会觉得他的帮助在这里会很有价值
    * 打消那些可能的解，有些时候我们问问题会得到一些回答，但是那些回答你看完了发现自己之前已经尝试过，但是问题没有解决，然后我们又要去追问，别人继续追答，这样很没效率不说，下次其他的人遇到相同的问题来看这个帖子的时候会阅读很多无关信息，很没效率
* 我们问的问题，要让别人能够很快给出解，这个解最好就是一个指示，告诉我们如何去做。不能说叫别人写出一长串代码，这样的帮助耗时耗力，相信愿意帮助的人不多。也不要让别人看你的 code 去帮你 debug，要问的话，最好是能大大缩短代码篇幅，方便别人，也方便自己
* 防止目的不明确或者宽泛的问题，每个问题都有一个最终目的，不管是想让找出 bug，或者是解决发现的 bug，再或是设计上面想要有多点的建议，需要让别人明白你想要什么。另外就是不要把问题加上 “紧急”，“快速” 这样的词汇，这样的话有一种在催别人的感觉，导致别人不愿意看你的问题


<br>


## Tip
这周接触了 openresty，记录一下它在 Mac 上的安装和启动

安装(使用 homebrew)：
```
>$ brew tap openresty/brew
>$ brew install openresty
```

启动：
```
>$ openresty -p `...` -c ...
```

停止：
```
>$ openresty -s quit -p `...` -c ...
```

<br>

再来记录几个之前比较模糊，但是现在懂了的 Python 基础知识点：
* Python 当中的对象的浅拷贝并不是说只是拷贝引用，而是**创建了一个新的对象，但是这个新对象里面的子对象还是之前对象里面的子对象**，下面的代码很好的解释了这一点：
```python
import copy

a = [[1,2],2,3,4]
b = copy.copy(a)

b.append(1)
b[0].append(3)

print(a)
print(b)

-----------------------------
[[1, 2, 3], 2, 3, 4]
[[1, 2, 3], 2, 3, 4, 1]
```
* **==** 运算符的定义，Python 和 Java 很不一样，这里 Python 比较的不是地址，而是值。Python 中比较地址用的是 **is** 运算符，顺便说一下 Python 中用 id 来区分表示对象，其实可以代表对象的地址了，is 运算符的运算速度会比 == 运算符快很多，因为 is 不会被重载，它仅仅比较两个对象的 id

<br>

## Share
这周最后还是来聊聊算法，拓扑排序

[拓扑排序原理和习题分析](./拓扑排序原理和习题分析)
