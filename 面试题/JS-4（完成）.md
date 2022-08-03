# JS重定向与路由

## Location方式

```js
location.reload()	   //重载
location.assign(url)   //跳转
location.replace(url)  //替换
```

## History方式

```js
history.go(number)   // 前进指定记录栈
history.back() 		 // 后退一个记录栈
history.forward()	 // 前进一个记录栈
history.pushState(stateObj, title[, url]);    // 向当前浏览器会话的历史堆栈中添加一个状态
history.replaceState(stateObj, title[, url]); // 替换当前浏览器会话的历史堆栈中的当前状态
```

## 监听

```js
//监听hash变化
window.onhashchange = callback
//监听state变化
window.onpopstate = callback
```

## 区别

# 框架的路由实现

