## Algorithm

[LeetCode 312. Burst Balloons 思路分析总结](./LeetCode312BurstBalloons思路分析总结.md)

<br>

## Review
两篇关于 JavaScript Promise 的文章:<br>

[Understanding promises in JavaScript](https://medium.com/@gokulnk/understanding-promises-in-javascript-13d99df067c1)

[Understanding async-await in Javascript](https://medium.com/@gokulnk/understanding-async-await-in-javascript-1d81bb079b2c)

<br>

第一篇文章讲的是关于 Promise 的工作原理和使用方法：
* Promise 里面所有的函数的返回值也都是 Promise
* Promise 有三个状态
    * pending
    * resolve
    * reject
* 对应到上面的 resolve 和 reject，promise 有三个响应机制(函数)
    * then（当 promise 的状态是 resolve 的时候执行）
    * catch（当 promise 的状态是 reject 的时候执行）
    * finally（不管 promise 状态是什么都执行）
* Promise.all(iterable)
    * 这个函数会异步执行多个 Promise
    * 返回 resolve 的 promise 当输入执行完所有输入promise或输入为空
    * 返回 reject 的 promise 当遇到第一个 reject
* Promise.race(iterable)
    * 返回第一个得到的 resolve 或者 reject 的promise 

<br>

第二篇文章讲的是 Promise 的语法糖 async 和 await 的工作原理和使用方法：
* await 表示的表达式只能被定义在 async 表示的函数中
* async 表示的函数的返回值是 promise 类型，即使你的返回值不是 promise，它也会自动将返回值包裹成 promise 的形式
* await 的作用只是阻断了程序的继续执行，直到其修饰的 promise 被 resolve
* 在循环中使用 await 时要非常小心，考虑是否可以使用 Promise.all 来 parallel 运行，不然的话，效率会受到很大影响
* 使用 await 注意使用 try catch 来进行异常的检测和处理


<br>

## Tip
最近学习了网络当中的 ABNF 定义范式，在这里罗列操作符的定义和一些核心规则

**操作符**：
* 空格 : 用来分隔定义中的各个元素
* / : 用来表示可选项
    * foo/bar   ; 表示 foo 和 bar 二者选一
* %c##-## : 表示范围
    * DIGIT = %x30-35 = "1" / "2" / "3" / "4" / "5"
* () : 表示组合，括号里面的东西视为单个元素
* ; : 注释
* \* : 用来表示变量重复的个数
    * 2*5element    ;表示 element 被允许的个数是 2-5 个
    * \*element     ;表示 element 被允许的个数是任意个，包括 0 个
    * 2*2element    ;表示 element 被允许的个数是 2 个，等效于 2element
* [] : 表示可选，等效于 *1()

**核心规则**： 规则名字是大小写不敏感的
* ALPHA     = %41-5A / %x61-7A  ; A-Z / a-z
* BIT       = "0" / "1"
* CHAR      = %x01-7F           ; excluding NUL
* CR        = %x0D              ; carriage return
* CRLF      = CR LF             ; Internet standard newline
* CTL       = %x00-1F / %x7F    ; controls
* DIGIT     = %x30-39           ; 0-9
* DQUOTE    = %x22              ; " double quote
* HEXDIG    = DIGIT / "A" / "B" / "C" / "D" / "E" / "F"
* HTAB      = %x09              ; horizontal tab
* LF        = %x0A              ; linefeed
* SP        = %x20
* WSP       = SP / HTAB         ; white space
* LWSP      = *(WSP / CRLF WSP) ; do not use when defining mail headers and use with caution in other contexts
* OCTET     = %x00-FF           ; 8 bits of data
* VCHAR     = %x21-7E           ; visible (printing) characters

<br>

## Share
这次写的是自己接触编程不到 3 年来，所看到的一些东西，希望对你们有所帮助。
<br>

[谈谈学习编程过程中的意识误区](./谈谈学习编程过程中的意识误区.md)