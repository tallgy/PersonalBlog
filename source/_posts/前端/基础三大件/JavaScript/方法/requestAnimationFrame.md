---
title: requestAnimationFrame
date: 2023-07-05 15:24:18
tags:
categories:
---

# requestAnimationFrame

## 定时器

定时器是通过给一个间隔时间去处理的，但是首先，因为 JavaScript 的是一个单线程的语言，所以，JavaScript 其实是内部有一个 事件循环机制 去执行这个 定时器的， 那么会有一个问题就是，如果我们的定时器的时间已经到了，但是因为本来的事件没有执行完成，那么就不会被让开，所以定时器执行的时间并不是说准确在那个时间点的。

## 特点

正常的定时器的时间是固定下来的，而这个的特点在于，我们的执行时间是由浏览器来决定调用的执行时机，可以避免出现页面执行 JavaScript 代码被卡住的问题。

## 注意

本质来说，他并不会说，我如果在下一次页面渲染的时候主动让出线程，只是说，执行的时间可以在渲染之后，不会影响正常的渲染。



# 下面的结论 应该是错误的，但是没有找到比较好的说明，通过自己的代码发现， requestAnimationFrame 内如果执行大型代码还是会造成卡顿。

## requestAnimationFrame 为什么不会掉帧

```
	首先，我们可以知道布局绘制和js执行都是在主线程进行的执行的，
	所以，如果当js拿到主线程执行权后，有可能会出现执行时间过长，导致在下一帧的开始js没有及时归还主线程，导致下一帧没有按时渲染，于是就会出现掉帧，而这个requestAnimationFrame方法就是解决这个问题的，
	他会在每一帧被调用，通过回调函数，会把js任务分成更小的任务快，并且会在每一帧时间用完暂停js的执行，归还主线程，所以布局绘制就会在每帧及时拿到主线程的使用权，所以不会掉帧。
```

```
requestAnimationFrame 最大的优势是由系统来决定回调函数的执行时机。
「requestAnimationFrame的步伐跟着系统的刷新步伐走」，
「它能保证回调函数在屏幕每一次的刷新间隔中只被执行一次」，
「这样就不会引起丢帧现象，也不会导致动画出现卡顿的问题」。
```

```
requestAnimationFrame还有以下两个优势：

「CPU节能」：
		使用setTimeout实现的动画，当页面被隐藏或最小化时，setTimeout 仍然在后台执行动画任务，由于此时页面处于不可见或不可用状态，刷新动画是没有意义的，完全是浪费CPU资源。而requestAnimationFrame则完全不同，当页面处理未激活的状态下，该页面的屏幕刷新任务也会被系统暂停，因此跟着系统步伐走的requestAnimationFrame也会停止渲染，当页面被激活时，动画就从上次停留的地方继续执行，有效节省了CPU开销。

「函数节流」：
		在高频率事件(resize,scroll等)中，为了防止在一个刷新间隔内发生多次函数执行，使用requestAnimationFrame可保证每个刷新间隔内，函数只被执行一次，这样既能保证流畅性，也能更好的节省函数执行的开销。一个刷新间隔内函数执行多次时没有意义的，因为显示器每16.7ms刷新一次，多次绘制并不会在屏幕上体现出来。
```

下面为《HTML5 Canvas 核心技术》给出的兼容主流浏览器的 requestNextAnimationFrame 和 cancelNextRequestAnimationFrame 方法

```
window.requestNextAnimationFrame = (function () {
    var originalRequestAnimationFrame = undefined,
        wrapper = undefined,
        callback = undefined,
        geckoVersion = null,
        userAgent = navigator.userAgent,
        index = 0,
        self = this;

    wrapper = function (time) {
        time = performance.now();
        self.callback(time);
    };

    /*!
     bug!
     below code:
     when invoke b after 1s, will only invoke b, not invoke a!

     function a(time){
     console.log("a", time);
     webkitRequestAnimationFrame(a);
     }

     function b(time){
     console.log("b", time);
     webkitRequestAnimationFrame(b);
     }

     a();

     setTimeout(b, 1000);


     so use requestAnimationFrame priority!
     */
    if(window.requestAnimationFrame) {
        return requestAnimationFrame;
    }


    // Workaround for Chrome 10 bug where Chrome
    // does not pass the time to the animation function

    if (window.webkitRequestAnimationFrame) {
        // Define the wrapper

        // Make the switch

        originalRequestAnimationFrame = window.webkitRequestAnimationFrame;

        window.webkitRequestAnimationFrame = function (callback, element) {
            self.callback = callback;

            // Browser calls the wrapper and wrapper calls the callback

            return originalRequestAnimationFrame(wrapper, element);
        }
    }

    //修改time参数
    if (window.msRequestAnimationFrame) {
        originalRequestAnimationFrame = window.msRequestAnimationFrame;

        window.msRequestAnimationFrame = function (callback) {
            self.callback = callback;

            return originalRequestAnimationFrame(wrapper);
        }
    }

    // Workaround for Gecko 2.0, which has a bug in
    // mozRequestAnimationFrame() that restricts animations
    // to 30-40 fps.

    if (window.mozRequestAnimationFrame) {
        // Check the Gecko version. Gecko is used by browsers
        // other than Firefox. Gecko 2.0 corresponds to
        // Firefox 4.0.

        index = userAgent.indexOf('rv:');

        if (userAgent.indexOf('Gecko') != -1) {
            geckoVersion = userAgent.substr(index + 3, 3);

            if (geckoVersion === '2.0') {
                // Forces the return statement to fall through
                // to the setTimeout() function.

                window.mozRequestAnimationFrame = undefined;
            }
        }
    }

    return window.webkitRequestAnimationFrame ||
        window.mozRequestAnimationFrame ||
        window.oRequestAnimationFrame ||
        window.msRequestAnimationFrame ||

        function (callback, element) {
            var start,
                finish;

            window.setTimeout(function () {
                start = performance.now();
                callback(start);
                finish = performance.now();

                self.timeout = 1000 / 60 - (finish - start);

            }, self.timeout);
        };
})();


    window.cancelNextRequestAnimationFrame = window.cancelRequestAnimationFrame
        || window.webkitCancelAnimationFrame
        || window.webkitCancelRequestAnimationFrame
        || window.mozCancelRequestAnimationFrame
        || window.oCancelRequestAnimationFrame
        || window.msCancelRequestAnimationFrame
        || clearTimeout;
```

