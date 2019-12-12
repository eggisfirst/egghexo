---
title: 'ES6 之 Promise（二）'
date: 2019-11-6 19:00:27
tags: [es6, 异步]
categories: 前端
cover: https://w.wallhaven.cc/full/lm/wallhaven-lmkkr2.jpg
---

## Promise的串联

当后续的Promise需要用到之前的Promise的处理结果时，需要Promise的串联。

Promise对象中，无论是then方法还是catch方法，它们都具有返回值，返回的是一个**全新的Promise对象**，它的状态满足下面的规则：

1. 如果当前的Promise是未决的，得到的新的Promise是挂起状态（异步：例如ajax请求、定时器）

```js
const pro1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(2);
  }, 3000);
})

console.log(pro1) //pendding 

pro1.then(result => {
  console.log(pro1,"结果出来了，得到的是一个Promise") //resolved
})
```

2. 如果当前的Promise是已决的，会运行响应的后续处理函数，并将后续处理函数的结果（返回值）作为resolved状态数据，应用到新的Promise中；如果后续处理函数发生错误，则把返回值作为rejected状态数据，应用到新的Promise中。

```js
const pro1 = new Promise((resolve, reject) => {
  resolve(1);
})

const pro2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(2); //把resolved作为状态数据到新的promise的数据中
  }, 3000);
})

pro1.then(result => {
  console.log("结果出来了，得到的是一个Promise",result) //pro1到resolved阶段，result为1
  return pro2;  
}).then(result => {
  console.log(pro2,result) //相当于pro2.then(result=>{}) pro2是已决状态，result为2
}).then(result => {
  console.log(result) //因为pro2没有return,相当于return undefined //输出undefined
})
```
```js
const pro1 = new Promise((resolve, reject) => {
  throw 1; //推向rejected状态
})

const pro2 = pro1.then(result => {
  return result * 2
}, err => {
  return err * 3;  //1*3 返回3 因为并没有报错，所以是作为下一个promise对象的resolved状态数据
});                 //如果是throw err*3 则是作为下一个promise对象的rejected状态数据

pro1.catch(err => {  //返回的是一个新的promise对象，跟pro2没关系
  return err * 2;
})

console.log(pro2) //pendding
//pro2类型：Promise对象
//pro2的状态：
pro2.then(result => console.log(pro2,result * 2), err => console.log(err * 3))

//输出：resolved,6
```

**后续的Promise一定会等到前面的Promise有了后续处理结果后，才会变成已决状态**
```js
const pro1 = new Promise((resolve, reject) => {
  resolve(1)
})

const pro2 = pro1.then(result => {
  return result * 2  //只要当pro1的这句话执行完后pro2的状态才会变成已决
}, err => {
  return err * 3;
});

```

## 解决回调地狱问题
```js
function biaobai(god) {
  return new Promise((resolve, reject) => {
    console.log(`向${god}发送短信`)
    setTimeout(() => {
      if (Math.random() < 0.1) {
        resolve(true)
      } else {
        resolve(false)
      }
    },500);
  })
}

let gods = ['女神1', '女神2']
let pro = null

for (let index = 0; index < gods.length; index++) {
  if (index === 0) {
    pro = biaobai(`${gods[0]}`)
  }

  pro = pro.then(res => {
    if (res === undefined) { //表示同意后或者最后一次都是return undefined
      return //return undefined
    }
    else if (res) {
      console.log(`${gods[index]}同意了`)
      return //return undefined
    }
    else {
      console.log(`${gods[index]}拒绝了`)
      if (index < gods.length - 1) {
        return biaobai(`${gods[index + 1]}`)
      }
    }
  })
}
```