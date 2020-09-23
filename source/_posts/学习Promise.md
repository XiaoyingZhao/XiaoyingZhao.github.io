---
title: 学习Promise
date: 2020-07-01 18:09:25
tags:
---

如果要弄懂promise，就必须弄懂什么是异步、什么是同步。
#### 什么是同步呢
你可以理解为同一个时间，你只能干一件事
```js
function second() {
    console.log('second')
}
function first(){
    console.log('first')
    second()
    console.log('Last')
}
first()
//first、second、last
```

##### 调用栈
```
当执行此代码时，将创建一个全局执行上下文并将其推到调用堆栈的顶部；// 这个不太重要，下面是重点
first()函数先上，现在他在顶部；
然后打印‘first’，然后执行完了，这个时候这个console.log会自动弹走，就是这个console.log虽然是后进来的，但是他先走了；
现在first函数仍然在顶部，他下面还有second函数，所以不会弹走；
执行second()函数,这时候second函数在顶部；
打印‘second’，然后执行完了，弹走这个console.log，这时候second在顶部；
这个时候second函数的事儿都干完了，他也弹走了，这时候first函数在顶部；
浏览器会问，first你还有事吗，first说我还有一个，执行打印‘last’

```
#### 什么是异步呢
```js
const getList = () => {
    setTimeout(() => {
        console.log('我执行了！');
    }, 2000);
};
console.log('Hello World');
getList();
console.log('哈哈哈');
//Hello World、哈哈哈、我执行了！（两秒以后执行最后一个）
```

##### 消息队列
刚才我们说了，同步的时候，浏览器会维护一个‘执行栈’，除了执行栈，在开启多线程的时候，浏览器还会维护一个消息列表，除了主线程，其余的都是副线程，这些副线程合起来就叫消息列表。 

#### 增加难度：
```js
setTimeout(function() {
    console.log('我是定时器！');
})
new Promise(function(resolve) {
    console.log('我是promise！');
    resolve();
}).then(function() {
    console.log('我是then！');
})
console.log('我是主线程！');

执行顺序：
我是promise！
我是主线程！
我是then！
我是定时器！
```

##### 事件轮询
上面我们说了，浏览器为了提升效率，为js开启了一个不太一样的多线程，因为js不能同时执行嘛，那副线程（注意是副线程里面哈）里面谁执行，这个选择的过程，就可以理解为事件轮询。我们先用事件轮询的顺序分析一下上面的代码，再来上概念：
```
promise函数肯定首先执行，他是主线程嘛，打印‘我是promise’；
然后继续走主线程，打印‘我是主线程’；
然后主线程走完了，开始走消息列表；
（宏任务和微任务一会再讲）
这个时候会先执行promise.then，因为他是微任务，里面的‘我是then！’
消息列表里面在上面的是定时器，但是定时器是宏任务，优先级比较低，所以会往后排；
```

##### 什么是宏任务？微任务？
```
 宏任务（Macrotasks）：js同步执行的代码块，setTimeout、setInterval、XMLHttprequest、setImmediate、I/O、UI rendering等。

微任务（Microtasks）：promise、process.nextTick（node环境）、Object.observe, MutationObserver等。

微任务比宏任务要牛逼一点

浏览器执行的顺序：
(1)执行主代码块，这个主代码块也是宏任务
(2)若遇到Promise，把then之后的内容放进微任务队列
(3)遇到setTimeout，把他放到宏任务里面
(4)一次宏任务执行完成，检查微任务队列有无任务 
(5)有的话执行所有微任务 
(6)执行完毕后，开始下一次宏任务。
```
下面用代码来深入理解上面的机制：

```js
setTimeout(function() {
    console.log('4')
})

new Promise(function(resolve) {
    console.log('1') // 同步任务
    resolve()
}).then(function() {
    console.log('3')
})
console.log('2')
```
这段代码作为宏任务，进入主线程。
先遇到setTimeout，那么将其回调函数注册后分发到宏任务Event Queue。
接下来遇到了Promise，new Promise立即执行，then函数分发到微任务Event Queue。
遇到console.log()，立即执行。
整体代码script作为第一个宏任务执行结束。查看当前有没有可执行的微任务，执行then的回调。 （第一轮事件循环结束了，我们开始第二轮循环。）
从宏任务Event Queue开始。我们发现了宏任务Event Queue中setTimeout对应的回调函数，立即执行。 执行结果：1 - 2 - 3 - 4


```js
console.log('1')
setTimeout(function() {
    console.log('2')
    process.nextTick(function() {
        console.log('3')
    })
    new Promise(function(resolve) {
        console.log('4')
        resolve()
    }).then(function() {
        console.log('5')
    })
})

process.nextTick(function() {
    console.log('6')
})

new Promise(function(resolve) {
    console.log('7')
    resolve()
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9')
    process.nextTick(function() {
        console.log('10')
    })
    new Promise(function(resolve) {
        console.log('11')
        resolve()
    }).then(function() {
        console.log('12')
    })
})
```
整体script作为第一个宏任务进入主线程，遇到console.log(1)输出1。
遇到setTimeout，其回调函数被分发到宏任务Event Queue中。我们暂且记为setTimeout1。
遇到process.nextTick()，其回调函数被分发到微任务Event Queue中。我们记为process1。
遇到Promise，new Promise直接执行，输出7。then被分发到微任务Event Queue中。我们记为then1。
又遇到了setTimeout，其回调函数被分发到宏任务Event Queue中，我们记为setTimeout2。
现在开始执行微任务，我们发现了process1和then1两个微任务，执行process1,输出6。执行then1，输出8。 第一轮事件循环正式结束，这一轮的结果是输出1，7，6，8。那么第二轮事件循环从setTimeout1宏任务开始：
首先输出2。接下来遇到了process.nextTick()，同样将其分发到微任务Event Queue中，记为process2。
new Promise立即执行输出4，then也分发到微任务Event Queue中，记为then2。
现在开始执行微任务，我们发现有process2和then2两个微任务可以执行输出3，5。 第二轮事件循环结束，第二轮输出2，4，3，5。第三轮事件循环从setTimeout2宏任务开始：
直接输出9，将process.nextTick()分发到微任务Event Queue中。记为process3。
直接执行new Promise，输出11。将then分发到微任务Event Queue中，记为then3。
执行两个微任务process3和then3。输出10。输出12。 第三轮事件循环结束，第三轮输出9，11，10，12。 整段代码，共进行了三次事件循环，完整的输出为1，7，6，8，2，4，3，5，9，11，10，12。 (请注意，node环境下的事件监听依赖libuv与前端环境不完全相同，输出顺序可能会有误差)