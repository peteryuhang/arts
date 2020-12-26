最近看了 [Pro Git](https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified) 中关于 Reset 和 Checkout 这两个指令的介绍，觉得可以从中学到一些 Git 的运行机制，在此做个记录。

## 工作流程

在具体讲这两个指令之前，先说一些 Git 的背景知识。Git 有 3 个文件存储区，分别对应的是 HEAD，Index，还有就是 Working Directory。HEAD 是 Git 中的头指针，指向的是当前分支的最近的一次 commit，HEAD 文件区里面的文件内容对应的是最近的一次 commit 的内容。index 中对应的是暂存区中的内容，表示的是下一次准备提交的内容，也就是你运行 `git add` 后加到暂存区的内容。而 working directory 就是我们的工作目录，也就是我们当前在编辑器中所看到的文件内容，这里我们简称工作区。一般的流程是，我们在工作目录中改了某个文件，或者做了某些改动，这时工作区中文件存储的内容会发生改变。当我们使用 `git add` 指令时，相关的代码会从工作区拷贝到 index 文件存储区，这时 index 区和工作区的相关内容就会是一样的。如果我们再使用 `git commit` 指令，那么 index 区中的相关内容就被拷贝到 HEAD 区，也就是生成了新的 commit。到这里，3 个区的内容保持一致。其实，我们熟知的 `git status` 指令就是通过比较这 3 个区中的内容来显示结果的，关于这 3 个区之间的关系，下图可以很好地解释：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0238ffabe1041b48477247323499717~tplv-k3u1fbpfcp-watermark.image)

关于上图的一个点，也就是 checkout 我们上面没提。这里说一下，我们熟知的一个指令是 `git checkout <branch>`，这个指令会让你去到另一个分支。跑这个指令后，发生的事情刚好是我们上面讲的逆过程，去到另一个分支，HEAD 区中的内容会发生改变，然后对应的内容会被拷贝到 index 区中去，然后再从 index 中拷贝到工作区中，这时你在编辑器中看到的文件内容就会发生变化，同时 3 个区的内容依旧保持一致。关于 checkout 我们后面还会提到，这里你只需要有个印象，它其实是 `git add` + `git commit` 的逆过程就好了。

<br>

## reset

`git reset` 是一个 Git 中非常常见的指令。我们经常用这个指令把当前的工作区的内容移除，回到当前分支最近的一次 commit 上。网上很多教程中提到 `git reset` 是一个非常危险的指令，理由是它会删除工作区的内容，并且无法撤销。于是我们脑海里就有了一个不完整的意识，“这个指令尽量少用”。

这种想法其实是我们不熟悉这个指令的一个表现，`git reset` 在很多场景下是安全的，安不安全取决于你怎么用它。废话不多说，我们直接进入主题。

reset 翻译过来是 “重置” 的意思，顾名思义，这个指令会对我们上面说的几个文件存储区的内容进行改动。一般使用 `git reset` 后面都会带标识符，有 `--soft`，`--mixed`，`--hard`。其中，`--mixed` 是默认的，也就是你不加上面 3 个的其中一个的话，会默认带上。你可能会问上面这 3 个标识符分别代表什么功能。其实这 3 个标识符表示的是 reset 的 3 个步骤，从 `--soft` 开始，到 `--hard` 结束。3 个步骤如下：

1. 移动 HEAD 和当前所指向的分支，注意这个和 checkout 不一样（checkout 仅改变 HEAD 自身），而它则是连同分支一起移动，比如 HEAD 现在指向的是 master 分支，移动过后 HEAD 还是指向 master，但是它们指向的 commit 会不一样。这一步过后， index 区和工作区的内容并没有发生改变，发生改变的仅仅是 HEAD 区中的内容。
2. 用 HEAD 区的内容更新 index 区，这一步过后，index 区的内容会和 HEAD 区保持一致，但是工作区，也就是你在编辑器中所看到的内容并不会改变。
3. 用 index 区的内容更新工作区，这一步过后，3 个区的内容保持一致

如果你使用 `--soft`，那么 reset 会完成第 1 个步骤然后停止，使用 `--mixed` 会完成前 2 个步骤然后停止，而使用 `--hard` 会完成全部的 3 个步骤。**导致 reset 是一个危险的指令其实只是这里的第 3 个步骤，更准确地说就是 `--hard` 标识符**。之所以危险，是因为一旦运行这个指令（`git reset --hard`），之前在工作区中改变的内容将无法恢复。而如果仅仅是使用 `--soft` 或是 `--mixed`，我们完全可以通过运行 `git add` 以及 `git commit` 来重新记录之前的改动，这里文件内容并没有丢失。

理解了上面的 3 个步骤，也就理解了 `git reset` 这个指令的基本原理，常见的一些 `git reset` 的用法想想也就不难理解。比如说对某个文件进行 reset 操作，`git reset <filename>`，把这个指令省略的地方补全，其实是 `git reset --mixed HEAD <filename>`。它会把当前 HEAD 区的内容更新到 index 区，工作区保持不变，仔细一想，这其实是 `git add` 的逆向操作。

再比如，通常在 GitHub 最后 merge 的时候都会有一个 "Squash and merge"，这里的 squash 是把几个 commits 整合成一个 commit，也就是 commit 的压缩。该如何做呢，其实只需要下面两步：

```
git reset --soft <previouscommit>
git commit
```

通过 `git reset --soft` 指令使 HEAD 退回到之前的某个 commit，这时 index 区的内容没有改变，还是最近一次 commit 的内容。然后重新 commit 就会将 index 区相较此时 HEAD 区不同的内容整合成一个 commit。

<br>

## checkout

和 reset 指令类似，`git checkout` 同样是操纵 3 个文件区，但是有所不同。我们先来看 `git checkout <branchName>` 如果是 checkout 到某个分支或者是某个 commit，Git 只会移动 HEAD 指针，并不会对 HEAD 所指向的分支带来任何的改变。运行这个指令会改变 HEAD 区，同时也会改变 index 区以及工作区的内容，这有点类似 `git reset --hard`。但是这个指令是完全安全的，当你运行这个指令时，Git 会检测你的工作区还有 index 区是否有未提交的内容。如果有的话，checkout 就会失败，然后提示你需要先进行提交保证 3 个区的内容是一致的。

另外，`git checkout` 后面还可以接文件或者文档路径。这时，这个指令就是不安全的，它等同于 `git reset HEAD --hard fileName`。运行后，工作区的内容会被改变。

<br>

## 总结

总的来说，关于 Git 中的 reset 和 checkout 就这么多的内容。这里面涉及到几个文件存储区之间的关系，以及 HEAD 指针的移动。我们解释了在什么情况下运行这两个指令是安全的，什么情况下又是不安全的，这主要看工作区中未提交的内容是否会被覆盖。同时，我们了解了这两个指令和 Git 中其他的指令，比如 `git add`，以及 `git commit` 的潜在关系，并知道如何灵活应用它们去实现一些很常见的机制，比如 commit 的压缩（squash）。下表列出了 `git reset` 和 `git checkout` 的一些基本用法，可以用作回顾：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ff9c6f58b1e4094bbc68cbfb7c0fe00~tplv-k3u1fbpfcp-watermark.image)


