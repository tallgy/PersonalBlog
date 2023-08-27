---
title: qiankun
date: 2023-08-23 15:36:44
tags:
 - qiankun
categories:
 - qiankun
---

# qiankun 微前端


## scope css css 隔离方式

原理比较简单

就是使用 cssRule 将 textContent 添加上 scope
首先因为微应用的设置，所以顶部会有一个 div 包裹，和一个 data-qiankun 的 appName
所以 结合起来将 css 拼接 添加上了 scope

```JavaScript

const styleNodes = appElement.querySelectorAll('style') || [];
styleNodes.forEach(styleNode => {
  // 详细请看 styleSheet https://developer.mozilla.org/zh-CN/docs/Web/API/StyleSheet
  const sheet = styleNode.sheet as any; // type is missing
  // 详情请看 https://developer.mozilla.org/zh-CN/docs/Web/API/CSSRule
  // arrayify 主要是变为数组效果
  const rules = arrayify<CSSRule>(sheet?.cssRules ?? []);
  // 这个可以让每一个 css 都添加上 scope
  const css = this.rewrite(rules, prefix);
  // eslint-disable-next-line no-param-reassign
  // 覆盖原来的 textContent 可以生效
  styleNode.textContent = css;

  (styleNode as any)[ScopedCSS.ModifiedTag] = true;
})

```


## shadow DOM

比较简单就是，直接创建 shadow DOM 

这里需要注意的是： style 需要放在 shadowDOM 里面，否则不会有作用。
粗略测试

```JavaScript

const { innerHTML } = appElement;
appElement.innerHTML = '';
let shadow;
console.log(appElement, appElement.attachShadow);
shadow = appElement.attachShadow({ mode: 'open' });
shadow.innerHTML = innerHTML;

```


## 沙箱模式 sandbox


### LegacySandbox 基于Proxy 支持单应用的代理沙箱

内部是一个 Proxy 

创建一个 fakeWindow 。然后使用 Proxy

简易源码
```JavaScript
/**
 * 基于 Proxy 实现的沙箱
 * name: 沙箱名字
 * context：沙箱所依赖的全局变量
 * TODO: 为了兼容性 singular 模式下依旧使用该沙箱，等新沙箱稳定之后再切换
 */
export default class LegacySandbox implements SandBox {
  /** 沙箱期间新增的全局变量 */
  private addedPropsMapInSandbox = new Map<PropertyKey, any>();
  /** 沙箱期间更新的全局变量 */
  private modifiedPropsOriginalValueMapInSandbox = new Map<PropertyKey, any>();
  /** 持续记录更新的(新增和修改的)全局变量的 map，用于在任意时刻做 snapshot */
  private currentUpdatedPropsValueMap = new Map<PropertyKey, any>();
  globalContext: typeof window;
  sandboxRunning = true;

  private setWindowProp(prop: PropertyKey, value: any, toDelete?: boolean) {
    if (value === undefined && toDelete) {
      delete (this.globalContext as any)[prop];
      return;
    }

    (this.globalContext as any)[prop] = value;
  }

  /** 激活 将 currentUpdatedPropsValueMap 更新到 全局 */
  active() {
    if (!this.sandboxRunning) {
      this.currentUpdatedPropsValueMap.forEach((v, p) => this.setWindowProp(p, v));
    }

    this.sandboxRunning = true;
  }

  /** 不激活 */
  inactive() {
    // 将全局道具恢复到初始快照
    // 这个将所有值恢复原始状态
    this.modifiedPropsOriginalValueMapInSandbox.forEach((v, p) => this.setWindowProp(p, v));
    // 这个将所有新增的值，删除
    this.addedPropsMapInSandbox.forEach((_, p) => this.setWindowProp(p, undefined, true));

    this.sandboxRunning = false;
  }

  constructor(name: string, globalContext = window) {
    this.globalContext = globalContext;
    const { addedPropsMapInSandbox, modifiedPropsOriginalValueMapInSandbox, currentUpdatedPropsValueMap } = this;

    const rawWindow = globalContext;
    // 创建一份 fake window
    const fakeWindow = Object.create(null) as Window;

    /**
     * 这种沙箱应该是类似于 将 新增的变量记录下来
     * 将如果存在的属性修改，就会记录原始数据
     * 在移除的时候，可以直接删除新增的变量，然后将原始数据覆盖新数据
     * 做到恢复的效果
     */
    const setTrap = (p: PropertyKey, value: any, originalValue: any, sync2Window = true) => {
      // 后面有对于这个的解析
      ...
    };

    // 通过使用 Proxy 代理。
    const proxy = new Proxy(fakeWindow, {
      // 后面也有对于 Proxy 的解析
      ...
    });

    this.proxy = proxy;
  }

  patchDocument(): void {}
}

```


