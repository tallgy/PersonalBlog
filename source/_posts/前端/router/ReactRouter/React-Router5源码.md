---
title: React-Router5源码
date: 2023-07-23 16:30:27
tags:
 - React
 - Router
categories:
 - React
 - Router
 - v5源码
---

# React-Router v5 源码分析

GitHub
fork 的 remix 
v5分支
```
https://github.com/tallgy/react-router
```


## Link && NavLink

router 跳转需要的 link

NavLink 内部也是使用 Link 只是主要 多了 active Class Style 等激活属性
表示如果当前页面属于当前的 link 那么就会有对应的样式展示 这里源码就没有贴出来了

Link 内部使用的跳转是 LinkAnchor

Link 核心代码逻辑
```JavaScript

const props = {
  ...,
  navigate() {
    // 判断是否为重复导航 createPath 通过location 生成一个完整的 url
    // 通过 当前页面的location 和 本Link的location比较是否同一个
    const isDuplicateNavigation = createPath(context.location) === createPath(normalizeToLocation(location));
    // 判断使用 replace 还是 push
    const method = (replace || isDuplicateNavigation) ? history.replace : history.push;

    method(location);
  }
};

return React.createElement(component = LinkAnchor, props);

```

LinkAnchor 核心逻辑
```JavaScript

let props = {
  ...rest,
  // onClick 事件
  onClick: event => {
    try {
      // 是否有用户自定的事件
      if (onClick) onClick(event);
    } catch (ex) {
      // catch 之后直接上抛，不执行后续的跳转逻辑
      event.preventDefault();
      throw ex;
    }

    // 如果 默认阻止了、按键不为左键、target跳转不是当前页面跳转、存在修饰符的点击，那么都不执行
    if (
      !event.defaultPrevented && // onClick prevented default onClick阻止默认
      event.button === 0 && // ignore everything but left clicks 忽略一切，除了左键
      // 如果存在 target 如果 target 为 _self 才能执行，代表了其他模式由浏览器自行处理
      (!target || target === "_self") && // let browser handle "target=_blank" etc. 让浏览器处理“target= blank”等。
      !isModifiedEvent(event) // ignore clicks with modifier keys // 忽略带有修饰符键的单击
    ) {
      event.preventDefault();
      // 执行 navigate 进行跳转
      navigate();
    }
  }
};

// 代表了 LinkAnchor 其实就是 a 链接。
return <a {...props} />;

```


## BrowserRouter && HashRouter

内部本质都是使用的 Router
Router 组件主要是将 history 等放在了 Context 环境中
所以这也是为什么，我们可以在 子组件里面直接获取 history ，因为这个是直接放在了 Context 里面了的。

Router 核心代码
```JavaScript


/**
 * 用于将历史记录放在上下文上的公共API。
 * The public API for putting history on context.
 */
class Router extends React.Component {
  // 构造函数 执行一次
  constructor(props) {
    this.state = {
      location: props.history.location
    };

    // 这是一个 hack。
    // 我们必须在构造函数中开始监听位置变化，以防在初始渲染中出现<Redirect>。
    // 如果有，它们将在挂载时 replace/push，并且由于 cDM 在子节点中先于父节点触发，我们可能会在<Router>被挂载之前得到一个新的位置。
    // 监听 location 将 pendingLocation 进行更新
    this.unlisten = props.history.listen(location => {
      this._pendingLocation = location;
    });
  }

  // 组件被挂载时 执行一次
  componentDidMount() {
    if (this.unlisten) {
      // 此时已经捕获了任何预挂载位置更改，因此取消侦听器的注册。
      this.unlisten();
    }
    this.unlisten = this.props.history.listen(location => {
      // 此时可以使用 setState 去进行更新 location
      if (this._isMounted) {
        this.setState({ location });
      }
    });
    // 这是由最开始 构造函数的时候进行的监听所获取的 _pendingLocation
    if (this._pendingLocation) {
      this.setState({ location: this._pendingLocation });
    }
  }

  // 组件被卸载 执行一次
  componentWillUnmount() {
    if (this.unlisten) {
      // 注销监听
      this.unlisten();
    }
  }

  render() {
    return (
      // 主要是为了创建 Provider context 用于内部组件去获取 history location 等数据
      // RouterContext 有 history、location、match、staticContext
      // HistoryContext 有 history
      <RouterContext.Provider
        value={{
          history: this.props.history,
          location: this.state.location,
          match: Router.computeRootMatch(this.state.location.pathname),
          staticContext: this.props.staticContext
        }}
      >
        <HistoryContext.Provider
          children={this.props.children || null}
          value={this.props.history}
        />
      </RouterContext.Provider>
    );
  }
}


```

## 

