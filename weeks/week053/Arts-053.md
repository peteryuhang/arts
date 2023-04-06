> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

**题目解析**：

题意很简单，就是让你在一个数组中找出子数组和等于 K 的所有子数组的个数。首先这道题并没有特别要求时间复杂度，我们可以大胆地去思考可行解。子数组是由一个区间组成，一个区间是由两个边界，或者说是左右两个元素确定，那么我们把所有元素两两配对的情况都遍历一遍即可，于是我们可以得到下面的代码：

```java
public int subarraySum(int[] nums, int k) {
    if (nums == null || nums.length == 0) {
        return 0;
    }

    int n = nums.length;

    int result = 0;
    for (int i = 0; i < n; ++i) {
        int sum = 0;
        for (int j = i; j < n; ++j) {
            sum += nums[j];
            if (sum == k) {
                result++;
            }
        }
    }

    return result;
}
```

首先要说明一点，上面的代码在 LeetCode 平台上是可以提交通过的，而且意图直接了当。就是**找出所有的 `[i,j]` 区间及其区间和 `sum`，然后和 k 进行比较计数**。但是你也可以发现，这个代码的时间复杂度比较高，两个嵌套 for 循环，最后就是 `O(n^2)` 的最差时间复杂度。

但有了这个可行解，我们可以基于此进行优化。上面的解法把一个区间的两头都遍历了一下，那么我们就可以思考，**能不能只知道区间的一头，就可以确定符合条件的区间个数？** 如果我们从左向右遍历数组，当我们遍历到一个元素的时候，如果以当前遍历到的元素作为区间的右边界，那么我们必须从已经遍历过的元素中找一个作为左边界。很明显，这个元素左边的元素我们全部都已经遍历过了，问题来了，怎么选择？这里可以用到一个 **前缀和** 的知识。数组中一个元素的前缀和是指这个元素和其前面所有的元素的加和，知道了这个我们可以很容易的求出一个区间的和，如果我们用一个哈希表保存前面元素的前缀和，那么当遍历到一个元素的时候，就可以去哈希表中寻找有没有符合条件的前缀和：

```
假设一个数组：[...i,...j...]
前缀和：prefixSum_i = sum([0,...,i])
前缀和：prefixSum_j = sum([0,...,i,...,j])
区间和：intervalSum[i + 1, j] = prefixSum_j - prefixSum_i

当我们知道了 prefixSum_j，我们要找的符合条件的值 x 就是：
prefixSum_j - x = k 
x = k - prefixSum_j
这个 x 是前面任意元素的前缀和
```

你可以看到，通过前缀和加哈希表，我们就可以把这道题的时间复杂度给降到 `O(n)`。总得来说这道题目不难，在这里我们首先给出了暴力的可行解，再在暴力可行解的基础上进行优化。

这种题很容易在面试的时候遇到，如果在面试的时候遇到，切记一定要展现出自己的思考过程。因为面试是一个交流的过程，你要给面试官的感觉是你在和他合作探讨解决一个问题，而不是仅仅给出答案。**在交流之中，不断地通过面试官给的需求和条件推出未知解，遇到不懂的也大胆说出自己的想法，你要展现出的思考过程需要是渐进的，连贯的。** 就像你展现出的自己是由慢到快，从走到跑的这么一个过程，这就会给人一种很自然的状态。总之，在面试中做出题目永远不是目的，而是要表现出自己是一个懂得倾听，善于合作的伙伴。

<br>

**参考代码**：

```java
public int subarraySum(int[] nums, int k) {
    if (nums == null || nums.length == 0) {
        return 0;
    }

    int n = nums.length;

    // map 用于记录前面遍历过的元素的前缀和
    // 因为可能存在相同的前缀和
    // 这里用 key 表示前缀和，value 表示前缀和的个数
    Map<Integer, Integer> map = new HashMap<>();
    
    int result = 0, prefixSum = 0;

	// 初始化 map，如果整个前缀和数组就是答案，需要减去的值就是 0
    map.put(0, 1);
    for (int i = 0; i < n; ++i) {
        prefixSum += nums[i];

        // 如果需要区间的值是 k，那么要找的左边界元素的前缀和就是 minuend
        int minuend = prefixSum - k;

        // 如果要找的前缀和存在，就记录答案
        if (map.containsKey(minuend)) {
            result += map.get(minuend);
        }

        // 记录当前元素的前缀和
        map.put(prefixSum, map.getOrDefault(prefixSum, 0) + 1);
    }

    return result;
}
```