#### 简易 Proxy 代码
```JavaScript

// 通过使用 Proxy 代理。
const proxy = new Proxy(fakeWindow, {
  set: (_: Window, p: PropertyKey, value: any): boolean => {
    const originalValue = (rawWindow as any)[p];
    // 返回值必须为 true
    return setTrap(p, value, originalValue, true);
  },

  get(_: Window, p: PropertyKey): any {
    // 代表都是 window 本身
    if (p === 'top' || p === 'parent' || p === 'window' || p === 'self') {
      return proxy;
    }
    // 这里有一个处理是考虑 方法的情况。将 this 指向进行了处理
    return value = (rawWindow as any)[p];
  },

  has(_: Window, p: string | number | symbol): boolean {
    return p in rawWindow;
  },

  /**
   * 获取属性描述
   * @param _ 
   * @param p 
   * @returns 
   */
  getOwnPropertyDescriptor(_: Window, p: PropertyKey): PropertyDescriptor | undefined {
    return Object.getOwnPropertyDescriptor(rawWindow, p);
  },

  /**
   * 这个更新属性 拦截对象的 Object.defineProperty() 操作。
   * Object.defineProperty 返回一个对象，或者如果属性没有被成功定义，抛出一个 TypeError。
   * 相比之下，Reflect.defineProperty 方法只返回一个 Boolean，来说明该属性是否被成功定义。
   * @param _ 
   * @param p 
   * @param attributes 
   * @returns 
   */
  defineProperty(_: Window, p: string | symbol, attributes: PropertyDescriptor): boolean {
    const originalValue = (rawWindow as any)[p];
    const done = Reflect.defineProperty(rawWindow, p, attributes);
    const value = (rawWindow as any)[p];
    setTrap(p, value, originalValue, false);

    return done;
  },
});

```

#### setTrap 方法

这个方法将更新了的属性都加入map
用于处理 snap 快照问题

简述就是
将新增的属性加入 addedPropsMapInSandbox 并记录最开始的值
将修改的属性加入 modifiedPropsOriginalValueMapInSandbox 并记录原始值
然后持续更新属性 currentUpdatedPropsValueMap 里面的值   用于 snap 快照
新增和修改的属性用于在 inactive 的时候知道如何修改数据。

```JavaScript
/**
 * 设置属性 p 的值为 value originValue
 * 首先，每次的值 会通过 currentUpdatedPropsValueMap 进行更新
 * 如果 sync2Window 为 true 那么就会 同步更新到 globalContext 里面
 * 如果 globalContext 没有，那么就会 addedPropsMapInSandbox 添加 value
 * 如果 modifiedPropsOriginalValueMapInSandbox 没有，那么就会添加 originalValue
 * 
 * if window no has p，那么属于新增变量
 * else if 不属于更新的变量，那么记录原始数据
 * 然后更新数据
 * 
 * 这种沙箱应该是类似于 将 新增的变量记录下来
 * 将如果存在的属性修改，就会记录原始数据
 * 在移除的时候，可以直接删除新增的变量，然后将原始数据覆盖新数据
 * 做到恢复的效果
 * @param p 
 * @param value 
 * @param originalValue 
 * @param sync2Window 
 * @returns true
 */
const setTrap = (p: PropertyKey, value: any, originalValue: any, sync2Window = true) => {
  // 如果本身数据没有，那么加入新增
  if (!rawWindow.hasOwnProperty(p)) {
    addedPropsMapInSandbox.set(p, value);
  } else if (!modifiedPropsOriginalValueMapInSandbox.has(p)) {
    // 如果当前 window 对象存在该属性，且 record map 中未记录过，则记录该属性初始值
    modifiedPropsOriginalValueMapInSandbox.set(p, originalValue);
  }

  // 持续更新数据
  currentUpdatedPropsValueMap.set(p, value);

  if (sync2Window) {
    // 必须重新设置 window 对象保证下次 get 时能拿到已更新的数据
    (rawWindow as any)[p] = value;
  }

  this.latestSetProp = p;

  // 在 strict-mode 下，Proxy 的 handler.set 返回 false 会抛出 TypeError，在沙箱卸载的情况下应该忽略错误
  // 所以直接 return true
  return true;
};

```


