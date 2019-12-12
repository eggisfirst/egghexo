---
title: 'ES6 之 块级绑定'
date: 2019-11-3 18:24:40
tags: es6
categories: 前端
cover:  https://w.wallhaven.cc/full/42/wallhaven-42j966.jpg
---

## 用var声明变量的问题
1. 允许重复的变量声明: 导致数据被覆盖
```js
var a = 1
var a = 2 //后者覆盖前者
```

2. 变量提升: 怪异的数据访问、闭包问题
```js
//怪异的数据访问
if (Math.random() < 0.5) {
    var a = "abc";
    console.log(a);
}
else {
    console.log(a); //没有声明a，但是不会报错  undefined
}

console.log(a); //没有声明a，但是不会报错  undefined


//闭包问题
var div = document.getElementById("divButtons")

for (var i = 1; i <= 10; i++) {
    var btn = document.createElement("button");
    btn.innerHTML = "按钮" + i;
    div.appendChild(btn);
    btn.onclick = function () {
        console.log(i); //输出11
    }
}

// 循环结束后，i：11  
// 因为var变量提升，每次var i = i，相当于把全局的i都覆盖了，而点击事件是异步的，点击的时候i全部为你11了
```



3. 全局变量挂载到全局对象: 全局对象成员污染问题
```js
var console = "abc";
console.log(console) 
//var的时候相当于在window上添加属性，所以如果window有这个属性，则会被覆盖。

```

es6不仅引入let、const关键字用于解决变量声明的问题，同时引入了块级作用域的概念。
> 块级作用域： 代码执行时遇到花括号，会创建一个块级作用域，花括号结束，销毁块级作用域。（if 或 for的花括号）

## 使用let声明变量
1. let声明的变量不会挂载到全局对象，解决全局对象被污染问题
```js
let a = 123;
console.log(window.a)  //undefined
```
   
2. let声明的变量，不允许当前作用域范围内重复声明，解决变量被覆盖问题
```js
let a = 123;
let a = 456; 
// ' Identifier 'b' has already been declared' 
//检查到，当前作用域（全局作用域）已声明了变量a
```


3. let声明的变量不会变量提升：解决怪异的数据访问、闭包问题
```js
if (Math.random() < 0.5) {
    let a = 123; //定义在当前块级作用域
    console.log(a) //当前块级作用域中的a
}
else {
    //这是另外一个块级作用域，该作用域中找不到a
    console.log(a)  //a is not defined
}

console.log(a); //a is not defined

//闭包
let div = document.getElementById("divButtons");
for (let i = 1; i <= 10; i++) {  //每次let都是创建一个新的块级作用域，里面i不会被覆盖。
    let button = document.createElement("button")
    button.innerHTML = "按钮" + i;
    button.onclick = function () {
        console.log(i) //使用的是当前作用域中的i
    }
    div.appendChild(button)
}
```

> 底层实现上，let声明的变量实际也会有提升，但是，提升后会将其放入‘暂时性死区’，如果访问的变量位于暂时性死区，则会报错‘Cannot access 'a' before initialization’,当代码运行到该变量的声明语句时，会将其从暂时性死区中移除。

> 在循环中，用let声明的循环变量，会特殊处理，每次进入循环体，都会开启一个新的作用域，并且将循环变量绑定到该作用域。在循环中使用的let声明的循环变量，在循环结束后会销毁。

## 使用const声明变量

和let完全相同，仅在于用const声明的变量，必须在声明时赋值，并且不可以重新赋值。

> 注意
1. 常量不可变，是指声明常量的内存空间不可变，并不保证内存空间中的地址指向的其他空间不可变。
```js
const a = {
  name: '123',
  age: 18
}
a.name = 456  //不会报错

a = {
  name: 456,
  age: 18
}             //赋值了一个新的地址，会报错
```
2. 常量的命名
   1. 特殊的常量：该常量从字面意义上，一定是不可变的。（圆周率、月地距离），通常**该常量的名称全部使用大写，多个单词之间用下划线分割（PI、MOON_DIS）**
   2. 普通的常量：使用和之前一样的命名即可

3. 在for循环中，循环变量不能使用常量