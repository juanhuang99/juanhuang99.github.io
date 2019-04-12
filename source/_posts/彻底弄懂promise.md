---
title: 彻底弄懂promise
date: 2019-04-12 14:03:25
tags: ES6
---
## 什么是promise
promise是es6的一个重要特性，用于实现异步。promise对象拥有一个叫做状态的属性，该属性不受外界影响，修改后不能再次变化。而Promise是一个构造函数，可以生成promise对象。

<!-- more -->

常见用法：
```
var a = new Promise(function (res, rej) {
  console.log('1')
  if (1) {
    res(3)
  } else {
    rej()
  }
})
a.then(function (val) {
  console.log(val)
})
console.log(2)
```
输出顺序是1,2,3,为什么呢？
Promise对象表示未来某个将要发生的事件，但在创建（new）Promise时，作为Promise参数传入的函数是会被立即执行的，只是其中执行的代码可以是异步代码。因此会马上输出1，而then中的方法是被放到mircotask队列中，在当前的代码执行完后在执行。因此先输出2，再输出3

## 构造函数的四个方法
Promise构造函数上带有resolve，race，all，reject四个方法，他们的共同点是都会返回一个promise。让我们来分别了解下。
##### resolve
resolve接受一个值或是promise对象，如果接受的是promise对象，会直接返回该promise对象，否则返回完成状态的promise对象。
```
Promise.resolve(3).then(res => console.log(res))// 3
var a = Promise.resolve(3)
console.log(Promise.resolve(a) === a) // true
```

##### reject
类似resolve，返回的是拒绝状态的promise

##### all
效果：谁跑的慢，以谁为准执行回调
Promise.all接受一个数组，分为三种情况:
1. 传入空数组，返回完成状态的promise
2. 传入的数组中没有promise，返回异步完成的prmise
3. 传入的数组中有promise，返回处于pending状态的promise对象，在数组中的promise都成功或者有一个失败时，变成完成状态或拒绝状态

##### race
效果：谁跑的快，以谁为准执行回调
同all有点接近，在某个promise完成后，返回该值，根据状态来决定返回的是完成状态还是拒绝状态的promise
如果传入的是空数组，返回的promise永远是等待

## promise对象的方法
promise带有then和catch两个方法

##### then
then接受两个参数，返回一个promise。第一个参数是promise在成功的情况下的回调函数，第二个参数是失败情况下的(可选)
- 如果then中的回调函数返回一个值，那么then返回的Promise将会成为接受状态，并且将返回的值作为接受状态的回调函数的参数值。
- 如果then中的回调函数抛出一个错误，那么then返回的Promise将会成为拒绝状态，并且将抛出的错误作为拒绝状态的回调函数的参数值。
- 如果then中的回调函数返回一个已经是接受状态的Promise，那么then返回的Promise也会成为接受状态，并且将那个Promise的接受状态的回调函数的参数值作为该被返回的Promise的接受状态回调函数的参数值。
- 如果then中的回调函数返回一个已经是拒绝状态的Promise，那么then返回的Promise也会成为拒绝状态，并且将那个Promise的拒绝状态的回调函数的参数值作为该被返回的Promise的拒绝状态回调函数的参数值。
- 如果then中的回调函数返回一个未定状态（pending）的Promise，那么then返回Promise的状态也是未定的，并且它的终态与那个Promise的终态相同；同时，它变为终态时调用的回调函数参数与那个Promise变为终态时的回调函数的参数是相同的。

##### catch
作用和then的第二个参数一样，用来指定reject的回调。还有另外一个作用：在执行resolve的回调（也就是then中的第一个参数）时，如果抛出异常了（代码出错了），那么并不会报错卡死js，而是会进到这个catch方法中。

参考： 
[大白话讲解Promise（一）](https://www.cnblogs.com/lvdabao/p/es6-promise-1.html)
[来聊聊promise](http://hpoenixf.com/posts/10947/)

