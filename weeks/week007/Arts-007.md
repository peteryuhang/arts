## Algorithm

**题目一**：<br>

[LC 41. First Missing Positive](https://leetcode.com/problems/first-missing-positive/)

**解答**：<br>

给一个整形数组，找出最小缺失的正整数，例如 [0,-1,2] 中最小缺失的正整数就是 1，[1,2,4,9] 中最小缺失的正整数就是 3。这道题如果不加上 $O(n)$ 时间和 $O(1)$ 空间这样的限定条件，应该再简单不过，但是加上了这两个要求，一下子使问题变得棘手。怎么来思考？首先这道题给定的条件很有限，输入参数就只有数组，如果非要用 $O(n)$ 时间和 $O(1)$ 空间来做的话，表示我们除了输入数组以外，不能借助任何其他的数据结构。数组应该是属于一类最最基础的数据结构，除去 length 之外，就只有两个属性 index 和 value，那这道题就变成了 “如何利用数组的 value 和 index 之间的关系来找到最小缺失正整数”，如果想到了这一点，就已经成功了一半。如果继续想下去有几点是可以明确的：
1. 缺失的正整数肯定在 [1, array.length + 1] 这个范围内
2. 我们可以交换输入数组中的元素的位置来让 index 和 value 的关系更加明确
3. 保证 index 和 value 的关系后，我们可以通过 index 来判定整数的存在性

第一点很好理解，一个数组总共有 array.length 这么多个数，全部排满，也就是 1,2,...array.length, 那么答案就是 array.length + 1，没有排满，那么在这之间肯定是有缺失元素的。第二点是说我们可以通过交换来让 index 和 value 形成对应，我们看的是 value，但是 index 可以辅助我们寻找。前两点明确了，第三点就是从头到尾寻找答案的过程。

**实现参考代码**：

```java
public int firstMissingPositive(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 1;
    }
    
    for (int i = 0; i < nums.length;) {
        if (nums[i] <= 0 || nums[i] > nums.length || nums[nums[i] - 1] == nums[i]) {
            i++;
            continue;
        }
        
        // swap
        int tmp = nums[nums[i] - 1];
        nums[nums[i] - 1] = nums[i];
        nums[i] = tmp;
    }
    
    for (int i = 0; i < nums.length; ++i) {
        if (nums[i] != i + 1) {
            return i + 1;
        }
    }
    
    return nums.length + 1;
}
```

**总结**：<br>

代码中 index 和 value 的对应关系是 index = value - 1，代码实现有两点需要注意，第一是，交换完后，需要判断交换过来的数是否需要被放到相应的地方，例如 `[2,3,1]`；第二点时，元素越界的话，以及元素 value 已经和 index 对应上了，那么就应该继续遍历，例如 `[0,-1,1]` 和 `[1,1]`。总的来说这道题并没有涉及什么算法和数据结构的应用，有点像脑筋急转弯的感觉，想到了就做的出，想不到的话就做不出，但是它给我们解数组问题提供了一个新的方向：利用 index 和 value 的对应关系来辅助求解

<br>

## Review
两篇关于 Linux 操作系统的启动和初始化的文章:<br>

[An introduction to the Linux boot and startup processes](https://opensource.com/article/17/2/linux-boot-and-startup)

[An introduction to GRUB2 configuration for your Linux machine](https://opensource.com/article/17/3/introduction-grub2-configuration-linux)

第一篇文章讲的是关于 Linux 操作系统的启动和初始化，有以下几个阶段：
* BIOS POST（Power On Self Test）
    * 检查硬件功能是否正确
    * 完毕后会触发 BIOS 中断，定位到引导扇区（为接下来的 Bootloader 作准备）
* GRUB2（GRand Unified Bootloader, version 2）
    * 启动管理器，帮助计算机能够找到操作系统内核，并把它加载到内存中运行
    * 配置文件目录在 `/boot/grub2/grub.cfg`
    * 三个阶段
        * 阶段 1，在 MBR（Master Boot Record） 扇区中找到 boot.img 文件，只有 446 Bytes，这一步只是为下一步作准备
        * 阶段 1.5，在 MBR 和第一扇区之间，对应到 core.img 文件，这是开始运行文件系统的驱动器，为下一个阶段作准备
        * 阶段 2，所有的相关文件在 `/boot/grub2/i386-pc` 目录下，这个阶段主要是找到并加载 Linux kernel 到 RAM，把计算机的控制权交给 kernel，有关 kernel 的文件，开头都是 vmlinuz，可以在 `/boot` 目录下查看现有的 kernel
        * kernel 可以自己提取和解压，kernel 运行后，加载 systemd，也就是最初始进程，并把控制权交给它，启动阶段到此结束
* 初始化阶段
    * systemd
        * 是所有进程的母进程
        * 加载文件系统，定义在 `/etc/fstab`
        * 配置文件在 `/etc/systemd/system/default.target` 里，这里面定义了什么状态和目标需要被初始化到当前用户
        * 根据需求来启动相应的 target 文件

<br>

第二篇文章讲的是如何配置 GRUB2
* 配置文件在 `/boot/grub2/grub.cfg`，产生于 Linux 的安装，每当新的 kernel 被安装，会重新生成
* grub.cfg 要找的配置文件在 `/etc/grub.d` 目录下，这些文件开头的数字是为了给 grub.cfg 文件一个参考顺序，以便对应正确。用户可以在这个目录下自己添加文件，主要目的是为了非 Linux 的操作系统，但注意必须紧跟着 10_linux 之前或之后
* 由于 grub.cfg 文件较为复杂，可以考虑更改 `/etc/default/grub` 文件来达到目的，这个文件里面存放的都是很简单的 key-value 对，下面是一些 keys 的定义：
    * **GRUB_TIMEOUT**：GRUB 启动时的 kernel 选择菜单持续时间，单位 /秒
    * **GRUB_DISTRIBUTOR**：kernel 选择菜单上面 kernel 对应的具体名称
    * **GRUB_DEFAULT**：启动时默认的 kernel
    * **GRUB_CMDLINE_LINUX**：kernel 启动时的传入参数，一经设置，对所有 kernels 生效
    * **GRUB_DISABLE_RECOVERY**：如果设成 “false”，恢复通道会在每个安装的 kernel 产生，如果设成 “true”，不会产生恢复通道
    * ...
* 使用命令 `grub2-mkconfig > /boot/grub2/grub.cfg` 来产生 grub.cfg 文件
* 建议把 GRUB_DISABLE_RECOVERY 设成 “false”，然后根据自己的定义生成 grub.cfg 文件





<br>

## Tip
在看极客时间上面操作系统专栏的时候，学习了一个 PPT 记笔记的方法，严格意义上来讲，这个不能算是一个技术技巧，应该算是一个学习技巧，但是用好了可以帮助我们更清晰高效地记笔记。步骤如下：
1. 为文章中一段知识（也可以是一个章节）做一张 PPT，标题就是段落大意，正文罗列这段当中的所有知识点，并给不清楚，不了解的知识点标号
2. 针对步骤 1 当中不清楚、不了解的知识点进行学习，查找资料补充，将资料和重要的点贴在接下来的几张 PPT 中，这里可能还会出现不清楚，不了解的嵌套知识，继续编号即可
3. 当第一张 PPT 中所有不清楚的地方都弄懂了，把所有以知识点为标题的 PPT 都放在最后当作附页

这么下来，回头看文章就会觉得非常清晰，忘了的地方也可很方便地去对应附页中寻找。这样的记笔记的方法可以让你深入一个知识点，把书读 “厚”，很适合精学。

<br>

## Share
最近写算法题写的比较多，分享的内容也都是和算法有关的，感觉题型总结、分类很重要，时常回顾也很重要。希望自己在这个阶段多多积累些基础知识，不光是算法，还有其他很多很多需要学的基础知识。
<br>

[LeetCode 滑动窗口（Sliding Window）类问题总结](LeetCode滑动窗口（SlidingWindow）类问题总结.md)