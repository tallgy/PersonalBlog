---
title: react学习
date: 2022-02-14 19:30:54
tags:
 - React
categories:
 - React
---



# React 学习 / v17.0.2

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

