---
layout:     post
title:      从babel-polyfill的一个坑而起
description: babel/polyfill tc39 Array.prototype.flat es2015
subtitle:   babel-polyfill
date:       2018-01-06
author:     Seize
header-img: img/home-bg-art.jpg
catalog: true
tags:
    - 前端
    - js
    - babel
    - babel-polyfill
---
# 问题
接手的系统几天前出现了兼容性问题，`Array.flat is not a function`,一看原来是没有引用babel-polyfill，直接import了babel-polyfill，发现并没有解决问题。

# 原因

 跑去看了一下babel-polyfill的[git](https://github.com/babel/babel/tree/master/packages/babel-polyfill/)

> [`@babel/polyfill`](http://babeljs.io/docs/usage/polyfill) [**IS** just the import of stable `core-js` features and `regenerator-runtime`](https://github.com/babel/babel/blob/c8bb4500326700e7dc68ce8c4b90b6482c48d82f/packages/babel-polyfill/src/index.js)

> This will emulate a full ES2015+ environment (no < Stage 4 proposals)

所以，babel-polyfill只是包括了稳定的corejs(小于stage 4的没有）和regerator-runtime（这个是用来实现generator的）。

```js
import "./noConflict";
import global from "core-js/library/fn/global"
```

noConflict：

```js
// Cover all standardized ES6 APIs.
import "core-js/es6";

// Standard now
import "core-js/fn/array/includes";
import "core-js/fn/array/flat-map";
import "core-js/fn/string/pad-start";
import "core-js/fn/string/pad-end";
import "core-js/fn/string/trim-start";
import "core-js/fn/string/trim-end";
import "core-js/fn/symbol/async-iterator";
import "core-js/fn/object/get-own-property-descriptors";
import "core-js/fn/object/values";
import "core-js/fn/object/entries";
import "core-js/fn/promise/finally";

// Ensure that we polyfill ES6 compat for anything web-related, if it exists.
import "core-js/web";

import "regenerator-runtime/runtime";

```

ok，先去看一下core-js是什么：模块化的标准库，可提供所有es5 es6 es7的新特性，可以全局污染使用或者避免污染使用。第一个core-js/es6 里面包括了![](http://ww4.sinaimg.cn/large/006tNc79ly1g3h1brivdyj30jt0dx76s.jpg)

这些对象扩展，点开Array

![](http://ww4.sinaimg.cn/large/006tNc79ly1g3h1d6ygenj30el05rt9r.jpg)

ok，whatever，flat是超纲了。（因为这里面是es6）



然后还有一个corejs/web，他用来兼容浏览器宿主对象，定时器等。

global是global对象。

看他后面那几行又补充了一些新增的state4属性

array新加includes和flatmap，what？？

这里重点来了：**为什么有flatmap，没有flat？？？** 

# So es2015+ 的state4 到底有啥？

先看一下什么是ECMA-262、ECMAScript

## ECMA-262和ECMAScript

> 首先解释一下ECMA(European Computer Manufactures   Association)欧洲计算机制造商协会。
>  TC39(Technical Committee #39)39号技术委员会
>    1. TC93制定了ECMA-262标准。
>
>    2. ECMA-262是ECMAScript的标准，ECMA-262定义了ECMAScript。
>
>    3. 由ECMA-262定义的ECMAScript与web服务器没有依赖关系。ECMA-262定义的只是ECMAScript的语言基础。我们常见的web浏览器只是ECMAScript实现可能的宿主环境之一。宿主环境不仅提供基本的ECMAScript实现，同时也会提供该语言的扩展，以便语言与环境之间对接交互。
>
>    4. javascript实现了ECMAScript。   
>
>    5. 尽管 ECMAScript 是一个重要的标准，但它并不是 JavaScript 唯一的部分，当然，也不是唯一被标准化的部分。实际上，一个完整的 JavaScript 实现是由以下 3 个不同部分组成的：   
>      1. 核心（ECMAScript）
>      2. 文档对象模型（DOM）
>      3. 浏览器对象模型（BOM）

## ES2015

> ECMAScript 2015 is an ECMAScript standard that was ratified in June 2015.

2015年六月获得批准的。

[完整版链接]([http://www.ecma-international.org/ecma-262/6.0/](http://www.ecma-international.org/ecma-262/6.0/))

[es6新增](https://github.com/lukehoban/es6features#readme):

> - [arrows](https://github.com/lukehoban/es6features#arrows)  箭头函数
> - [classes](https://github.com/lukehoban/es6features#classes) 类
> - [enhanced object literals](https://github.com/lukehoban/es6features#enhanced-object-literals) 增强对象字面量
> - [template strings](https://github.com/lukehoban/es6features#template-strings) 模板字符串
> - [destructuring](https://github.com/lukehoban/es6features#destructuring) 解构
> - [default + rest + spread](https://github.com/lukehoban/es6features#default--rest--spread) 默认值，拓展运算符，剩余运算符
> - [let + const](https://github.com/lukehoban/es6features#let--const)
> - [iterators + for..of](https://github.com/lukehoban/es6features#iterators--forof) 
> - [generators](https://github.com/lukehoban/es6features#generators)
> - [unicode](https://github.com/lukehoban/es6features#unicode)
> - [modules](https://github.com/lukehoban/es6features#modules) 模块
> - [module loaders](https://github.com/lukehoban/es6features#module-loaders)
> - [map + set + weakmap + weakset](https://github.com/lukehoban/es6features#map--set--weakmap--weakset)
> - [proxies](https://github.com/lukehoban/es6features#proxies)
> - [symbols](https://github.com/lukehoban/es6features#symbols)
> - [subclassable built-ins](https://github.com/lukehoban/es6features#subclassable-built-ins) 内建对象可被继承
> - [promises](https://github.com/lukehoban/es6features#promises) 没有finally
> - [math + number + string + array + object APIs](https://github.com/lukehoban/es6features#math--number--string--array--object-apis) 
>   1. acosh、hypot平方和的平方根、imul相乘	
>   
>   2. isNaN、isInteger
>   
>   3. includes、repeat
>   
>   4. from 返回真正的数组、of一组值变数组、fill填充、find、findIndex、copyWithin复制替换
>   
>   5. assign
> - [binary and octal literals](https://github.com/lukehoban/es6features#binary-and-octal-literals)
> - [reflect api](https://github.com/lukehoban/es6features#reflect-api)
> - [tail calls](https://github.com/lukehoban/es6features#tail-calls) 尾调用优化，有问题
>

所以es2015也不多，array就加了from、of等5个，也没有flatMap，那为什么polyfill里面加了它呢，而不加flat呢？

我们用的是babel 7.4，babel 7是core-js@2

> Right now `@babel/polyfill` is mostly just an alias of `core-js` v2. [Source](https://github.com/babel/babel/blob/master/packages/babel-polyfill/src/index.js)

babel 6时代，babel/polyfill包括所有state，它这么引：

``` js
import "core-js/shim"; // included < Stage 4 proposals
import "regenerator-runtime/runtime";
```

core-js/shim的文档：

> Below is a list of Stage < 3 proposal polyfills in `core-js` v2.

``` js
// core-js v2

// Stage 3
import "core-js/fn/string/trim-left";
import "core-js/fn/string/trim-right";
import "core-js/fn/string/match-all";
import "core-js/fn/array/flat-map";
import "core-js/fn/array/flatten"; // RENAMED
import "core-js/fn/global";

// Stage 1
import "core-js/fn/symbol/observable";
import "core-js/fn/promise/try";
import "core-js/fn/observable";

// Stage 1 Math Extensions
import "core-js/fn/math/clamp";
import "core-js/fn/math/deg-per-rad";
import "core-js/fn/math/degrees";
import "core-js/fn/math/fscale";
import "core-js/fn/math/iaddh";
import "core-js/fn/math/isubh";
import "core-js/fn/math/imulh";
import "core-js/fn/math/rad-per-deg";
import "core-js/fn/math/radians";
import "core-js/fn/math/scale";
import "core-js/fn/math/umulh";
import "core-js/fn/math/signbit";

// Stage 1 "of and from on collection constructors"
import "core-js/fn/map/of";
import "core-js/fn/set/of";
import "core-js/fn/weak-map/of";
import "core-js/fn/weak-set/of";
import "core-js/fn/map/from";
import "core-js/fn/set/from";
import "core-js/fn/weak-map/from";
import "core-js/fn/weak-set/from";

// Stage 0
import "core-js/fn/string/at";

// Nonstandard
import "core-js/fn/object/define-getter";
import "core-js/fn/object/define-setter";
import "core-js/fn/object/lookup-getter";
import "core-js/fn/object/lookup-setter";
// import "core-js/fn/map/to-json"; // Not available standalone
// import "core-js/fn/set/to-json"; // Not available standalone

import "core-js/fn/system/global";
import "core-js/fn/error/is-error";
import "core-js/fn/asap";

// Decorator metadata? Not sure of stage/proposal
import "core-js/fn/reflect/define-metadata";
import "core-js/fn/reflect/delete-metadata";
import "core-js/fn/reflect/get-metadata";
import "core-js/fn/reflect/get-metadata-keys";
import "core-js/fn/reflect/get-own-metadata";
import "core-js/fn/reflect/get-own-metadata-keys";
import "core-js/fn/reflect/has-metadata";
import "core-js/fn/reflect/has-own-metadata";
import "core-js/fn/reflect/metadata";
```

Core-js @2的时候flat和flatmap都是state3，然而我又去看了tc39 esma 262的[git](https://github.com/tc39/proposals/blob/master/finished-proposals.md)

Soga,这几个state3 现在已经是state4了，

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3i0is3smkj30oj06zdgw.jpg)

所以babel/polyfill补充了这些：

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3i0jqwam8j30s40i2whl.jpg)

说明紧跟state4走，没问题。

我们的问题还没解决，那为什么没有flat，只有flat-map。

这又是另一个故事了

# what is wrong with flat ？

flat原来名叫flatten，和国外一个叫Mootools的框架命名冲突了。造成了smooshGate，[smoosh门事件](https://developers.google.com/web/updates/2018/03/smooshgate)。

咋回事呢？

Mootools框架定义了自己的`Array.prototype.flatten` 与标准不同，并且会覆盖原生实现，这虽然会导致新的网站同时使用这两者会出问题，但不会导致之前的网站出问题。但是，mootools还会把自定义的方法复制到他自己的一个api对象Element：

```js
for(var key in Array.prototype){
	Element.prototype[key] = Array.prototype[key]
}
```

本来Element能得到flatten，因为它是自己定义的属性，可以被for in枚举。但是flatten被标准实现之后，变成不可枚举了，所有使用Element.prototype.flatten的代码就完犊子了。

作者开了个玩笑，不如把flatten改成smooth，造成了更大的影响。。。

18年3月 TC39开会讨论这一问题

![](http://ww1.sinaimg.cn/large/006tNc79ly1g3i2539kukj30ii00ujrh.jpg)

18年5月 TC39开会再讨论：

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3i278g2p2j30kn01maa7.jpg)

not smoosh～ be serious～

所以啊，归根结底竟然是一个名字造成的锅。
