---
title: ES6 之 Symbol
date: 2019-11-12 11:46:17
tags: [es6]
categories: 前端
cover: https://w.wallhaven.cc/full/73/wallhaven-73x33o.png
---

# 普通符号

符号是ES6新增的一个**数据类型**，它通过使用函数 ```Symbol(符号描述)``` 来创建

符号设计的初衷，是为了给对象设置私有属性
- 私有属性：只能在对象内部使用，外面无法使用

## 符号的特点

1. 没有字面量（数字字面量：12312）
2. 使用 typeof 得到的类型是字符串```symbol``` 
3. **每次调用 Symbol 函数得到的符号永远不相等，无论符号描述是否相同**
```js
var a = Symbol('abc')
var b = Symbol('abc')
a === b //false
```
4. 符号可以作为对象的属性名存在，这种属性称之为符号属性
  - 开发者可以通过精心的设计，让这些属性无法通过常规方式被外界访问 ----**私有变量**
```js
const Hexo = (() => {
  const getRandom = Symbol('getRandom')
  return  class Hexo {
    constructor(attack, defence, hp) {
      this.attack = attack
      this.defence = defence
      this.hp = hp
    }
    gongji() {
      let dmg = this.attack * this[getRandom](4,1)
      console.log(111,dmg)
    }
    //定义了一个内部的实现方法不希望被外部访问的时候。
    [getRandom](max, min) {
      return Math.random()*(max - min) + min
    }
  }
})()
const hexo = new Hexo(10, 20, 200)
hexo.gongji()
hexo.getRandom() //访问不到
```
  - 符号属性是不能枚举的，因此在 for-in 循环中无法读取到符号属性，Object.keys 方法也无法读取到符号属性
  - Object.getOwnPropertyNames 尽管可以得到所有无法枚举的属性，但是仍然无法读取到符号属性
  - ES6 新增 **Object.getOwnPropertySymbols** 方法，可以读取符号,放入一个数组里面
```js
//如果写了symbol又想访问的，也是可以的
const sy1 = Symbol()
const obj = {
  a: 1,
  b: 2,
  [sy1]: 3
}
const sy = Object.getOwnPropertySymbols(obj) //拿到obj里面含有所有符号的数组
obj[sy[0]] //3
```
5. 符号无法被隐式转换，因此不能被用于数学运算、字符串拼接或其他隐式转换的场景，但符号可以显式的转换为字符串，通过 String 构造函数进行转换即可，console.log 之所以可以输出符号，是它在内部进行了显式转换 
```js
const a = Symbol('a')
String(a) // 字符串Symbol(a)
console.loh(a) //字符串Symbol(a)
a + '' //Cannot convert a Symbol value to a string
+ a //Cannot convert a Symbol value to a number
```

# 共享符号
根据某个符号名称（符号描述）能够得到**同一个符号**

```js
Symbol.for("符号名/符号描述")  //获取共享符号
```
```js
const syb1 = Symbol.for("abc");
const syb2 = Symbol.for("abc");
console.log(syb1 === syb2) //true
//符号可以被访问
const obj = {
  a: 1,
  b: 2,
  [Symbol.for("c")]: 3
}
console.log(obj[Symbol.for("c")]);
```
## 实现原理
```js
const Symbolfor = (() => {
  //定义一个全局变量
  const global = {}
  return function (name) {
    //如果变量名没有值，则赋值
    if(!global[name]) {
      global[name] = Symbol(name)
    }
    //有则返回对应属性名的属性值
    return global[name]
  }
})()

var a = Symbolfor('abc')
var b = Symbolfor('abc')
console.log(a === b) //true
```