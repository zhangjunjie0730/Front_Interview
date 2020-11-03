# 前端面试题目汇总

## 目录

- [手写代码系列](#手写代码系列)
  - [实现一个 `JSON.parse()` 和 `JSON.stringify()`](#1-实现一个-JSON.parse()-和-JSON.stringify())
  - [防抖节流原理并用 js 实现](#2-防抖节流原理并用-js-实现)
  - [Promise函数](./Javascript/Promise.md)





## 手写代码系列

### 1. 实现一个 `JSON.parse()` 和 `JSON.stringify()`

####  `JSON.parse()` 

**核心：**用到 `eval("(" + json + ")") ` 实现对 json 的解析。但是 `eval()` 是个危险的函数。

```js
function _parse(jsonStr){
  return new Function("return" + jsonStr)
}
```



### 2. 防抖节流原理并用 js 实现

#### 1. 防抖 debounce

- 原理：事件被触发 n 秒后才能再一次执行回调，如果 n 秒内被再次触发，则重新计时。
- 适用场景：按钮提交；搜索框联想场景（只发送最后一次输入的值）。
- 代码

1. 简易版本，执行一次，n秒后才能执行下一次。**问题：timeout 函数执行是先等待 wait 秒再执行。第一次也会有等待时间。**

   ```js
   function _debounce(func, wait) {
     let timeout;
     // return的是一个未执行的函数, 所以执行的时候才知道this是指向执行者的。
     return function () {
       const context = this; // 把func中的this指向改为_debounce的调用者.
       const args = arguments; // 同样把参数变成_debounce的调用者.例如要用到event参数
       clearTimeout(timeout);
       timeout = setTimeout(func.apply(context, args), wait);
     };
   }
   ```

   

2. 改进，传入 immediate，表示可以先立即执行。默认也是立即执行

   ```js
   function _debouncce(func, wait, immediate) {
     let timeout;
     // return的是一个未执行的函数, 所以执行的时候才知道this是指向执行者的。
     return function () {
       const context = this; // 把func中的this指向改为_debounce的调用者.
       const args = arguments; // 同样把参数变成_debounce的调用者.例如要用到event参数
       if(timeout) clearTimeout(timeout);
       if (immediate) {
         // 立即执行
         const callNow = !timeout; // 让执行和timeout挂钩，如果timeout存在说明执行过，就不调用立即执行。如果没有timeout就立即执行
         timeout = setTimeout(function () {
           timeout = null;
         }, wait); // wait时间内，timeout都有值，所以使得callNow在这段时间内都是false，就不执行函数；如果超过wait时间，timout为false，此时执行。
         if (callNow) func.apply(context, args);
       } else {
         // 不会立即执行
         timeout = setTimeout(func.apply(context, args), wait);
       }
     };
   }
   ```

3. 带返回值以及可以取消防抖。

   ```js
   function _debounce(func, wait, immediate) {
     let timeout, result;
     let debounced = function () {
       const context = this;
       const args = arguments;
       if (timeout) clearTimeout(timeout);
       if (immediate) {
         const callNow = !timeout;
         timeout = setTimeout(function () {
           timeout = null;
         }, wait);
         if (callNow) result = func.apply(context, args);
       } else {
         timeout = setTimeout(func.apply(context, args), wait);
       }
       return result;
     };
     // 取消防抖操作，用对象形式赋予方法 → 实际应用场景：用户觉得等待时间太长，取消提交操作
     debounced.cancel = function () {
       clearTimeout(timeout);
       timeout = null;
     };
     return debounced;
   }
   ```



#### 2. 节流 throttle

原理：规定时间内，只能触发一次函数。即使时间内多次触发函数，但只生效一次。注意要保证，首次进入时立即执行。

- 利用时间戳实现 —— 第一次进入立即触发，但是离开之后不会执行最后一次（未满wait最后不触发）

  ```js
  function _throttle(func, wait) {
    let context, args;
    let previous = 0; // 上一次的时间戳
    return function () {
      let now = +new Date(); // 获取当前时间戳 或者new Date().valueOf()
      context = this;
      args = arguments;
      if (now - previous > wait) {
        func.apply(context, args);
        previous = now;
      }
    };
  }
  ```

   

- 利用定时器实现

  ```js
  // 第一次进入不触发，最后一次离开会触发 没头有尾
  function _throttle(func, wait) {
    let timeout;
    return function () {
      const context = this;
      const args = arguments;
      if (!timeout) {
        timeout = setTimeout(function () {
          timeout = null;
          func.apply(context, args);
        }, wait);
      }
    };
  }
  ```

  



































