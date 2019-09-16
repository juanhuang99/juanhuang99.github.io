---
title: JavaScript 类型转换
date: 2019-09-06 14:52:01
tags:
---
## 什么是ToPrimitive抽象操作
该操作主要是将对象类型转换为基本类型。

<!-- more -->
### ToPrimitive(obj,preferredType)

JS引擎内部转换为原始值ToPrimitive(obj,preferredType)函数接受两个参数，第一个obj为被转换的对象，第二个preferredType为希望转换成的类型（默认为空，接受的值为Number或String）

在执行ToPrimitive(obj,preferredType)时如果第二个参数为空并且obj为Date的事例时，此时preferredType会被设置为String，其他情况下preferredType都会被设置为Number如果preferredType为Number，ToPrimitive执行过程如下：
1. 如果obj为原始值，直接返回；
2. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
3. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
4. 否则抛异常。

如果preferredType为String，将上面的第2步和第3步调换，即：
1. 如果obj为原始值，直接返回；
2. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
3. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
4. 否则抛异常。

## 什么是封装对象
var a = new String('abc')，a被叫做封装了基本类型的封装对象，还原一个封装对象的值，可以调用valueOf方法。

## 显式强制类型转换
### 转换为字符串
1.null转换为'null',undefined转换为'undefined',其他基本类型都调用基本类型的包装对象属性toString()并返回值。
```
const a = 123;
const _a = new Number(123);
console.log(String(a), _a.toString());//'123' '123'
const b = true;
const _b = new Boolean(true);
console.log(String(b), _b.toString());//'true' 'true'
```
2.数字的字符串化遵循通用规则，但是极小极大数字使用指数形式
```
const a = 1.07*1000*1000*1000*1000*1000*1000*1000;
console.log(String(a));//'1.07e+21'
```
3.对于普通对象来说，除非自行定义，否则toString()返回Object.prototype.toString()的值,其他对象有自己的toString()方法则调用自己的该方法，但是如果 toString 返回的不是基本类型值，转换时会报 TypeError。
```
const b = {};
console.log(String(b));//[object object]
```
4.数组的默认toString()方法进行了重新定义，将所有单元字符串化以后再用‘,’连接起来
```
const a = [1, 2, 3];
console.log(String(a));//'1,2,3'
```
### 转换为数字
1.true转换为1，false转换为0，undefined转换为NaN，null转换为0
```
console.log(Number(null));//0
console.log(Number(undefined));//NaN
console.log(Number(true));//1
console.log(Number(false));//0
```
2.对字符串的处理遵循数字常量的相关规定/语法，处理失败时返回NaN
```
console.log(Number('123'));//123
console.log(Number('0b111'));//7
console.log(Number('0o123'));//83
console.log(Number('0x123'));//291
console.log(Number('123a'));//NaN
console.log(Number('a123'));//NaN
```
3.对象（包括数组）会首先按照ToPrimitive抽象操作被转换为相应的基本类型值，再按照前两条规则处理；如果某个对象即不存在valueOf方法也不存在toString方法，则会产生TypeError错误（例如Object.create(null)不存在以上两种方法）

```
//obj1,obj2分别调用toString和valueOf方法，obj3调用原型上的toString()方法
const arr = [1, 2, 3];
console.log(Number(arr));//NaN
console.log(Number(arr.toString()));//NaN


const num = new Number(123);
console.log(Number(num));//123
console.log(Number(num.valueOf()));//123

const bool = new Boolean(true);
console.log(bool.valueOf());//true
console.log(Number(bool));//1
console.log(Number(bool.valueOf()));//1

const obj1 = {
  toString:()=>"21"
}

const obj2 = {
  valueOf:()=>"42",
  toString:()=>"21"
}

const obj3 = {
  a:1
}
console.log(Number(obj1));//21
console.log(Number(obj2));//42
console.log(obj3.toString());//[object Object]
console.log(Number(obj3));//NaN


const obj = Object.create(null);
console.log(Number(obj));//TypeError
```
### 转换为布尔值
- 可以被转换为false的值（undefined，null，false， +0、-0和NaN，''）
- 除条件1的其他都被转换为true（切记：封装对象均被转为true）

