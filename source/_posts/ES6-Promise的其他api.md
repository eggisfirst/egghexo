---
title: 'ES6 之 Promise的其他api'
date: 2019-11-7 08:37:32
tags: [es6,异步]
categories: 前端
cover: https://w.wallhaven.cc/full/8x/wallhaven-8xlz51.jpg
---
# Promise的其他api

## 原型成员（实例成员）

- **then**：注册一个后续处理函数，当Promise为resolved状态时运行该函数
- **catch**：注册一个后续处理函数，当Promise为rejected状态时运行该函数
- finally：[ES2018]注册一个后续处理函数（无参），当Promise为已决时运行该函数（无论是resoleved或者rejected）
```js
  const pro = new Promise((res,rej) => {
    res(1)
  })
  pro.finally(() => {console.log('finally1')}) 
  pro.finally(() => {console.log('finally2')})
  pro.then((res) => {console.log('then',res)})
  // finally1 finally2 then,1
```

## 构造函数成员（静态成员）

- **resolve(数据)**：该方法返回一个resolved状态的Promise，传递的数据作为状态数据
  - 特殊情况：如果传递的数据是Promise，则直接返回传递的Promise对象
```js
const pro = new Promise((res,rej) => {
  res(1)
})
pro.then(res => {
  console.log('then',res)
})
//如果promise里面只有res没有其他操作可以等效于下面的写法
const pro1 =  Promise.resolve(2)
pro1.then(res => {
  console.log('then1',res)
})
```
```js
//特殊情况
const pro = new Promise((res,rej) => {
  res(1)
})
const p = Promise.resolve(1)

console.log(pro === p) //true
```

- **reject(数据)**：该方法返回一个rejected状态的Promise，传递的数据作为状态数据
```js
//同上
const pro1 =  Promise.reject(3)
pro1.catch(err => {
  console.log('catch',err)
})

```

- **all(iterable)**：这个方法返回一个新的promise对象，该promise对象在iterable参数对象里所有的promise对象都成功的时候才会触发成功，一旦有任何一个iterable里面的promise对象失败则立即触发该promise对象的失败。这个新的promise对象在触发成功状态以后，会把一个包含iterable里所有promise返回值的数组作为成功回调的返回值，顺序跟iterable的顺序保持一致；如果这个新的promise对象触发了失败状态，它会把iterable里第一个触发失败的promise对象的错误信息作为它的失败错误信息。Promise.all方法常被用于处理多个promise对象的状态集合。
```js
const proms = []
for (let i = 0; i < 10; i++) {
  const pro = new Promise((res,rej) => {
    setTimeout(() => {
      if(Math.random() < 0.1) {
        console.log(i,'success')
        res(i)
      }else {
        console.log(i,'fail')
        rej(i)
      }
    }, Math.floor(Math.random(3000 - 1000) + 1000));
  })    
  proms.push(pro)  
}
const pros = Promise.all(proms)
pros.then(res => {
  console.log(res,'都成功了')  //全部成功才会返回都成功，res为每一个pro的返回值组成的数组。
})
pros.catch(err => {
  console.log(err,'有失败的') //有一个失败即返回有失败的，err为第一个失败的返回值
})
```


- **race(iterable)**：当iterable参数里的任意一个子promise被成功或失败后，父promise马上也会用子promise的成功返回值或失败详情作为参数调用父promise绑定的相应句柄，并返回该promise对象
```js
const proms = []
for (let i = 0; i < 10; i++) {
  const pro = new Promise((res,rej) => {
    setTimeout(() => {
      if(Math.random() < 0.1) {
        console.log(i,'success')
        res(i)
      }else {
        console.log(i,'fail')
        rej(i)
      }
    }, Math.floor(Math.random(3000 - 1000) + 1000));
  })    
  proms.push(pro)  
}
const pros = Promise.race(proms)
pros.then(res => {
  console.log(res,'有一个成功了')  //有一个成功就会返回成功
})
pros.catch(err => {
  console.log(err,'有一个失败了') //有一个失败即返回失败
})
```

## 处理异步之间的联系
- 有一个需求需要其他异步全部处理完之后做的
```js
function biaobai(god) {   //异步发送短信
  return new Promise((res, rej) => {
    setTimeout(() => {
      console.log(`向女神${god}发送短信`)
      if (Math.random() < 0.3) {
        console.log(`${god}同意`)
        res(true)
      }
      else {
        console.log(`${god}拒绝`)
        res(false)
      }
    }, Math.floor(Math.random(3000 - 1000) + 1000));
  })
}

let hasAgree = false  //全局变量记录第一个同意的女神
const proms = []     
for (let i = 0; i < 10; i++) {
  const pro = biaobai(`女神${i}`).then(res => {
    if (res) {
      if (hasAgree) {
        console.log('发错了短信')   //已经有同意的女神
      } else {
        hasAgree = true
        console.log(`太开心了`)   //第一个同意的女神
      }
    }
    return res                //返回resolved状态的promise
  })  
  proms.push(pro)               //把所有的promise装入一个数组
}

const pros = Promise.all(proms)

pros.then(res => {        //所有的promise都到了resolve状态后打印日志
  console.log('日志', res)
})
```