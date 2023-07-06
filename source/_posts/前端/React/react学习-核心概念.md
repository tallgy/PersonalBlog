---
title: react学习-核心概念
date: 2022-02-14 19:30:54
tags:
 - React
categories:
 - React
---



# React 学习-核心概念 / v17.0.2

## jsx

### 属性值

​		你可以通过使用引号，来将属性值指定为字符串字面量：

```
const element = <a href="https://www.reactjs.org"> link </a>;
```

​		也可以使用大括号，来在属性值中插入一个 JavaScript 表达式：

```
const element = <img src={user.avatarUrl}></img>;
```

> **警告：**
>
> 因为 JSX 语法上更接近 JavaScript 而不是 HTML，所以 React DOM 使用 `camelCase`（小驼峰命名）来定义属性的名称，而不使用 HTML 属性名称的命名约定。
>
> 例如，JSX 里的 `class` 变成了 [`className`](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)，而 `tabindex` 则变为 [`tabIndex`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/tabIndex)。

### jsx表示对象

babel会把jsx转移为一个 React.createElement的函数调用

```
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);

const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

```
这个是创建的对象的简化结构
// 注意：这是简化过的结构
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```



## 元素渲染

### 更新已渲染的元素

​		React 元素是[不可变对象](https://en.wikipedia.org/wiki/Immutable_object)。一旦被创建，你就无法更改它的子元素或者属性。一个元素就像电影的单帧：它代表了某个特定时刻的 UI。

​		根据我们已有的知识，更新 UI 唯一的方式是创建一个全新的元素，并将其传入 [`ReactDOM.render()`](https://zh-hans.reactjs.org/docs/react-dom.html#render)。

```
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```

​		通过每秒调用 render来重新渲染

**注意：**

* 在实践中，大多数 React 应用只会调用一次 [`ReactDOM.render()`](https://zh-hans.reactjs.org/docs/react-dom.html#render)。在下一个章节，我们将学习如何将这些代码封装到[有状态组件](https://zh-hans.reactjs.org/docs/state-and-lifecycle.html)中。

* 我们建议你不要跳跃着阅读，因为每个话题都是紧密联系的。

### React 只更新它需要更新的部分

​		React DOM 会将元素和它的子元素与它们之前的状态进行比较，并只会进行必要的更新来使 DOM 达到预期的状态。

​		就是说，我们虽然多次运行了 render，但是实际上只会更改必要的更新。



## 组件 && props

### 函数组件与 class 组件

这两个组件是等效的。

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

### 渲染组件

```
const element = <div />;

