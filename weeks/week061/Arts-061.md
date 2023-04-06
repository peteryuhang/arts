> ARTS是什么？<br>
**Algorithm**：每周至少做一个leetcode的算法题；<br>
**Review**：阅读并点评至少一篇英文技术文章；<br>
**Tip**：学习至少一个技术技巧；<br>
**Share**：分享一篇有观点和思考的技术文章。

## Algorithm

[LC 380. Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1/)

[LC 381. Insert Delete GetRandom O(1) - Duplicates allowed](https://leetcode.com/problems/insert-delete-getrandom-o1-duplicates-allowed/)

**题目解析**：

两道题目，第二道题目是第一道题目的递进。

先看第一道题，让你实现一个数据结构，这个数据结构是一个集合，它支持三种操作，分别是 `插入元素`，`删除元素` 以及 `随机选择一个元素`。这三种操作的时间复杂度均为 `O(1)`。 这道题目的难点在于第三个操作，如果不需要实现这个操作的话，那其实和普通的 set 差不多。随机选择一个元素是需要保证集合中的每个元素被选到的概率相同，这个操作是需要数组才能完成的，但是仅仅使用数组是不够的，虽然数组支持插入以及删除元素，但是数组删除元素的时间复杂度是 `O(n)`。于是，这道题目的思考重点就变成了，**如何在 O(1) 的时间内删除数组指定位置上的元素**，这里有个小技巧，我们知道，如果是删除数组的最后一个元素，可以保证删除的时间复杂度 `O(1)`，那么如果我们知道要删除的元素在数组上的位置，其实可以把要删除的元素和数组最后一个元素进行交换，然后删除，这样就可以保证时间复杂度符合要求。另外，要实现这个数据结构，我们需要通过元素反向找到该元素在数组上的位置，因此，我们还需要一个哈希表来记录这个信息，说到这里，这道题就算是解决了，这里的难点主要在于能不能想到如何在 `O(1)` 时间删除数组上的一个元素，另外主要就是实现上的细节。

有了第一题的基础，第二题不会太难。第二题也是让你实现一个和第一题类似的数据结构，但是这个结构是允许重复元素的，其他的没有太大区别。思路还是差不多，**但是通过元素反向找到位置的这个结构不能再是普通的哈希表**，因为一个元素可能存在多个位置，这里我用了一个嵌套的哈希表来表示，知道了这一点，基本上剩下的就是实现的细节了。

<br>

**参考代码（380）**：

```go
type RandomizedSet struct {
    ValueMapIndex map[int]int
    ValueArray []int
}

/** Initialize your data structure here. */
func Constructor() RandomizedSet {
    return RandomizedSet{ValueArray: []int{}, ValueMapIndex: map[int]int{}}
}

/** Inserts a value to the set. Returns true if the set did not already contain the specified element. */
func (this *RandomizedSet) Insert(val int) bool {
    if _, ok := this.ValueMapIndex[val]; ok {
        return false
    }

    index := len(this.ValueArray)
    this.ValueArray = append(this.ValueArray, val)
    this.ValueMapIndex[val] = index

    return true
}

/** Removes a value from the set. Returns true if the set contained the specified element. */
func (this *RandomizedSet) Remove(val int) bool {
    index, ok := this.ValueMapIndex[val];
    if !ok {
        return false
    }

    lastIndex := len(this.ValueArray) - 1

    // 要删除的元素和数组最后一个元素交换位置，同时更新哈希表
    this.ValueArray[index] = this.ValueArray[lastIndex]
    this.ValueMapIndex[this.ValueArray[lastIndex]] = index

    // 删除数组最后一个元素，同时更新哈希表
    this.ValueArray = this.ValueArray[:lastIndex]
    delete(this.ValueMapIndex, val)
    
    return true
}

/** Get a random element from the set. */
func (this *RandomizedSet) GetRandom() int {
    return this.ValueArray[rand.Intn(len(this.ValueMapIndex))]
}
```

<br>

**参考代码（381）**：

```go
type RandomizedCollection struct {
    ValueMapIndexes map[int]map[int]int
    ValueArray []int
}

/** Initialize your data structure here. */
func Constructor() RandomizedCollection {
    return RandomizedCollection{ ValueMapIndexes: map[int]map[int]int{}, ValueArray: []int{} }
}

/** Inserts a value to the collection. Returns true if the collection did not already contain the specified element. */
func (this *RandomizedCollection) Insert(val int) bool {
    valueIndexes, ok := this.ValueMapIndexes[val]

    if !ok {
        this.ValueMapIndexes[val] = map[int]int{}
        valueIndexes = this.ValueMapIndexes[val]
    }

    index := len(this.ValueArray)
    this.ValueArray = append(this.ValueArray, val)
    valueIndexes[index] = 1

    if ok {
        return false
    }

    return true
}

/** Removes a value from the collection. Returns true if the collection contained the specified element. */
func (this *RandomizedCollection) Remove(val int) bool {
    valueIndexes, ok := this.ValueMapIndexes[val];

    if !ok || len(valueIndexes) == 0 {
        return false
    }

    lastIndex := len(this.ValueArray) - 1

    // 如果数组的最后一个元素就是当前要删除的元素
    // 直接删除即可，同时更新哈希表
    if this.ValueArray[lastIndex] == val {
        this.ValueArray = this.ValueArray[:lastIndex]
        delete(valueIndexes, lastIndex)
    } 
    // 如果数组的最后一个元素不是当前要删除的元素
    else {
        index := -1

        // 在要删的元素对应的位置中随机选择一个位置删除
        for k := range valueIndexes {
            index = k
            break
        }

        // 要删除的位置上的元素和数组最后一个元素交换，然后删除，同时更新哈希表
        this.ValueArray[index] = this.ValueArray[lastIndex]
        delete(this.ValueMapIndexes[this.ValueArray[lastIndex]], lastIndex)
        this.ValueMapIndexes[this.ValueArray[lastIndex]][index] = 1

        // 删除交换后的元素
        this.ValueArray = this.ValueArray[:lastIndex]
        delete(valueIndexes, index)
    }

    return true
}

/** Get a random element from the collection. */
func (this *RandomizedCollection) GetRandom() int {
    return this.ValueArray[rand.Intn(len(this.ValueArray))]
}

```

<br>

## Review

[To My Programmer Self 20 Years Ago: Do These 4 Things More](https://levelup.gitconnected.com/to-my-programmer-self-20-years-ago-do-these-4-things-more-fb562cf7d309)

一个有着 20 年编程经验的程序员分享关于程序员如何成长的文章，主要是如下的四点：

* **更多的自动化**

  计算机其实并没有人类聪明，对于哪些人类没有办法解决或者是没有想到解决方案的事情，换到计算机也无法解决。但是计算机相比之下的优势在于，可以快速完成一些重复的事情，你只需要告诉它基本的规则即可。
  
  如果不想让自己做重复的劳动，那么就多多地去让机器自动化地完成一些事情。毕竟对我们来说，一件同样的事情做上十遍与做上一百遍对自己能力的提升并没多大区别，那何不把这些时间省下来花在更有意义的事情上呢？

  当然，把一件事情自动化需有一定的前期投入，比如说你写个脚本自动加载并展现每天的更新数据，写个自动化的工具自动帮你提交代码等等，这些都需要一定的时间和精力才能完成，但是放在长期来看还是值得的。**自动化省下来的不仅仅是时间**，它可以释放你的大脑，让你可以专注到其它值得专注的事情上去，而不受打扰和中断。而且，自动化过后，甚至说都不需要自己亲自来完成这件事，你可以让别人帮你做，因为你已经把一件相对复杂的事情简化成运行一个命令那么简单，这就好像按一个按钮就可以把要做的事情做完，可以说是零门槛，只要其它人有空都可以来做这个事情。这个时候，你完全可以不想这个事情，**要知道我们觉得累往往是因为想得太多，而不是做的太多**，解放大脑所带来的价值远不止节省时间。
 
* **更多的测试**

  这个就不用多说了，很多的书籍都在不断强调测试的重要性，并且写测试是写出可维护的代码的必要一步。回到文章中，作者给出了一个很有意思的观点，他说**更多的测试可以给你带来更多的自信**，理由也很好解释，往往代码中都藏着很多细节，特别是一个庞大的项目，但显而易见的是，我们是无法记住并注意到代码中所有的细节，这就意味着我们需要机器自动地帮我们记住并检测这些细节，那怎么做呢？当然就是多写测试，你会发现，写了测试后你不需要小心翼翼地去审视你写的每行代码，或者反复提醒自己要细心，因为你知道你有一个强大的保护伞，你不需要担心的太多，这样自信就来了。说到这里，你可能发现了，写测试其实也是实现自动化的一种，目的是让计算机反复检测代码中的细节和各个方面的逻辑。
  
  除了上面这些，写测试还可以帮助你认识到你写的东西，让你带着审视的眼光来设计你的代码，更多的你可以参考 [之前 ARTS 的 Share 部分](https://juejin.im/post/5d716604f265da03d7283d18)。

* **更多的求助他人**

  老话说得好，“当局者迷，旁观者清”。不管怎样，多几双眼睛来帮助你来看问题，发现问题总是好的。但是我们总有一个惯性思维就是，“在别人面前暴露太多的问题显得自己特别的弱小”，这样会让我们觉得犯错是不好的，甚至是可耻的，但其实不然，**进步往往都源于找到并正确地看待问题**。因此，首先需要消除 “犯错可耻” 这么样一个内心想法。然后每当自己工作或者思考中遇到瓶颈了，及时地向他人求助，要知道，没有一个完美的想法，一个东西从不同角度看都会有其优劣之处，不要想着一个东西在给别人看之前要多么地完善，我们的目的并不在于做出一个十全十美的的东西，而是要早点找出问题，总结出方法，让自己学到得更多，变得更优秀。
  
* **更多的教授**

  学习也是有效率、层级之分的。像阅读，看视频，或者听讲这种形式的学习都属于低层次的学习，或者是输入层面的学习，这样的学习形式很难将你看到的，听到的转化成为自己拥有的。特别是对于编程这种，实践性特别强的技能，看书看视频远远不能让你成为一个高手。你还需要多实践，多总结，相比之下，帮别人解决问题，将自己知道的东西输出出去可以让你懂得更多，因为，在你和别人交流的过程中，你可以从不同的角度去看待一个问题，甚至从别人的问题之中可以看到自己之前看不到的东西，这样可以让你对一个问题有一个更深层次的理解，对一个知识点从不同维度地加深印象，同时，**教授他人也是增加自己的影响力的一种方法**。
  
  在当前互联网时代，教授或者说是输出知识的方式有很多，比如是去到类似 Stack Overflow 这样的网站去回答别人提出的问题，亦或是写自己的博客，把自己掌握的东西通过文字的形式写出来，再或是给自己开发的东西写一个说明性的文档，这些都是教授，都是知识输出，是非常高效的一种学习模式。
  
你可以看到，上面这几条建议并不是说你做了马上就可以明显地看到自己能力的变化。上面说的都是一些认知，一些好的习惯，都是从长期来看的，毕竟我们一直都在路上，优秀的人的目标永远是让自己变得更优秀。

<br>

## Tip

[Stop Using Objects as Hash Maps in JavaScript](https://medium.com/better-programming/stop-using-objects-as-hash-maps-in-javascript-9a272e85f6a8)

这周无意间知道了一个关于 JavaScript 的小知识点，或者说是之前的一个错误。之前总习惯把 JavaScript 中的 Object 当作哈希表来用，这样做一来是方便，二来受某些动态语言潜移默化的影响，比如 Python。

这次算是涨知识了，JavaScript 中其实是有哈希表这样一个数据结构的，而且把 Object 当作哈希表来使用其实是有很多的弊端的。这次就依次来看看 JavaScript 中使用哈希表的优势有哪些：

* **更多的键值类型**

  在 Object 中，只允许 string 或者 symbol 作为 key。但是哈希则不同，可以用 object，function，primitives 来作为 key:
  
  ```javascript
    const map = new Map();
    const myFunction = () => console.log('I am a useful function.');
    const myNumber = 666;
    const myObject = {
        name: 'plainObjectValue',
        otherKey: 'otherValue'
    };
    map.set(myFunction, 'function as a key');
    map.set(myNumber, 'number as a key');
    map.set(myObject, 'object as a key');
    
    console.log(map.get(myFunction)); // function as a key
    console.log(map.get(myNumber)); // number as a key
    console.log(map.get(myObject)); // object as a key
  ```

* **判断大小更加方便**

  如果经常使用 Object 的话，你应该清楚，如果要统计 Object 里存放了多少元素，我们一般是要把 Object 所有的 Key 拿到才能获得大小，这是及其不方便的，而且程序的运行效率也会打折扣。
  
  ```javascript
  const map = new Map();
  map.set('someKey1', 1);
  map.set('someKey2', 1);
  ...
  map.set('someKey100', 1);
  console.log(map.size) // 100, Runtime: O(1)
  
  const plainObjMap = {};
  plainObjMap['someKey1'] = 1;
  plainObjMap['someKey2'] = 1;
  ...
  plainObjMap['someKey100'] = 1;
  console.log(Object.keys(plainObjMap).length) // 100, Runtime: O(n)
  ```

* **效率更高**

  相对于 Object，Map 对于频繁的插入或者是删除操作都较为快速。主要是因为 Map 可以支持不同类型的 Key 值，作者在自己的电脑上做实验，还有就是前面提到的长度判断，对于一千万个键值对，在判断长度上二者所花费的时间：
  
  Object：~ 1.6 s
  
  Map: < 1 ms

* **遍历更加方便和快捷**

  Object 在遍历之前需要取到所有的 key，然后再通过 key 获得到 value，而 Map 可直接遍历：
  
  ```javascript
  for (let [key, value] of map) {
    console.log(`${key} = ${value}`);
  }

  for (let key of Object.keys(plainObjMap)) {
    const value = plainObjMap[key];
    console.log(`${key} = ${value}`);
  }
  ```

* **Key 的顺序**

  对大多数语言来说，哈希表里面存放的元素都是无序的，但是自从 ECMAScript 2015 后，Map 中元素被遍历到的先后顺序与元素被插入的先后顺序一致。不懂 JavaScript 中 Map 的底层实现原理是怎样的，估计用了一些额外的空间来记录元素的存放位置？感觉这又是一道算法题。。。不管怎么样，Object 是做不到这一点


<br>

## Share

继续和皓叔学习 makefile 的知识，这次是 [第二篇文章](https://blog.csdn.net/haoel/article/details/2887)

通过之前的第一篇文章理解了 make 是如何工作的，主要就是层级地根据目标文件去找依赖文件，然后执行相应的命令，make 会根据文件的更新时间来决定是否执行命令。

第二篇文章主要讲了几个优化技巧，来帮助我们写出简洁易懂的 makefile

* **使用变量来表示依赖关系**

  有些时候一个文件依赖一堆文件，然后执行的命令中又重复带上这一堆文件，在一个庞大的项目之中，如果我们需要新增一个依赖，往往变得比较棘手。我们可以用变量来表示一些依赖项，这样可以使内容模块化，方便我们修改和维护。
  
  makefile 中定义变量也十分简单，跟 Python 中定义变量是一样的，不需要修饰符，直接 `varName = ...` 即可。在使用变量的地方可以用 `$(varName)` 将变量所表示的内容取出。

* **让 make 自动推导命令**

  make 有一些隐藏的规则，这就是其一。只要 make 看到一个 [.o] 文件，它就会自动的把 [.c] 文件加在依赖关系中，并且相关的命令你也只需写一次，不需要重复写，make 会自动推导出来，这可以让 makefile 更加地简洁。
  
* **清空目标文件的规则**

  makefile 中一般都有一个清空目标文件（.o 和执行文件）的规则，这方便文件的重编译，比如下面的表示：
  
  ```
  .PHONY : clean
  clean :
      -rm edit $(objects)
  ```
  
  这里的 clean 后面没有接任何的依赖，也没有被目标文件关联，因此在编译的时候不起作用，也就是在编译阶段，其后面的命令不会被执行，我们可以手动执行 `make clean` 以此来执行 clean 之后的命令。
  
  `.PHONY` 表示 clean 是一个 “伪目标”。而在 rm 前面加一个减号的意思是，即使某些文件出了问题，也要执行后面的命令。