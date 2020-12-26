> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 241. Different Ways to Add Parentheses](https://leetcode.com/problems/different-ways-to-add-parentheses/)

**题目解析**：

题意是对一个计算式子增加括号，比如输入的式子是 `2-1-1`，那么可以对其增加括号，使其变成 `2-(1-1)` 或者是 `(2-1)-1`，可以看到的是括号的位置的不同会使这个式子最终的结果不同。题目需要列出所有可能的结果。

刚看到这道题，可能比较直接的思路是不断地去试着化简式子，比如 `2-1-1` 会变成 `1-1`，或者是 `2-0`。依照这样的思考方式，到最后会发现如果式子一旦变长，逻辑会变得比较混乱。**其实，这里我们只需要关注运算符即可**，一个运算符表示的就是一次化简操作，有 n 个运算符，那么这个式子就会被化简 n 次，**只是说这 n 个运算符的先后顺序决定了最后的结果**。如果是这样子，那么这个问题其实就等同于一个排列的问题，解题的思路也就有了。

在实现的时候，我们可以倒着来想，并递归拆分这个问题。什么意思？还是拿 `2-1-1` 来举例，当我们看到第一个减号的时候，我们假设这个减号是**最后一个**被执行的运算符，由于是最后一个被执行的运算符，那么这个减号左右两边的式子已经有了结果。这时，将这个减号左右两边分成两个等式依次递归拿到结果，由于这两个等式被这个运算符隔离来，这两个等式先后并不受影响。这样不断地递归拆分下去，最终我们就可以得到当前考虑的运算符最后被执行的所有可能性。要把所有的情况考虑到，我们也只需要遍历，然后用同样的方式拆分加递归即可。

<br>

**参考代码**：
```java
public List<Integer> diffWaysToCompute(String input) {
    List<Integer> results = new ArrayList<>();

    int n = input.length();

    for (int i = 0; i < n; ++i) {
    	// 仅仅考虑运算符
        if (input.charAt(i) == '+' || input.charAt(i) == '-' || input.charAt(i) == '*') {
            // 将当前式子以运算符为分割，拆分成两个式子
            String p1 = input.substring(0, i);
            String p2 = input.substring(i + 1);

            // 递归去拿到左边式子的所有结果
            List<Integer> r1 = diffWaysToCompute(p1);
            // 递归去拿到左边式子的所有结果
            List<Integer> r2 = diffWaysToCompute(p2);

            // 把两边式子的结果组合起来形成当前式子的结果
            calAndRecTheAnswer(r1, r2, results, input.charAt(i));
        }
    }

    // 特殊情况，当前式子仅仅是一个数字，直接将数字放入结果列表中返回
    if (results.size() == 0) {
        results.add(Integer.parseInt(input));
        return results;
    }

    return results;
  }

  private void calAndRecTheAnswer(List<Integer> r1, 
                                List<Integer> r2, 
                                List<Integer> result,
                                char oper) {
    for (int r11 : r1) {
        for (int r22 : r2) {
            if (oper == '+') {
                result.add(r11 + r22);
            } else if (oper == '-') {
                result.add(r11 - r22);
            } else if (oper == '*') {
                result.add(r11 * r22);
            }
        }
    }
  }
```

<br>

## Review

[Pro Git CH7](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection)

继续 Pro Git，这次看到了第七章。对于这章，内容比较多，分成两次来记录。个人认为这一章介绍的很多工具或指令可以帮助我们更高效地来运用 Git。

* **修正和检索**

  我们知道，`git log` 指令可以帮助我们查看提交日志。这个指令只是查看所有的提交，更具体来说，其实是看这个代码仓库的提交树是什么样的。如果说要具体看到某一个提交，我们可以使用 `git show` 这个指令，这个指令可以接 commit hash，或者是分支名称，如果是接分支名称，那么查看的就是最近的一个提交，如：

  ```
  git show a96600c4ee40b380860a5724288274cf29fb81be
  git show topic_branch
  ```
  
  另外，这里还介绍了一个 `relog` 的概念。按字面意思翻译过来就是 “引用日志”，更准确一点的解释是，HEAD 以及分支的引用变化，每当你改变分支历史的时候，比如 `git merge`，`git rebase`，`git commit`，这些指令都会对分支的历史带来改变。这些改变会被 log 起来，这里的 log 就是 relog。你可以通过 relog 查看某项操作之后的代码状况。relog 可以看成是 git 的操作日志，这个在有些时候非常有帮助。但是 relog 并不会保存所有的操作信息，只会保存最近几个月的，所以仅仅依赖于 relog 来保存和回顾历史肯定是不够的，只能说 relog 可以在有些时候为我们查阅仓库历史提供遍历。
  
  这里再介绍几个 git 中常见的符号：
  
  * `^`
	
    用于表示当前 commit 的 parent commit。这个符号后面可以接数字，但是这个只是对 merge commit 有用，用于横向的表示 merge 之前的 commits。
    
    上面说的是这个符号放 commit 或分支后面的情况，当这个符号放在前面，则可以表示非的意思，等同于 `--not`，如下面两个指令都表示展示存在于 `refA`，`refB`，但是不存在于 `refC` 的所有 commits
    
    ```
    git log refA refB --not refC
    git log refA refB ^refC
    ```

  * `～`
  
    和前面一个类似，也是用于表示当前 commit 的 parent commit。这个符号后面也可以接数字，适用于所有的 commit，这时就是表示一个 commit 在同一个分支上的 parent。
  
  * `..`
  
    表示的是一个分支上有，另一个分支上没有的所有 commit，比如：
    
    ```
    git log master..exp         # 存在于 exp 分支上，但是不存在于 master 上的所有 commits
    git log origin/master..HEAD # 你将要 push 的 commits
    git log origin/master..     # 上面的简略写法
    ```

  * `...`
  
    这个用的比较少，用法和双点类似，但是表示的意思是不同时存在于两个分支上的 commits。可以用于找出两个 branch 的差异

* **交互式暂存**

  `git add` 相信用过 Git 的人都不会陌生，但是我们用的多的功能往往是 `git add -A`。这个指令会把工作区的所有内容添加到暂存区。但很多时候，我们的需求不知这一个，我们可能仅仅想添加改动的某一部分到暂存，有时我们甚至想把暂存区的内容重新放回工作区。这里，`git add -i` 就派上用场了。操作也非常直观和简单，只需要按照命令行界面给的提示进行下去就行。

* **存储与清理**
  
  另外说到存储，有一个场景是。我们想要去到另外一个分支进行开发，但是当前分支的工作区的内容并没有完成，我们不想对当前工作区的内容进行提交。这时就可以用到 `git stash` 这个指令将工作区和暂存区的内容缓存起来，让我们可以方便地跳到其他分支而不用担心历史修改丢失。具体的使用方法如下：
  
  ```
  git stash or git stash push # 把当前工作区和暂存区的内容推入到缓存栈中
  git stash list              # 列出当前缓存栈中的内容
  git stash apply <stash@{..}>  # 把缓存栈中的制定内容拉出来，后面不指定表示最近的一次
  git stash drop <stash@{..}> # 删除指定的缓存
  git stash pop               # 把最近一次缓存的内容拉出，并将其删除
  git stash --patch           # stash 部分内容  
  ```

  另外 git 也可以帮助我们清理代码仓库中不需要的目录和文件，指令如下
  
  ```
  git clean -f -d
  ```
  
  这个指令会清除仓库中所有未追踪的文件和子目录。使用这个指令的时候需要格外小心，如果你不确定它会删除什么，可以试着运行 `--dry-run` 或 `-n` 选项来做一次演习，git 会告诉你当前指令将会移除什么。

* **搜索**

  git 当中有一些非常便利的搜索操作，主要是 `git log` 以及 `git grep`，可以方便我们查找到指定的内容：
  
  ```
  git grep -n/--line-number str # 在工作区中寻找 str 出现的位置，并打印出行号
  git grep -c/--count str       # 在工作区中寻找 str，并统计 str 在每个文件中出现的次数
  git grep -n/--line-number str # 在工作区中寻找 str 出现的位置，并打印出行号
  ```
  
  另外，`--break` 以及 `--heading` 可以让打印出来的东西更整洁。
  
  `git grep` 指令主要是查找工作区，或者说当前工作目录中的内容。如果需要查找提交历史相关的内容，我们可以用到 `git log` 相关的操作：
  
  ```
  git log -S str # 找出那些对 str 的出现次数有改变的 commits
  git log -L :funcName:"filepath" # 找出文件中某个函数相关的 commit 记录
  ```
  



  
<br>

## Tip

这次记录下在使用 Ant Design Pro 时的几个注意点：

* 在测试开发的过程中，我们是可以在手机上查看效果的。方法也很简单，在每次 `npm start` 启动 Ant Design Pro 后，一般会给出局域的 IP 地址，如：

    ```
     App running at:
     - Local:   http://localhost:8000 (copied to clipboard)
     - Network: http://10.0.0.118:8000
    ```
   这里，我们只需要在手机上浏览器中输入 `http://10.0.0.118:8000` 即可访问页面。但是这里有个问题，如果说是后台服务器也是在 `localhost` 本地的话，手机端是无法发送请求的。这里只能是测试一下网页页面在手机端的显示，至于说是服务器的测试还是移到电脑端做吧，毕竟是一样的。
 
* Ant Design Pro 的权限管理可以在 `config.js` 文件中配置。具体的权限获取的逻辑在 `src/utils/authority.js` 的 `getAuthority` 函数中。其实就是把从后台得到的用户数据放到 `localStorage` 中，然后框架会根据 `getAuthority` 函数返回结果以及 `config.js` 文件中的配置型来进行判断。当然这是最初始的逻辑，如果需要复杂一点的验证，则需要自己改对应的逻辑来达到目的。

<br>

## Share

看完来 Pro Git 中关于 reset 与 checkout 指令的介绍，记录一下，写写自己的一些理解

[Git 中的 Reset 与 Checkout](./Git中的Reset与Checkout.md)


