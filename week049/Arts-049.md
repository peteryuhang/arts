> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。


## Algorithm

[LC 1354. Construct Target Array With Multiple Sums](https://leetcode.com/problems/construct-target-array-with-multiple-sums/)

**题目解析**：

题目让你用一个元素全为 1 的数组构造一个 target 数组，规则是每一次操作可以将数组中所有元素的和赋值给其中一个元素，然后反复迭代进行，问你能否构建 target 数组。

这道题如果顺着题目想很难得到可行解，你可能会尝试使用暴力的深度优先搜索，举个例子，你要构建 `[9,3,5]`，起始数组是 `[1,1,1]`，第一次迭代你得到了数组和为 3，那么这个 3 放在数组下标 0,1,2 三个位置都是可行的，你需要每个位置都尝试放一下，这么做是一种解法，但是时间复杂度是指数级别的，对这道题目的输入数据大小来说，不适用。暴力搜索走不通，你或许会想当然的往动态规划的方向思考，但是这个方向也很难走通，原因在于你很难拆解问题，定义状态。因此这里我们应该换一种思考方式了。

**既然从前往后思考问题得不出答案，那我们就倒着想**。我们从我们最后想要构建的数组出发（结果），往前推，看能不能推得出全为 1 的初始数组。怎么推？在回答这个问题之前，我们首先要知道的是，数组元素之间的联系，根据题目描述，我们可以得知数组当前的所有元素和 `sum` 与最大的元素存在着一定的关系，`sum` 是下一次迭代的最大值，那么之前迭代的最大值是否可以根据这个计算出来呢？答案是肯定的，我们来根据例子看看：
```
[a_max,a_2,a_3,...]

为了简便，我们令 a_other = a_2 + a_3 + ...

sum_prev = a_max              // 之前的 sum 就是当前所有元素的最大值
sum_cur = a_max + a_other     // 当前的 sum 就是当前所有元素之和

// 因为数组中的元素是在不断改变的，每次只变一个元素
// 如果要知道之前所有元素的最大值，我们必须还原当前改变的元素
// 当前改变的元素就是 a_max
sum_prev = a_prev + a_other = a_max
a_prev = sum_prev - a_other
       = a_max - (sum_cur - a_max)
       = 2 * a_max - sum_cur
```

通过上面的一系列推导，我们可以通过当前改变的值（最大值）推得出这个元素在改变之前的值，然后不断地反向推导，就能得出数组里面元素最原始的状态，我们再看是否能得到全为 1 的最原始的数组。那么这里又存在一个问题，如何动态找出所有元素中的最大值？这个简单，我们可以借助**最大堆**这个数据结构，它可以帮助我们在 `logn` 的时间维护一个排序好的序列。有了这些，这道题基本上就可以解决了。

以上就是最为基本的解题思路，当然这道题目会存在一些比较极端的 case，比如 `[1, 100000000]` ，因此我们要对之前的解法进行优化。我们之前说过，我们需要考虑 `sum` 还有就是 `max`，但是当 `max` 相对于其他元素过于大的时候，我们需要对这一个元素反复做同样的事情很多次，这个过程能否简化呢？这里我们可以用一个取余的方法做到：

```
[2, 100]

sum = 102, max = 100, other = 2  
sum = 100, max = 98, other = 2
sum = 98, max = 96, other = 2
...

// 如果 max 与 other 差距过大，每次迭代我们其实都在更新 max
// 因为 other 没有变，每次 max 减去的都是相同的值
// 这里，我们可以利用取余的方式来使得 max 一次性更新到刚好比 other 小的值
// end_max = max % other
```

取余其实是一个很常用的数学技巧，能够很好地帮助我们节省时间。这道题目有点偏数学了，它的思想其实有点贪心的意思在里面，也就是把局部解衍生到全局。**通过这道题，其实告诉我们以后思考问题正着想不通，我们完全可以从结果出发，倒着想，说不定有奇效**。

<br>

**参考代码（TLE）**：

```java
public boolean isPossible(int[] target) {
    PriorityQueue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder());

    long sum = 0;
    for (int i : target) {
        pq.offer(i);
        sum += i;
    }

    while (pq.peek() != 1) {
        int max = pq.poll();

        long prev = max * 2 - sum;
        if (prev <= 0) {
            return false;
        }

        pq.offer((int) prev);

        sum = max;
    }

    return true;
}
```


**参考代码**：

```java
public boolean isPossible(int[] target) {
    PriorityQueue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder());

    long sum = 0;
    for (int i : target) {
        pq.offer(i);
        sum += i;
    }

    while (pq.peek() != 1) {
        int max = pq.poll();

        // 获得 other
        sum -= max;

        // other = 1，不管 max 是多少，都可以通过 1 不断迭代而成
        if (sum == 1) {
            return true;
        }

        // 由于是反向推，只减不加
        // other 小于 1，说明无法构建
        // other 必须小于 max 来保证 max 的正常迭代递减，不然 max 无法递减到 1
        if (sum <= 0 || max <= sum) {
            return false;
        }

        // 通过取余操作将 max 迭代到比 other 小的地方
        max %= sum;

        // other + prev_max = prev_sum
        sum += max;

        // prev_max 放入队列，继续迭代
        pq.offer(max);
    }

    return true;
```
<br>

## Review

[Being a Programmer is Not About Writing Code](https://medium.com/the-heart-of-programming/being-a-programmer-is-not-about-writing-code-84aa54cb1c61)

做一个程序员是不是就是写好代码就好了？这篇文章讲述了一个老生常谈的问题，那就是作为程序员的我们很多时候太过于依赖于框架。框架并不是不好，框架帮助我们掩盖了很多的技术细节，往往我们只需要实现好框架的一小部分就可以完成我们期待要完成的任务。框架会给我们一种幻觉，这种幻觉让我们很简单的认为掌握了很多很有用的知识，完成了很多很有意义的项目。**但其实我们仅仅是在配置一些高级程序员的杰作，我们是站在别人的肩膀上看世界，却觉得自己非常的高大**。

文章中把学习编程和学习下棋做了一个对比，刚学习下棋，我们会先学习每个子的走法，比如兵只能向前一步，马走日等等。等到这些熟练了之后，我们就需要走出去学习一些新的东西，因为下棋远不止这些，这只是第一步，你还需要学习策略，记忆棋谱，大局判断等等。学习编程也是一样的道理，会用了一两个框架，学会了一两门编程语言，绝对不等同于你掌握了编程的全部，会这些这也仅仅是入门。你还需要学习很多，比如设计你的 class、学会创建一些可以复用的模块、学习函数式编程、学习并应用一些算法、创建一些省时省力的自动化工具、从零构建一个网络应用。。。

总之，学习没有终点，写下那些你还没有掌握的东西，慢慢设定学习计划，不要止于框架。

<br>

## Tip

这周继续 go 语言

Go 中的面向对象：

* 结构体（类）定义
    ```go
    type Employee struct {
    	Id string
    	Name string
    	Age int
    }
    ```
* 实例创建以及初始化
    ```go
    e := Employee{"0", "Bob", 20}
    e1 := Employee{Name: "Mike", Age: 30}
    e2 := new(Employee)		// 这里返回的引用/指针，相当于 e := &Employee{}
    e2.Id = "2"
    e2.Age = 22
    e2.Name = "Rose"
    ```
* 行为（方法）定义
    ```go
    func (e *Employee) String() string {
        return fmt.Sprintf("ID:%s-Name:%s-Age:%d", e.Id, e.Name, e.Age)
	}
    ```
* 接口定义与实现
    ```go
    type Programmer interface {
	    WriteHelloWorld() string
    }
    
    type GoProgrammer struct {}

    func (p *GoProgrammer) WriteHelloWorld() string {
        ...
    }
    ```
    
包和依赖管理：

首先我们需要增加环境变量，Mac 下 `~/.bashrc` 文件中添加：
```
export GOPATH="/Users/pyh/go
```
Go 的编译器会自动在 GOPATH 后面添加 `src` 目录

* init 方法
    * 在 main 被执行前，所有依赖的 package 的 init 方法都会被执行
    * 不同包的 init 函数按照包导入的依赖关系决定执行顺序
    * 每个包可以有多个 init 函数（意味着可以重名）
    * 包的每个源文件也可以有多个 init 函数
* 安装第三方包（注意不能有前缀 http:// 也不能有后缀 .git）
    ```bash
    $> go get -u github.com/easierway/concurrent_map
    ```
* 最新版本的 Go 中查找依赖包路径的解决方案
	* 当前包下的 vendor 路径
	* 向上级目录查找，直到找到 src 下的 vendor 目录
	* 在 GOPATH 下面查找依赖包
	* 在 GOROOT 下面查找依赖包
* 常用的依赖管理工具：
    * godep
    * glide
    * dep

<br>

## Share

这次想向大家推荐一本书 《[Clean Code](https://book.douban.com/subject/4199741/)》，可能你已经读过，也可能你没有读过，不管读没读过都没有关系。我相信你一定知道这本书，我读过的关于这本书的文章和感想都不下 10 篇，可以算作是编程界的必读之书。花了一些银子弄到了这本书的纸质的英文原版，准备细嚼慢咽。我看了一下，全书大概有 16 章，准备用 5 ～ 7 周的时间来读这本书，这次我的目的是**精读 + 剖析**，我每周会把我读的部分给分享出来，写写学到的东西，总结一下这些新学的东西和之前掌握的东西的一些关联，再说说感想。因为新书刚刚到手，这次我仅仅是立个 Flag，下周正式开始。**如果你也想要一起读书，一起分享，随时欢迎，相信通过交流分享，我们能够学到很多我们完全没有听说，或者不知道的东西，这是非常难能可贵的，因为这会扩大你的视野，而不仅仅是知识的堆叠**。