const element = <Welcome name="Sara" />;
```

​		当 React 元素为用户自定义组件时，它会将 JSX 所接收的属性（attributes）以及子组件（children）转换为单个对象传递给组件，这个对象被称之为 “props”。

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

这里传递的属性attributes 将会作为props对象里面的属性进行传递。class，className 都会使用props传递。
const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

**注意：** 组件名称必须以大写字母开头。

* React 会将以小写字母开头的组件视为原生 DOM 标签。例如，`<div />` 代表 HTML 的 div 标签，而 `<Welcome />` 则代表一个组件，并且需在作用域内使用 `Welcome`。

* 你可以在[深入 JSX](https://zh-hans.reactjs.org/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized) 中了解更多关于此规范的原因。



### 组合组件

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

### Props的只读性

​		**所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。**



## State & 生命周期

### 将函数组件转换为class组件

* 创建同名class，继承于 React.Component
* 添加一个空的 render() 方法
* 将函数体移动到 render() 中
* 在render() 中，使用 this.props 替换 props
* 删除剩余的空函数声明。

​		每次组件更新时 `render` 方法都会被调用，但只要在相同的 DOM 节点中渲染 `<Clock />` ，就仅有一个 `Clock` 组件的 class 实例被创建使用。这就使得我们可以使用如 state 或生命周期方法等很多其他特性。



### 向 class 组件中添加局部的 state

* 把 `render()` 方法中的 `this.props.date` 替换成 `this.state.date` ：

* 添加一个 [class 构造函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes#Constructor)，然后在该函数中为 `this.state` 赋初值：

* 通过以下方式将 `props` 传递到父类的构造函数中：
  * Class 组件应该始终使用 `props` 参数来调用父类的构造函数。
* 移除 `<Clock />` 元素中的 `date` 属性：

```
class Clock extends React.Component
    <any,
        {
            date ?: any
        }> {
    constructor(props: any) {
        super(props);
        this.state = {
            date: new Date()
        }
    }

    render() {
        return (
            <div>
                <h1>Hello</h1>
                <h2>it is { this.state.date.toLocaleTimeString() }</h2>
            </div>
        )
    }
}
```



### 生命周期方法添加到 Class 中

* 当 `Clock` 组件第一次被渲染到 DOM 中的时候，就为其[设置一个计时器](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval)。这在 React 中被称为“挂载（mount）”。
* 同时，当 DOM 中 `Clock` 组件被删除的时候，应该[清除计时器](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval)。这在 React 中被称为“卸载（unmount）”。

#### 生命周期方法

*  `componentDidMount()` 方法会在组件已经被渲染到 DOM 中后运行
*  `componentWillUnmount()` 生命周期方法中清除



##### 创建 clock 组件

```
class Clock extends React.Component
    <
        any,
        {
            date : any
        }> {
    private timerId: any;
    constructor(props: any) {
        super(props);
        this.state = {
            date: new Date()
        }
    }

    componentDidMount() {
        this.timerId = setInterval(
            () => this.tick(),
            1000
        )
    }

    componentWillUnmount() {
        clearInterval(this.timerId)
    }

    tick() {
        this.setState({
            date: new Date()
        })
    }

    render() {
        return (
            <div>
                <h1>Hello</h1>
                <h2>it is { this.state.date.toLocaleTimeString() }</h2>
            </div>
        )
    }
}
```

##### 调用顺序

* 当 `<Clock />` 被传给 `ReactDOM.render()`的时候，React 会调用 `Clock` 组件的构造函数。
* 之后 React 会调用组件的 `render()` 方法。
* 当 `Clock` 的输出被插入到 DOM 中后，React 就会调用 `ComponentDidMount()` 生命周期方法。
* 浏览器每秒都会调用一次 `tick()` 方法。 
* 一旦 `Clock` 组件从 DOM 中被移除，React 就会调用 `componentWillUnmount()` 生命周期方法。



### 正确的修改 State

#### 不要直接修改state里面的属性

```
// wrong, 这样写不会重新渲染组件。
this.state.comm = 'he'

// correct
this.setState({comm: 'he'})
```

​		构造函数是唯一可以给 `this.state` 赋值的地方。

#### state更新可能是异步的

​		出于性能考虑，React 可能会把多个 `setState()` 调用合并成一个调用。

​		因为 `this.props` 和 `this.state` 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。

```
// Wrong
例如，此代码可能会无法更新计数器：
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

​		要解决这个问题，可以让 `setState()` 接收一个函数而不是一个对象。这个函数用上一个 state 作为第一个参数，将此次更新被应用时的 props 做为第二个参数：

```
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

#### State 的更新会被合并

​		当你调用 `setState()` 的时候，React 会把你提供的对象合并到当前的 state。

​		这里的合并是浅合并，所以 `this.setState({comments})` 完整保留了 `this.state.posts`， 但是完全替换了 `this.state.comments`。



### 数据是向下流动的

​		组件可以选择把它的 state 作为 props 向下传递到它的子组件中：

```
<FormattedDate date={this.state.date} />
```

​		`FormattedDate` 组件会在其 props 中接收参数 `date`，但是组件本身无法知道它是来自于 `Clock` 的 state，或是 `Clock` 的 props，还是手动输入的：

```
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```



## 事件处理

- React 事件的命名采用小驼峰式（camelCase），而不是纯小写。
- 使用 JSX 语法时你需要传入一个函数作为事件处理函数，而不是一个字符串。

​		例如，传统的 HTML：

```
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

