> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 299. Bulls and Cows](https://leetcode.com/problems/bulls-and-cows/)

**题目解析**：

题意是在一个游戏中你写下一个数，然后让朋友来猜。在这里我们并不是猜大小，而是按位比较，在知道数字位数的前提下，朋友给会给出一个数字。位置相同且大小相同，该位就被称为 `bull`。如果位置不同，但是该位上的数在给定数字中出现，并且不是 `bull`，那么这个位就可以称为 `cow`。看个例子就理解了：

```
secret = 1807 guess = 7801
```

你写下的数是 `1807`，然后朋友猜 `7801`。挨个比较，只有第 3 位上的 0 是相同的，所以只有 1 个 `bull`。剩下的 `8`，`1`，`7` 虽然没有对位匹配，但是都在给定的数中出现过，因此他们都可以算作 `cow`。最后输出答案 `1A3B`，表示有 1 个 `bull`，3 个 `cow`。

一眼看去，这道题貌似并不难，可以说马上有思路，无非就是遍历。但是写着写着，你发现这之中还是有很多细节。首先一个问题是，需要遍历几次？只遍历 1 次的话貌似只能找出 `bull`。但是对于 `cow` 来说，在不知道整体的数字分布的情况下，很难做判断。因此我们至少需要遍历两次。另外我们需不需要辅助的数据结构呢？对于 `bull` 来说，比较简单，单纯的遍历即可。但是 `cow` 的话，我们需要记录数字出现的个数，这里需要一个哈希表。另外，对于已经成为 `bull` 的数字，我们不应该再把它考虑成 `cow`。所以，**我们还需要一个结构来记录 `bull` 出现的位置**，这一点容易被忽视。通过上面这个思考过程，我写下了下面的代码：

```java
public String getHint(String secret, String guess) {
    int n = secret.length();

    Map<Character, Integer> counts = new HashMap<>();
    boolean[] visited = new boolean[n];

    int bulls = 0;
    for (int i = 0; i < n; i++) {
        char s = secret.charAt(i), g = guess.charAt(i);
        if (s == g) {
            bulls++;
            visited[i] = true;
        } else {
            counts.put(s, counts.getOrDefault(s, 0) + 1);
        }
    }

    int cows = 0;
    for (int i = 0; i < n; i++) {
        char g = guess.charAt(i);
        if (!visited[i] && counts.containsKey(g) && counts.get(g) >= 1) {
            cows++;
            counts.put(g, counts.get(g) - 1);
        }
    }

    return bulls + "A" + cows + "B";
}
```

上面的时间空间复杂度均为 `O(n)`。那有没有可以优化的地方呢？答案是肯定的，因为这里的数字种类是 `0 ~ 9`，我们完全可以用一个长度是 10 的数组来代替哈希表。另外，这里还有一个巧妙的方法能够让你不用记录 `bull` 出现的位置，而且只需要遍历 1 次。在我们遍历的过程中，无非两种情况，

* 当前位置下，`guess` 出现的数字出现在 `secret` 中，已经遍历过
* 当前位置下，`guess` 出现的数字出现在 `secret` 中，但还没遍历过

这里我们只需要交替记录的方法即可，就是如果当前遍历到的数字不配对，我们就记录，为了区别 `guess` 和 `secret`。在 `guess` 中出现的数字，每出现 1 次，我们算作 `+1`。在 `secret` 中出现的数字，每出现 1 次，我们算作 `-1`。具体细节可参见代码。

这样下来，我们把空间复杂度降到了 `O(1)`。另外，我们还使用了一个小技巧将代码变得更加的简洁。当然，最后的答案并不是一下子想到的，而是在有了一开始的代码后，慢慢优化而来。


<br>

**参考代码**：
```java
public String getHint(String secret, String guess) {
    int n = secret.length();

    int[] nums = new int[10];

    int bulls = 0, cows = 0;
    for (int i = 0; i < n; i++) {
        int s = Character.getNumericValue(secret.charAt(i));
        int g = Character.getNumericValue(guess.charAt(i));

        if (s == g) {
            bulls++;
        } else {
            // nums[s] 为正，表示这个数字在 guess 中出现过，并且没有被记录
            if (nums[s] > 0) {
                cows++;
            }
            // nums[s] 为负，表示这个数字在 secret 中出现过，并且没有被记录
            if (nums[g] < 0) {
                cows++;
            }

            // 记录 secret
            nums[s]--;
            // 记录 guess
            nums[g]++;
        }
    }

    return bulls + "A" + cows + "B";
}
```

<br>

## Review

[Pro Git CH8](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection)

这一章主要讲如何配置 Git，这里介绍一些常用的知识点

Git 主要有 3 个级别的配置，每个配置对应的不同的配置文件：

```
--system, 配置文件路径：[path]/etc/gitconfig 对所有的用户生效
--global, 配置文件路径：~/.gitconfig 仅对当前用户生效
--local，配置文件路径：.git/config 仅对当前代码仓库生效
```

3 个级别的配置优先级 `system < global < local`。意思如果配置项在这 3 个文件中的 2 个或多个文件出现，优先级高的会覆盖优先级低的。

举例一些常见的配置项：

* `git config core.editor`

   设置默认的编辑器。当你书写类似 commit message 一类东西时，如果不指定编辑器，那么 git 就会采用你在这里的配置。

* `git config commit.template`

   commit 模版，也可以认为是默认的 commit 信息。设定这个的目的主要是为了提醒开发者需要遵守的 commit 的格式。如果团队中有一些标准或者 commit 信息风格要求的话，配置上这个是在好不过的。
   
* `git config core.excludefile`

   git 忽略的文件。可能有人觉得这个和 `.gitignore` 干的事情重复了。但是别忘了 `.gitignore` 仅仅是对当前代码仓库有效，而配置这个可以作用于当前用户，甚至是系统。像是 macOS 中的 `.DS_Store`，以及 vim 中的 `.swp` 都是默认被忽略的。配置好后，我们就不需要将这些常见的忽略文件放入 `.gitignore` 中，省时省力。
  
<br>

## Tip

这里分享一个读书上的技巧。平时读书的时候是习惯跟着作者的思路走。作者告诉我们什么，我们就相信什么。但这仅仅是被动学习，就是被动的获取接收到的知识。这种学习方式的效率不高。更好的读书方式是不断地提问。看书的时候，每到一个章节，或是每看到一个知识点，都可以在心里默默提出一些相关的问题。这更像是在与作者互动，交流想法。这种主动学习的方式能够时刻督促自己不断寻找问题，然后不断针对这些问题在书中或者书外寻找答案。这样学到的知识会更加牢固，而且通过这种方式，我们的独立思考能力，以及寻找答案的能力也可以得到不错的锻炼。

生活中处处皆学问。同样是看书，方式方法不一样，最后的结果也会大相径庭。应该把看书看作是和作者交流互动的一种方式，而不是单纯的去刻意记住书中的知识点。

<br>

## Share

这周继续写写 JavaScript 相关的东西。对这门语言了解多了，慢慢你会发现很多和其他语言不一样的东西。记录一下，加深认识

[JavaScript避坑指南](./JavaScript避坑指南.md)