### ProxySandbox 基于Proxy 支持多应用的代理沙箱

基于 Proxy 的沙箱
这个沙箱的实现是 首先将 不可配置的属性 赋值到 fakeWindow 里面
然后在 set 的时候，判断是否是原有属性存在的，存在将其 getOwnPropertyDescriptor 记录下来
然后在讲值记录到 updatedValueSet 里面。 这个的作用没有理解有什么作用

在 inactive 的时候 使用 getOwnPropertyDescriptor 属性将其 描述赋值回去，实现恢复的效果

源码
```JavaScript
/**
 * 基于 Proxy 实现的沙箱
 */
export default class ProxySandbox implements SandBox {
  /** window 值变更记录 */
  private updatedValueSet = new Set<PropertyKey>();
  private document = document;
  proxy: WindowProxy;
  sandboxRunning = true;

  active() {
    if (!this.sandboxRunning) activeSandboxCount++;
    this.sandboxRunning = true;
  }

  inactive() {
    if (--activeSandboxCount === 0) {
      // 将全局值重置为前一个值
      Object.keys(this.globalWhitelistPrevDescriptor).forEach((p) => {
        const descriptor = this.globalWhitelistPrevDescriptor[p];
        if (descriptor) {
          // 如果存在描述，直接重新定义描述就行
          Object.defineProperty(this.globalContext, p, descriptor);
        } else {
          // 否则直接删除这个属性就行
          // @ts-ignore
          delete this.globalContext[p];
        }
      });
    }

    this.sandboxRunning = false;
  }

  // 白名单中未被修改的全局变量的描述符
  // the descriptor of global variables in whitelist before it been modified
  globalWhitelistPrevDescriptor: { [p in (typeof globalVariableWhiteList)[number]]: PropertyDescriptor | undefined } =
    {};
  globalContext: typeof window;

  constructor(name: string, globalContext = window, opts?: { speedy: boolean }) {
    this.globalContext = globalContext;
    const { updatedValueSet } = this;
    const { speedy } = opts || {};

    const { fakeWindow, propertiesWithGetter } = createFakeWindow(globalContext, !!speedy);

    const descriptorTargetMap = new Map<PropertyKey, SymbolTarget>();

    const proxy = new Proxy(fakeWindow, {
      // 后面存在 源码解析
      ...
    });

    this.proxy = proxy;

    activeSandboxCount++;

    function hasOwnProperty(this: any, key: PropertyKey): boolean {
      // calling from hasOwnProperty.call(obj, key)
      if (this !== proxy && this !== null && typeof this === 'object') {
        return Object.prototype.hasOwnProperty.call(this, key);
      }

      return fakeWindow.hasOwnProperty(key) || globalContext.hasOwnProperty(key);
    }
  }

  /**
   * 更新目前被运行的app
   * 同时每次 微任务之后都要去 清除 不知道这个的原因是什么。。
   * @param name 
   * @param proxy 
   */
  private registerRunningApp(name: string, proxy: Window) {
    if (this.sandboxRunning) {
      const currentRunningApp = getCurrentRunningApp();
      // 更新 目前被运行的 app 如果是这个，表明了，同时只能一次只能运行一个app
      if (!currentRunningApp || currentRunningApp.name !== name) {
        setCurrentRunningApp({ name, window: proxy });
      }
      // 如果你有任何其他好的想法，请删除下一个tick中的标记，
      // 这样我们就可以确定它是否在微应用中，这种方法只是一种变通方法，它不能涵盖所有复杂的情况，例如微应用在某些情况下与master运行在相同的任务上下文中
      // 微任务
      nextTask(clearCurrentRunningApp);
    }
  }
}

```

#### createFakeWindow

创建一份 fake window
fake window 里面保存的属性是 不可配置的属性