​		在 React 中略微不同：

```
<button onClick={activateLasers}>  Activate Lasers
</button>
```

​		在 React 中另一个不同点是你不能通过返回 `false` 的方式阻止默认行为。你必须显式的使用 `preventDefault`。

​		在这里，`e` 是一个合成事件。React 根据 [W3C 规范](https://www.w3.org/TR/DOM-Level-3-Events/)来定义这些合成事件，所以你不需要担心跨浏览器的兼容性问题。React 事件与原生事件不完全相同。

​		但是，这样进行调用时，将无法使用 this，所以需要使用 bind 来进行绑定context上下文。

```
正常的使用时。这里的this，因为是在onClick里面进行的调用，this将没有正确的指向。所以需要使用bind进行绑定。
click(e ?: any) {
    console.log(this.props);
    e.preventDefault()
    console.log('you are click')
}

render() {
    return (
        <div>
            <h1 onClick={this.click}>Hello</h1>
            <h2>it is { this.state.date }</h2>
        </div>
    )
}
```

```
第一种，在constructor里面进行绑定this并返回
constructor(props: any) {
    super(props);
    this.state = {
        date: 1
    }

    this.click = this.click.bind(this)
}
```

```
第二种，在onClick里面使用bind进行绑定上下文。
render() {
    return (
        <div>
            <h1 onClick={this.click.bind(this)}>Hello</h1>
            <h2>it is { this.state.date }</h2>
        </div>
    )
}
```



如果不使用bind，可以使用两种方式。

​		如果你正在使用实验性的 [public class fields 语法](https://babeljs.io/docs/plugins/transform-class-properties/)，你可以使用 class fields 正确的绑定回调函数：

```
  // 此语法确保 `handleClick` 内的 `this` 已被绑定。
  // 注意: 这是 *实验性* 语法。
  // this。箭头函数来将this进行了定向指定。
  handleClick = () => {
    console.log('this is:', this);
  }
```

​		如果你没有使用 class fields 语法，你可以在回调中使用[箭头函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)：

```
  render() {
    // 此语法确保 `handleClick` 内的 `this` 已被绑定。
    return (
      <button onClick={() => this.handleClick()}>
        Click me
      </button>
    );
  }
```

​		这个方式和上面是基本上是一样的。区别就是在于上面的语法是一个实验性语法。



### 向事件处理程序传递参数

​		第一种，事件参数需要进行显式的传递

​		第二种，事件参数是使用的bind进行的绑定，所以会隐式的放在最后一个。

```
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```



## 条件渲染

​		React 中的条件渲染和 JavaScript 中的一样，使用 JavaScript 运算符 [`if`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) 或者[条件运算符](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)去创建元素来表现当前的状态，然后让 React 根据它们来更新 UI。

对于条件渲染我们可以使用几种方式：

* 元素变量，使用if

  * ```
    if () {
    	<LogoutButton onClick={this.handleLogoutClick} />
    } else {
    	<LoginButton onClick={this.handleLoginClick} />
    }
    ```

* 与运算符 &&， 注意，同时我们也可以知道，{} 里面如果是布尔类型，那么将不会生成dom，只有在属性里面attributes里面使用Boolean类型才会没有问题。

  * ```
    { message && <h2>
              You have {unreadMessages.length} unread messages.
            </h2>}
    ```

* 三目运算符

  * ```
    The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    
    {isLoggedIn
            ? <LogoutButton onClick={this.handleLogoutClick} />
            : <LoginButton onClick={this.handleLoginClick} />
          }
    ```

    

```
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}

ReactDOM.render(
  // Try changing to isLoggedIn={true}:
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
);
```

```
class Greeting extends React.Component<any, any> {
    constructor(props ?: any) {
        super(props);
    }

    render() {
        return (
            this.props.loginState ?
                <h1>login</h1> :
                <h1>logout</h1>
        );
    }
}

class Clock extends React.Component<any, {
    loginState ?: boolean
}> {
    // private loginState: any;

    constructor(props ?: any) {
        super(props);
        this.state = {
            loginState: false,
        }
    }

    componentDidUpdate(prevProps: Readonly<{}>, prevState: Readonly<{}>, snapshot?: any) {
        console.log(prevState)
    }

    handleLoginClick() {
        this.setState({
            loginState: true
        })
    }

    handleLogoutClick() {
        this.setState({
            loginState: false
        })
    }

    render() {
        let loginState = this.state.loginState
        let button = loginState ?
            <button onClick={this.handleLogoutClick.bind(this)}>logout</button> :
            <button onClick={this.handleLoginClick.bind(this)}>login</button>

        return (
            <div>
                <Greeting loginState={loginState}/>
                {button}
            </div>
        );
    }
}

ReactDOM.render(
    <Clock></Clock>,
    document.getElementById('root')
)
```

### 阻止组件渲染

​		可以通过render方法直接返回 null，来不进行渲染。

​		在组件的 `render` 方法中返回 `null` 并不会影响组件的生命周期。例如，上面这个示例中，`componentDidUpdate` 依然会被调用。



## 列表 & key

### 渲染多个组件

​		下面，我们使用 Javascript 中的 [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 方法来遍历 `numbers` 数组。将数组中的每个元素变成 `<li>` 标签，最后我们将得到的数组赋值给 `listItems`：

```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```



### 基础列表组件

​		当我们运行这段代码，将会看到一个警告 `a key should be provided for list items`，意思是当你创建一个元素时，必须包括一个特殊的 `key` 属性。我们将在下一节讨论这是为什么。

​		key值用于diff算法的优化，Vue上面也有，所以不多赘述。

​		我们这里只看一下 key 值是怎么进行使用的。

```
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```



### key

​		key 帮助 React 识别哪些元素改变了，比如被添加或删除。因此你应当给数组中的每一个元素赋予一个确定的标识。

​		写法：

```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

​		一个元素的 key 最好是这个元素在列表中拥有的一个独一无二的字符串。

​		当元素没有确定 id 的时候，万不得已你可以使用元素索引 index 作为 key。

​		如果列表项目的顺序可能会变化，我们不建议使用索引来用作 key 值，因为这样做会导致性能变差，还可能引起组件状态的问题。可以看看 Robin Pokorny 的[深度解析使用索引作为 key 的负面影响](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318)这一篇文章。如果你选择不指定显式的 key 值，那么 React 将默认使用索引用作为列表项目的 key 值。

​		要是你有兴趣了解更多的话，这里有一篇文章[深入解析为什么 key 是必须的](https://zh-hans.reactjs.org/docs/reconciliation.html#recursing-on-children)可以参考。



### 用key提取组件

​		元素的 key 只有放在就近的数组上下文中才有意义。

​		比方说，如果你[提取](https://zh-hans.reactjs.org/docs/components-and-props.html#extracting-components)出一个 `ListItem` 组件，你应该把 key 保留在数组中的这个 `<ListItem />` 元素上，而不是放在 `ListItem` 组件中的 `<li>` 元素上。

​		简单来说，虽然listItem 里面就只有一个li，但是因为就近的数组上下文是使用的 listItem，所以我们就需要给listItem 来使用key，而不是把key写在li标签里面。

```
function ListItem(props) {
  const value = props.value;
  return (
    // 错误！你不需要在这里指定 key：
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 错误！元素的 key 应该在这里指定：
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

```
function ListItem(props) {
  // 正确！这里不需要指定 key：
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 正确！key 应该在数组的上下文中被指定
    <ListItem key={number.toString()} value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

​		一个好的经验法则是：在 `map()` 方法中的元素需要设置 key 属性。



### key值在兄弟节点之间必须唯一

​		数组元素中使用的 key 在其兄弟节点之间应该是独一无二的。然而，它们不需要是全局唯一的。当我们生成两个不同的数组时，我们可以使用相同的 key 值。

​		key 会传递信息给 React ，但不会传递给你的组件。如果你的组件中需要使用 `key` 属性的值，请用其他属性名显式传递这个值：

```
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```



### 在jsx中嵌套map()

​		JSX 允许在大括号中[嵌入任何表达式](https://zh-hans.reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx)，所以我们可以内联 `map()` 返回的结果：

```
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
      )}
    </ul>
  );
}
```



## 表单

### 受控组件

​		在 HTML 中，表单元素（如`<input>`、 `<textarea>` 和 `<select>`）通常自己维护 state，并根据用户输入进行更新。而在 React 中，可变状态（mutable state）通常保存在组件的 state 属性中，并且只能通过使用 [`setState()`](https://zh-hans.reactjs.org/docs/react-component.html#setstate)来更新。

​		我们可以把两者结合起来，使 React 的 state 成为“唯一数据源”。渲染表单的 React 组件还控制着用户输入过程中表单发生的操作。被 React 以这种方式控制取值的表单输入元素就叫做“受控组件”。

```
<input type="text" value={this.state.value} onChange={this.handleChange} />

handleChange(event) {
    this.setState({value: event.target.value});
}
```

​		由于在表单元素上设置了 `value` 属性，因此显示的值将始终为 `this.state.value`，这使得 React 的 state 成为唯一数据源。由于 `handlechange` 在每次按键时都会执行并更新 React 的 state，因此显示的值将随着用户输入而更新。

​		这里就是将value值给锁定了，然后使用了change方法来进行修改value，修改了state的value就会修改input的value，然后就会页面的修改。

```
<input type="text" value={this.state.value} onChange={this.handleChange} />

handleChange(event) {
  this.setState({value: event.target.value});
}
```

​		同时，我们也发现了，react的 onChange 事件，其实就是input事件。

​		同时 text area 文本输入框，也是一样的使用 value 来进行值的操作。



#### select 标签

​		React 并不会使用 `selected` 属性，而是在根 `select` 标签上使用 `value` 属性。这在受控组件中更便捷，因为您只需要在根标签中更新它。

```
<select value={this.state.value} onChange={this.handleChange}>
    <option value="grapefruit">葡萄柚</option>
    <option value="lime">酸橙</option>
    <option value="coconut">椰子</option>
    <option value="mango">芒果</option>
</select>
```

​		通过value属性的变化，来确定selected的显示。

> 注意
>
> 你可以将数组传递到 `value` 属性中，以支持在 `select` 标签中选择多个选项：
>
> ```
> <select multiple={true} value={['B', 'C']}>
> ```



#### 文件 input 标签

​		在 HTML 中，`<input type="file">` 允许用户从存储设备中选择一个或多个文件，将其上传到服务器，或通过使用 JavaScript 的 [File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications) 进行控制。

```
<input type="file" />
```

​		因为它的 value 只读，所以它是 React 中的一个**非受控**组件。将与其他非受控组件[在后续文档中](https://zh-hans.reactjs.org/docs/uncontrolled-components.html#the-file-input-tag)一起讨论。



### 处理多个输入

​		当需要处理多个 `input` 元素时，我们可以给每个元素添加 `name` 属性，并让处理函数根据 `event.target.name` 的值选择要执行的操作。

​		我开始的方法是，通过传递一个参数，来代表是哪个input输入，然后再进行一个处理判断。

​		这里记住，这个value是不能接收Boolean类型的。但是checked可以。

```
通过传递参数来判断是哪个输入。然后再在handleInputChange事件里面进行判断处理

<input type="number" id={'input'} onChange={(e) => this.handleInputChange(e, 'input')} value={this.state.numberOfGuests}/>
```

​		我们通过设置input的name，然后通过type来获取不同的值，并且通过name来进行赋值给state。

```
handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
        [name]: value
    });
}
```

​		这里使用了 ES6 [计算属性名称](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names)的语法更新给定输入名称对应的 state 值。

```
interface IAddPlayerFormState {
    isGoing: boolean,
    numberOfGuests: number
}

