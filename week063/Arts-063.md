> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 1044. Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/)

**题目解析**：

题意用一句话可以概括：“**找到一个字符串中最长的重复子串**”。这里还有一个重要的隐含条件就是，比较的两个子串允许有重叠的部分。

拿到一道题目，除非题目有特别的要求，不然还是先不考虑时间复杂度，先考虑如何解决问题，因为先有思路是关键。那么这道题目，我们可以用两层嵌套循环找出所有的子串，在寻找的过程中，把找到的子串存入一个哈希表中，如果发现有相同的子串，并且该子串比我们之前记录的子串的长度要长，则更新答案，通过这样的方式最终我们可以找到最长的重复子串。

从实际的运行结果上看，上面的方法会超时，我们需要进行优化或者是换其它的思路。对于这道题目来说，需要利用一个名叫 **Robin Karp** 的算法，如果不知道这个算法，那么这道题目对你来说可能会比较棘手。但 Robin Karp 算法其实并不难理解，其核心的思想和哈希的原理有点类似，这里通过题目给的例子简单介绍一下这个算法。假设输入的字符串是 "banana"，这个字符串有很多的子串，比如说下面这些：

```
b, ana, an, ban, bana, ...
```

直觉告诉我们，如果要比较两个字符串是否相等，意味着必须挨个比较两个字符串上的字符，假设字符串的长度是 L，那么每次比较就需要花费 O(L) 的时间。那有没有更快的方法呢？我们知道，如果仅仅是比较两个整数是否相等，我们可以在 O(1) 的时间做到，那么我们是否可以把字符串转化为整数呢？答案是肯定的，Robin Karp 算法就是运用这样的思路，将字符串转化为整数，然后通过比较整数来替代字符串的比较操作，比如例子中的字符串，我们可以运用下面的转化：
```
"banana"
((b * 31 + a) * 31 + n) * 31 + a ...
```
表达式中的字母是对应字符的 ASCII 码，这个表达式其实就是一个哈希函数，我们从左到右遍历字符串，每多遍历一位就把之前遍历到的数字乘上个固定数字。这个说直白点就是把字符串当作是 31 进制的数，如果这个不理解，你可以想想我们是如何表示一个十进制数的，比如 213131 可以表示成：

$$2 * 10^5 + 1 * 10^4 + 3 * 10^3 + 1 * 10^2 + 3 * 10 + 1$$

同样的思想，字符串 banana 就可以表示成：

$$b * 31^5 + a * 31^4 + n * 31^3 + a * 31^2 + n * 31 + a$$

你可能会问为什么这里的取的是 31 而不是其它数字，首先其它的数字也是可以的，但是尽量建议选取质数，不然产生冲突的概率会变大，这里说的产生冲突的意思是，两个不同的字符串用上面这个式子得出同样的整数。

上面说的是 Robin Karp 算法的核心，但是这道题目仅有 Robin Karp 算法还不够，因为 Robin Karp 并不能帮助我们在 O(n) 的时间里找到所有的字符串，但是如果我们给定一个长度，Robin Karp 算法可以在 O(n) 时间帮助我们找到所有该长度的子串，并计算出哈希值，换句话说，**如果我们给定长度，Robin Karp 可以在 O(n) 的时间告诉我们这个长度的子串有没有存在重复的情况**。那我们就可以不断的去猜这个最长的字符串的长度是多少，这个猜数字的过程可以用二分来实现，不断逼近最终的长度，这样，整个算法的时间就是 O(nlgn)。

另外，代码实现中需要注意的一些点是，因为如果字符串的长度比较长，哈希值可能会超出整数所能表示的范围，因此我们要取模。另外，我们还需要去处理哈希冲突的情况，这些都是代码的细节。

<br>