```JavaScript

/**
 * 将global的不可配置属性复制到fakeWindow 
 * 并且将属性存在 get 方法的属性 放入 propertiesWithGetter 值为 true
 * @param globalContext 
 * @param speedy 
 * @returns 
 */
function createFakeWindow(globalContext: Window, speedy: boolean) {
  // map always has the fastest performance in has check scenario
  // Map 总是有最快的性能在检查场景
  // see https://jsperf.com/array-indexof-vs-set-has/23
  const propertiesWithGetter = new Map<PropertyKey, boolean>();
  const fakeWindow = {} as FakeWindow;

  /*
   将global的不可配置属性复制到fakeWindow 但是只是自身的属性，没有考虑到 prototype
   如果属性不作为目标对象的自有属性存在，或者作为目标对象的可配置自有属性存在，则不能将其报告为不可配置的属性。
   */
  Object.getOwnPropertyNames(globalContext)
    // 获取不可配置的属性
    .filter((p) => {
      // configurable ：当且仅当此属性描述符的类型可以更改且该属性可以从相应对象中删除时，为 true。
      const descriptor = Object.getOwnPropertyDescriptor(globalContext, p);
      return !descriptor?.configurable;
    })
    .forEach((p) => {
      const descriptor = Object.getOwnPropertyDescriptor(globalContext, p);
      // 如果存在 descriptor
      if (descriptor) {
        // 特殊值处理
        ...

        // 存在 getter 方法
        if (hasGetter) propertiesWithGetter.set(p, true);

        // 冻结描述符以避免被 zone.js 修改    rawObjectDefineProperty 就是 defineProperty
        rawObjectDefineProperty(fakeWindow, p, Object.freeze(descriptor));
      }
    });

  return {
    fakeWindow,
    propertiesWithGetter,
  };
}

```

#### Proxy


```JavaScript

const proxy = new Proxy(fakeWindow, {
  set: (target: FakeWindow, p: PropertyKey, value: any): boolean => {
    if (this.sandboxRunning) {

      // 如果是在 白名单里面，那么就直接赋值到全局属性里面
      // 否则，赋值到 target fakeWindow 里面，这里考虑到了 保存原有描述的情况
      // 将属性同步到 globalContext 
      if (typeof p === 'string' && globalVariableWhiteList.indexOf(p) !== -1) {
        this.globalWhitelistPrevDescriptor[p] = Object.getOwnPropertyDescriptor(globalContext, p);
        // @ts-ignore
        globalContext[p] = value;
      } else {
        // 当属性之前存在于 globalContext 中时，我们必须保留它的描述
        if (!target.hasOwnProperty(p) && globalContext.hasOwnProperty(p)) {
          const descriptor = Object.getOwnPropertyDescriptor(globalContext, p);
          const { writable, configurable, enumerable, set } = descriptor!;
          // 这里我们忽略了 globalContext的 accessor descriptor，
          // 因为触发它的逻辑没有意义(这可能会导致沙箱转义)，我们强制通过数据描述符设置值
          if (writable || set) {
            Object.defineProperty(target, p, { configurable, enumerable, writable: true, value });
          }
        } else {
          target[p] = value;
        }
      }

      // 维护一个更新的 Set
      updatedValueSet.add(p);

      return true;
    }

    // 在 strict-mode 下，Proxy 的 handler.set 返回 false 会抛出 TypeError，在沙箱卸载的情况下应该忽略错误
    return true;
  },

  get: (target: FakeWindow, p: PropertyKey): any => {

    // 特殊情况考虑
    ...

    // 存在 propertiesWithGetter 或者 不属于 target 里面 就会使用 globalContext 否则使用 target
    const actualTarget = propertiesWithGetter.has(p) ? globalContext : p in target ? target : globalContext;
    const value = actualTarget[p];

    // 冻结的值应该直接返回
    if (isPropertyFrozen(actualTarget, p)) {
      return value;
    }

    /*
    某些dom api必须绑定到本机窗口，否则会导致类似
    “TypeError: Failed to execute 'fetch' on' window ': Illegal invocation”的异常。 
        See this code:
          const proxy = new Proxy(window, {});
          const proxyFetch = fetch.bind(proxy);
          proxyFetch('https://qiankun.com');
    */
    const boundTarget = useNativeWindowForBindingsProps.get(p) ? nativeGlobal : globalContext;
    // 考虑的是方法的特殊情况需要 this 指向问题，正常情况还是 直接返回 value
    return getTargetValue(boundTarget, value);
  },

  has(target: FakeWindow, p: string | number | symbol): boolean {
    // property in cachedGlobalObjects must return true to avoid escape from get trap
    return p in cachedGlobalObjects || p in target || p in globalContext;
  },

  getOwnPropertyDescriptor(target: FakeWindow, p: string | number | symbol): PropertyDescriptor | undefined {
    if (target.hasOwnProperty(p)) {
      const descriptor = Object.getOwnPropertyDescriptor(target, p);
      descriptorTargetMap.set(p, 'target');
      return descriptor;
    }

    if (globalContext.hasOwnProperty(p)) {
      const descriptor = Object.getOwnPropertyDescriptor(globalContext, p);
      descriptorTargetMap.set(p, 'globalContext');
      if (descriptor && !descriptor.configurable) {
        descriptor.configurable = true;
      }
      return descriptor;
    }

    return undefined;
  },

  ownKeys(target: FakeWindow): ArrayLike<string | symbol> {
    return uniq(Reflect.ownKeys(globalContext).concat(Reflect.ownKeys(target)));
  },

  defineProperty: (target: Window, p: PropertyKey, attributes: PropertyDescriptor): boolean => {
    const from = descriptorTargetMap.get(p);

    switch (from) {
      case 'globalContext':
        return Reflect.defineProperty(globalContext, p, attributes);
      default:
        return Reflect.defineProperty(target, p, attributes);
    }
  },

  deleteProperty: (target: FakeWindow, p: string | number | symbol): boolean => {
    this.registerRunningApp(name, proxy);
    if (target.hasOwnProperty(p)) {
      // @ts-ignore
      delete target[p];
      updatedValueSet.delete(p);

      return true;
    }

    return true;
  },

  // 确保' window instanceof window '在微应用中返回true
  getPrototypeOf() {
    return Reflect.getPrototypeOf(globalContext);
  },
});

```