handleInputChange(event ?: any) {
    const target = event.target
    const value = target.type==='checkbox' ? target.checked : target.value
    const name = target.name

    this.setState({
        [name]: value
    } as Pick<IAddPlayerFormState, keyof IAddPlayerFormState>)
}
```



### 受控输入空值

​		在[受控组件](https://zh-hans.reactjs.org/docs/forms.html#controlled-components)上指定 `value` 的 prop 会阻止用户更改输入。如果你指定了 `value`，但输入仍可编辑，则可能是你意外地将 `value` 设置为 `undefined` 或 `null`。

​		下面的代码演示了这一点。（输入最初被锁定，但在短时间延迟后变为可编辑。）

```
ReactDOM.render(<input value="hi" />, mountNode);

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode);
}, 1000);
```



### 受控组件的替代品

​		有时使用受控组件会很麻烦，因为你需要为数据变化的每种方式都编写事件处理函数，并通过一个 React 组件传递所有的输入 state。当你将之前的代码库转换为 React 或将 React 应用程序与非 React 库集成时，这可能会令人厌烦。在这些情况下，你可能希望使用[非受控组件](https://zh-hans.reactjs.org/docs/uncontrolled-components.html), 这是实现输入表单的另一种方式。



## 状态提升

​		状态提升，其实简单来说就是子组件可以修改数据传递给父组件。

​		方法其实也是和vue的差不多。vue子组件可以通过 $emit 通知父组件进行方法的调用。而react是通过直接调用父组件传递过来的方法进行调用。this.props.onTemperatureChange(event.target.value) 这个方法就是父组件传递过来的方法。父组件的attributes 属性。onTemperatureChange={this.handleCelsiusChange}。

```
const scaleNames : ScaleState = {
    c: 'Celsius',
    f: 'Fahrenheit'
}

