> ARTS是什么？<br>
> **Algorithm**：每周至少做一个leetcode的算法题；<br>
> **Review**：阅读并点评至少一篇英文技术文章；<br>
> **Tip**：学习至少一个技术技巧；<br>
> **Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 763. Partition Labels](https://leetcode.com/problems/partition-labels/)

**题目解析**：

题目给定一个字符串，让你根据字符的出现位置对字符串进行分割，保证相同字符只会出现在同一个块中。一开始的时候，大致的思路是，第一遍遍历字符串拿到每个字符出现的次数，然后接着第二遍遍历去找答案。这里可以借助一个哈希来判断是否可以分割，如果当前遍历的元素是不是最后一次出现，就把当前元素放到哈希中去，如果是最后一次出现，那么就把该元素从哈希中移除。然后，根据哈希是否为空来记录答案，于是我得到了下面的代码：

```java
public List<Integer> partitionLabels(String s) {
    int[] counts = new int[26];
    List<Integer> results = new ArrayList<>();
    Set<Character> rep = new HashSet<>();

    for (char c : s.toCharArray()) {
        counts[c - 'a']++;
    }

    int len = 0;
    for (char c : s.toCharArray()) {
        len++;
        if (counts[c - 'a']-- == 1) {
            rep.remove(c);
        } else {
            rep.add(c);
        }

        if (rep.isEmpty()) {
            results.add(len);
            len = 0;
        }
    }

    return results;
}
```

本来以为这道题目就完事了，但看过答案才发现还有另外一种思路。这种思路下，第一遍遍历记录的不是字符串中的字符出现的个数，而是记录每个字符最后出现的位置。第二遍遍历的时候，我们用一个整型变量来记录此时遍历的块中遍历过的元素最后的位置的最大值。只要这个最大值等于当前值就说明我们找到了答案。这样，我们就不需要额外维护一个哈希，但是记录最后出现的位置确实挺难一下想到的，也可能是我们习惯记录个数了，偶尔换换思路也是挺好的。

<br>

**参考代码**：
```java
public List<Integer> partitionLabels(String s) {
    List<Integer> results = new ArrayList<>();
    int[] lastIndex = new int[26];

    for (int i = 0; i < s.length(); ++i) {
       lastIndex[s.charAt(i) - 'a'] = i;
    }

    int maxLastIndex = 0, start = 0;
    for (int i = 0; i < s.length(); ++i) {
       maxLastIndex = Math.max(maxLastIndex, lastIndex[s.charAt(i) - 'a']);

       if (i == maxLastIndex) {
           results.add(i - start + 1);
           start = i + 1;
       }
    }

    return results;
}
```

<br>

## Review

[Signs that You are a Bad Programmer](https://javascript.plainenglish.io/signs-that-you-are-a-bad-programmer-6bfb8fb0a33)

文章的主题是，**只有当你对编程感兴趣，编程才会有意义**。作者同时给出了一些编程中的坏的习惯和一些不好的想法以及做法。这些点在很多的书中或者文章中都有提到。比如，“不写测试”、“不以解决问题为目的”、“害怕请教别人问题”、“不遵从编程中的一些标准或最佳实践”。。。作者列出的这些问题点可能在每个程序员身上都会存在。很多时候我们会犯这些问题并不是因为我们不知道，而是我们懒，总是想着怎么样才能让当下的自己更省力，缺少一个长远的规划。久而久之，潜在的问题变成了真正的问题，我们因此会觉得编程带给我们的麻烦比好处多，从而丧失了对编程的兴趣。

作者最后提到，学习编程需要的是持之以恒，因为需要学习和积累的经验很多，不能一蹴而就，至少需要上十年来持续学习。如果你在一开始的时候就不喜欢编程，何不花些时间寻找自己感兴趣的事情来做。


<br>

## Tip

最近在研究 socket 套接字编程。平时我们的 RESTful API 是基于 HTTP 协议的。但是有些时候，我们需要考虑的是传输效率，特别是对于一些小型的设备，比如门锁、水表、烟雾报警器等等，这些设备是用电池供电，需要考虑传输的功率损耗。因此，这些设备往往都是传输二进制流，而且基于更加底层的协议，比如 TCP 或者是 UDP。这里，参照网上的一些案例，我在本机上用 JS 实现了一个 socket 接口：

**服务端**：
```javascript
const net = require('net');

const server = net.createServer();

server.listen(3326, () => {
  console.log('TCP Server is listening');
});

server.on('connection', (socket) => {
  // 读取数据
  socket.on('data', (buffer) => {
    const msg = buffer.toString();
    console.log(msg);

    // write 方法写入数据，发回给客户端
    // socket.write(Buffer.from(`你好 Pyh`));
  });
});

server.on('close', () => {
  console.log('Server Close!');
});

server.on('error', (err) => {
  console.error('服务器异常：', err);
});
```

**设备端**：
```javascript
const net = require('net');

const client = net.createConnection({
  host: '127.0.0.1',
  port: 3326,
});

client.on('connect', () => {
  // 向服务器发送数据
  client.write('Nodejs 技术栈');
  client.end();
});

client.on('data', (buffer) => {
  console.log(buffer.toString());
});

client.on('error', (err) => {
  console.error('服务器异常：', err);
});

client.on('close', (err) => {
  console.log('客户端链接断开！', err);
});
```

这里主要是用到了 NodeJS 中的 `net` 模块来对事件进行监听，其中可以看到的是，不管是服务端，还是设备端接收到的数据都是二进制流（Buffer）。我们需要简单地对其进行转换，如果是二进制流有特殊含义的话，在服务器端，还需要自己实现函数来进行解析。

<br>

## Share

[深入理解ES6(二)](./深入理解ES6(二).md)