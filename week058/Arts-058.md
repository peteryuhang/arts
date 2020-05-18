> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 151. Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/)

**题目解析**：

题目的输入条件是一个字符串，让你以单词为基本单位反转这个字符串，这里的重点是 **以单词为基本单位**。这道题目有很多种解法，特别是使用 Java 或者 Python 这种有很多 String 的内置函数的高级语言。比如，我想到的一个直接的解法就是将整个字符串以空格作为分隔符进行拆分，然后遍历拆分后的单词数组，过滤掉那些空的单词，然后对单词数组直接反转，最后再从新 join 即可。

但是如果你对 Java 的字符串的底层原理稍有了解的话，你就会发现其实在这里我们浪费了很多的空间。字符串在 Java 中是不可变的，当你对一个字符串进行删改操作，底层实现其实会生成一个新的字符串，比如说对单词使用 split 操作，其空间复杂度就是 `O(n)`。你可以看到上面的解法中，我们用了 split 操作，join 操作，还用容器对单词进行存储，这里面都有空间的不必要的浪费。

这道题目是否可以用 `O(1)` 的空间复杂度解决呢？如果是 Java 语言的话，答案是不行的，原因上面也解释了，Java 没法直接对 String 进行操作，因为 String 不可变，必须先把 String 转换为 char 数组，首先转化这一步就会用到 `O(n)` 的空间。但如果是 C++ 这种用 char 数组来表示 string 的语言来说，我们当然可以考虑用 `O(1)` 的空间将其实现。我对 C++ 不是特别熟悉，这里我就假设 string 转换为 char 数组这一步以及 char 数组转化为 string 这一步不存在空间的消耗，主要是看看代码逻辑该怎么写，这里并不涉及特别高深的算法，主要还是考察写代码的基本功。

还有，这里有一个认识就是，**如果要对一个数组以区间段的形式进行反转的话，可以先将整个数组反转，然后再对每个区间段进行反转**。这算是一个技巧吧，其实也很好理解，首先整个数组反转会将数组中所有元素进行反转，如果以区间段为单位，区间段也被反转了，但是，这样做区间段里面的元素也被反转了，所以我们要恢复每个区间段之前的元素顺序，就要对区间段进行再次的反转。

<br>

**参考代码**：
```java
public String reverseWords(String s) {
    String[] words = s.split(" ");

    List<String> nonEmptyWords = new ArrayList<>();
    for (String word : words) {
        if (word.length() > 0) {
            nonEmptyWords.add(word);
        }
    }

    Collections.reverse(nonEmptyWords);

    String result = String.join(" ", nonEmptyWords);

    return result;
}
```

**参考代码（模拟 C++，O(1) 空间）**：
```java
public String reverseWords(String s) {
    if (s == null || s.length() == 0) {
        return s;
    }

    int n = s.length();

    char[] sArr = s.toCharArray();
    // 对整个字符数组进行反转
    reverse(sArr, 0, n - 1);

    // 对每个单词进行反转
    int i = 0, j = 0;
    while (j < n) {
        while (j < n && sArr[j] == ' ') {
            j++;
        }

        i = j;

        while (j < n && sArr[j] != ' ') {
            j++;
        }
        
        if (i < n) {
            reverse(sArr, i, j - 1);
        }
    }

    // 重构数组，主要是为了去除不必要的空格
    i = 0; j = 0;
    while (j < n) {
        while (j < n && sArr[j] == ' ') {
            j++;
        }

        while (j < n && sArr[j] != ' ') {
            sArr[i++] = sArr[j++];
        }
        
        while (j < n && sArr[j] == ' ') {
            j++;
        }

        if (j < n) {
            sArr[i++] = ' ';
        }
    }

    return new String(sArr).substring(0, i);
}

public void reverse(char[] sArr, int start, int end) {
    while (start < end) {
        sArr[start] ^= sArr[end];
        sArr[end] ^= sArr[start];
        sArr[start++] ^= sArr[end--];
    }
}
```

<br>

## Review