interface ScaleState {
    [key : string]: string
}
interface valueState {
    scale ?: string,
    temperature : number,
}

class TemperatureInput extends React.Component<any, valueState> {
    constructor(props?:any) {
        super(props);

        this.state = {
            temperature: 0
        }

        this.handleChange = this.handleChange.bind(this)
    }

    handleChange(event : any) {
        // this.setState({
        //     temperature: event.target.value
        // })
        this.props.onTemperatureChange(event.target.value)
    }

    render() {
        const temperature = this.props.temperature
        const scale = this.props.scale
        return (
            <fieldset>
                <legend>Enter temperature in {scaleNames[scale]}: </legend>
                <input
                    type="text"
                    value={temperature}
                    onChange={this.handleChange} />
            </fieldset>
        );
    }
}

interface valueStateCalculator {
    temperature: string,
    scale: string
}
class Calculator extends React.Component<any, valueStateCalculator> {
    constructor(props ?: any) {
        super(props);

        this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this)
        this.handleCelsiusChange = this.handleCelsiusChange.bind(this)

        this.state = {temperature: '', scale: 'c'};
    }

    handleCelsiusChange(temperature : string) {
        this.setState({
            scale: 'c',
            temperature
        })
    }
    handleFahrenheitChange(temperature : string) {
        this.setState({
            scale: 'f',
            temperature
        });
    }

    toCelsius(fahrenheit : number) {
        return (fahrenheit - 32) * 5 / 9;
    }
    toFahrenheit(celsius : number) {
        return (celsius * 9 / 5) + 32;
    }
    tryConvert(temperature:string, convert:Function) {
        const input = parseFloat(temperature);
        if (Number.isNaN(input)) {
            return '';
        }
        const output = convert(input);
        const rounded = Math.round(output * 1000) / 1000;
        return rounded.toString();
    }

    render() {
        const scale = this.state.scale
        const temperature = this.state.temperature
        const celsius = scale==='f' ?
            this.tryConvert(temperature, this.toCelsius) :
            temperature;
        const fahrenheit  = scale==='c' ?
            this.tryConvert(temperature, this.toFahrenheit) :
            temperature;

        return (
            <div>
                <TemperatureInput
                    scale={'c'}
                    temperature={celsius}
                    onTemperatureChange={this.handleCelsiusChange}></TemperatureInput>
                <TemperatureInput
                    scale={'f'}
                    temperature={fahrenheit}
                    onTemperatureChange={this.handleFahrenheitChange}></TemperatureInput>
            </div>
        );
    }
}
```



## 组合 vs 继承

### 包含关系

​		有些组件无法提前知晓它们子组件的具体内容。在 `Sidebar`（侧边栏）和 `Dialog`（对话框）等展现通用容器（box）的组件中特别容易遇到这种情况。

​		我们建议这些组件使用一个特殊的 `children` prop 来将他们的子组件传递到渲染结果中：

```
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

