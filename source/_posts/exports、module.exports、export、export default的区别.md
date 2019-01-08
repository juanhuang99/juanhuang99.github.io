---
title: exports、module.exports、export、export default的区别
date: 2019-01-07 17:22:11
tags: nodeJs   ES6
---

exports、module.exports、export、export default都是模块导出，它们之间又有什么区别，为了不继续混乱下去，是时候把它们的关系捋捋清楚了。

<!-- more -->
## 使用范围
require: node 和 es6 都支持的引入
export / import : 只有es6 支持的导出引入
module.exports / exports: 只有 node 支持的导出

## node中的模块导入导出
**exports 和 module.exports**
在一个node执行一个文件时，会给这个文件内生成一个 exports和module对象，
而module又有一个exports属性。他们之间的关系如下图，都指向一块{}内存区域。
![](/img/node-export.png)
看个代码例子：

    //utils.js
    let a = 100;
    console.log(module.exports); //能打印出结果为：{}
    console.log(exports); //能打印出结果为：{}

    exports.a = 200; //这里辛苦劳作帮 module.exports 的内容给改成 {a : 200}

    exports = '指向其他内存区'; //这里把exports的指向指走

    //test.js
    var a = require('/utils');
    console.log(a) // 打印为 {a : 200}  
>从上面可以看出，其实require导出的内容是module.exports的指向的内存块内容，并不是exports的。
他们之间的区别就是exports只辅助module.exports操作内存中的数据，最后真正被require出去的内容还是module.exports。简而言之就是exports 只是 module.exports的引用。

## ES6中的模块导入导出
**export 和 export default**
它们之间的区别是：
1.export与export default均可用于导出常量、函数、文件、模块等
2.在一个文件或模块中，export、import可以有多个，export default仅有一个
3.通过export方式导出，在导入时要加{ }，export default则不需要
4.export能直接导出变量表达式，export default不行。

看个代码例子：

    //testEs6Export.js
    'use strict'
    //导出变量
    export const a = '100';  

    //导出方法
    export const dogSay = function(){ 
        console.log('wang wang');
    }

    //导出方法第二种
    function catSay(){
    console.log('miao miao'); 
    }
    export { catSay };

    //export default导出
    const m = 100;
    export default m; 
    //export defult const m = 100;// 这里不能写这种格式。


    //index.js
    'use strict'
    var express = require('express');
    var router = express.Router();

    import { dogSay, catSay } from './testEs6Export'; //导出了 export 方法 
    import m from './testEs6Export';  //导出了 export default 

    import * as testModule from './testEs6Export'; //as 集合成对象导出



    /* GET home page. */
    router.get('/', function(req, res, next) {
    dogSay();
    catSay();
    console.log(m);
    testModule.dogSay();
    console.log(testModule.m); // undefined , 因为  as 导出是 把 零散的 export 聚集在一起作为一个对象，而export default 是导出为 default属性。
    console.log(testModule.default); // 100
    res.send('恭喜你，成功验证');
    });

    module.exports = router;