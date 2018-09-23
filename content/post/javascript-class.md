+++

title = "Javascript实现类的几种方法"
date = 2018-09-18T14:01:11+08:00
toc = true
tags = ["Javascript"]
categories = ["Javascript"]
banner = "img/avatar.png"
author = "Tacey Wong"

+++

**参考：**[阮一峰-Javascript定义类（class）的三种方法](http://www.ruanyifeng.com/blog/2012/07/three_ways_to_define_a_javascript_class.html)

编写和维护如此复杂的代码，必须使用模块化策略。目前，业界的主流做法是采用"面向对象编程"。因此，Javascript如何实现面向对象编程，就成了一个热门课题。麻烦的是，Javascipt语法不支持"类"（class）（ES6已经引入class关键字），导致传统的面向对象编程方法无法直接使用。程序员们做了很多探索，研究如何用Javascript模拟"类"。本文总结了Javascript定义"类"的几种方法。

## 一、构造函数法

使用构造函数模拟类，在其内部用`this`关键字只带实例对象。

```javascript
function Car(){
    this.brand = "Tesla";
}
```
生成实例的时候，使用`new`关键字。
```javascript
var one_car = new Car();
console.log(one_car);
```
类的属性和方法，可以定义在构造函数的prototype对象之上。
```javascript
Car.prototype.makeSound = function(){
    console.log("滴滴");
}
```

## 二、Object.create()方法

为了解决"构造函数法"的缺点，更方便地生成对象，Javascript的国际标准ECMAScript第五版（目前通行的是第三版），提出了一个新的方法`Object.create()`。

用这个方法 “类”就是一个对象，不是函数。

```javascript
var Car = {
    brand : "Tesla",
    makeSound : function(){console.log("滴滴")}
}
```

然后直接用`Object.create()`生成实例，不需要用到`new`

```javascript
var one_car = Object.create(cat);
console.log(one_car.brand);//Tesla
one_car.makeSound();//滴滴
```
如果遇到浏览器不支持`Object.create()`可以自己模拟实现
```javascript
if (!Object.create){
    Object.create = function(o){
        function F(){}
        F.prototype = o;
        return new F();
    };
}
```
这种方法比“构造函数方法“简单，但是不能实现私有属性和私有方法，实例对象之间也不能共享数据，对”类“的模拟不够全面。

## 三、极简主义法

荷兰程序员Gabor de Mooij提出了一种比Object.create()更好的新方法，他称这种方法为"极简主义法"（minimalist approach）。

### 3.1 封装

这种方法不使用`this`和`prototype`，代码部署起来非常简单。

首先，他也是一个对象模拟”类“。在这个类里面定义一个构造函数`createNew()`用来生成实例。

```javascript
    var Car = {
        createNew:function(){
        //some code
        }
    };
```

然后，在`createNew()`里面，定义一个实例对象，把这个实例对象作为返回值
```javascript
var Car = {
    createNew:function{
        var car = {};
        car.brand = "Tesla";
        car.makeSound = function(){console.log("滴滴");};
        return car;
    };
}
```
使用的时候，调用`createNew()`方法，就可以得到实例对象。

```javascript
var one_car = Car.createNew();
one_car.makeSound();//滴滴
```
这个中方法的好处是容易理解、结构清晰优雅，符号传统的”面向对象编程“的构造，因此可以方便地部署下面的特性。

### 3.2 继承

让一个类继承另一个类，实现起来很方便。只要在前者的`createNew()`方法中，调用后者的`createNew()`方法即可。
先定义一个Animal类。

```javascript
var Animal = {
    createNew: function(){
        var animal = {};
        animal.sleep = function(){console.log("sleep");};
        return animal;
    }
};
```
然后在Cat的`createNew()`方法中，调用Animal的createNew()方法。

```javascript
var Cat = {
    createNew:function(){
        var cat = Animal.createNew();
        cat.name = "大猫";
        cat.makeSound:function(){console.log("喵喵");};
        return cat;
    }
}
```

这样得到的Cat实例，就会同时继承Cat类和Animal类。
```javascript
var cat1 = Cat.createNew();
cat1.sleep();//sleep
```

### 3.3私有属性和私有方法

在`createNew()`方法中，只要不是定义在cat对象上的方法和属性，都是私有的。
```javascript
var Cat = {
    createNew:function(){
        var cat={};
        var sound = "喵喵";
        cat.makeSound = function(){console.log(sound);};
        return cat;
    }
}
```

上面的内部变量sound，外部无法读取，只有通过cat的共有方法makeSound()的读取。
```javascript
var cat1 = Cat.createNew();
console.log(cat1.sound);//undefined
```

### 3.4数据共享

有时候我们需要所有的实例对象能够读写同一项内部数据。这个时候，只要把这个内部数据封装在类对象的里面、createNew()方法的外面即可。
```javascript
var Cat = {
    sound : "喵喵喵",
    createNew: function(){
        var cat = {};
　　    cat.makeSound = function(){ alert(Cat.sound); };
　　    cat.changeSound = function(x){ Cat.sound = x; };
        return cat;
　　 }
};
```

然后，生成两个实例对象：
```javascript
var cat1 = Cat.createNew();
var cat2 = Cat.createNew();
cat1.makeSound();//喵喵
```
这是如果有一个实例对象，修改了共享的数据，另一个实例对象也会受到影响
```javascript
cat2.changeSound("啦啦啦");
cat1.makeSound();//啦啦啦
```

## 四、ES6 class

曙光来临！ES6引入了`class`关键字(本质上还是基于`function` & `prototype`))

```javascript
class Car{
    constructor(brand){
        this.brand = brand;
    }
    makeSound(){
        console.log(this.brand);
    }
}
```
创建实例
```javascript
let one_car = new Car("Tesla");
console.makeSound();
```

继承直接使用`extends`关键字即可

```javascript
class SmallCar extends Car{
    constructor(brand){
        super(brand);
    }
}
```