```
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

​		这个就类似于vue组件的插槽式写法。



​		命名props传递

​		具名插槽，写法和vue不同

```
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```

​		定义时，通过直接调用 props.left 来进行调用，注意，这个插槽和props属性是共通的。所以其实对于react来说，并没有分插槽这个属性。reac内部会使用。自己的方法进行处理。不用我们进行操作。

​		同时 react的 ‘具名插槽‘ 的使用形式是直接在组件里面进行调用，但是传值需要在属性里面进行传值，所以这就是说为什么react没有插槽这一概念的原因。



### 特例关系

​		在 React 中，我们也可以通过组合来实现这一点。“特殊”组件可以通过 props 定制并渲染“一般”组件：

```
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```

​		简单来说就是显示是通过props的内容进行的显示，同时传递时通过传递这个对应的内容，让页面进行渲染。



​		Props 和组合为你提供了清晰而安全地定制组件外观和行为的灵活方式。注意：组件可以接受任意 props，包括基本数据类型，React 元素以及函数。

​		如果你想要在组件间复用非 UI 的功能，我们建议将其提取为一个单独的 JavaScript 模块，如函数、对象或者类。组件可以直接引入（import）而无需通过 extend 继承它们。



## react哲学

### 第一步：将设计好的 UI 划分为组件层级

​		划分组件，分层。抽离组件。

### 第二步：用 React 创建一个静态版本

### 第三步：确定 UI state 的最小（且完整）表示

通过问自己以下三个问题，你可以逐个检查相应数据是否属于 state：

1. 该数据是否是**由父组件通过 props 传递而来**的？如果是，那它应该不是 state。
2. 该数据**是否随时间的推移而保持不变**？如果是，那它应该也不是 state。
3. 你**能否根据其他 state 或 props 计算出该数据的值**？如果是，那它也不是 state。

### 第四步：确定 state 放置的位置

对于应用中的每一个 state：

- 找到根据这个 state 进行渲染的所有组件。
- 找到他们的共同所有者（common owner）组件（在组件层级上高于所有需要该 state 的组件）。
- 该共同所有者组件或者比它层级更高的组件应该拥有该 state。
- 如果你找不到一个合适的位置来存放该 state，就可以直接创建一个新的组件来存放该 state，并将这一新组件置于高于共同所有者组件层级的位置

### 第五步：添加反向数据流





​		先继承 react.component 组件的内容，然后再在在自己的方法里面添加一个render 方法。

```
class ShoppingList extends React.Component {
  render() {
    return (
      <div className="shopping-list">
        <h1>Shopping List for {this.props.name}</h1>
        <ul>
          <li>Instagram</li>
          <li>WhatsApp</li>
          <li>Oculus</li>
        </ul>
      </div>
    );
  }
}

// 用法示例: <ShoppingList name="Mark" />
```

​		上述代码等同于:

```
return React.createElement('div', {className: 'shopping-list'},
  React.createElement('h1', /* ... h1 children ... */),
  React.createElement('ul', /* ... ul children ... */)
);
```

