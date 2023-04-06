> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 1297. Maximum Number of Occurrences of a Substring](https://leetcode.com/problems/maximum-number-of-occurrences-of-a-substring/)

**题目解析**：

给定一个字符串，找出出现次数最多的子串，但是有两个限制条件：
* 子串里面的不同的字符的个数不能超过 maxLetters
* 子串的长度必须在 minSize 和 maxSize 之间

这道题目，最初的想法就是使用 **滑动窗口**，但是这里有个问题，子串的长度既有上限也有下限，如果同时带着这两个限制条件去做滑动窗口，你会发现我们其实行不通，举个例子，假如 `minSize` 是 3，`maxSize` 是 6，如果当前滑动窗口的大小是 4，而且子串里面不同字符的个数也不超过 maxLtters，也就是说两个限制条件均满足，那么你是考虑移动左指针还是右指针？仔细去想，如果按这种思路，其实窗口的两头并没有严格的限制条件去控制。

现在我们可以思考这样一个问题，**是不是我们必须考虑 `minSize` 和 `maxSize`，还是说只用考虑其中一个？** 这道题目有一个很巧妙的地方在于，我们只需要考虑 `minSize` 即可，举个例子：
```
s = "aabcaabcaab", maxLetters = 2, minSize = 2, maxSize = 3

aab 出现次数最多，且满足限制条件
只要 aab 满足限制条件，它的子串 ab 也必定满足限制条件，且出现次数必定不低于 aab
```

总的说来，我们只需要考虑长度的下限即可，这样的话，这道题就变成了普通的滑动窗口问题。

<br>

**参考代码**：
```java
 public int maxFreq(String s, int maxLetters, int minSize, int maxSize) {
    Map<String, Integer> counts = new HashMap<>();

    int n = s.length();
    char[] sArr = s.toCharArray();
    int[] hash = new int[26];

    int l = 0, result = 0, unique = 0;
    for (int r = 0; r < n; ++r) {
        if (hash[sArr[r] - 'a']++ == 0) {
            unique++;
        }
        
        // 如果条件不满足，移动左指针，缩小窗口，使条件满足
        while (unique > maxLetters || r - l + 1 > minSize) {
            if (hash[sArr[l++] - 'a']-- == 1) {
                unique--;
            }
        }

        // 条件满足，更新答案，这里只考虑 minSize
        if (r - l + 1 == minSize) {
            String cur = s.substring(l, r + 1);
            counts.put(cur, counts.getOrDefault(cur, 0) + 1);
            result = Math.max(result, counts.get(cur));
        }
    }
    
    return result;
}
```

<br>

## Review
[97 Things Every Programmer Should Know](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/)

[Automate Your Coding Standard](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_04/)

这篇短文主要讲述 **代码标准**，比如代码的风格，还有测试覆盖率之类的，但是通常是，在项目之初我们定好的这些标准，在项目进行当中就会被渐渐忽略，原因很直接，那就是我们希望短时间高产出，代码标准这些东西不会直接影响用户的体验和整个产品的短期质量。这也就会导致最后产品的代码一团糟，后期维护很困难。对此，作者给出了一些建议，我们应该列一些约定俗成的规定，并且使用一些工具来检测我们的代码风格以及测试覆盖率，如果测试工具显示的指标不通过，那么代码就不能够上产品，这样一来代码标准的检测就自动化了，不需要人工审查，省时省力，大家也都会遵守这一约定。当然，这些标准也不是一成不变的，有些我们之前认为的东西，在产品开发过程中发生了变化，也需要做一些调整，但是最好是能自动化的东西就自动化，不能自动化的东西，我们也需要和一起合作的人达成共识。

<br>

[Beauty Is in Simplicity](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_05/)

文章的观点和直接，简洁的代码才是美的，你可能会问 **“什么样的代码才是简洁的代码”**，就是表意明确，指代清晰，让人一看就懂，如果项目简单，做到这一点不难，但是如果项目的体量增大，系统变得复杂，做到这一点就不容易了，如果在一个项目中，可以做到函数职责清晰，简单，一个函数只有不超过 10 行的代码量，函数命名，变量命名让人只看名称不看具体实现就已知晓其作用和目的，那么这就可以被称作简洁的代码。想要写出简洁的代码并不只是我们闭关修炼就可以做到，我们需要多看一些好的开源代码，多思辨，多总结，只有这样我们才能够写出真正的简洁的代码。

<br>

[Before You Refactor](https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_06/)

当你需要去重构现有代码的时候，这篇文章或许对你有帮助，作者给出了几点建议：
* 在开始具体实作之前，最好是参考一些现有的测试和代码，看看需要重构的部分的优缺点是什么，哪些地方存在比较严重的问题？哪些地方是可改可不改的？这样分析一下，才会更有针对性。
* **避免改写所有的代码**。改写所有的现有代码意味着重写，之前的功劳全部白费，而且我们还需要重新设计，重新定计划，这是一个巨大的工程，不能轻易就做出这样的决定。
* **许多小改动比一个大改动要好的多**。代码往往是相互关联的，即使再牛逼的代码，耦合性还是存在的，大的改动意味着你需要考虑的更多，例如相关联的模块是否受影响，需不需要也重写，而局部的小改动则会好很多。
* **每次修改后要确保之前的 testcases 能够通过**。这个就不多说了，改动或者重构不应该影响之前已经实现好的逻辑功能。
* **避免因为个人喜好而修改代码**。我们之所以重构代码，主要目的是让代码变得更加健壮，逻辑更加清晰，我们不能因为个人的主观感受而去修改代码，要修改，需要给出明确，直观的理由。
* **新的技术也不应该成为修改代码的理由**。这一点和上一点类似，就是我们应该更看重代码的可靠性，而不是技术的知名度。
* **是人总是会犯错**。最后一点，我觉的也是最重要的，我们看别人写的代码的时候应该保有一种宽容的心态，不要觉得，产品都上线了，代码中存在问题就不能忍，要知道，是人都会犯错，我们也都是从不会到慢慢熟悉，然后才到会，给他人的技术成长留有空间，同样也是对自己。


<br>

## Tip

推荐一个基于 Evernote 的 markdown 软件，[Marxico](http://marxi.co/) 不知道为什么 Evernote 的国际版里没有 markdown 的支持，因此记录笔记只好另想办法

<br>

## Share

这次推荐一篇极客时间上关于 JavaScript 中代码执行顺序的文章，总所周知，JS 里面的代码都是异步的，这篇文章讲述 JS 的编译和运行机制，看完感觉挺有收获的

[变量提升：JavaScript代码是按顺序执行的吗？](https://time.geekbang.org/column/article/119046)