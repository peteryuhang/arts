> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 368. Largest Divisible Subset](https://leetcode.com/problems/largest-divisible-subset/)

**题目解析**：

题意是给定一个整形数组，让你从数组中选出一些元素组成一个集合，必须保证这些元素之间可整除或者可被整除，问找出最大的符合上述条件的子集。

这是一道数学相关的题目，如果 i 可被 j 整除意味着 `i % j == 0`，那我们可以再进一步想，类似，如果 j 可被 k 整除，`j % k == 0`。那么 i 和 k 是什么关系呢？没错，在这种情况下，等式 `i % k == 0` 也成立。到这个时候我们应该已经得出了一个递进关系式

```
count(i) = count(j) + 1, 如果 i % j == 0
```

这里的 `count(i)` 和 `count(j)` 分别表示的是数组中可整除 i，以及可整除 j 的元素个数，如果是题目求的是最大的子集大小，那么我们就可以写出动态规划的表达式了，如下：

```
dp[i] = max(dp[j]) + 1, 其中 i > j && i % j == 0
ans = max(dp[0]...dp[n])
```

但是，这里有两个地方需要注意，首先是数组是无序的，因为 DP 定义式中需要确定比较的元素的大小关系，因此我们需要对输入数组进行排序。另外有一点是，题目需要我们输出最后的集合，而不仅仅是集合的大小。对于这一点，我们就还需要在 DP 状态数组中保存和当前元素可约分的上一个元素的位置信息，这样的话，最后我们可以从一个元素倒回去找到其它的元素。

<br>

**参考代码**：
```go
func largestDivisibleSubset(nums []int) []int {
    if len(nums) <= 1 {
        return nums
    }

    // dp[i][0] 表示的是以 nums[i] 结尾的最大集合的大小
    // dp[i][1] 表示的是 nums[i] 上一个可约分的元素位置
    dp := make([][]int, len(nums))

    for i := 0; i < len(nums); i++ {
        dp[i] = make([]int, 2)
    }

    sort.Ints(nums)

    maxLen, index := 0, 0
    for i := 0; i < len(nums); i++ {
        // 状态初始化，nums[i] 可以被自己约分，最大集合大小是 1
        dp[i][0], dp[i][1] = 1, i
        // 在 0...i-1 之中找之前的集合
        for j := i - 1; j >= 0; j-- {
            // 如果说找到可约分的元素，并且当前集合小于之前的集合，就记录答案
            // 这里需要记录两个东西：大小，和上一个元素的位置
            if nums[i] % nums[j] == 0 && dp[i][0] < dp[j][0] + 1 {
                dp[i][0], dp[i][1] = dp[j][0] + 1, j
                
                // index 记录的是最大的集合中的最大元素的位置
                // maxLen 记录的是最大的集合的长度
                if dp[i][0] > maxLen {
                    maxLen, index = dp[i][0], i
                }
            }
        }
    }

    result := []int{}
    result = append(result, nums[index])
    // 从最大元素开始往回找到集合中的所有元素
    for dp[index][1] != index {
        index = dp[index][1]
        result = append(result, nums[index])
    }

    return result
}
```

<br>

## Review

[Hardening Your HTTP Security Headers](https://www.keycdn.com/blog/http-security-headers)

<br>

### 文章想要解决的问题

近些年来，网络安全的话题一直被反复提起。但是弄好网络安全并不是一个轻松容易的事情，这篇文章主要介绍如何通过设置一些 HTTP 响应头属性来让网络连接更加安全，这些属性会告诉浏览器和服务器通信的一些基本原则，这些原则会帮助屏蔽掉一些恶意的操作。要做到这些也非常容易，只需要知道一些基本概念和属性即可。

### 几个关键的 HTTP 安全头属性

文章介绍了几个在 HTTP 响应头属性，以及为什么我们需要设定这些属性。最后，文章还给出了检测网站是否安全的一些方法，一起来看看：

* **Content Security Policy**

  内容安全条款主要是为了防止跨域脚本文件的攻击。在安全条款中指定了被批准的信息源，也就是请求的路径，当设置了这个属性后，它定义浏览器能加载来自哪的资源，下面的例子允许从当前域名或者是 google-analytics.com 的信息源
  
  ```
  Content-Security-Policy: script-src 'self' https://www.google-analytics.com
  ```
  
* **X-XSS-Protection**

  主要用来防止 XSS 的攻击，一般的配置如下：
  
  ```
  X-XSS-Protection: 1; mode=block
  ```

* **HTTP Strict Transport Security (HSTS)**

  `Strict-Transport-Security` 头属性强制浏览器必须使用 HTTPS 与服务器之间建立通信连接
  
  ```
  Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  ```

* **X-Frame-Options**

  这个属性主要是防止 **点击劫持**（clickjacking），点击劫持是一种在网页中将恶意代码等隐藏在看似无害的内容（如按钮）之下，并诱使用户点击的手段。
  
  该属性告诉浏览器这个网页是否可以放在 iFrame 内，下面的例子告诉浏览器只有当架设 iFrame 的网站与发出 X-Frame-Options 的网站相同，才能显示发出 X-Frame-Options 网页的内容：
  
  ```
  X-Frame-Options: SAMEORIGIN
  ```

* **Expect-CT**

  Expect-CT 头允许站点选择性报告和/或执行证书透明度 (Certificate Transparency) 要求，来防止错误签发的网站证书的使用不被察觉。当站点启用 Expect-CT 头，就是在请求浏览器检查该网站的任何证书是否出现在公共证书透明度日志之中。主要用于防止欺骗的 SSL 证书被提供给客户端
  
  ```
  Expect-CT: max-age=604800, enforce, report-uri="https://www.example.com/report"
  ```

* **X-Content-Type-Options**

  这个 HTTP 头是一个提示标志，被服务器用来提示客户端一定要遵循在 `Content-Type` 首部对 MIME 类型的设定，不能对其进行修改。这就禁用了 MIME 类型嗅探行为（嗅探行为下，浏览器可以对看似错误的 MIME 类型进行修正）
  
  ```
  X-Content-Type-Options: nosniff
  ```

* **Feature-Policy**

  这个属性允许开发人员有选择的启用或者禁止浏览器的功能和 API 的规范，比如限制网站使用相机或麦克风这样的敏感 API，比如阻止网站使用像 document.write 这样过时的 API 等等。

  ```
  Feature-Policy: autoplay 'none'; camera 'none'
  ```

### 检测网站的安全性

文章给出了几个检测你的网站头属性的方法：

* 通过一个 [网站](https://tools.keycdn.com/curl) 输入你的域名地址，然后获得 HTTP 头的信息
* 利用 Chrome 的开发者工具，通过查看浏览器对你的域名请求来检测对应的 HTTP 头信息
* 通过一个叫做 [ScurityHeader](https://chrome.google.com/webstore/detail/securityheadersio-analyse/hbghndjigmobckggakgcalcfbohgkgog) 的工具帮助你分析，这个工具非常的强大，它不仅可以根据 HTTP 头信息对你的网站安全性进行打分，还会帮你总结你的 HTTP 安全相关的关键属性的内容，以及还会指出缺少什么头信息。

<br>

## Tip

记录一下这周在极客专栏中学到的有关平均负载的概念以及检测方法：

**平均负载**：单位时间内系统处于 **可运行状态** 和 **不可中断状态** 的平均进程数。这里的可运行状态指的是可执行的进程，在 `ps` 命令下的 R 的进程，不可中断状态是指一些 I/O 请求，是系统对进程和硬件设备的一种保护机制。

**平均负载和 CPU 使用率的关系**：平均负载不仅包括了使用 CPU 的进程，还包括等待 CPU 以及等待 I/O 的进程，因此在很多情况下这两个概念之间不能简单地画上等号。我们需要用到一些指令进行帮助判断：

```
>$ uptime # 显示 1 分钟，5 分钟，15 分钟内的平均负载
>$ grep 'model name' /proc/cpuinfo | wc -l # 获取逻辑 CPU 数
```

我们可以用 stress 指令来模拟进程请求
```
>$ stress --cpu 1 --timeout 600
```

另外 sysstat 里的 mpstat 和 pidstat 可以用来监控 CPU 和进程
```
# -P ALL 表示监控所有CPU，后面数字5表示间隔5秒后输出一组数据
>$ mpstat -P ALL 5
```

```
# 间隔5秒后输出一组数据
>$ pidstat -u 5 1
```

有时候遇到性能问题的时候，可以先用上述的方法查看平均负载。如果发现平均负载超过逻辑 CPU 数，这时还不能完全确定 CPU 过载，也有可能和 I/O 更繁忙了，这时需要进一步查看进程和 CPU 的使用情况。

<br>

## Share

最近读完了一本讲述如何培养习惯的书籍：[《微习惯》](https://book.douban.com/subject/26877306/)

里面提到的方法感觉对那些想要培养习惯，但是意志力和毅力又不足的人非常有帮助，这里写写读书笔记总结一下书中的核心内容，和自己的体会。

[如何轻松培养好习惯](./如何轻松培养好习惯.md)