[How to Write Log Files That Save You Hours of Time](https://medium.com/better-programming/how-to-write-log-files-that-save-you-hours-of-time-1ff0cd9ae2ed)

这篇文章主要是想告诉你怎么样编写 log 以及错误日志。不知道你在实际工作中是否遇到过这样的情景：发现程序出错，但是查看 log 发现很难定位错误，还有就是当你对整个系统不熟悉时，log 没有给你完整的错误信息，导致你很难 debug，然后你又着急修复问题，这会不会弄得你焦头烂额。

每当程序出错，如果我们能够得到很明确的错误信息，会不会让我们的做事效率事半功倍？对于如何写 log，文章给出了下面几点建议：

* 不要将无关信息写入 log

  **出错时，显示的信息并不是越多越好**，信息量越大，只会让人花更多的时间去理解信息背后的意义。信息要明确，一些和错误不相关的信息可以不提及，比如因为 HTTP 请求失败导致用户无法登陆，你并不用显示用户账号的创建时间这样的无关信息。

* 要说明错误具体是什么

  如果日志当中仅仅显示 “请求失败”，没有人知道程序是具体因为什么而出错，正常人从这种信息里面唯一能获取的就是程序出错了，但是这个显而易见，说了等于没有说。你需要解释清楚这里出错的原因，是因为 HTTP 请求导致的请求失败？还是连接数据库超时？亦或是请求的数据有问题？。。。总之，出错总有原因，你需要尽可能地描述清楚，不要仅仅是给出一个很宽泛的原因，这样不仅帮不到别人，有时还会产生误导。

* 错误发生的地方

  仅仅知道错误是什么还是不够的，特别是当系统特别庞大的时候。一个错误可能在系统的很多地方出现，因此我们需要指明错误发生的地点，比如说具体的类库，甚至是具体的函数，这样可以帮助缩小 debug 的范围，进而节省 debug 的时间。
 
* 给出相关的数据

  当一个错误发生，我们需要知道某些变量或者对象的一个状态，这样我们才能对错误排查彻底，比如还是说请求失败，如果这个时候，日志中能够给出请求的具体用户，时间，还有一些相关变量，这可以给我们一个参照对比的样例，辅助我们去对错误进行一个更深层次的排查
 
* 异常的类型要尽可能地小

  如果不管任何错误，我们都用 Exception 这个类型，那我们就很难从异常的类型去判断出错的原因，我们需要对异常不断细分，这样更有利于我们去直观理解错误。还有就是，当错误发生时，我们可以打印出整个函数的调用栈，方便我们找到根问题。

<br>

## Tip

记录一下，这周工作当中遇到的一个问题。有一个业务需求是将 UTC 00:00 的 timestamp 转化为洛杉矶时间的 00:00 的 timestamp，一开始写出了下面的 Python 代码：

```python
utc_time = dt.replace(tzinfo=pytz.timezone("UTC"))
pdt_time = dt.replace(tzinfo=pytz.timezone("America/Los_Angeles")) # 2020-05-07 00:00:00-07:53

pdt_start_date = utc_time + relativedelta(pdt_time, utc_time)
start_date = pdt_start_date.replace(tzinfo=None)
start_timestamp = to_timestamp(start_date)
```

主要的错误是在第二行，你会发现它多了一个 53 分钟的后缀。查相关文档发现了下面这句话：

> Unfortunately using the tzinfo argument of the standard datetime constructors ‘’does not work’’ with pytz for many timezones

修改成下面的做法：

```python
utc_time = dt.replace(tzinfo=pytz.timezone("UTC"))
pdt = pytz.timezone("America/Los_Angeles")
pdt_time = pdt.localize(dt)

pdt_start_date = utc_time + relativedelta(pdt_time, utc_time)
start_date = pdt_start_date.replace(tzinfo=None)
start_timestamp = to_timestamp(start_date)
```

<br>

## Share

[Machine Learning is Fun!](https://medium.com/@ageitgey/machine-learning-is-fun-80ea3ec3c471)

这周推荐一篇文章，如果你还不了解机器学习或者人工智能，不妨读一读。这篇文章主要的目的是想告诉你 “机器学习到底是什么” 以及 “机器学习能够帮助我们解决哪些问题”。让你对机器学习产生兴趣和想要去学的冲动就是这篇文章的目的。

如果你对机器学习没有半点概念的话，你可以把其当作是一种算法，准确点说，一系列的算法。**这种算法能够基于数据帮你解决一些特定的问题，我们可以把机器学习算法当作一个黑箱子，当你把输入数据导入，它就可以导出一个输出数据**。比如说你将一个手写的数字图片导入，它就可以基于你的输入图片，在屏幕上显示对应的阿拉伯数字。这么说或许有些神奇，但是如果你仔细想想我们人类是怎样认知世界的就不会感到陌生，当我们第一次看到一个圆圆的、红红的水果，我们并不知道它具体是什么，当别人告诉我们这个是苹果，我们才知道，当我们第二次看到形状相似的水果，比如橙子，我们可能会误认为这是苹果，因为我们能够叫得上名的也就只有苹果，但是当我们见过各种各样的水果，每种水果还不止见了一次，也都知道它们各自的名字。这时如果我们再看到一个水果，我们能够叫对名字的概率就会大大增加，因为我们见到过足够多的样本数据，基于之前的数据，这里说经验可能好些，我们的大脑可以将当前的水果的特征和之前我们记忆中的水果的特征进行对比，得出答案。机器学习说白了就是将上面的过程让电脑去完成，因为电脑可以在短时间里处理相对大量的数据，从而减轻了一些记忆性、重复性地操作，结果会比我们凭经验想出来的结果更准确。我想你应该至少理解，人工智能其实就是让电脑去模拟人脑去做一些更为复杂的计算和预测。

文章中给出了两种机器学习，**监督学习** 与 **非监督学习**。这两者的主要区别就是 **是否有标准答案作为参照**。比如当我们被告知一个红红的，圆圆的，味道酸甜的水果是苹果，这里的苹果就是答案，当我们有了足够多的训练数据，当具有一系列特征的物体出现在我们面前的时候，我们就可以基于这些特征和之前出现的特征还有特征对应的答案来做预测，这就是监督学习，**往往监督学习有明确的预测目标**。而非监督学习则是给出你某个东西的一系列特征，并不会告诉你特征之间的相互关系，这就好像别人给了你一堆数据，但是有些数据为什么会出现，在什么情况下会出现，这些问题你都不知道，你需要去自己发现。监督学习与非监督学习没有常用和不常用之分，它们都是基于不同的应用场景而出现的。

其实机器学习就是建立模型，然后通过足够多的训练数据不断地预估模型的参数，从而保证整个算法的预测成功率。但是你要知道的是，机器学习算法并不能帮你做完所有的事情，换句话说，机器学习不会无中生有，机器只能帮你快速地完成一些重复的工作。在使用机器学习之前，你需要很清楚地知道怎么样手动地，人工地基于当前的一些数据来做预测，你也需要知道某些数据之间必定多多少少存在联系，不然，机器很可能会产生风马牛不相及的预测结果，这也是不符合我们的预期的。

换个角度看，**其实人工智能并没有那么的神奇，最主要的还是靠人脑，电脑只是做了它该做的事情！**