## 隐式强制类型转换
### 转换为字符串
一元运算符加号（+）首先把非基本类型通过ToPrimitive抽象操作转换为基本类型，如果加号中的两项有一项是字符串，另一项则进行ToString操作。
```
console.log([1, 2] + [1, 2]); //1,21,2
//等同于
console.log([1, 2].toString() + [1, 2].toString()); //1,21,2
```
### 转换为数字
通过一元运算符-、/、*转换，常见的转换方法：
```
+ '2'  // 2
'2' - 0  // 2
'2' / 1   // 2
'2' * 1   // 2


+ 'x'  // NaN
'x' - 0 // NaN
'x' / 1 // NaN
'x' * 1  // NaN

1 + '2'  // '12'
1 + + '2'  // 3    即：1 + (+ '2')
```
### 转换为布尔值
- if(..)语句中的条件判断表达式
- for(..;..;..)语句的第二个条件判断表达式
- while(..)和do..while(..)的条件判断表达式
- ?:中的条件判断表达式
- 逻辑运算符||和&&左边的操作数（a||b等同于a?a:b，a&&b等同于a?b:a）

## == 比较
记住以下 5 条规则，任何相等比较都不怕。
### 1、字符串和数字之间的相等比较
字符串先转换为数字，然后再比较。

### 2、其他类型和布尔类型之间的相等比较
布尔类型转换为数字，再进行比较。

### 3、对象和非对象之间的相等比较
对象转换成基本类型值，按转换为数字的流程转换后进行比较。对象转换优先级最高。

### 4、null 和 undefined
null == undefined， 其他类型和 null 均不相等，undefined 也是如此。

### 5、特殊情况
```
NaN == NaN  // false
-0  == +0   // true
```
两个对象比较，判断的是两个对象是否指向同一个值。
下面是一些看起来“不可思议”的相等比较，是否可以按上面的规则解释清楚？
```
"0" == false // true
// 第2条规则，false 转换为数字，结果为 0，等式变为 "0" == 0
// 两边类型不一致，继续转换，第1条规则，"0" 转换为数字，结果为 0，等式变为 0 == 0

false == [] // true
// 第3条规则，[] 转换基本类型值，[].toString()，结果为 ""，等式变为 "" == false
// 两边类型不一致，继续转换，第2条规则，false 转换为数字，结果为 0，等式变为 "" == 0
// 两边类型不一致，继续转换，第1条规则，"" 转换为数字，结果为 0，等式变为 0 == 0

0 == []    // true
// 第3条规则，[] 转换基本类型值，[].toString()，结果为 ""，等式变为 0 == ""
// 两边类型不一致，继续转换，第1条规则，"" 转换为数字，结果为 0，等式变为 0 == 0
```
## 抽象关系比较
关系比较都会转换为 a < b 进行比较，a > b 会被转换成 b < a，a <= b 会被转换成 !(b<a)，a >= b 转换为 !(a<b)
### 比较规则
1.如果双方是其他情况首先调用ToPrimitive转换为基本类型;
2.若有一方不是字符串，则将其转换为数字再进行比较；
3.若有一方是 NaN，结果总是 false。
```
null >= 0 // true
null <= 0 // true
null == 0 // flase
//按照第二条规则，需要将 null 转换为数字，结果为 0，因此 null >= 0 和 null <= 0 是成立的。但是在相等比较规则中（第 4 条），其他类型和 null 均不相等，因此 null == 0 是不成立的。

var obj = {}
obj >= 0 // false
obj <= 0 // false
obj == 0 // false
//按照第 1 条规则，obj 转换结果为 "[object Object]"，接着按第 2 条规则，"[object Object]" 转换为数字，结果为 NaN，按照第 3 条，无论怎么比较结果都是 false。
```


参考： 
[JavaScript 类型转换](https://www.qinshenxue.com/article/javascript-type-conversion.html)
[深入理解 JavaScript 的类型转换](https://juejin.im/post/5d1587f4e51d4510664d1715)
[JavaScript各种蛋疼的类型转换](https://segmentfault.com/a/1190000008432611)