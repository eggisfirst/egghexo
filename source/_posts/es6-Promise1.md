---
title: 'ES6 之 Promise（一）'
date: 2019-11-5 19:14:50
tags: [es6,异步]
categories: 前端
cover:  https://w.wallhaven.cc/full/p8/wallhaven-p8523p.jpg
---

## 前提
我们习惯于使用传统的回调或事件处理来解决异步问题，但是随着前端工程越来越复杂，该模式面临以下两个问题：
1. 回调地狱：某个异步操作需要等待之前的异步操作完成，无论用回调还是事件，都会陷入不断的嵌套
```js
//小A心中有三个女神
//有一天，小A决定向第一个女神表白，如果女神拒绝，则向第二个女神表白，
//直到所有的女神都拒绝，或有一个女神同意为止
function biaobai(god, callback) {
    console.log(`小A向女神【${god}】发出了表白短信`);
    setTimeout(() => {
        if (Math.random() < 0.1) {
            callback(true);
        } else {
            callback(false);
        }
    }, 1000);
}

biaobai("女神1", function(result) {
    if (result) {
        console.log("女神1答应了，小A很开心!")
    } else {
        console.log("女神1拒绝了，小A表示无压力，然后向女神2表白");
        biaobai("女神2", function(result) {
            if (result) {
                console.log("女神2答应了，小A很开心!")
            } else {
                console.log("女神2十分感动，然后拒绝了小A，小A向女神3表白");
                biaobai("女神3", function(result) {
                    if (result) {
                        console.log("女神3答应了，小A很开心!")
                    } else {
                        console.log("小A表示生无可恋!!");
                    }
                })
            }
        })
    }
})
```

2. 异步之间的联系：某个异步操作要等待多个异步操作的结果，对这种联系的处理，会让代码的复杂度剧增
```js
//当所有的女神回复完成后，他要把所有的回复都记录到日志进行分析
//因为回复是异步的，所以不能放到函数末尾执行。
function biaobai(god, callback) {
    console.log(`小A向女神【${god}】发出了表白短信`);
    setTimeout(() => {
        if (Math.random() < 0.05) {
            callback(true);
        } else {
            callback(false);
        }
    }, Math.floor(Math.random() * (3000 - 1000) + 1000)); //模拟异步，每个女神回复的时间不同
}

let agreeGod = null; //同意小A的第一个女神 （只能建立一个全局变量）
const results = []; //用于记录回复结果的数组
for (let i = 1; i <= 20; i++) {
    biaobai(`女神${i}`, result => {
        results.push(result);  //每次向数组添加一个女神
        if (result) {
            console.log(`女神${i}同意了`)
            if (agreeGod) {
                console.log(`小A回复女神${i}: 不好意思，刚才朋友用我手机，乱发的`)
            } else {
                agreeGod = `女神${i}`;
                console.log(`小A终于找到了真爱`);
            }
        } else {
            console.log(`女神${i}拒绝了`)
        }
        if (results.length === 20) {
            console.log("日志记录", results) //终于得到所有的日志
        }
    })
}
//记录日志是一个单独的功能，这样穿插到代码中，会使代码易读性变差，并且日后添加其他模块的时候不好扩展或维护
```
ES官方参考了大量的异步场景，总结出了一套异步的通用模型，该模型可以覆盖几乎所有的异步场景，甚至是同步场景，也就是Promise。

## 通用模型
![Promise](es6-Promise/1.png)

### Promise的基本使用
```js
const pro = new Promise((resolve, reject)=>{
    // 未决阶段的处理（同步，立即处理）
    // 通过调用resolve函数将Promise推向已决阶段的resolved状态或者调用reject函数将Promise推向已决阶段的rejected状态
    // resolve和reject均可以传递最多一个参数，表示推向状态的数据
    // resolve和reject均不可逆，resolve后再reject，reject不会执行
})
//第一种写法
pro.then(data=>{
    //这是thenable函数，如果当前的Promise已经是resolved状态，该函数会立即执行
    //如果当前是未决阶段，则会加入到作业队列，等待到达resolved状态后执行
    //data为传出来的数据
}, err=>{
    //这是catchable函数，如果当前的Promise已经是rejected状态，该函数会立即执行
    //如果当前是未决阶段，则会加入到作业队列，等待到达rejected状态后执行
})
//第二种写法
pro.then(data=>{
    //同上
}).catch(err=>{
  //同上
})
```
使用promise后：
```js
function biaobai(god) {
  return new Promise(resolve => {
    console.log(`小A向${god}发出了表白短信`);
    setTimeout(() => {
        if (Math.random() < 0.1) {
            resolve(true) //不管是同意还是拒绝都是resolve
        } else {
            resolve(false);
        }
    }, 3000);
  })
}

biaobai("女神1").then(result => {
  console.log(result);
})
```


## 细节
1. 未决阶段的处理函数是同步的，会立即执行
```js
const pro = new Promise((res, rej) => {
  console.log('未决阶段') //立即执行  
  res('data') 
})
```

2. thenable和catchable函数是异步的，就算是立即执行，也会加入到事件队列中等待执行，并且，加入的队列是微队列
```js
const pro = new Promise((res, rej) => {
  res(1) 
})
pro.then(res => {
    console.log(res) //异步，在微队列里
})
console.log(2) //输出 2 1
```

3. pro.then可以只添加thenable函数，pro.catch可以单独添加catchable函数
4. 在未决阶段的处理函数中，如果发生未捕获的错误，会将状态推向rejected，并会被catchable捕获
```js
const pro = new Promise((res, rej) => {
  throw new Error('ERROR') //可以把状态推向rejected
  res('data') //不会执行
})

pro.catch(err => {
  console.log(err) //可以捕获到错误信息
})
```

5. 一旦状态推向了已决阶段，无法再对状态做任何更改
```js
const pro = new Promise((res, rej) => {
  res(1) //已经推向已决阶段
  res(2)//不会执行
  rej(3)//不会执行
})

```


6. **Promise并没有消除回调，只是让回调变得可控**

如何处理回调地狱请看下一篇 ES6 之 Promise（二）