### SnapshotSandbox 基于 diff方式实现 沙箱 快照沙箱

这个是存在问题的。
虽然 active 记录快照，同时恢复变更没有问题
但是如果在 微应用 active 之后，新增了属性，就无法被记录
然后 在 inactive 中无法删除掉
同时没有时刻记录当前微应用沙箱的快照，所以存在很多缺陷

```JavaScript
/**
 * 基于 diff 方式实现的沙箱，用于不支持 Proxy 的低版本浏览器
 */
export default class SnapshotSandbox implements SandBox {
  proxy: WindowProxy;
  sandboxRunning = true;
  private windowSnapshot!: Window;
  private modifyPropsMap: Record<any, any> = {};

  constructor(name: string) {
    this.proxy = window;
    this.type = SandBoxType.Snapshot;
  }

  /**
   * 两步：
   * 记录快照
   * 恢复变更
   */
  active() {
    // 记录当前快照
    this.windowSnapshot = {} as Window;
    iter(window, (prop) => {
      this.windowSnapshot[prop] = window[prop];
    });

    // 恢复之前的变更
    Object.keys(this.modifyPropsMap).forEach((p: any) => {
      window[p] = this.modifyPropsMap[p];
    });

    this.sandboxRunning = true;
  }

  /**
   * 两步：
   * 记录变更
   * 恢复环境
   */
  inactive() {
    this.modifyPropsMap = {};

    iter(window, (prop) => {
      if (window[prop] !== this.windowSnapshot[prop]) {
        // 记录变更，恢复环境
        this.modifyPropsMap[prop] = window[prop];
        window[prop] = this.windowSnapshot[prop];
      }
    });

    this.sandboxRunning = false;
  }

  patchDocument(): void {}
}

```



## Deferred 类

是一个非常有意思的类，一开始还没有理解这个是有什么作用
后面看了别人的解说，发现这个是一个很有用的方法
他主要可以用来保证了 其他方法的执行完成，才能执行当前方法

Deferred 源码
```JavaScript

class Deferred {
  promise;
  resolve;
  reject;
  constructor() {
    this.promise = new Promise((resolve, reject) => {
      this.resolve = resolve;
      this.reject = reject;
    });
  }
}

```

案例
```JavaScript
const frameworkStartedDefer = new Deferred();

const start = () => {
  console.log(1);
  // 直到这个调用了 resolve 下面的才可以调用
  frameworkStartedDefer.resolve();
}

const b = async () => {
  console.log(2);
  // 这里会使用 await 保证了 defer 的等待，如果 这个实例没有调用 resolve 那么这个就会一直等待
  await frameworkStartedDefer.promise;
  console.log(3);
}

b();
start();

/**
2
1
3
 */
```
