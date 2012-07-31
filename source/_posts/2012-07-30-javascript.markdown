---
layout: post
title: "JavaScript, The Hard Parts"
date: 2012-07-30 11:20
comments: true
categories: 
---

## 序

---

>  In JavaScript, there is a beautiful, elegant, highly expressive language that is buried under a steaming pile of good intentions and blunders.
> <small>"JavaScript, The Good Parts", by Douglas Crockford</small>

JavaScript，做为目前Github上[最为热门的语言](https://github.com/languages)，在无论是web开发还是mobile开发方面都已经不可或缺，尤其是最近node.js带来的web app风潮，使得js开发的规模和复杂度又上了一个台阶。

但正如Douglas Crockford所说，JavaScript美丽的本质被隐藏在一堆错误和草率的设计之下，在他的经典著作[JavaScript, The Good Parts](http://shop.oreilly.com/product/9780596517748.do)中，明确提出了一个简洁，优雅的语言子集，只使用一门语言最好的部分，可以产生更好的代码，也能造就一个更好的程序员。

这里讨论的，就是在js的The Good Parts中，相对更难，也更重要的部分，即如何设计代码的结构和组织，在保证可读性和易维护性的情况下，做到最大限度地DRY。

本文**不涉及**:

* 变量，对象，表达式，语句等基础要素
* 性能优化
* 宿主环境(Host Environment)，如浏览器
* 核心库之外的库和框架

本文**只探讨**:

* 语言最精华和最费解的要素，闭包和原型
* 运用以上要素的常用编程模式，面向对象，或非面向对象

在一切开始之前，我建议在每个js项目中，都使用*严格模式*进行开发

``` javascript
"use strict"
```

严格模式是一个ECMA5标准提出的js子集，在未来会逐渐成为js的语言标准，详细介绍在[这里](https://developer.mozilla.org/en/JavaScript/Strict_mode#Changes_in_strict_mode)

使用严格模式，可以部分地避免一些js设计不好的部分，比如严格模式的如下限制：

* 所有的变量必须先定义再使用
* 在作用域内`this`没有找到明确值的情况下，`this`为`undefined`，而不是全局对象

这两条规则可以让js语言最大的设计失误**隐式全局变量(Implied Globals)**的危害降低很多很多

---

## 要素(Features)

---

### 闭包(Closure)

#### 函数(Function)

js中的函数

* 函数是对象
* 函数有自己的作用域

函数的属性

this  
    `this` in these context
    method
    function
    constructor
    apply
    *this* context in closure
    var _this = this;

scope chain
prototype

#### 作用域(Scope)

作用域链(scope chain)

    Activation Object
    how to implement closure with scope chain

Hoisting

在作用域中，无论变量的定义在什么地方，js会在解释的时候把定义提到作用域的最前面，这称为*Hoisting*

``` javascript
var foo = "global";
function f() {
    console.log(foo); // => undefined
    var foo = "local";
    console.log(foo); // => "local"
}
```
上述代码等效
``` javascript
var foo = "global";
function f() {
    var foo;
    console.log(foo); // => undefined
    foo = "local";
    console.log(foo); // => "local"
}
```
因此，在作用域中不会出错的方法是将变量(包括嵌套函数)的定义语句提升到最前

Closure application

use closures access private attributes

this and arguments in closures
closures as constructors

closure module
var serial_maker = function() {
    var seq = 0;
    return {
        gensym: function() {return ++seq;} }; };

Immediate Function
包装模块，使它们不污染全局空间，比如javascript书签
``` javascript
(function () {
    //... code
}())   
```


---

### 原型(Prototype)
  
js是一门面向对象的语言，但与继承于smalltalk的基于类的面向对象语言不同，js的面向对象使用来自self语言的*原型继承*的方法实现，在js中，只有对象(object)，而没有对象的模板(class)

题外话，smalltalk和self语言都产生于Xerox PARC(施乐的帕洛阿托研究中心)，这里也是点阵图，图形界面，激光打印和以太网的故乡，致敬一下:)

#### 实现

几乎每一个js对象都和另一个对象相关联，这另一个对象就是我们说的原型，每一个对象都从原型继承属性

所有字面量对象(literal object)都具有同一个原型，她是`Object.prototype`，通过构造器生成的对象原型则是构造器的`prototype`属性的值

对象的原型对象也有可能有自己的原型，形成了一条原型链(prototype chain)，直到`Object.prototype`为止，她没有原型

查询对象的属性时，从此对象本身向原型链上依次向上查询，这也称为*委托(Delegation)*

指定原型生成对象的方法有两种：

1. 使用构造器生成
2. 在ECMA5中，使用`Object.create()`方法

Douglas Crockfold实现了一种利用构造器方便地继承原型的方法

``` javascript
if (typeof Object.beget !== 'function') {
    Object.beget = function (o) {
        var F = function () {};
        F.prototype = o;
        return new F();
    };
}
var another_stooge = Object.beget(stooge);
```

获取一个对象原型则并不容易，方法也有两种:

1. 查询`o.constructor.prototype`来检测，不过往往原型上的construct属性会丢失掉
2. 在ECMA5中，使用`Object.getPrototypeOf()`方法

#### 构造器(Constructor)

不得不说，使用`new`来初始化对象，这在基于原型继承的js语言中，带来了严重的违和感，根据作者本人的解释，当时的这个设计应该是为了贴近正流行的java语言，让更多人更容易上手，结果造成了js中一个不良和晦涩的设计，不过，在js中，这也是指定原型的必要方法。

``` javascript
var Person = function (name) {this.name = name;};
Person.prototype.getName = function() {return this.name;};
var john = new Person("john");
```

使用`new`调用一个函数时:

1. 一个空对象被创建
2. 将此空对象的原型指向`Person.prototype`
3. 将此空对象赋值给`this`
4. 为`this.name`赋值
5. 如果没有明确`return`其他对象，则返回`this`

为了让构造函数得以实现，**每一个**js中的函数对象都不得不：

* 拥有一个`prototype`属性，指向一个对象
* 这个对象有唯一一个属性`constructor`, 指向此对象所属的函数对象

这是复杂，冗余，而不容易理解的，有可能出现这些问题:

1. 如果忘记了`new`去调用构造函数，会造成不可预期的行为，尤其是在非严格模式下，这时的`this`是全局对象
2. 很多情况下`prototype`可能会在重构中被完全覆盖，此时`constructor`属性也会丢失掉

在后面我们会探讨创建对象更好的模式

---

## 模式(Patterns)

### 创建(Object create)

object create
* literal
  Object.prototype
* new 
  constructor.prototype
* Object.create()
  Douglas Crockford's inherit function, for ECMA3

### 类(Class)

constructor, instance method, class method
function Complex(r, i) {}
Complex.prototype.add = function(that) {};
Complex.ZERO = new Complex(0, 0);
Complex.parse = function(s) {return new Complex(r, i);}
Complex.prototype.constructor

### 继承(Inheritance)

inheritance
    child.prototype = inherit(parent.prototype);
method chaining
    Set.prototype.add.apply(this, arguments);
prototype inheritance (from good parts)
functional inheritance (from good parts)

### 组合(Composite)

### 扩充(Augmenting)

js中，所有的对象，包括原型，都可以在运行时动态地修改和添加新的行为，修改内部类和对象的特性一般称为*monkey patching*

以下是为函数对象加上ECMA5中的`bind`方法的代码
``` javascript
if (!Function.prototype.bind) {
    Function.prototype.bind = function(o) {
        var _this = this, _bound_args = arguments;
        return function() {
            var args = [], i;
            for(i = 1; i < _bound_args.length; i++) args.push(_bound_args[i]);
            for(i = 0; i < arguments.length; i++) args.push(arguments[i]);
            return _this.apply(o, args);
        };
    };
}
```

一般来说，为了保证代码的可预期性，对于原生对象，尽量只添加一些向后兼容的方法，比如上边就为ECMA3代码实现ECMA5中的方法，并进行了检测

### 混入(Mixin)

Range.prototype.equals = generic.equals;

### 模块(Module)

使用一个全局对象作为namespace，简单直接

``` javascript
var MYAPP = {};
MYAPP.events = {/*...code...*/};
MYAPP.doms = {/*...code...*/};
MYAPP.DATA = "CONST DATA";

var myFunc = function () {
    //使用本地引用，清晰表达使用了什么模块，效率也高
    var events = MYAPP.events;
    /* ... */
};
```

modules
objects as modules
    sets.SingletonSet = sets.AbstractSet.extend();
functions as modules

### Curry

再举个闭包实际运用的例子，实现函数的curry化功能，即生成一个绑定部分参数的新函数，这在函数式编程中很常见。

Curry这个名称来自于数学家Haskell Curry(著名的Haskell语言也来自于他)
``` javascript
if (!Function.prototype.curry) {
    Function.prototype.curry = function () {
        //arguments并不是真正的数组，所以没有concat方法，我们使用了slice将其转换成了数组
        var slice = Array.prototype.slice,
            args = slice.apply(arguments),
            _this = this;
        return function () {
            return _this.apply(null, args.concat(slice.apply(arguments)));
        };
    };
}

var add = function (x, y) {return x+y;}
var add1 = add.curry(1);
console.log(add1(2)); //=> 3
```


---

## 结论

