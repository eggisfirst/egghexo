---
title: ES6 之 async/await
date: 2019-11-10 10:27:31
tags: [es6, 异步]
categories: 前端
cover: https://w.wallhaven.cc/full/j5/wallhaven-j5y3op.jpg
---
# async 和 await

async 和 await 是 ES2016 新增两个关键字，它们借鉴了 ES2015 中生成器在实际开发中的应用，目的是简化 Promise api 的使用，并非是替代 Promise。

## async

目的是简化在函数的返回值中对Promise的创建

async 用于修饰函数（无论是函数字面量还是函数表达式），放置在函数最开始的位置，被修饰函数的返回结果一定是 Promise 对象。

```js

async function test(){
    console.log(1);
    return 2;
}

//等效于
// return 等效于 resolve
function test(){
    return new Promise((resolve, reject)=>{
        console.log(1);
        resolve(2);
    })
}

```

## await

**await关键字必须出现在async函数中！！！！**

await用在某个表达式之前，如果表达式是一个Promise，则得到的是thenable中的状态数据。

```js

async function test1(){
    console.log(1);
    return 2;
}

async function test2(){
    const result = await test1();
    console.log(result);
}

test2();

//等效于
//await后面的语句全部是放在then里面

function test1(){
    return new Promise((resolve, reject)=>{
        console.log(1);
        resolve(2);
    })
}

function test2(){
    return new Promise((resolve, reject)=>{
        test1().then(data => {
            const result = data;
            console.log(result);
        })
    })
}

test2();

```

如果await的表达式不是Promise，则会将其使用Promise.resolve包装后按照规则运行
```js
async function test() {
  const result = await 1;
  console.log(result)
}
test();
//等效于
function test() {
  return  Promise.resolve(1).then(res => {
    const result = res
    console.log(result)
  })
}
test()
```
## 当有错误状态时
```js
async function getPromise() {
  if (Math.random() < 0.5) {
    return 1;
  } else {
    throw 2;
  }
}

async function test() {
  try {
    const result = await getPromise();
    console.log("正常状态", result)
  } catch (err) {
    console.log("错误状态", err);
  }
}
```

## 优化之前的练习
```js
//获取李华所在班级的老师的信息
//1. 获取李华的班级id   Promise
//2. 根据班级id获取李华所在班级的老师id   Promise
//3. 根据老师的id查询老师信息   Promise
async function findTeacher() {
  const stus = await ajax({
    url: "./data/students.json"
  })
  let cid;
  for (let i = 0; i < stus.length; i++) {
    if(stus[i].name === '李华') {
      cid = stus[i].classId
    }        
  }
  const clas = await ajax({
    url: "./data/classes.json"
  })
  let tid;
  for (let i = 0; i < clas.length; i++) {
    if(clas[i].id === cid) {
      tid = clas[i].teacherId
    }        
  }
  const teachers = await ajax({
    url: "./data/teachers.json"
  })
  let teacher;
  for (let i = 0; i < teachers.length; i++) {
    if(teachers[i].id === tid) {
      console.log(teachers[i])
    }        
  }
}

findTeacher()
```