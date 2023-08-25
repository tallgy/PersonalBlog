---
title: import-html-entry
date: 2023-08-21 19:59:10
tags:
 - 微前端
 - qiankun
 - import-html-entry
categories:
 - 微前端
---

# import-html-entry

这个里面主要是一个 importEntry 方法
用于将 html 里面的内容 最终分割成 script 、 style html 三个部分
其中貌似有使用类似于 模块化+proxy 的方式处理了js的隔离问题。
css 只是将全部放入了 style 标签中

最终返回的结构是
```JavaScript
res = {
  /** 一份 style、html 内容被取出来 但是 script 没有取出来的string */
  template,
  /** public path */
  assetPublicPath: getPublicPath(entry),
  /** get script 方法、获取 script */
  getExternalScripts: () => getExternalScripts(scripts, fetch),
  /** get style 方法 */
  getExternalStyleSheets: () => getExternalStyleSheets(styles, fetch),
  /** 执行 script */
  execScripts: (proxy, strictGlobal, opts = {}) => {
    if (!scripts.length) {
      return Promise.resolve();
    }
    return execScripts(scripts[scripts.length - 1], scripts, proxy, {
      fetch,
      strictGlobal,
      ...opts,
    });
  },
}

```


## html

### 如果是一个 url string

是一个url，然后进行请求获取响应的文字，然后通过
1、获取数据
获取响应 fetch(url)
针对不同响应内容，比如 content-type charset 和流式数据流 来获取内容的文字 readResAsString(response, autoDecodeResponse)
2、获取不同属性
const { template, scripts, entry, styles } = processTpl(getTemplate(html), assetPublicPath, postProcessTemplate);

processTpl 方法 步骤
1、删除注释
2、link css 处理，加入 import-html-entry 的link注释（用于后面替换回来），加入 styles 数组
3、style css 处理, ignore 但是只处理 style ignore 的，不是很清楚为什么要这样。
4、处理 script 标签 同时 放入 scripts 数组里面，然后如果 entry 没有，那么就将最后一个 scripts 数组作为 entry

方法返回值
```JavaScript
let tplResult = {
  template,
  scripts,
  styles,
  // set the last script as entry if have not set
  entry: entry || scripts[scripts.length - 1],
};
```

### 如果是一个对象

通过这个解构，分成不同的属性
const { scripts = [], styles = [], html = '' } = entry;



## 然后调用 getEmbedHTML 方法

将 styles 替换回来 作为 embedHTML
然后通过 then 方法返回数据

```JavaScript

getEmbedHTML(template, styles, { fetch }).then(embedHTML => ({
  // 将 styles 替换回来了的 html
  template: embedHTML,
  // publicPath
  assetPublicPath,
  // 获取 scripts Promise.all(scripts).text 将 script 文字返回
  getExternalScripts: () => getExternalScripts(scripts, fetch),
  // 获取 styles 同上
  getExternalStyleSheets: () => getExternalStyleSheets(styles, fetch),
  // 执行 scripts 代码 里面使用 eval
  execScripts: (proxy, strictGlobal, opts = {}) => {
    if (!scripts.length) {
      return Promise.resolve();
    }
    return execScripts(entry, scripts, proxy, {
      fetch,
      strictGlobal,
      ...opts,
    });
  },
}))

```

### execScripts 方法

执行 scripts 将 proxy 作为这个 window 进行执行

