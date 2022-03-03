---
title: react-高级指引
date: 2022-02-17 11:00:05
tags:
 - React
categories:
 - React
---



#  React 高级指引

## 无障碍

### 语义化的 HTML

​		简单来说，就是如果对于比如 dl dt dd，其中这样的构造。

​		正常的使用 ，就是如同下面这样，因为return需要一个根组件，但是我们有两个标签，所以需要将其用div包含起来。

​		但是这样就会破坏语义性。所以就需要使用一个 Fragment 标签，其语义和template标签可以说是一样的。

```
function ListItem({ item }) {
  return (
    <div>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </div>
  );
}

function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        <ListItem item={item} key={item.id} />
      ))}
    </dl>
  );
}
```

```
import React, { Fragment } from 'react';

function ListItem({ item }) {
  return (
    <Fragment>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </Fragment>
  );
}

function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        <ListItem item={item} key={item.id} />
      ))}
    </dl>
  );
}
```



​		和其他的元素一样，你可以把一系列的对象映射到一个 fragment 的数组中。

```
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // Fragments should also have a `key` prop when mapping collections
        <Fragment key={item.id}>          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>      ))}
    </dl>
  );
}
```

​		当你不需要在 fragment 标签中添加任何 prop 且你的工具支持的时候，你可以使用 [短语法](https://zh-hans.reactjs.org/docs/fragments.html#short-syntax)：

```
function ListItem({ item }) {
  return (
    <>      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </>  );
}
```



### 控制焦点

​		我们可以使用 tab 键来进行焦点的切换。其中可以切换的是 a标签，input标签等可以获取focus的标签。



### 使用程序管理焦点

​		简单来说，就是我们可以创建一个DOM元素ref，然后将其赋值给了ref。于是就可以通过 this.textInput.current 来获取当前DOM节点。

```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // 创造一个 textInput DOM 元素的 ref
    this.textInput = React.createRef();
  }
  render() {
  // 使用 `ref` 回调函数以在实例的一个变量中存储文本输入 DOM 元素
  //（比如，this.textInput）。
    return (
      <input
        type="text"
        ref={this.textInput}
      />
    );
  }
}
```

​		然后我们就可以在需要时于其他地方把焦点设置在这个组件上：

```
focus() {
  // 使用原始的 DOM API 显式地聚焦在 text input 上
  // 注意：我们通过访问 “current” 来获得 DOM 节点
  this.textInput.current.focus();
}
```

​		其大意和vue的ref一样，但是这里我们需要先通过React.createRef 来创建一个ref。然后再将其用ref赋值给标签则可以获得对应的DOM节点。

​		我们需要使用 current 来获取当前的节点。同时我们如果对一个值多次使用ref赋值，只有第一次的会有作用

```
我这里，div ref={this.textInput}
			CustomInput ref={this.textInput}
其最后的结果就是 div，current就只有这一个对象，并没有生成对应的多个对象出来。

class App extends React.Component<any, any> {
    private textInput : any

    constructor(props : any) {
        super(props);

        this.textInput = React.createRef();
    }

    render() {

    return (
        <div ref={this.textInput}>
            <CustomInput ref={this.textInput} inputRef={this.textInput}></CustomInput>
        </div>
    );
  }
}
```

​		通过暴露ref，来让父组件的ref传向子节点。

```
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.inputElement = React.createRef();
  }
  render() {
    return (
      <CustomTextInput inputRef={this.inputElement} />
    );
  }
}

// 现在你就可以在需要时设置焦点了
this.inputElement.current.focus();
```



<img src="react-高级指引/image-20220217140334480.png" alt="image-20220217140334480" style="zoom:67%;" />

​		通过点击事件来让 ul 进行显示和隐藏，一般的想法就是使用click事件，设置一个Boolean值来进行显示和隐藏。

​		但是这样对于使用键盘的就不会过于的友好。因为这样点击事件启动之后，就需要其他的点击事件来进行操作会隐藏当前 ul 标签。

```
button的点击事件
onClickHandler() {
    this.setState(currentState => ({
      isOpen: !currentState.isOpen
    }));
  }

window对象监听点击事件。通过判断点击事件是否属于toggleContariner元素来确定是否需要隐藏。
  onClickOutsideHandler(event) {
    if (this.state.isOpen && !this.toggleContainer.current.contains(event.target)) {
      this.setState({ isOpen: false });
    }
```

​		所以我们可以使用 onBlur 和 onFocus 来完成这个效果。

​		我的思考是，直接添加 onBlur 和 onFocus 来进行判断，虽然这样对于button的点击事件就会一直处于focus状态。

​		官网的思路就是，click事件还是添加在button上面，然后使用focus和blur来进行事件的处理。但是这里存在一个问题，那就是对于focus和blur的事件需要有能够进行事件的才会使用，但是官网案例的显示是ul标签，不存在焦点事件，所以出现了问题。

​		通过官网并不是获取焦点就直接为true，而是使用了 settimeout方法。在get focus的时候清除定时器。

```
class BlurExample extends React.Component {
  constructor(props) {
    super(props);

    this.state = { isOpen: false };
    this.timeOutId = null;

    this.onClickHandler = this.onClickHandler.bind(this);
    this.onBlurHandler = this.onBlurHandler.bind(this);
    this.onFocusHandler = this.onFocusHandler.bind(this);
  }

  onClickHandler() {
    this.setState(currentState => ({
      isOpen: !currentState.isOpen
    }));
  }

  // 我们在下一个时间点使用 setTimeout 关闭弹窗。
  // 这是必要的，因为失去焦点事件会在新的焦点事件前被触发，
  // 我们需要通过这个步骤确认这个元素的一个子节点
  // 是否得到了焦点。
  onBlurHandler() {
    this.timeOutId = setTimeout(() => {
      this.setState({
        isOpen: false
      });
    });
  }

  // 如果一个子节点获得了焦点，不要关闭弹窗。
  onFocusHandler() {
    clearTimeout(this.timeOutId);
  }

  render() {
    // React 通过把失去焦点和获得焦点事件传输给父节点
    // 来帮助我们。
    return (
      <div onBlur={this.onBlurHandler}
           onFocus={this.onFocusHandler}>
        <button onClick={this.onClickHandler}
                aria-haspopup="true"
                aria-expanded={this.state.isOpen}>
          Select an option
        </button>
        {this.state.isOpen && (
          <ul>
            <li>Option 1</li>
            <li>Option 2</li>
            <li>Option 3</li>
          </ul>
        )}
      </div>
    );
  }
}
```



### 开发辅助

​		我们可以直接在 JSX 代码中检测一些无障碍复制功能。通常支持 JSX 的 IDE 会针对 ARIA roles，states 和 properties 提供智能检测。我们也可以使用以下工具：

#### eslint-plugin-jsx-a11y

​		ESLint 中的 [eslint-plugin-jsx-a11y](https://github.com/evcohen/eslint-plugin-jsx-a11y) 插件为你的 JSX 中的无障碍问题提供了 AST 的语法检测反馈。许多 IDE 都允许你把这些发现直接集成到代码分析和源文件窗口中。

​		 [Create React App](https://github.com/facebookincubator/create-react-app)中使用了这个插件中的一部分规则。如果你想启用更多的无障碍规则，你可以在项目的根目录中创建一个有如下内容的 `.eslintrc` 文件：

```
{
  "extends": ["react-app", "plugin:jsx-a11y/recommended"],
  "plugins": ["jsx-a11y"]
}
```

### 

## 代码分割

​		为了避免搞出大体积的代码包，在前期就思考该问题并对代码包进行分割是个不错的选择。 代码分割是由诸如 [Webpack](https://webpack.docschina.org/guides/code-splitting/)，[Rollup](https://rollupjs.org/guide/en/#code-splitting) 和 Browserify（[factor-bundle](https://github.com/browserify/factor-bundle)）这类打包器支持的一项技术，能够创建多个包并在运行时动态加载。

​		对你的应用进行代码分割能够帮助你“懒加载”当前用户所需要的内容，能够显著地提高你的应用性能。尽管并没有减少应用整体的代码体积，但你可以避免加载用户永远不需要的代码，并在初始加载的时候减少所需加载的代码量。

### import()

​		在你的应用中引入代码分割的最佳方式是通过动态 `import()` 语法。

​		**使用之前：**

```
import { add } from './math';

console.log(add(16, 26));
```

​		**使用之后：**

```
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

​		当 Webpack 解析到该语法时，会自动进行代码分割。如果你使用 Create React App，该功能已开箱即用，你可以[立刻使用](https://create-react-app.dev/docs/code-splitting/)该特性。[Next.js](https://nextjs.org/docs/advanced-features/dynamic-import) 也已支持该特性而无需进行配置。

​		如果你自己配置 Webpack，你可能要阅读下 Webpack 关于[代码分割](https://webpack.docschina.org/guides/code-splitting/)的指南。你的 Webpack 配置应该[类似于此](https://gist.github.com/gaearon/ca6e803f5c604d37468b0091d9959269)。

​		当使用 [Babel](https://babeljs.io/) 时，你要确保 Babel 能够解析动态 import 语法而不是将其进行转换。对于这一要求你需要 [@babel/plugin-syntax-dynamic-import](https://classic.yarnpkg.com/en/package/@babel/plugin-syntax-dynamic-import) 插件。



### React.lazy

> 注意:
>
> `React.lazy` 和 Suspense 技术还不支持服务端渲染。如果你想要在使用服务端渲染的应用中使用，我们推荐 [Loadable Components](https://github.com/gregberge/loadable-components) 这个库。它有一个很棒的[服务端渲染打包指南](https://loadable-components.com/docs/server-side-rendering/)。

**使用之前：**

```
import OtherComponent from './OtherComponent';
```

**使用之后：**

```
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```

​		此代码将会在组件首次渲染时，自动导入包含 `OtherComponent` 组件的包。

​		`React.lazy` 接受一个函数，这个函数需要动态调用 `import()`。它必须返回一个 `Promise`，该 Promise 需要 resolve 一个 `default` export 的 React 组件。

​		然后应在 `Suspense` 组件中渲染 lazy 组件，如此使得我们可以使用在等待加载 lazy 组件时做优雅降级（如 loading 指示器等）。

```
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

​		主要有几个点：

* React.lazy() 进行promise加载
* suspense 使用 里面的 fallback 来进行 加载过程中的优雅降级。

​		`fallback` 属性接受任何在组件加载过程中你想展示的 React 元素。你可以将 `Suspense` 组件置于懒加载组件之上的任何位置。你甚至可以用一个 `Suspense` 组件包裹多个懒加载组件。

```
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </div>
  );
}
```



### 异常捕获边界

​		如果模块加载失败（如网络问题），它会触发一个错误。你可以通过[异常捕获边界（Error boundaries）](https://zh-hans.reactjs.org/docs/error-boundaries.html)技术来处理这些情况，以显示良好的用户体验并管理恢复事宜。

```
import React, { Suspense } from 'react';
import MyErrorBoundary from './MyErrorBoundary';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

​		具体的后续会说。



### 基于路由的代码分割

​		这里是一个例子，展示如何在你的应用中使用 `React.lazy` 和 [React Router](https://reactrouter.com/) 这类的第三方库，来配置基于路由的代码分割。

```
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  </Router>
);
```



### 命名导出

​		简单来说就是使用 React.lazy 方法。他只会使用默认导出。

​		`React.lazy` 目前只支持默认导出（default exports）。如果你想被引入的模块使用命名导出（named exports），你可以创建一个中间模块，来重新导出为默认模块。这能保证 tree shaking 不会出错，并且不必引入不需要的组件。

```
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;

// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";

// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```



## Context

​		Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。

​		简单来说就是，因为props的是需要通过组件进行传递，那么此时，如果我们需要传递的属性极其繁琐。所以就会导致很麻烦，此时就可以使用context的形式。

​	**例子：**

```
class ThemeButton extends React.Component<any, any> {
    render() {
        return <button>{this.props.theme}</button>;
    }
}

class Toolbar extends React.Component<any, any> {
    render() {
        return <ThemeButton theme={this.props.theme}></ThemeButton>;
    }
}

class App extends React.Component<any, any> {
    render() {
        return (
            <div>
                <input type="text"/>
                <Toolbar theme={'qqq'}></Toolbar>
            </div>
        );
  }
}
```

​		我们可以发现，App为了传递一个theme给ThemeButton ，从中间传递了几次，类似这种，传递次数又长，又冗余的。

​		使用 context 进行传递。

* 首先，需要使用 const ThemeContext = React.createContext('light'); 进行创建一个context
* 然后，对于要传递的地方。使用 <ThemeContext.Provider value="dark"> xxx </ThemeContext.Provider> 标签进行包裹。这里的ThemeContext 就是创建context的命名。这里的 provider 就是代表了会将value进行传递给以下的组件树。
* 然后中间组件不需要传递 
* 对于要使用的，我们直接创建一个 static contextType = ThemeContext; 这里 contextType 是固定的。不可修改的。然后只需要再在标签里面通过 this.context 就可以访问了。
* 虽然感觉很奇怪。。

```
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');
class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    // 无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 指定 contextType 读取当前的 theme context。
  // React 会往上找到最近的 theme Provider，然后使用它的值。
  // 在这个例子中，当前的 theme 值为 “dark”。
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```



​		**如果你只是想避免层层传递一些属性，[组件组合（component composition）](https://zh-hans.reactjs.org/docs/composition-vs-inheritance.html)有时候是一个比 context 更好的解决方案。**

​		一种 **无需 context** 的解决方案是[将 `Avatar` 组件自身传递下去](https://zh-hans.reactjs.org/docs/composition-vs-inheritance.html#containment)，因为中间组件无需知道 `user` 或者 `avatarSize` 等 props：

```
function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}

// 现在，我们有这样的组件：
<Page user={user} avatarSize={avatarSize} />
// ... 渲染出 ...
<PageLayout userLink={...} />
// ... 渲染出 ...
<NavigationBar userLink={...} />
// ... 渲染出 ...
{props.userLink}
```

​		简单来说就是，我们传递的时候，将组件传递过去，然后中间层，只需要传递组件props，而不会传递其他的，就轻松很多，如同一种插槽一样的感觉。



### `React.createContext` API

```
const MyContext = React.createContext(defaultValue);
```

​		创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 `Provider` 中读取到当前的 context 值。

​		**只有**当组件所处的树中没有匹配到 Provider 时，其 `defaultValue` 参数才会生效。此默认值有助于在不使用 Provider 包装组件的情况下对组件进行测试。注意：将 `undefined` 传递给 Provider 的 value 时，消费组件的 `defaultValue` 不会生效。

​		具体的查看官方文档进行了解。



## 错误边界

​		错误边界是一种 React 组件，这种组件**可以捕获发生在其子组件树任何位置的 JavaScript 错误，并打印这些错误，同时展示降级 UI**，而并不会渲染那些发生崩溃的子组件树。错误边界可以捕获发生在整个子组件树的渲染期间、生命周期方法以及构造函数中的错误。

> 注意
>
> 错误边界**无法**捕获以下场景中产生的错误：
>
> - 事件处理（[了解更多](https://zh-hans.reactjs.org/docs/error-boundaries.html#how-about-event-handlers)）
> - 异步代码（例如 `setTimeout` 或 `requestAnimationFrame` 回调函数）
> - 服务端渲染
> - 它自身抛出来的错误（并非它的子组件）

​		如果一个 class 组件中定义了 [`static getDerivedStateFromError()`](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromerror) 或 [`componentDidCatch()`](https://zh-hans.reactjs.org/docs/react-component.html#componentdidcatch) 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界。当抛出错误后，请使用 `static getDerivedStateFromError()` 渲染备用 UI ，使用 `componentDidCatch()` 打印错误信息。

* static getDerivedStateFromError()
  * 渲染备用UI
* componentDidCatch()
  * 打印错误信息

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染能够显示降级后的 UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 你同样可以将错误日志上报给服务器
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 你可以自定义降级后的 UI 并渲染
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

​		然后你可以将它作为一个常规组件去使用：

```
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

​		错误边界的工作方式类似于 JavaScript 的 `catch {}`，不同的地方在于错误边界只针对 React 组件。只有 class 组件才可以成为错误边界组件。大多数情况下, 你只需要声明一次错误边界组件, 并在整个应用中使用它。

​		注意**错误边界仅可以捕获其子组件的错误**，它无法捕获其自身的错误。如果一个错误边界无法渲染错误信息，则错误会冒泡至最近的上层错误边界，这也类似于 JavaScript 中 `catch {}` 的工作机制。



### 未捕获错误的新行为

​		这一改变具有重要意义，**自 React 16 起，任何未被错误边界捕获的错误将会导致整个 React 组件树被卸载。**



### 组件栈追踪

> 注意
>
> 组件名称在栈追踪中的显示依赖于 [`Function.name`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/name) 属性。如果你想要支持尚未提供该功能的旧版浏览器和设备（例如 IE 11），考虑在你的打包（bundled）应用程序中包含一个 `Function.name` 的 polyfill，如 [`function.name-polyfill`](https://github.com/JamesMGreene/Function.name) 。或者，你可以在所有组件上显式设置 [`displayName`](https://zh-hans.reactjs.org/docs/react-component.html#displayname) 属性。



## refs转发

​		Ref 转发是一项将 [ref](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html) 自动地通过组件传递到其一子组件的技巧。对于大多数应用中的组件来说，这通常不是必需的。但其对某些组件，尤其是可重用的组件库是很有用的。最常见的案例如下所述。

```
class App extends React.Component<any, any> {
    render() {
        const FancyButton = React.forwardRef((props, ref) => (
            <button ref={ref as any} className="FancyButton">
                {props.children}
            </button>
        )) as any;

        // 你可以直接获取 DOM button 的 ref：
        const ref = React.createRef();
        return <FancyButton ref={ref}>Click me!</FancyButton>;
    }
}
```



## Fragments

​		React 中的一个常见模式是一个组件返回多个元素。Fragments 允许你将子列表分组，而无需向 DOM 添加额外节点。

​		还有一种新的[短语法](https://zh-hans.reactjs.org/docs/fragments.html#short-syntax)可用于声明它们。

```
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}

render() {
  return (
    <>
      <ChildA />
      <ChildB />
      <ChildC />
    </>
  );
}
```

​		作用，就是可以在使用的时候不会影响到DOM树，因为他不会渲染成DOM树。和template相似。

​		同时，可以携带key值，用于循环。



## 高阶组件

​		高阶组件（HOC）是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。

​		具体而言，**高阶组件是参数为组件，返回值为新组件的函数。**

​		同时高阶组件是一个纯函数。不会修改原来的参数。



​		`CommentList` 和 `BlogPost` 不同 - 它们在 `DataSource` 上调用不同的方法，且渲染不同的结果。但它们的大部分实现都是一样的：

- 在挂载时，向 `DataSource` 添加一个更改侦听器。
- 在侦听器内部，当数据源发生变化时，调用 `setState`。
- 在卸载时，删除侦听器。

​		你可以想象，在一个大型应用程序中，这种订阅 `DataSource` 和调用 `setState` 的模式将一次又一次地发生。我们需要一个抽象，允许我们在一个地方定义这个逻辑，并在许多组件之间共享它。这正是高阶组件擅长的地方。

​		对于订阅了 `DataSource` 的组件，比如 `CommentList` 和 `BlogPost`，我们可以编写一个创建组件函数。该函数将接受一个子组件作为它的其中一个参数，该子组件将订阅数据作为 prop。让我们调用函数 `withSubscription`：

```
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
);
```

```
// 此函数接收一个组件...
function withSubscription(WrappedComponent, selectData) {
  // ...并返回另一个组件...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ...负责订阅相关的操作...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... 并使用新数据渲染被包装的组件!
      // 请注意，我们可能还会传递其他属性
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```



## 深入JSX

​		实际上，JSX 仅仅只是 `React.createElement(component, props, ...children)` 函数的语法糖。如下 JSX 代码：

```
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

会编译为：

```
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

### 指定 React 元素类型

​		JSX 标签的第一部分指定了 React 元素的类型。

​		大写字母开头的 JSX 标签意味着它们是 React 组件。这些标签会被编译为对命名变量的直接引用，所以，当你使用 JSX `<Foo />` 表达式时，`Foo` 必须包含在作用域内。

#### React 必须在作用域内

​		由于 JSX 会编译为 `React.createElement` 调用形式，所以 `React` 库也必须包含在 JSX 代码作用域内。

​		例如，在如下代码中，虽然 `React` 和 `CustomButton` 并没有被直接使用，但还是需要导入：

```
import React from 'react';import CustomButton from './CustomButton';
function WarningButton() {
  // return React.createElement(CustomButton, {color: 'red'}, null);  return <CustomButton color="red" />;
}
```

#### 在 JSX 类型中使用点语法

​		在 JSX 中，你也可以使用点语法来引用一个 React 组件。当你在一个模块中导出许多 React 组件时，这会非常方便。例如，如果 `MyComponents.DatePicker` 是一个组件，你可以在 JSX 中直接使用：

```
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;}
```

#### 用户定义的组件必须以大写字母开头

#### 在运行时选择类型

​		你不能将通用表达式作为 React 元素类型。如果你想通过通用表达式来（动态）决定元素类型，你需要首先将它赋值给大写字母开头的变量。这通常用于根据 prop 来渲染不同组件的情况下:

```
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // 错误！JSX 类型不能是一个表达式。  return <components[props.storyType] story={props.story} />;}
```

​		要解决这个问题, 需要首先将类型赋值给一个大写字母开头的变量：

```
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // 正确！JSX 类型可以是大写字母开头的变量。  const SpecificStory = components[props.storyType];  return <SpecificStory story={props.story} />;}
```

### JSX 中的 Props

#### 字符串字面量

​		你可以将字符串字面量赋值给 prop. 如下两个 JSX 表达式是等价的：

```
<MyComponent message="hello world" />

<MyComponent message={'hello world'} />
```

​		当你将字符串字面量赋值给 prop 时，它的值是未转义的。所以，以下两个 JSX 表达式是等价的：

```
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

#### Props 默认值为 “True”

​		如果你没给 prop 赋值，它的默认值是 `true`。以下两个 JSX 表达式是等价的：

```
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

​		通常，我们不建议不传递 value 给 prop，因为这可能与 [ES6 对象简写](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_2015)混淆，`{foo}` 是 `{foo: foo}` 的简写，而不是 `{foo: true}`。这样实现只是为了保持和 HTML 中标签属性的行为一致。

#### 属性展开

​		如果你已经有了一个 props 对象，你可以使用展开运算符 `...` 来在 JSX 中传递整个 props 对象。以下两个组件是等价的：

```
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

#### JSX 子元素

```
render() {
  // 不需要用额外的元素包裹列表元素！
  return [
    // 不要忘记设置 key :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

#### 函数作为子元素

​		通常，JSX 中的 JavaScript 表达式将会被计算为字符串、React 元素或者是列表。不过，`props.children` 和其他 prop 一样，它可以传递任意类型的数据，而不仅仅是 React 已知的可渲染类型。例如，如果你有一个自定义组件，你可以把回调函数作为 `props.children` 进行传递：

```
// 调用子元素回调 numTimes 次，来重复生成组件
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}    </Repeat>
  );
}
```

#### 布尔类型、Null 以及 Undefined 将会忽略

​		`false`, `null`, `undefined`, and `true` 是合法的子元素。但它们并不会被渲染。以下的 JSX 表达式渲染结果相同：

​		值得注意的是有一些 [“falsy” 值](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)，如数字 `0`，仍然会被 React 渲染。例如，以下代码并不会像你预期那样工作，因为当 `props.messages` 是空数组时，`0` 仍然会被渲染：

```
<div>
  {props.messages.length &&    <MessageList messages={props.messages} />
  }
</div>
```



## 性能优化

​		UI 更新需要昂贵的 DOM 操作，因此 React 内部使用了几种巧妙的技术来最小化 DOM 操作次数。对于大部分应用而言，使用 React 时无需做大量优化工作就能拥有高性能的用户界面。尽管如此，你仍然有办法来加速你的 React 应用。

### 虚拟化长列表

​		如果你的应用渲染了长列表（上百甚至上千的数据），我们推荐使用“虚拟滚动”技术。这项技术会在有限的时间内仅渲染有限的内容，并奇迹般地降低重新渲染组件消耗的时间，以及创建 DOM 节点的数量。

​		[react-window](https://react-window.now.sh/) 和 [react-virtualized](https://bvaughn.github.io/react-virtualized/) 是热门的虚拟滚动库。 它们提供了多种可复用的组件，用于展示列表、网格和表格数据。 如果你想要一些针对你的应用做定制优化，你也可以创建你自己的虚拟滚动组件，就像 [Twitter 所做的](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3)。

### 避免调停

​		React 构建并维护了一套内部的 UI 渲染描述。它包含了来自你的组件返回的 React 元素。该描述使得 React 避免创建 DOM 节点以及没有必要的节点访问，因为 DOM 操作相对于 JavaScript 对象操作更慢。虽然有时候它被称为“虚拟 DOM”，但是它在 React Native 中拥有相同的工作原理。

​		当一个组件的 props 或 state 变更，React 会将最新返回的元素与之前渲染的元素进行对比，以此决定是否有必要更新真实的 DOM。当它们不相同时，React 会更新该 DOM。

​		即使 React 只更新改变了的 DOM 节点，重新渲染仍然花费了一些时间。在大部分情况下它并不是问题，不过如果它已经慢到让人注意了，你可以通过覆盖生命周期方法 `shouldComponentUpdate` 来进行提速。该方法会在重新渲染前被触发。其默认实现总是返回 `true`，让 React 执行更新：

```
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

​		如果你知道在什么情况下你的组件不需要更新，你可以在 `shouldComponentUpdate` 中返回 `false` 来跳过整个渲染过程。其包括该组件的 `render` 调用以及之后的操作。

​		在大部分情况下，你可以继承 [`React.PureComponent`](https://zh-hans.reactjs.org/docs/react-api.html#reactpurecomponent) 以代替手写 `shouldComponentUpdate()`。它用当前与之前 props 和 state 的浅比较覆写了 `shouldComponentUpdate()` 的实现。

### shouldComponentUpdate 的作用

​		这是一个组件的子树。每个节点中，`SCU` 代表 `shouldComponentUpdate` 返回的值，而 `vDOMEq` 代表返回的 React 元素是否相同。最后，圆圈的颜色代表了该组件是否需要被调停。

[![should component update](react-高级指引/should-component-update.png)](https://zh-hans.reactjs.org/static/5ee1bdf4779af06072a17b7a0654f6db/cd039/should-component-update.png)

​		节点 C2 的 `shouldComponentUpdate` 返回了 `false`，React 因而不会去渲染 C2，也因此 C4 和 C5 的 `shouldComponentUpdate` 不会被调用到。

​		对于 C1 和 C3，`shouldComponentUpdate` 返回了 `true`，所以 React 需要继续向下查询子节点。这里 C6 的 `shouldComponentUpdate` 返回了 `true`，同时由于渲染的元素与之前的不同使得 React 更新了该 DOM。

​		最后一个有趣的例子是 C8。React 需要渲染这个组件，但是由于其返回的 React 元素和之前渲染的相同，所以不需要更新 DOM。

​		显而易见，你看到 React 只改变了 C6 的 DOM。对于 C8，通过对比了渲染的 React 元素跳过了渲染。而对于 C2 的子节点和 C7，由于 `shouldComponentUpdate` 使得 `render` 并没有被调用。因此它们也不需要对比元素了。

#### 示例

```
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```



## Portals

​		Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。

```
ReactDOM.createPortal(child, container)
```

​		第一个参数（`child`）是任何[可渲染的 React 子元素](https://zh-hans.reactjs.org/docs/react-component.html#render)，例如一个元素，字符串或 fragment。第二个参数（`container`）是一个 DOM 元素。



## Profiler API

​		`Profiler` 测量一个 React 应用多久渲染一次以及渲染一次的“代价”。 它的目的是识别出应用中渲染较慢的部分，或是可以使用[类似 memoization 优化](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-to-memoize-calculations)的部分，并从相关优化中获益。

> 注意：
>
> Profiling 增加了额外的开支，所以**它在[生产构建](https://zh-hans.reactjs.org/docs/optimizing-performance.html#use-the-production-build)中会被禁用**。
>
> 为了将 profiling 功能加入生产环境中，React 提供了使 profiling 可用的特殊的生产构建环境。 从 [fb.me/react-profiling](https://fb.me/react-profiling)了解更多关于如何使用这个构建环境的信息。



## 协调

​		React 提供的声明式 API 让开发者可以在对 React 的底层实现并不了解的情况下编写应用。在开发者编写应用时，可以保持相对简单的心智，但开发者无法了解其内部的实现原理。本文描述了在实现 React 的 “diffing” 算法过程中所作出的设计决策，以保证组件更新可预测，且在繁杂业务场景下依然保持应用的高性能。

### 设计动机

​		在某一时间节点调用 React 的 `render()` 方法，会创建一棵由 React 元素组成的树。在下一次 state 或 props 更新时，相同的 `render()` 方法会返回一棵不同的树。React 需要基于这两棵树之间的差别来判断如何高效的更新 UI，以保证当前 UI 与最新的树保持同步。

​		通常一个最优算法里面。时间复杂度依然很高。

​		所以React在以下的两个基础之上提出了另外一套算法。同时也是diffing算法的核心。（我认为）

* 两个不同类型的元素会产生出不同的树
* 开发者可以通过设置 key 属性，来告知那些子元素在不同的渲染下可以保存不变。



### diffing 算法

```
https://zh-hans.reactjs.org/docs/reconciliation.html#the-diffing-algorithm
```

#### 对比不同类型的元素

​		当根节点为不同类型的元素时，React 会拆卸原有的树并且建立起新的树。举个例子，当一个元素从 `<a>` 变成 `<img>`，从 `<Article>` 变成 `<Comment>`，或从 `<Button>` 变成 `<div>` 都会触发一个完整的重建流程。

#### 对比同一类型的元素

​		当对比两个相同类型的 React 元素时，React 会保留 DOM 节点，仅比对及更新有改变的属性。比如：

```
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

​		通过对比这两个元素，React 知道只需要修改 DOM 元素上的 `className` 属性。

​		当更新 `style` 属性时，React 仅更新有所更变的属性。比如：

```
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

​		通过对比这两个元素，React 知道只需要修改 DOM 元素上的 `color` 样式，无需修改 `fontWeight`。

​		在处理完当前节点之后，React 继续对子节点进行递归。

#### 对比同类型的组件元素

​		当一个组件更新时，组件实例会保持不变，因此可以在不同的渲染时保持 state 一致。

​		下一步，调用 `render()` 方法，diff 算法将在之前的结果以及新的结果中进行递归。

#### 对子节点进行递归

​		默认情况下，当递归 DOM 节点的子元素时，React 会同时遍历两个子元素的列表；当产生差异时，生成一个 mutation。

* 在子元素列表末尾新增元素时，更新开销比较小。

* 如果只是简单的将新增元素插入到表头，那么更新开销会比较大。
  * 因为React 不会意识到应该保留其他元素。所以会导致开销比较大。
  * 所以需要使用keys

#### Keys

​		为了解决上述问题，React 引入了 `key` 属性。当子元素拥有 key 时，React 使用 key 来匹配原有树上的子元素以及最新树上的子元素。

​		当基于下标的组件进行重新排序时，组件 state 可能会遇到一些问题。由于组件实例是基于它们的 key 来决定是否更新以及复用，如果 key 是一个下标，那么修改顺序时会修改当前的 key，导致非受控组件的 state（比如输入框）可能相互篡改，会出现无法预期的变动。

​		在 Codepen 有两个例子，分别为 [展示使用下标作为 key 时导致的问题](https://zh-hans.reactjs.org/redirect-to-codepen/reconciliation/index-used-as-key)，以及[不使用下标作为 key 的例子的版本，修复了重新排列，排序，以及在列表头插入的问题](https://zh-hans.reactjs.org/redirect-to-codepen/reconciliation/no-index-used-as-key)。

#### 权衡

​		我们定期优化启发式算法，让常见用例更高效地执行。在当前的实现中，可以理解为一棵子树能在其兄弟之间移动，但不能移动到其他位置。在这种情况下，算法会重新渲染整棵子树。

​		由于 React 依赖启发式算法，因此当以下假设没有得到满足，性能会有所损耗。

1. 该算法不会尝试匹配不同组件类型的子树。如果你发现你在两种不同类型的组件中切换，但输出非常相似的内容，建议把它们改成同一类型。在实践中，我们没有遇到这类问题。
2. Key 应该具有稳定，可预测，以及列表内唯一的特质。不稳定的 key（比如通过 `Math.random()` 生成的）会导致许多组件实例和 DOM 节点被不必要地重新创建，这可能导致性能下降和子组件中的状态丢失。



## Refs and the DOM

​		Refs 提供了一种方式，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素。



### 回调Ref和普通ref的使用

#### 普通ref

* 需要使用 React.createRef(); 进行创建，否则不能ref成功
* 同时我们查看了 createRef 的方法里面实现了什么，发现就只是创建了一个 current 对象，然后再使用 Object.seal 将其对象进行了封闭操作。
* 同时如果我们创建一个对象。只要对象里面存在current属性，那么便可以存储ref

```
class Main extends React.Component<any, any> {
    private inputRef:any
    
    constructor(props:any) {
        super(props);
        
        this.inputRef = React.createRef();
    }

    render() {
        return (
            <div>
                <input type="text" ref={this.inputRef}/>
            </div>
        );
    }
}
```

#### 回调ref

* 不需要使用 React.createRef(); 进行初始化
* 通过使用回调函数，函数的第一个参数就是对应的DOM节点。
* 和普通ref一样使用ref属性attributes。

```
class Main extends React.Component<any, any> {
    private inputRef:any

    constructor(props:any) {
        super(props);
    }

    render() {
        const setInputRef = (el:any) => {
            this.inputRef = el
        }
        return (
            <div>
                <input type="text" ref={setInputRef}/>
            </div>
        );
    }
}
```

#### 关于回调 refs 的说明

​		如果 `ref` 回调函数是以内联函数的方式定义的，在更新过程中它会被执行两次，第一次传入参数 `null`，然后第二次会传入参数 DOM 元素。这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，但是大多数情况下它是无关紧要的。



## Render Props

​		术语 [“render prop”](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) 是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术

### 使用 Render Props 来解决横切关注点（Cross-Cutting Concerns）

​		简单来说就是vue的插槽，vue的插槽，就是说，可以在使用这个组件的时候，自己定义组件内部的显示为自己定义的组件。然后同时还需要使用作用域插槽来解决作用于的问题。貌似

​		首先，我们知道可以使用 this.props.children 的属性可以有一个插槽的效果

```
class Main extends React.Component<any, any> {
    render() {
        return (
            <div>
                {this.props.children}
            </div>
        );
    }
}
```

​		但是，这样写，有一个作用域的问题，以及不能调用Main里面的属性。

​		所以此时就要使用 this.props.render 来代替 children 了

```
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

        {/*
          使用 `render`prop 动态决定要渲染的内容，
          而不是给出一个 <Mouse> 渲染结果的静态表示
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>移动鼠标!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

​		其思想就是，首先，this.props.render 这个是一个props属性，所以render是调用组件时传递过来的。然后传递这个参数this.state 这个参数就会作为 mouse，然后传递给Cat组件的mouse属性。

​		然后这个的返回值又会渲染。标签。

```
this.props.render(this.state)

Mouse render={mouse => (
	Cat mouse={mouse}	
)}
```

​		关于 render prop 一个有趣的事情是你可以使用带有 render prop 的常规组件来实现大多数[高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html) (HOC)。 例如，如果你更喜欢使用 `withMouse` HOC而不是 `<Mouse>` 组件，你可以使用带有 render prop 的常规 `<Mouse>` 轻松创建一个：

```
// 如果你出于某种原因真的想要 HOC，那么你可以轻松实现
// 使用具有 render prop 的普通组件创建一个！
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )}/>
      );
    }
  }
}
```

​		重要的是要记住，render prop 是因为模式才被称为 *render* prop ，你不一定要用名为 `render` 的 prop 来使用这种模式。事实上， [*任何*被用于告知组件需要渲染什么内容的函数 prop 在技术上都可以被称为 “render prop”](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce).

​		尽管之前的例子使用了 `render`，我们也可以简单地使用 `children` prop！

```
<Mouse children={mouse => (
  <p>鼠标的位置是 {mouse.x}，{mouse.y}</p>
)}/>
```

​		记住，`children` prop 并不真正需要添加到 JSX 元素的 “attributes” 列表中。相反，你可以直接放置到元素的*内部*！

```
<Mouse>
  {mouse => (
    <p>鼠标的位置是 {mouse.x}，{mouse.y}</p>
  )}
</Mouse>
```

​		这里，就是使用了 children 属性，我们可以知道，this.props.children 属性将作为组件的插槽形式进行传递。我们在调用组件的时候，直接作为内容，然后就会作为租价的children属性。

### 注意事项

#### 将 Render Props 与 React.PureComponent 一起使用时要小心

​		如果你在 render 方法里创建函数，那么使用 render prop 会抵消使用 [`React.PureComponent`](https://zh-hans.reactjs.org/docs/react-api.html#reactpurecomponent) 带来的优势。因为浅比较 props 的时候总会得到 false，并且在这种情况下每一个 `render` 对于 render prop 将会生成一个新的值。



## 静态类型检查

​		像 [Flow](https://flow.org/) 和 [TypeScript](https://www.typescriptlang.org/) 等这些静态类型检查器，可以在运行前识别某些类型的问题。他们还可以通过增加自动补全等功能来改善开发者的工作流程。出于这个原因，我们建议在大型代码库中使用 Flow 或 TypeScript 来代替 `PropTypes`。

```
npx create-react-app my-app --template typescript
```

​		如需将 TypeScript 添加到**现有的 Create React App 项目**中，[请参考此文档](https://facebook.github.io/create-react-app/docs/adding-typescript).

### 添加 TypeScript 到现有项目中

```
https://zh-hans.reactjs.org/docs/static-type-checking.html#adding-typescript-to-a-project
```

* 先下载typescript的安装包
* 然后在package.json 里面添加scripts指令。tsc就是将ts编译的指令。
* 创建tsconfig.json 的文件，用于typescript的编译配置
  * 可以使用 npm tsc --init
  * `tsconfig.json` 文件中，有许多配置项用于配置编译器。查看所有配置项的的详细说明，[请参考此文档](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)。



## 严格模式

​		`StrictMode` 是一个用来突出显示应用程序中潜在问题的工具。与 `Fragment` 一样，`StrictMode` 不会渲染任何可见的 UI。它为其后代元素触发额外的检查和警告。

> 注意：
>
> 严格模式检查仅在开发模式下运行；*它们不会影响生产构建*。

​		你可以为应用程序的任何部分启用严格模式。例如：

```
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>      <Footer />
    </div>
  );
}
```

​		在上述的示例中，*不*会对 `Header` 和 `Footer` 组件运行严格模式检查。但是，`ComponentOne` 和 `ComponentTwo` 以及它们的所有后代元素都将进行检查。

​		`StrictMode` 目前有助于：

- [识别不安全的生命周期](https://zh-hans.reactjs.org/docs/strict-mode.html#identifying-unsafe-lifecycles)
- [关于使用过时字符串 ref API 的警告](https://zh-hans.reactjs.org/docs/strict-mode.html#warning-about-legacy-string-ref-api-usage)
- [关于使用废弃的 findDOMNode 方法的警告](https://zh-hans.reactjs.org/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage)
- [检测意外的副作用](https://zh-hans.reactjs.org/docs/strict-mode.html#detecting-unexpected-side-effects)
- [检测过时的 context API](https://zh-hans.reactjs.org/docs/strict-mode.html#detecting-legacy-context-api)

​		未来的 React 版本将添加更多额外功能。



## 使用 PropTypes 进行类型检查

> 注意：
>
> 自 React v15.5 起，`React.PropTypes` 已移入另一个包中。请使用 [`prop-types` 库](https://www.npmjs.com/package/prop-types) 代替。
>
> 我们提供了一个 [codemod 脚本](https://zh-hans.reactjs.org/blog/2017/04/07/react-v15.5.0.html#migrating-from-reactproptypes)来做自动转换。



## 非受控组件

​		在大多数情况下，我们推荐使用 [受控组件](https://zh-hans.reactjs.org/docs/forms.html#controlled-components) 来处理表单数据。在一个受控组件中，表单数据是由 React 组件来管理的。另一种替代方案是使用非受控组件，这时表单数据将交由 DOM 节点来处理。

### 默认值

​		在 React 渲染生命周期时，表单元素上的 `value` 将会覆盖 DOM 节点中的值。在非受控组件中，你经常希望 React 能赋予组件一个初始值，但是不去控制后续的更新。 在这种情况下, 你可以指定一个 `defaultValue` 属性，而不是 `value`。在一个组件已经挂载之后去更新 `defaultValue` 属性的值，不会造成 DOM 上值的任何更新。



## Web Components

​		React 和 [Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components) 为了解决不同的问题而生。Web Components 为可复用组件提供了强大的封装，而 React 则提供了声明式的解决方案，使 DOM 与数据保持同步。两者旨在互补。作为开发人员，可以自由选择在 Web Components 中使用 React，或者在 React 中使用 Web Components，或者两者共存。