**参考代码（暴力解，超时）**：
```go
func longestDupSubstring(S string) string {
    if len(S) == 0 {
        return S
    }

    subs := map[string]bool{}
    result := ""
    for i := 0; i < len(S); i++ {
        for j := i + 1; j < len(S); j++ {
            cur := S[i:j + 1]
            if _, ok := subs[cur]; ok {
                if len(result) < len(cur) {
                    result = cur
                }
            }

            subs[cur] = true
        }
    }

    return result
}
```

**参考代码**：
```go
func longestDupSubstring(S string) string {
    if len(S) == 0 {
        return S
    }

    start, end := 0, len(S) - 1

    for start + 1 < end {
        mid := start + (end - start) / 2
        ans := robinKarp(S, mid, (1 << 28) - 1)
        if len(ans) != 0 {
            start = mid
        } else {
            end = mid
        }
    }

    endStr := robinKarp(S, end, (1 << 28) - 1)
    startStr := robinKarp(S, start, (1 << 28) - 1)

    if len(endStr) > len(startStr) {
        return endStr
    }

    return startStr
}


func robinKarp(S string, n, mod int) string {
    if len(S) == 0 {
        return ""
    }

    t, d, h := 0, 31, 1

    hashSet := map[int][]string{}
    for i := 0; i < n; i++ {
        // fmt.Println(t)
        t = (t * d + int(S[i])) % mod
    }

    for i := 1; i < n; i++ {
        h = h * d % mod
    }

    hashSet[t] = []string{ S[0:n] }

    for i := n; i < len(S); i++ {
        t = ((t - int(S[i - n]) * h) * d + int(S[i])) % mod

        if t < 0 {
            t += mod
        }

        cur := S[i - n + 1: i + 1]
        if strList, ok := hashSet[t]; ok {
            for _, v := range strList {
                if cur == v {
                    return v
                }
            }
            hashSet[t] = append(hashSet[t], cur)
        } else {
            hashSet[t] = []string{ cur }
        }
    }

    return ""
}
```

<br>

## Review

