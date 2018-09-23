+++
title = "Javascript面向对象编程"
date = 2018-09-15T13:35:28+08:00
toc = true
tags = ["Javascript","面向对象"]
categories = ["Javascript"]
banner = "img/avatar.png"
author = "Tacey Wong"
+++

## 前言

每当需要些JS代码的时候都会想到阮籍的一句话：
> 时无英雄，竖子成名

有一段时间实在是想不到为什么是Javascript，就像疑惑上帝为什么会选择耶路撒冷一样。后来懵懵懂懂的认识到Javascript的简单语法和几乎无限的自由度（逻辑功能实现上）让其在早期迅速占领了大片疆域，当时间线走到二十一世纪之后的十几个年头，传统的“精英型语言”开发者回首发现浏览器天然跨平台的优势以及HTTP(S)无限自由的链接各种系统的时候，Javascript已经成了既定事实。虽然跨平台方案诸侯遍地，也只有Javascript能够在其中自由穿梭，默认支持。

时也命也！

上帝选择耶路撒冷也许是因为偏爱，但对Javascript的选择也仅仅是历史时间线的选择。Javascript的自由度为其自身开疆拓土，成就了Javascript如今一家独大的地位，但塞翁福祸岂可谬哉？Javascript语法简单，所以要实现复杂的功能就必须大量依赖宿主环境的各种外部API，而且外部API经常不受Javascript规范所约束；Javascript的无类型对象让人随便就能写几行代码，但是质量的保证惨不忍睹；Javascript基于原型链的系统自由度突破天际，也造成了纷繁复杂，上下不一的各种问题。

就以我目前对其的认知，发现Javascript有如下缺点：

+ 重度依赖object这种数据类型，可对这种数据类型的规则限定却是各种反人类
+ 作用域及生存周期的限定是个谜
+ 过多依赖外部环境
+ 语义上不一致（与Python相比，Javascript的各种语义就像是个随意的玩具）

Javascript的难点与核心在Object（静）与异步（动），要充分发挥Javascript语言的功能必须弄清楚这两点。


http://bonsaiden.github.io/JavaScript-Garden/zh/
http://www.ruanyifeng.com/blog/2012/07/three_ways_to_define_a_javascript_class.html
http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html


## 封装
http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html

## 继承


## ES6 面向对象