<br>

## Review

[5 JavaScript Tricks That Are Good To Know](https://levelup.gitconnected.com/5-javascript-tricks-that-are-good-to-know-78045dea6678)

文章介绍了 5 个 JavaScript 的语法，合理运用这些语法可以让代码变得更加地简洁，一起来看看：

* every 和 some

  这两个内置函数都是对数组中的每个元素进行操作，类似 map，输入参数是一个函数，every 表示函数作用于所有元素后的返回值均为 true，every 的返回值才会是 true，some 的话，只要函数作用于数组中的某个元素返回 true，那么它的返回值就是 true。
  ```javascript
  const a = [ 13, 2, 37, 18, 5 ];
  const b = [ 0, -1, 30, 22 ];
  
  // every
  a.every(isPositive); // returns true
  b.every(isPositive); // returns false
  
  // some
  a.some(isPositive); // returns true
  b.some(isPositive); // returns true
  ```

* 依据条件设置一个变量

  在日常开发中，往往我们会遇到这么一个情况。就是一个变量的值取决于另外一个变量，当另外一个变量不存在的话，我们需要给这个变量一个默认值，这在设置系统变量的时候经常会遇到。一般都会用 if 判断一下，但是我们可以借助 `||` 或运算符来让代码变得更为简洁
  
  ```javascript
  env = process.env.ENVIRONMENT || 'developer';
  ```

* 改变一个数组中的变量类型

  在一些开发场景下，一个数组里面的数字都是以字符串的形式保存，在运用的时候我们需要将这些字符串表示的数字转换成整形，一个简洁的写法就是借助 map 和强制类型转换的函数
  
  ```javascript
  let a = [ '1', '5', '8' ];
  selected_values = a.map(Number) // [ 1, 5, 8 ]
  ```
 
* 对象拆解

  这个就不用多说了，就是通过赋值语句，将一个对象中需要的 entry 拿出来。对象拆解其实在 JS 中应用还是很广泛的，比如函数的参数传值，这么做的好处是，不管是接收放，还是传入方，均不需要考虑参数的前后顺序，而且传入方只需传入一个包裹好的对象，接收方只需要选择其需要的参数即可，双方都省去了很对细节。

* 效率测试工具

  往往我们在 debug 的时候都只会用到 `console.log` 但是 console 这个内置对象其实还是有很多其他的函数，比如 `time` 和 `timeEnd`，它可以帮助测试代码运行的效率。这两个函数的输入参数是一个 string，time 表示该 string 代表的任务开始执行，timeEnd 表示同 string 代表的任务结束，统计时间。
  
  ```javascript
  console.time('code');
  // code...
  console.timeEnd('code');
  ```

<br>

## Tip

这周学习了一下正则表达式，发现它还有很多高阶的用法，不得不说还是自己知道的太少。正则在实际开发中运用的还是非常广泛的，特别是在文本的搜索、校验和提取中。

在记录之前，推荐两个网站：

* https://regex101.com/ 这个是一个测试网站，基本上覆盖的所有版本和所有语言下的正则
* https://github.com/ziishaned/learn-regex 是一个 GitHub 上的正则的教程，跟完基本上可以对付工作中 80% 的场景，包括下面一些例子都是选取于该网站

元字符（基于字符）：
* 基础（单个字符）
	*  `.` 任意字符，换行除外
	*  `\d` 任意数字，`\D` 任意非数字
	*  `\w` 任意数字字母下划线，`\W` 数字字母下划线以外的任意字符
	*  `\s` 任意空白符，`\S` 任意非空白符
* 空白符（单个字符）
	* `r` 回车符
	* `n` 换行符
	* `f` 换页符
	* `t` 制表符
	* `v` 垂直制表符
* 量词
	* `*` 0 到多次
	* `+` 1 到多次
	* `?` 0 到 1 次
	* `{m}` 出现 m 次
	* `{m,n}` m 到 n 次
	* `{,n}` 至多 n 次
	* `{m,}` 至少 m 次
* 范围
	* `|` 如 ab|bc 代表 ab 或 bc
	* `[...]` 多选一，括号中任意单个元素
	* `[a-z]` 匹配 a 到 z 之间任意单个元素（按 ASCII 表，包含 a，z）
	* `[^...]` 取反，不能是括号中的任意单个元素
* 边界
	* `^` 匹配行的开始，多行模式下可以匹配任意行开头
	  
        ^(T|t)he => **The** car is parked in the garage.

	* `$` 匹配行的结束，多行模式下可以匹配任意行的结尾
	
        (at\.)$ => The fat cat. sat. on the m**at.**

	* `\b` 匹配单词边界
	* `\A` 仅匹配整个字符串的开始，不支持多行模式
	* `\Z` 仅匹配整个字符串的结束，不支持多行模式

贪婪与非贪婪:
* 贪婪：正则中表示次数的量词默认是贪心的，它会尽可能多的去匹配符合要求的内容

    "/(.*at)/" => **The fat cat sat on the mat**.     

* 非贪婪：“数量” 元字符后加 ？（英文问号）找出长度最小且满足要求的

    "/(.*?at)/" => **The fat** cat sat on the mat. 

环视：
* `X(?<=Y)` 匹配前面是 Y 的 X

    (?<=(T|t)he\s)(fat|mat) => The **fat** cat sat on the **mat**.

* `X(?<!Y)` 匹配前面不是 Y 的 X

    (?<!(T|t)he\s)(cat) => The cat sat on **cat**.

* `X(?=Y)` 匹配后面是 Y 的 X

    (T|t)he(?=\sfat) => **The** fat cat sat on the mat.

* `X(?!Y)` 匹配后面不是 Y 的 X

    (T|t)he(?!\sfat) => The fat cat sat on **the** mat.

子组：
* `(正则)` 将 regex 保存成一个子组
* `(?P<name>正则)` 命名子组，将 regex 保存成名称为 name 的子组
* `(?:正则)` 仅分组，不保存这个子组
* `\分组编号` 重复某个子组
* eg. find: `(\w+) \1` replace: `\1`

标识：
* i 用来取消大小写匹配，g 表示全局，匹配整个字符串

    /The/gi => **The** fat cat sat on **the** mat.

* m 表示多行匹配

    "/.at(.)?$/gm" => The **fat** <br>
                    cat **sat**<br>
                  on the **mat**.


感觉掌握上面这些正则最好的方法还是多练，平时工作的时候搜索代码库的时候记得尝试着去用一用，然后多多回顾，时间长了，自然就熟练了。

<br>

## Share

clean code 的更进暂停一周，这周来分享一篇文章：

[Geek to Live: The art of asking](https://lifehacker.com/geek-to-live-the-art-of-asking-191451)

这篇文章讲的是怎样去问问题，这对那些新入职，或者是刚步入社会不久的人非常有帮助。概括来说就是**问的问题要具体、自己要尝试着去解决、问对人，还有最重要的一点就是，要让被问的人感觉他的回答是有价值的**。这些东西说到底其实是意识的问题，注意了，就会得到很大的改善。

但是我想说的其实是，反过来，被问的人的回答也很重要。一个潦草敷衍的回答不仅会让提问人感到困惑，往往还会给人不尊重的感觉。可能你会说，谁会牺牲自己的时间来思考或者帮助你解决烦人的问题的同时还面带微笑？这么说也没错，**但是往往工作上面强调的是共赢**。当你同事问了一个低级问题，你不予理睬，那么除非你是他的领导，不然下次他会尽可能地减少问你问题甚至和你合作的频率。反过来，你帮助他解决了问题，那么正常来说他也会尽可能地和你分享他所了解的信息。这里的关键就是，如何回答才能不浪费自己的时间，还能最大程度地帮助到别人，说到底是一个沟通的艺术。