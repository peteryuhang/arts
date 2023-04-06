> 说明一下，自己对前端的了解比较欠缺，很早就听说过前端框架 React，但是最近才开始认真学习，看了一个 [React 教程](https://jscomplete.com/learn/complete-intro-react)，感觉学到了不少，这里就写写总结，加上自己的理解，可能会对刚刚接触前端或者 React 的人有所帮助，说的不好的地方也请指出。

<br>

## 什么是 React
我们经常提到 React，就会说这是一个前端框架，我们通常意义上面理解的后端框架，比如 Spring，Django 等等，这些框架是 “重量级的”，里面提供的服务应有尽有，让你能够集中去写应用层的代码，不用特别纠结于底层逻辑信息，但是这些框架非常的不灵活，你必须按照框架的要求去进行每一步设计和操作，即使这一步可能你并不能很好地解释为什么要这么做，另外，即使你仅仅用了框架的一小部分，整个框架你也得带上。但 React 和这些框架不同之处在于，**React 非常的轻量级**，其实你完全可以把 React 当作是 JS 当中的一个库，它可以帮助我们去更好地声明 UI，但是前端的完成并不能仅仅靠它，我们依旧需要引入其他的代码库，只是说 **React 改变了我们声明 UI 的方式，让 UI 的声明，以及 UI 之间的联系更加的清晰** ，它并没有限制我们其他的设计，我们也不需要为了它去额外学习特别多的东西。

**早期的前端，如果要改变页面，我们需要去直接操纵 DOM 节点**，这里简单提及以下 DOM，写过 HTML 都会知道，里面是分层的，比如 `<html>` 下面是 `<body>`，然后是 `<div>` 等等，你可以把整个 HTML 文件理解为一个 DOM 树，当然你会说，这并不难啊，但是要知道的是，如果这个 HTML 文件比较大，你可能对应起 DOM 节点会比较困难，另外层与层之间会有状态的改变和影响，比如说 `<div>` 里面发生了一个事件，我需要去改变 `<body>` 当中的属性，这些相互关联的逻辑都需要在程序当中写明，项目一大，逻辑复杂，往往就会乱。**这个时候我们不仅仅需要知道我们需要呈现什么样的界面，我们还要清楚知道该怎么样去做**。React 等前端框架的出现改变了这一点，相比之前我们需要知道怎么去描述一个个的事件，以及其背后的逻辑，这时我们只需要关注一个 UI 的**最终状态**，当用户操纵 UI 到了某个状态，React 会去相应地更新 DOM 节点，也就是说我们现在**只需要知道我们需要在什么时候呈现什么样的界面，怎么去做的事情直接让 React 去做就好**。

<br>

## React 中的两个重要的函数
一般来说我们通过修改 DOM 节点来改变 UI，传统的前端技术是使用 DOM API 函数来做到这一点，可以看下面的例子：
```js
document.getElementById('domNode').innerHTML = `
  <div>
    Using DOM API to change DOM
  </div>
`;
```
上面的例子写在 React 中，就会是：
```js
ReactDOM.render(
  React.createElement(
    'div',
    null,
    'Using React to change DOM',
  ),
  document.getElementById('domNode'),
);
```
你可以看到我们其实并没有使用任何的 HTML 语句，整个过程我们都在调用 React 中的函数，看上去貌似使用 React 的方式更为复杂些，但请先不要一看到 React 代码量稍长一点，就急着下这个结论，这个问题我们先放一下，我们来看一下这里出现的两个函数，这两个函数也是 React 成功运行的基石：
* **React.createElement()** 函数会创建一个 React 元素，你可以把其理解为 HTML 中的对应信息，这个函数的第一个参数是 tag，也就是 HTML 的标签名，第二个参数是标签的属性，null 就表示没有属性，第三个是需要放到 DOM 上去的 data，第三个元素之后还可以有其他的 React 元素，这些 React 元素就是当前 React 元素的 Children，还是举个例子更容易理解：
    ```js
    ReactDOM.render(
      React.createElement(
        "div",
        null,
        "Using React to change DOM ",
        React.createElement("input"),
        React.createElement("input")
      ),
      document.getElementById('domNode'),
    );
    
    document.getElementById('domNode').innerHTML = `
      <div>
        Using DOM API to change DOM
        <input />
        <input />
      </div>
    `;
    ```
    上面的两种表示方式是等效的

* **ReactDOM.render()** 这个函数是整个 React 应用的入口，它带有两个参数，第一个是 React 的元素（使用 React.createElement() 函数创建），也就是 React 需要放到 DOM 节点上的东西，第二个是 DOM 的位置。也就是说这个函数会根据第一个参数，去对应改变第二个参数中提到的 DOM 节点。这里说一下，这里并不是说只能更新一个 DOM 节点，第一个参数是可以层级嵌套的，第二个参数只是说明 DOM 树中 root 节点的位置，root 后面的 children 等会参照第一个参数的内容。

上面的两个函数是 React 运行的基础，但是在实际的使用中，你可能很少看到它们的出现，这里面是有原因的，先说说第二个函数，这个函数可以等同于 main 函数，可以把其看作是程序的入口，**一般来说一个应用程序仅仅会使用一次这个函数**。第一个函数是会经常被调用的，很少看到是因为这样子嵌套着去一个个调用函数确实看起来很不值观，代码也看起来非常的累赘，而且我们习惯了 HTML 的声明方式，所以，这里，**JSX** 出现了，要特别强调一点的是，**JSX并不是一个模版语言，它只 JavaScript 的一个扩展应用，或者说是一个语法糖，主要目的是允许程序把 JavaScript 中的函数调用写成声明 HTML 语句的形式**，这样一来，我们可以在 JavaScript 中写 HTML 了。在来个例子
```html
function Button (props) {
  return <button type="submit">{props.label}</button>;
}

ReactDOM.render(<Button label="Save" />, domNode);
```

你可以看到上面的 ReactDOM.render() 函数中直接将 tag 名写成了函数名，如果我们把上面的写法还原成前面讲的函数调用的形式，也是等效的，代码就会变成：
```js
function Button (props) {
  return React.createElement(
    "button",
    { type: "submit" },
    props.label
  );
}

ReactDOM.render(
  React.createElement(Button, { label: "Save"}),
  domNode
);
```
JSX 在这里起到的作用仅仅是把 HTML 的 syntax 转换成 React 中的相应函数调用，并无其他的功能。JSX 的出现让代码变得更加直观、简洁，但是其本身并不影响函数的功能。这里有一点需要强调的是，函数名的首字母记得一定要大写，因为原生的 HTML tag 名也会在这里起作用，为了不混淆，一般自己定义的 “tag” 的首字母要大写。

<br>

## React 组件
如果你熟悉 React，那么对组件这个概念一定不会陌生，在 React 中，一切都以组件的形式呈现，其实我们刚刚已经在示例代码中展示了组件，就是上面示例中的 Button 函数，这里我们传入了参数，这个参数是一个 object，你可以在这个 object 中定义任意多的变量传入组件，但是实际应用中**组件的输入参数部分最好使用 object 拆解的方式来定义**，比如：
```html
const Button = ({ label }) => (
  <button type="submit">{label}</button>
);
```
**这样做可以保证仅仅接收那些组件中需要的参数，也可以直观地展示组件用到了哪些输入参数，让代码更加地清晰、明了**。

除了以函数的形式来定义组件，我们还可以使用 class 的形式，例子：
```html
class Button extends React.Component {
  render() {
    return (
      <button>{this.props.label}</button>
    );
  }
}

ReactDOM.render(<Button label="Save" />, domNode);
```
如果使用 class 的方式来定义组件，其中必须要有一个 **render() 函数**，这个函数负责 return 组件的输出结果。props 这个参数也是在组件对象构建的时候就生成的。当然，用户也可以不传参数到组件，**一个组件可以没有输入，但是必须要有输出，不然的话，这个组件并没有任何意义**。有状态和无状态的组件在于，**无状态的组件相当于一个纯函数，输入是什么，输出就一定是什么**，这样的组件是最推荐使用的，因为其被重用的话不需要过多的考虑其副作用。但是，DOM 节点之间是相互关联的，有时组件之间也是相互关联的，在设计过程中，我们也需要让 React 知道那些组件发生了变化，所以组件当中还是需要有状态，**有状态的组件，最后的输出会受到组件中的状态的影响**。当用户在 UI 上操作过后，React 会去看组件中的状态是否发生变化来决定是否去更新 DOM 树节点。

在之前（2019以前），如果要建立**有状态的组件**，则必须使用 class 来定义组件，因为 class 当中会有内置的一些函数组件中没有的 React 生命周期函数，但是新版的 React 中增加了 Hooks 函数，这些函数只能用于函数定义的组件中，不能用于 class 定义的组件中，**Hooks 函数的函数名会以 use 开头，它使得函数中可以定义状态，以及对应的改变和操纵状态**，说了这么多，你可能会问，到底什么是状态，简单点说就是你组件中定义的除输入参数以外的变量，这些变量表示着组件的状态，但是如果你想让这些变量的改变被 React 所侦测到，那么你就需要 Hook 函数了，这里给出了一个简单的例子：
```html
const Button = ({ clickAction }) => {
  return (
    <button onClick={clickAction}>
      +1
    </button>
  );
};

const Display = ({ content }) => (
  <pre>{content}</pre>
);

const CountManager = () => {
  const [count, setCount] = useState(0);

  const incrementCounter = () => {
    setCount(count + 1);
  };

  return (
    <>
      <Button clickAction={incrementCounter} />
      <Display content={count} />
    </>
  );
};
```
上面的程序会设定一个 button 的 UI，每点击这个 button，就会使下面的数字加 1，为了**让每个组件仅仅负责单一的职责**，这里，我们建立了三个组件，Button 组件负责显示 button 的 UI，Display 组件负责显示输出的数字结果，CountManager 组件负责计算，并将 Button 和 Display 组件串接起来，可以理解为 Button 和 Display 组件是 CountManager 的 children。可以看到，逻辑部分都在 CountManager 组件中，在 CountManager 中，我们使用了 Hook 函数 **useState()**，这个函数负责在 React 中设定状态，它的输入是状态的初始值，输出一个数组，其中有两个值，第一个是这个状态，第二个是一个函数，我们可以使用这个函数去改变状态。同时，我们在 CountManager 中定义了 incrementCounter() 函数，这个函数很简单，就是去改变状态值，我们将这个函数传入 Button 中，**重点来了，Button 组件会通过 JavaScript 函数闭包来调用 CountManager 中的 incrementCounter() 函数**，你可以看到 CountManager 组件和 Button 组件之间的联系。除了 useState() 函数之外，还有一些其他的 Hook 函数，可以把其对应到 class 中的生命周期函数去看，这里就不一一列举了。

有一点需要指出的是，返回值我们使用`<></>`，而不是 `<div></div>` 来包裹多个组件，这两个是等效的，但是前者表示的是空节点，在 React 中，我们还可以使用 `<React.Fragment></React.Fragment>` 去包裹多个同级输出组件。

**建议尽量使用函数来作为组件，因为函数比较轻便，也比较清晰，直观，好理解**。

<br>

## React 渲染机制
之前提到，页面的更新和改变都必须更新 DOM 节点。在常规的 DOM 操作中，每当 DOM 中的节点改变，整个 DOM 树就会被刷新，但是 React 并不会这样做，**React 只会刷新那些有变化的节点及其子树**，这样可以节省不少效率。知道这个有什么好处呢？一个好处就是，我们在设计 React 程序的时候，尽量让 DOM 树变得横向扁平些，反之，纵向深了，节点之间的影响范围就会增大，从而状态的改变而导致的无关节点数目的更新就会增多。还有就是尽量不要使用全局变量当状态，尽量让一个状态的影响范围变窄。

实际当中 React 是怎样去渲染的？其实大致的流程也很简单，**每当我们改变组件中的状态时（之前的例子中的 setCount()），React 就会重新渲染组件，刷新组件中的所有变量（包括更新的状态），状态的更新会使得 React 建立一个虚拟的 DOM 树，React 会用这个虚拟的 DOM 树去和当前实际的 DOM 树进行比较，仅仅去改变需要改变的 DOM 子树。**

这里解释一下 React 中特别容易混淆的两个概念，**组件和 React 元素**，组件可以看作是一个模版，一个全局的定义，它可以是类，也可以是函数。React 元素是组件的输出，它其实是一个 object，一个实例，它用来表示组件中所描述的 DOM 节点。

<br>

## 总结
看完了以上的内容，不知道你对 React 是否有了初步的了解，其实它的内容并不是特别的多，你只需要理解组件的具体含义，以及 React 大致是怎么样通过状态去更新 DOM 树的即可。当然，我知道上面的内容只是最最基础的一些东西，想要知道的更全，更细的话，则需要去阅读 [React 的官方文档](https://zh-hans.reactjs.org/tutorial/tutorial.html)。另外，更好的方式就是直接找一个项目上手。