```JavaScript

export function execScripts(entry, scripts, proxy = window, opts = {}) {
	const {
		fetch = defaultFetch, strictGlobal = false, success, error = () => {
		}, beforeExec = () => {
		}, afterExec = () => {
		},
		scopedGlobalVariables = [],
	} = opts;

	return getExternalScripts(scripts, fetch, error)
		.then(scriptsText => {

			const geval = (scriptSrc, inlineScript) => {
        // beforeExec 和 afterExec 有点像是一个回调函数
				const rawCode = beforeExec(inlineScript, scriptSrc) || inlineScript;

        // 获取 代码，同时也进行了 function 封装。
				const code = getExecutableScript(scriptSrc, rawCode, { proxy, strictGlobal, scopedGlobalVariables });

        // 类似于 eval 只是外层加了一个 function 包裹
				evalCode(scriptSrc, code);

				afterExec(inlineScript, scriptSrc);
			};

			function exec(scriptSrc, inlineScript, resolve) {

        // 如果 src === entry 如果在没有 entry 的情况下，这个值为 最后一个 src
				if (scriptSrc === entry) {
          // 更新 第一个、第二个 和 最后一个 数据
					noteGlobalProps(strictGlobal ? proxy : window);
            
          geval(scriptSrc, inlineScript);
          // 其实这个 getGlobalProp 方法还是有点没有理解的。但是里面有用到 iframe 
          const exports = proxy[getGlobalProp(strictGlobal ? proxy : window)] || {};
          resolve(exports);
          
				} else {
          geval(scriptSrc, inlineScript)
				}
			}

			function schedule(i, resolvePromise) {
        // 调用 exec 方法执行对应的 scripts[i]
        exec(scripts[i], scriptsText[i], resolvePromise)
        // 然后递归 
        schedule(i+1, resolvePromise)
        // 如果结束了，且 没有 entry 那么就调用 ，否则应该是给 内部 exec 方法调用
        resolvePromise()
			}

			return new Promise(resolve => schedule(0, success || resolve));
		});
}

```

#### getExecutableScript

拼接成字符串

通过查看可以知道 先使用 bind 将 this 指向 window.proxy
然后将 window.proxy 参数都作为参数传递
所拼接的字符串
;(function(window, self, globalThis) {
  ;${scriptText}\n
  ${sourceUrl}
}).bind(window.proxy)
(window.proxy, window.proxy, window.proxy);

发现的问题
首先 window.proxy 并不是 Proxy
其次发现，如果直接查看 window.proxy 是 undefined
但是在方法里面 使用 call 方法 输出 this 的 window.proxy 就是 window  

```JavaScript

/**
 * 生成一个代码字符串，类似于模块化一样将方法包裹一层。
 * @param {*} scriptSrc 
 * @param {*} scriptText 
 * @param {*} opts 
 * @returns 
 */
function getExecutableScript(scriptSrc, scriptText, opts = {}) {
	const { proxy, strictGlobal, scopedGlobalVariables = [] } = opts;

	const sourceUrl = isInlineCode(scriptSrc) ? '' : `//# sourceURL=${scriptSrc}\n`;

	// 将 scopedGlobalVariables 拼接成变量声明，用于缓存全局变量，避免每次使用时都走一遍代理
	const scopedGlobalVariableDefinition = scopedGlobalVariables.length ? `const {${scopedGlobalVariables.join(',')}}=this;` : '';

	// 通过这种方式获取全局 window ，因为 script 也是在全局作用域下运行的，所以我们通过 window.proxy 绑定时也必须确保绑定到全局 window 上
	// 否则在嵌套场景下， window.proxy 设置的是内层应用的 window ，而代码其实是在全局作用域运行的，会导致闭包里的 window.proxy 取的是最外层的微应用的 proxy
	const globalWindow = (0, eval)('window');
	globalWindow.proxy = proxy;
	// TODO 通过 strictGlobal 方式切换 with 闭包，待 with 方式坑趟平后再合并
	return strictGlobal
		? (
			scopedGlobalVariableDefinition
				? `;(function(){with(this){${scopedGlobalVariableDefinition}${scriptText}\n${sourceUrl}}}).bind(window.proxy)();`
				: `;(function(window, self, globalThis){with(window){;${scriptText}\n${sourceUrl}}}).bind(window.proxy)(window.proxy, window.proxy, window.proxy);`
		)
		: `;(function(window, self, globalThis){;${scriptText}\n${sourceUrl}}).bind(window.proxy)(window.proxy, window.proxy, window.proxy);`;
}

```