[30 Magical Python Tricks to Write Better Code](https://towardsdatascience.com/30-magical-python-tricks-to-write-better-code-e54d1642c255)

在这两年的工作中，都有用 Python 这门语言，给我的感觉是 Python 如果写的好的话可以让代码变得非常简洁。这篇文章主要就是介绍 Python 中的一些小技巧，这些技巧可以让你的代码变得更少，也更清晰，我选取了文章中我不知道的技巧进行记录：

* 在一行中给多个变量赋值

  ```python
  a, b = 12, 31
  a, *b = 12, 31, 34 # Python3 支持， Python2 不支持，b 会是一个 list，里面存放着 31, 34
  a = b = c = 10
  ```

* 反转 list

  ```python
  a = [1, 2, 3, 4]
  a_reverse = a[::-1]
  ```

* 在 list 中找到出现频率最高的元素

  ```python
  my_list = [1,2,3,1,1,4,2,1]
  most_frequent = max(set(my_list),key=my_list.count)
  ```

* 找到 list 中每个元素的出现频率

  ```python
  from collections import Counter
  my_list = [1,2,3,1,4,1,5,5]
  ele_freq_dict = Counter(my_list)
  ```

* 创建一个等间隔的序列

  ```python
  my_list = list(range(2,20,2)) # 间隔为 2 的序列，这里的 20 是取不到的
  ```

* 将可变，且可迭代的变量转化为不可变

  ```python
  my_list = [1,2,3,4,5]
  my_list_immutable = frozenset(my_list)
  ```

* python 中的 map 内置函数，让函数作用到 list 的每个变量上

  ```python
  my_list = ["felix", "antony"]
  new_list = map(str.capitalize,my_list)
  
  # 或是可以用匿名函数
  my_list_2 = [1, 2, 3, 4, 5]
  new_list = map(lambda x: x*x, my_list_2)
  ```

* 合并两个字典

  ```python
  dict_1 = {'One': 1, 'Two': 2}
  dict_2 = {'Two': 2, 'Three': 3}
  dictionary = {**dict_1, **dict_2}
  ```

* 将两个 list 转化为字典

  ```python
  list_1 = ["One","Two","Three"]
  list_2 = [1,2,3]
  dictionary = dict(zip(list_1, list_2))
  print(dictionary) # {'Two': 2, 'One': 1, 'Three': 3}
  ```

* 同时遍历两个 list

  ```python
  list_1 = ['a','b','c']
  list_2 = [1,2,3]
  for x,y in zip(list_1, list_2):
      print(x,y)
  ```

感觉 Python 非常的灵活，和 Java 这样的规规整整的语言比起来，有时真的很方便，简简单单的几行代码就可以抵得上其他语言十几行的代码量。但是话说回来，不管用什么样的编程语言，最终都是要落实到解决问题上来，语言只是工具，算法和设计才是重点。


<br>

## Tip

这周学习了 CSS 的一些基础知识，为接下来要做的一个前端项目做准备。这里主要是记录一些原则和关键属性，至于一些其它的属性，遇到的时候可以查文档，不需要特别的记录。

* **CSS 基本语法**

  CSS 的语法很简单，属性的定义由 selector 和 block 组成。比如下面的例子：
  
  ```css
  html {
      background-color: red;
  }
  ```
  
  这里的 `html` 就是 selector，`background-color` 是选择的属性，`red` 是属性被赋的值。

* **CSS 文件在浏览器中处理的大致流程**
  
  我这里画了一张图，主要是 CSS 文件在浏览器段的处理流程
  
  ![](https://user-gold-cdn.xitu.io/2020/6/29/172fd35daa85323e?w=937&h=491&f=png&s=35051)

  这里先说说大致的流程，CSS 一般是跟着 HTML 走的。HTML 会生成 DOM 树，CSS 也会生成对应的 CSSDOM 树结构，然后两个树再进行合成。有了合成树，页面上的每个模块的基本要展示的信息都有了，剩下需要做的就是排版，浏览器会用一系列的算法计算出每个模块出现的位置，然后是最终呈现。
  
  这里我们重点关注 CSS 加载之后的两项处理，第一项是解决 CSS 文件中的定义冲突问题，一个模块的属性可能会被其父节点的属性所影响，比如下面的例子：
  
  ```html
  <div class="cba">
    <h2 class="abc">
        hello word
    </h2>
  </div>
  ```
  这里父节点定义的属性 cba 会影响子模块定义的属性 abc，这一项中就是要确定每个模块的属性。第二项是计算出每个属性最后的值，我们在定义 CSS 的值的时候会使用 %, em, rem，而不是直接给出具体的 px，但是最终还是要转化成 px 的，这里要解决的问题就是将相对值转化为绝对值。

* **CSS 中的盒子模型**

  所有的 HTML 中的节点元素在 CSS 中都被看作是一个盒子，用于封装和更好的修饰 HTML 元素：
  
  ![](https://user-gold-cdn.xitu.io/2020/6/29/172fd7a1c41eee99?w=1040&h=656&f=png&s=38577)

  可以看到 CSS 中的很多属性都是直接改变盒子的属性，之所以要使用盒子来表示一个模块大概是因为能很好的在布局上描述 HTML 模块。

* **解决属性定义冲突**

  在上面提到了，CSS 的属性定义中，父节点定义的属性会影响子节点定义的属性，那么这里浏览器作何取舍呢？一般可以参照下面的考虑优先级：
  
  1. 是否有 `!important` 关键字
  2. selector 的特异性
  3. 属性定义出现在 CSS 文件中的位置
  
  这里解释一下，如果某个属性定义后面加了 `!important`，那么就表明这个定义有最高的优先级，如果作用在该模块上的其它相同属性的定义没有加 `!important`，那么这个定义就会覆盖其它的定义。如果都没有加 `!important`，那么就需要看属性定义所在的 selector 的特异性，特异性优先级如下:
  
  1. inline -> 表示属性直接加载 HTML 上，没有写在 CSS 文件里
  2. \# id
  3. . class
  4. 选取 HTML 上的某个元素，比如 `html`
  
  如果特异性也一样，那就要看定义在 CSS 文件中出现的位置，后面的位置会覆盖前面的位置。

* **计算属性的值**  

  我们经常在 CSS 文件中用 %, em, rem, vh, vw 来表示属性的值，那么这些符号分别代表什么，如何转化成 px？我们依次来看看：
  
  * **x% 作用在 font-size 上**：`当前的 font-size = 父节点的 font-size * x%`
  * **x% 作用在 lengths 上（ex. padding, width...）**：`当前的 lengths = 父节点的 width * x%`
  * **x em 作用在 font-size 上**：`当前的 font-size = 父节点的 font-size * x`
  * **x em 作用在 lengths 上（ex. padding, width...）**：`当前的 lengths = 当前节点的 font-size`
  * **x rem 作用在 font-size 上**：`当前的 font-size = root 的 font-size * x`，root font-size 如果没有定义的话，默认浏览器的 font-size，一般是 16 px
  * **x rem 作用在 lengths 上（ex. padding, width...）**：`当前的 lengths = root 的 font-size * x`，root font-size 如果没有定义的话，默认浏览器的 font-size，一般是 16 px
  * **x% vh** 表示的是相对于当前屏幕高度的百分之 x
  * **x% vw** 表示的是相对于当前屏幕宽度的百分之 x
  
  这里你可以看到的是 em, rem 是基于 font-size 进行转换，二者的区别只不过是相对的节点不一样，推荐使用 rem，这样我们当设备不一样时，模块大小会自动根据浏览器的初始值改变大小。另外 vh, vw 是基于浏览器屏幕大小的变量。


<br>

## Share

[The XY Problem](http://xyproblem.info/)

X-Y 问题。最初看到这个是在 coolshell 上。讲的是一个误区，一个人在遇到一个问题 X 时，觉得 Y 方法可以解决问题 X，但是又对 Y 方法不了解，于是去向他人请教 Y 方法，但其实 Y 方法不是 X 问题的解决方案。这种现象会导致怎样的问题呢？首先这会浪费他人的宝贵时间，另外，这么做会导致不能很好的解决问题 X。

那如何避免这种问题呢？说到底，这仅仅是意识层面的问题，在向别人请教问题的时候，或者是别人向你请教问题的时候，**双方一定要清楚地知道做某件事的目的是什么**。注意，这里说的目的一定是最终的目的，那如何才能知道最终的目的？可以向别人或者是向自己不断地问 “为什么”。比如说有一个人过来问你在 golang 中如何取字符串的后三位？“取字符串的后三位” 是他现在的目的，这个时候你就可以反问他 “为什么要取字符串的后三位？”，他给了解释之后，如果你还有不清楚的话，可以在他给的解释的基础上进一步追问。这么做的好处是可以尽可能的减少信息不对称，换句话说，就是保证沟通的双方始终在一个频道上。

通过这件事情，我想到了在工作中，我们经常会接收上级分发下来的任务。有些时候，任务可能对于你来说非常的明确，但是这个时候我们可以思考一下做这个任务的目的是什么？上级具体是要解决什么样的问题？这个任务是一个完整的任务还是部分，如果是部分的话，那么还需要跟谁协作？我做的这部分东西在整个系统中扮演的是什么角色？这些问题的答案，都需要在和上级沟通的过程中明确，通过这样的沟通，我们能够减少与上级的信息不对称，并获取更多的上下文信息，有时候甚至就可以站在上一级的视角来看待问题，提出上级没有考虑到的因素。如果仅仅是被动地接收需求，然后大概地按自己想象的那样去完成，做出来的东西很可能不会是领导想看到的。因此，在工作中，沟通还是很必要的。