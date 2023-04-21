---
title: Javascript中的函数节流与函数防抖
tags:
  - JS
  - 面试题
categories:
  - 前端
  - JavaScript
date: 2022-04-20 21:45:58
---
> 函数节流和函数防抖都是用来优化性能，以及避免短时间内连续调用某个函数的方案。我们通过以下两个例子，来理解两种方案，以及它们的应用场景。

# 函数节流

函数节流即为，一个函数执行一次后，只有大于设定的执行周期后，才会执行第二次。

这里我们可以理解为当一个函数立即执行后，它需要一个冷却时间才能被执行第二次，也就是我们需要去节制函数的调用次数，即为节流。

我们可以通过检测两次函数调用的时间差，如果在设定的函数冷却时间之内，则不能执行，如果在冷却时间之外则可以执行。通过函数节流可以优化Javascript的性能，防止一个函数被无差别的多次反复执行。

```javascript
// JS核心/01.js
/**
 * 函数节流
 * @param fn 要被节流的函数
 * @param delay 规定的时间（函数执行的冷却时间）
 */
function throttle(fn, delay) {
  var lastTime = 0;
  // 需要通过闭包来保存lastTime的状态，否则每次调用lastTime都会被初始化为0
  return function () {
    var nowTime = Date.now();
    if (nowTime - lastTime > delay) {
      fn.call(this); // 解决fn函数内this指向问题，如果不绑定this，函数的调用者为window（因为在这里执行函数函数前没有执行者），如果绑定了this，函数的this就指向了调用者本身
      lastTime = Date.now();
    }
  };
}

var fun = throttle(function () {
  console.log("触发了！");
}, 500);

fun();

setTimeout(function () {
  fun();
}, 400)

setTimeout(function () {
  fun();
}, 600)
```

输出结果:
```
触发了！
触发了！
```

# 函数防抖

函数防抖即为，一个需要频繁触发的函数，在规定时间内，只让最后一次生效，前面的不生效。

也就是说说一个方法将执行时，它会在一段时间内等待有没有事件第二次触发这个方法，如果有它就不执行了，如果没有才执行。

我们可以通过定时器，在方法第一次调用时，设置一个定时器，然后触发方法，假如在方法被触发前，该方法又被调用了，那在第二次调用前，会清除第一次调用方法而生成的定时器，重新再生成一个定时器去执行方法。

当我们页面上有一个按钮，希望用户在多次快速点击按钮时，仅触发一次按钮效果，我们就可以使用函数防抖机制，来避免用户在快速点击按钮时，连续触发多次方法。

```javascript
/**
 * 函数防抖
 * @param fn 添加防抖的函数
 * @param delay 防抖间隔时间
 */
function debounce(fn, delay) {
  // 记录上一次的延时器
  var timer = null;
  return function () {
    // 清除上一次的延时器
    clearTimeout(timer);
    // 获取传入方法内部的参数
    var args = Array.prototype.slice.apply(arguments);
    // 重新设定新的延时器
    timer = setTimeout(function () {
      fn.apply(this, args);
    }, delay);
  }
}

var fun = debounce(function (a, b) {
  console.log(a);
  console.log(b);
  console.log('触发了！');
}, 1000)

setTimeout(function () {
  fun()
}, 200)

setTimeout(function () {
  fun()
}, 300)

setTimeout(function () {
  fun()
}, 400)

setTimeout(function () {
  fun(111, 222)
}, 1402)
```

输出结果:
```
undefined
undefined
触发了！
111
222
触发了！
```

# 二者区别

函数节流是给函数执行设定一个冷却时间，函数被触发后在某固定一时间段内无法被触发第二次，它响应第一个触发者而忽略后面的触发者。

函数防抖是推迟了函数的执行，只响应后面的触发者，而抛弃前面的触发者，它的执行时间可以被无限推迟。
