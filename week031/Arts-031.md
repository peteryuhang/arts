## Algorithm

[LC 75. Sort Colors](https://leetcode.com/problems/sort-colors/)

**Follow up**:

如果是 K 种颜色的话，该如何处理

<br>

**题目解析**：

给定一个输入的数组，数组当中只有 0, 1, 2 这三种元素，让你对这个数组进行排序。我们先不急着去想最优解，来看看如果不加限制地看这道题，会有哪些解法：
* 利用归并排序，时间 O(nlogn)，空间 O(n)
* 利用快速排序，时间 O(nlogn)，空间 O(1)
* 利用计数排序，时间 O(n)，空间 O(1)

三种排序算法，显然计数排序会更优，因为这里只有 3 种元素，因此计数排序的空间复杂度也是常数级别的。但是这道题最后问你的是能不能仅仅使用 One-Pass 来完成题目，**One-Pass 的意思是仅仅只有一次 for 循环遍历**，带着这个条件再来看这道题是不是会比较没有想法？思路可以从 3 种颜色这里作为突破口，3 种颜色意味着排序好的数组，存在 3 个区域，每个区域存放的元素的值都是一样的：
```
[0...0,1...1,2...2]
```
我们可以想到用两个指针去维护其中的 2 个区域：
```
[0,...0,1...1,2...2]
 ------i     j-----
```

思路大概就有了，3 根指针，第一根维护 0 的区域，第二根维护 2 的区域，另外一根**从头遍历数组**，遇到 0 就和第一个指针指向的元素交换，遇到 2 就和第二个指针指向的元素交换，当遍历的指针和第二根指针相遇了就结束遍历。这里有一个小细节就是，**遍历的那根指针在交换完元素之后需不需要马上往前移动？** 如果是和维护 0 的那根指针交换的话，因为遍历的这根指针已经遍历过这之前的所有元素了，因此交换完可以马上往前移动一个位置，但是如果是和维护 2 的那根指针交换的话，遍历的指针没有遍历过从那边交换过来的元素，交换过来的元素有可能是 0，有可能是 2，因此不能马上往前移动。

LeetCode 的上面这道题我们算是解决了，但是**如果说这里不再是 3 种颜色，而是 K 种颜色，该如何处理呢**？如果是这种情况，像上面这种指针的做法就行不通了，你可能会想到那就直接排序吧，没错，排序的思路是对的，一般的快速排序，**平均时间复杂度是 O(nlogn)**，那能不能让他变得更快些，计数排序的话可以做到 O(n) 的时间复杂度，但是空间复杂度就会是 O(K)，**如果这里还是要求你用常数级别的空间复杂度，该如何解决**？这里有一个点可能不太容易想到，平时我们想到快速排序，一般都知道，它的做法其实是利用分治的思想，把输入数组进行分割，对于这道题，需要换一种思路，就是**我们基于颜色对数组进行分割，在分割数组的同时，我们也在分割颜色**，这种做法可以把时间复杂度变成 O(nlogK)，因为颜色的数目肯定是小于元素的数目的，因此这个方法优于 O(nlogn)，具体可以参考下面的代码。


<br>

**参考代码（一）**：Sort Color
```java
public void sortColors(int[] nums) {
    if (nums == null || nums.length == 0) {
        return;
    }
    
    int pointer0 = 0, pointerTraverse = 0, pointer2 = nums.length - 1;
    
    while (pointer2 >= pointerTraverse) {
        if (nums[pointerTraverse] == 0) { // 和维护 0 的指针交换元素，遍历指针往前移动
            swap(nums, pointer0++, pointerTraverse++);
        } else if (nums[pointerTraverse] == 2) {  // 和维护 2 的指针交换元素，遍历指针暂时不往前移动
            swap(nums, pointer2--, pointerTraverse);
        } else {
            pointerTraverse++;
        }
    }
}

private void swap(int[] nums, int i, int j) {
    int tmp = nums[i];
    nums[i] = nums[j];
    nums[j] = tmp;
}
```

<br>

**参考代码（二）**：
Follow Up
```java
public void sortColors2(int[] colors, int k) {
    if (colors == null || colors.length == 0 || colors.length <= k) {
        return;
    }
    
    quickSort(colors, 0, colors.length - 1, 1, k);
}

private void quickSort(int[] colors, 
                       int start, 
                       int end, 
                       int startColor, 
                       int endColor) {
    if (startColor >= endColor || start >= end) {
        return;
    }
    
    // 对颜色进行分割，并且每次都等分
    // 并利用中间的颜色作为数组的切分元素
    int midColor = (startColor + endColor) / 2;
    
    // 快速排序的思想，只不过这里 pivot 元素变成了上面选择的颜色
    int l = start, r = end;
    while (l <= r) {
        while (l <= r && colors[l] <= midColor) {
            l++;
        }
        
        while (l <= r && colors[r] > midColor) {
            r--;
        }
        
        if (l <= r) {
            swap(colors, l++, r--);
        }
    }
    
    // 同时基于数组和颜色进行分治
    quickSort(colors, start, r, startColor, midColor);
    quickSort(colors, l, end, midColor + 1, endColor);
}

private void swap(int[] nums, int i, int j) {
    int tmp = nums[i];
    nums[i] = nums[j];
    nums[j] = tmp;
}
```

<br>

## Review
[Four Big Mistakes Every Software Engineer Has Made](https://medium.com/better-programming/four-big-mistakes-every-software-engineer-has-made-b009e7f92b5c)

程序员常犯的四大错误，文章开头导言的一句话我很认同，搞砸事情，遭遇困难都没事，**最重要的是通过这些经历去反思、去学习、去成长**。

我们一一来看文章中提到的几大常见错误：
* **想方设法试着去变得聪明**。说到这一点，很多人可能觉得会有点违反常识，聪明是好事啊，为什么在这里变成错误了呢？作者以写代码为例，如果写出过于聪明的代码确实可以提高程序运行的效率，但是他强调代码是给人看的，而且代码也是需要人来不断维护和改进的，过于强调聪明的代码往往有些地方会让人思考很久，这从长远来看并不是件好事，另外有一个 **KISS** 原则，也就是 **Keep It Simple And Stupid**，在满足了需求的情况下，设计简单，实现清晰的程序代码往往更加稳定和可靠。
* **从不锻炼**。程序员这个职业往往和各种生理健康等迹象联系在一起，比如我们常常会认为大牛级程序员大多都是秃头，弱不禁风等等。其实仔细想想程序员大多数情况下一坐就是一整天，而且也不怎么锻炼，因此身体健康会出问题。文章举了一个例子，假如说你这一生只允许买一辆车，那么你会对你的车非常的爱惜，经常保养，做各项指标检查，不轻易借给别人开，自己使用的时候也会加倍呵护。但是回头看我们的身体，终其一生，我们也只有一副身体呀，为何不好好爱护呢？时常锻炼，时常注意自己身体的状态变化是非常重要的。
* **没有充分记录**。在做软件项目的开发时，我们往往会遇到各种各样的问题，我们花了很多时间去解决这些问题，但是我们有些时候只专注于当下我们已经解决的事情，忘记去记录自己是如何一步步解决的。**人总是健忘的**。对于一个复杂的问题的解法，你如果不把每一步清晰地记录下来，到时候你可能会面对相同的问题花一样多的时间来思考和分析，我们说要学会从过去的经历中学习与成长，而记录则是这里面很重要的一个环节。同样，如果你很清晰地记录了你的项目如何部署，如何调试，API 文档之类的东西，相信这些东西也会帮助到跟你一起合作做项目的人，他们遇到问题，大概率会从你记录的文档中获知答案，因此不会过来向你询问，反过来看，这也是节约了你的时间。
* **缺乏坚持，过早放弃**。当我们面对几个复杂、耗时、或者不知从何下手的任务时，我们有很大的概率放弃，当然放弃就以为着注定得不到我们所期待的结果，文章的建议是可以把手头大的任务给拆解成一个个小的子任务，这样可以让我们更加客观地去分析一个任务，同时我们也知道自己要做什么事情，花多少时间等等。总之，程序员越走到后面，挑战会越来越大，坚持下去，因为优秀的人只有一个目标，就是让自己变得更优秀。

<br>

## Tip

这次记录一个比较实用的 Linux 挂载命令 **screen**，screen 可以帮你新开一个命令行窗口，然后你也可以通过 screen 的命令让程序在服务器上挂载，很是方便

首先检查 screen 是否安装：
```
>$ screen --version
```

在 Ubuntu 上面安装：
```
>$ sudo apt install screen
```

开一个新的 screen：
```
>$ screen
```

开一个带有命名的 screen，如果有多个 screen 的话，这个命名功能可以用作区分：
```
>$ screen -S session_name
```

screen 窗口中的命令一般是 ctrl + a ？的形式

```
ctrl + a + c          -> 在当前 screen 下新开一个 window
ctrl + a + p          -> 切换 window
ctrl + a + d          -> 挂载这个 screen，退出 screen 并回到主 shell 界面
ctrl + d              -> 删除当前的 window
```

进入一个 screen:
```
>$ screen -r <session_name | session_id>
```

我们也可以查看我们创建过的 screen，这里会显示 screen 编号还有名称（没有给名字，系统会自带），screen 创建时间，还有当前 screen 的状态，Attached 表示这个 screen 是被打开的状态，Detached 表示是挂载状态，比如我在名字为 pyh 这个 screen 中运行 `screen -ls`:
```
>$ screen -ls
There are screens on:
        14237.pyh       (10/17/2019 10:00:59 PM)        (Attached)
        19177.pts-4.ip-172-31-30-164    (08/16/2019 08:13:11 PM)        (Detached)
2 Sockets in /var/run/screen/S-ubuntu.
```

另外，有一些个性化或者是更强大的配置，你也可以通过更改 /etc/screenrc 文件来实现

一开始我想到的是如果在 screen 中开 screen 是不是可以一直嵌套下去，试了一下，发现不行，screen 只能在主 terminal 上运行，screen 只会有一层。




<br>

## Share
听池老师讲的一个发现自我的方法，感觉对自己挺有帮助的，写写文章分享出去

[One Way To Know Yourself better](https://medium.com/@peter.yuhangpeng/one-way-to-know-yourself-better-7d4ea44ce2f)