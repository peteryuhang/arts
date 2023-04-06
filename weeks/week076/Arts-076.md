> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 187. Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/)

**题目解析**：

题意算是简单明了，输入是一个字符串，这个字符串表示的是整个 DNA 序列。让你找出重复的长度为 10 的子序列。

拿到题目后，可以尝试找出可行解。很明显的是，我们可以把一个 DNA 序列的所有子序列给枚举出来，比如说输入的字符串是 `AAATTTGGGCC`，那么长度为 10 的子序列也就 `AAATTTGGGC` 以及 `AATTTGGGCC`。同时，你也可以很容易地发现规律，`子序列的个数 = DNA序列的长度 - 9`。可见的是，总的 DNA 子序列并不算多，因此暴力解也是可行的。我们只需要用两个哈希表，从左到右遍历 DNA 序列，把出现过的子序列存到其中一个哈希表中，如果这个子序列再次出现就把它放到另一个哈希表中，最后，第二个哈希表中存放的就是我们想要的答案。由于子序列的长度固定为常数 10，而且子序列的总个数可以看似等同于 DNA 序列的长度，因此，最后的时间复杂度就是 `O(n)`。

上面的解法的确是可行的，但是也不难发现其中的问题。首先，在这之中我们需要频繁地去使用类似 `substring` 这样的函数，由于字符串的不可变性，使用这些函数等同于重新生成一个新的字符串。这会大大增加程序运行时的消耗。另外，**这道题目限定了子序列的长度是 10，如果说子序列的长度不限定又该怎么办**？这里就不得不说一个叫做 **滚动哈希** 的技巧，[Robin-Karp 算法](https://en.wikipedia.org/wiki/Rabin%E2%80%93Karp_algorithm) 中运用了这个技巧，把常规的字符串比较的时间复杂度从 `O(n^2)` 降至 `O(n)`。其核心的原理是把一个字符串表示成正整数，并在遍历的过程中不断地更新这个正整数，这个正整数被称之其所表示的字符串的哈希值。

当然了，即使是我们使用了滚动哈希，最后的时间复杂度仍然是 `O(n)`。但是总体来说，我们会减少新的字符串被创建的次数，也就是说这个时间复杂的常数项会比前面一个解法来的小。在面试当中，面对同一个问题，能够有不同的想法和思路，相信对自己的自信也会有不小的提升。

<br>

**参考代码**：

**解法 1（暴力枚举）O(n)**
```java
public List<String> findRepeatedDnaSequences(String s) {
    Set<String> hash = new HashSet();
    Set<String> repeated = new HashSet();

    for (int i = 10; i <= s.length(); ++i) {
        String cur = s.substring(i - 10, i);
        if (!hash.add(cur)) {
            repeated.add(cur);
        }
    }

    return new ArrayList<>(repeated);
}
```

**解法 2（滚动哈希）O(n)**
```java 
public List<String> findRepeatedDnaSequences(String s) {
    if (s.length() <= 10) {
        return new ArrayList<>();
    }

    Map<Long, Boolean> hash = new HashMap<>();

    Map<Character, Long> mapping = new HashMap<>();

    mapping.put('A', 1L);mapping.put('C', 2L);
    mapping.put('G', 3L);mapping.put('T', 4L);

    char[] sArr = s.toCharArray();
    long curHashValue = 0L;
    for (int i = 0; i < 10; ++i) {
        curHashValue = curHashValue * 10L + mapping.get(sArr[i]);
    }

    hash.put(curHashValue, true);

    List<String> results = new ArrayList<>();
    for (int i = 10; i < sArr.length; ++i) {
        curHashValue %= 1e9 * mapping.get(sArr[i - 10]);
        curHashValue = curHashValue * 10 + mapping.get(sArr[i]);

        if (hash.containsKey(curHashValue)) {
            if (hash.get(curHashValue)) {
                results.add(s.substring(i - 9, i + 1));
                hash.put(curHashValue, false);
            }
        } else {
            hash.put(curHashValue, true);
        }
    }

    return results;
}
```

<br>

## Review

[Pro Git CH6](https://git-scm.com/book/en/v2/GitHub-Account-Setup-and-Configuration)

继续读 Pro Git，第 6 章主要讲的是 GitHub 的使用和技巧。

在 GitHub 上很多时候都是协作开发，在 GitHub 上贡献代码的一般流程如下（比如合作开发一个开源项目）：

1. **fork 想要开发的项目**

   这一步主要是把一个项目拷贝一份到自己的仓库中，注意这里的仓库是 GitHub 的仓库，也就是说这一步的拷贝是在远端，和本地没有关系。fork 的本意是叉子，但是这里指的是创建一个分流，方便合作开发，想想叉子的形状你就理解了。

2. **clone 并创建分支**

   这一步是把上一步自己 fork 的仓库在本地克隆一份，并基于需求创建一个分支。

3. **开发，创建 commit**

   这一步就是开发了，有代码改动就会有 commit 生成。

4. **提交 pull request**

   这里其实有两步，第一步是把自己的代码改动 push 到自己的 GitHub 仓库。然后是向原仓库提交 pull request。因为原仓库并不是自己的，如果想要自己的代码改动被纳入原仓库，那么就需要向原仓库的管理员提交申请。pull request 就是提交这么样一个申请的意思，从字面意思也很好理解，pull 就是指 `git pull`，表示更新分支，request 就是请求，合在一起表示请求更新当前主分支（仓库）。

5. **讨论**

   有可能提交完 pull request 后，管理员会有一些疑问或者是不太理解的地方。这个时候你们就需要针对当前的改动进行讨论，这个过程也称为 “**代码审查（code review）**”。

6. **merge 到主分支**

   如果说上面的讨论进行的还顺利，原仓库的管理员同意你的变更，那么这时就可以把当前 Pull Request 分支上的改动合并到原仓库的主分支上去。这一步结束后，你就成功为原仓库贡献了代码。

上面的这 6 步是常见的一个开发流程，GitHub 在这当中的每一步都有具体的提示和工具。这里我们只需要理解每一步的意义即可，实际做过几次就基本上熟练了。这里说几个比较有用的技巧：

* **对已经提交 Pull Request 的分支最好是使用 merge 而不是 rebase 进行更新**
  
  这一点在一般的开发中比较常见，场景是你提交了 Pull Request，但是在讨论中发现自己的代码存在着一些问题，于是需要在本地更改代码后再次提交。但是在这个过程中，其它人往主分支上 merge 了一些新的 commits，你需要把这些 commits 更新到当前的分支。这个时候可以 merge 进当前分支生成一个 merge commit，然后在这个 merge commit 上增加 commit；还有一种方式是直接使用 rebase，如果是使用 rebase 的话，会导致整个 Pull Request 分支的起始点变更，那么 push 的时候就需要带上参数 `-f` 表示强制 push。在这里是不推荐使用 rebase 的，原因在于如果其他人已经把你的 pull request 给拉到本地进行测试和审查了，那么由于分支的起始节点发生了变化，再次更新就会存在问题。

* **引用链接**

  GitHub 上的引用非常简单，也非常使用。比如你需要链接到某个 Pull Request 或者 issue，那么只需要简单地用 `#` 跟上数字即可，在一个仓库中，一个数字不会对应到多个 PR 或者是 issue。当然你也可以在前面带上用户名和仓库名：
  ```
  userName#<num>
  userName/repo#<num>
  ```

* **将某个还没有 merge 的 pull request 拉到本地测试**

  其实 pull request 也是分支。只是通常来说这个分支一般都会是在远端，我们在本地是无法更改的，但是我们其实可以 checkout 到这个分支去测试和审查代码。一般我们可以用 fetch 去拉特定的 PR，这里只需要给定 PR 的编号即可，比如：
  ```
  git fetch origin refs/pull/<pr#>/head
  ```
  上面这个指令只会把特定的 PR 给拉到本地，其中 `refs/pull/<pr#>/head` 表示的是特定的 PR 在远端的链接。如果我们想要去到某个 PR 上，我们可以先在 `.git/config` 文件中配置一下：
  ```
  [remote "origin"]
    url = ...
    fetch = +refs/heads/*:refs/remotes/origin/*
    fetch = +refs/pull/*/head:refs/remotes/origin/pr/*
  ```
  上面的第一个 fetch 是默认的，表示的是把远端 `refs/heads/` 下的内容存入到本地 `.git/refs/remotes/origin/` 中去，但是 `refs/heads/` 并不包括正在进行的 PR。我们在其下面加了另一个 `fetch`，意义也是类似的，但是这次我们是把 PR 给拉到了本地仓库。配置完后，如果 `fetch` 顺利，我们就可以 checkout 到对应的 PR 上了：
  
  ```
  git checkout pr/#..
  ```

* **GitHub 的一些额外工具**

  除了上面说的这些最基本的 git 相关的服务。GitHub 上面还提供了很多工具，主要有 3 个，services, webhook，还有就是 api。前面两个比较类似，就是当你 push 代码到仓库的时候，会触发你在 GitHub 上面配置的一些工具，比如让 GitHub 去根据 push 上来的东西，发送请求到特定的 URL，然后根据 URL 返回的结果来确定 push 上来的代码是否符合规范，这个有时候可以帮助我们自动地检测代码中存在的问题。
  
  另外，GitHub 提供了丰富的 API，基本上你能在 UI 上干的所有事情都可以通过 GitHub API 来完成。运用这些 API 可以帮助我们自动化，从而降低开发的难度，提升开发的效率。

<br>

## Tip

这次来记录一下 **双因素认证（Two Factor Authentication/2FA）** 这个东西。这个主要是用于用户身份验证，从字面上很好理解，就是双重验证。在传统的验证中，用户只需要输入用户名和密码即可，但是这种单因素验证有时并不可靠，如果密码泄漏那么就会存在账户被盗的风险。因此目前都会提倡使用多种因素来进行验证，因素越多就越安全。双因素认证下的身份验证，除了你的密码外，还需要另一个因素来保证身份的合法性，这个东西往往就是指手机，当然有些情况下还指代其它物品，比如网上银行的 U 盾。

有些时候，我们输入密码后，还需要通过手机的短信验证。但是短信这东西并不靠谱，容易被拦截，而且 SIM 卡也容易被克隆。于是就有了 **TOTP（Time-based one-time password）**，在用户开通双因素验证后，服务器会生成一个密钥，然后发到用户的手机中，这个密钥用于生成 hash。每当用户端生成了一个 hash，把 hash 发送给服务器端，服务器端会用相同的密钥来生成一个 hash，然后把用户发送过来的 hash 和生成的 hash 进行比较。关于这部分的具体实现，可以看看这篇文章: [双因素认证（2FA）教程](https://www.ruanyifeng.com/blog/2017/11/2fa-tutorial.html)

<br>

## Share

这周读了《全神贯注的方法》的作者写的另外一本书，写写读书笔记

[练习的心态](./练习的心态.md)
