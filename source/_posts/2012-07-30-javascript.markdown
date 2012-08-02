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

* 变量，表达式，语句等语言基础要素
* 性能优化
* 宿主环境(Host Environment)，如浏览器
* 核心库之外的库和框架

本文**只探讨**:

* 语言中较为费解的部分，如闭包，原型和构造函数
* 如何进行代码复用
* 常见编程模式

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

## 对象(Object)

javascript中的对象，和传统面向对象语言中的对象有本质的不同

* 对象是属性的容器，每个属性都拥有名字(key)和值(value)
* 对象没有也不需要类(class)

对象可以用字面量的方式直接生成，由于js中函数也是对象，可以作为对象的属性，因此js中的对象结合了通常键值對(在别的语言中也称为hash/dictionary)的数据存储能力和一般语言中由类生成的对象的逻辑表达能力，可以说是js最优雅的设计之一

``` javascript
var o = {
    foo: "bar",
    hello: function () {return "hello, world";}
};
```

---

## 闭包(Closure)

要了解闭包的概念，首先来检视一下js中的函数(function)到底是什么

* 函数是对象，可以有自己的属性和方法
* 函数有自己的作用域

### 函数作用域

js中没有块级作用域(block scope)，取而代之的是函数作用域(function scope)，即只有函数可以构建出一个上下文环境，变量在声明他们的函数体及之内嵌套的任意函数体内都是有定义的

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

### 作用域对象

在函数作用域中，某个用`var`声明的变量实际上是一个与此函数相关的对象的属性，这个对象在ECMA3标准中称为*call object*，在ECMA5中称为*declarative environment record*，通常也有一种叫法是*Activation Object*，这个对象是一种内部实现，没有方法可以引用

如果没有用`var`而直接使用了某变量，则此变量视为是全局作用域相关对象(global object)的属性，这也是为什么严格模式要做变量必须定义的限制的原因所在

一种可行的利用函数作用域的方法被称为*Immediate Function*，被普遍使用在包装模块上，使内部变量不污染全局空间，典型的情景比如javascript书签
``` javascript
(function () {
    //... code
}())   
```

当函数内部又嵌套定义了其他函数时，他们的作用域对象会形成一个**作用域链**，这一组对象定义了代码作用域中的变量，当内部函数使用某个变量时，她会根据作用域链向上查找

当定义一个函数时，这个作用域链得以保存下来，当此函数被调用时，*Activation Object*被创建，保存局部变量和参数，并被添加至此作用域链上

### 闭包的实现

作用域链和其上保存的变量，让函数对象形成了**闭包**，她可以捕捉到局部变量和参数，并一直保存下来，并可以作为值进行传递

只有一个*Activation Object*失去所有引用的时候，即她不被包含在任何作用域链内的时候，才会被垃圾回收，这时候这个闭包就永久消失了

一个典型的闭包形成的计数器
``` javascript
var serial_maker = function() {
    var seq = 0;
    return {
        gensym: function() {return ++seq;} 
    }; 
};

var s = serial_maker();
s.gensym(); //=> 1
s.gensym(); //=> 2
```        

### 函数的隐藏属性

函数对象有`call`，`apply`，`length`之类的属性和方法，*Activation Object*也有默认的属性，最常见的是函数的参数列表`arguments`，而较令人费解的，是`this`

`this`在不同的函数调用中，指向不同的对象

* 函数作为方法调用时，`this`指向方法所属的对象
* 函数单独被调用时(没有所属对象)，`this`指向全局对象，在严格模式中，`this`是`undefined`
* 函数作为构造函数被调用时`new f()`，`this`是构造函数新生成并默认返回的对象
* 函数使用`call`或`apply`方式被调用时，`this`由传入的参数决定

对于闭包来说，无论是`this`，还是`arguments`，都是在调用时来决定的，这些隐藏属性闭包无法保存，所以很常见的一种方法是把`this`保存在局部变量中
``` javascript
var o = {
    m: function () {
        var self = this;
        var f = function () {
            this === o; //=> false
            self === o; //=> true
        };
        f();
    }
};

o.m();
```

---

## 原型(Prototype)
  
js是一门面向对象的语言，但与继承于smalltalk的基于类的面向对象语言不同，js的面向对象使用来自self语言的*原型继承*的方法实现，在js中，只有对象(object)，而没有对象的模板(class)

题外话，smalltalk和self语言都产生于Xerox PARC(施乐的帕洛阿托研究中心)，这里也是点阵图，图形界面，激光打印和以太网的故乡，致敬一下:)

### 原型的实现

几乎每一个js对象都和另一个对象相关联，这另一个对象就是我们说的原型，每一个对象都从原型继承属性

所有字面量对象(literal object)都具有同一个原型，她是`Object.prototype`，通过构造器生成的对象原型则是构造器的`prototype`属性的值

对象的原型对象也有可能有自己的原型，形成了一条原型链(prototype chain)，直到`Object.prototype`为止，她没有原型

查询对象的属性时，从此对象本身向原型链上依次向上查询，这也称为*委托(Delegation)*

### 原型的操作

指定原型生成对象的方法有两种：

1. 使用构造器生成
2. 在ECMA5中，使用`Object.create()`方法

其他的指定原型方法我们将在后面讲述

获取一个对象原型则并不容易，方法也有两种:

1. 查询`o.constructor.prototype`来检测，不过往往原型上的construct属性会丢失掉
2. 在ECMA5中，使用`Object.getPrototypeOf()`方法

---

## 构造函数(Constructor)

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

## 类(Class)

在js中，代码复用的方法有传统的(基于类的)，也有现代的(基于原型的)，我们先看一下在js中，如何模拟传统面向语言中类的行为

``` javascript
// constructor
var Gadget = function (name) {this.name = name;};

//instance method
Gadget.prototype.setPrice = function (price) {this.price = price;};

//static method
Gadget.isShiny = function () {return "you bet";};

//static property
Gadget.IPHONE = new Gadget('iphone');
```

Douglas Crockford也提到过另一种方法来构建类，不过他本人并不提倡这种做法

首先Monkey patching(之后会再提到这个概念)一下函数对象，新增加`method()`方法
``` javascript
if (!Function.prototype.method) {
    Function.prototype.method = function (name, implementation) {
        this.prototype[name] = implementation;
        return this;
    };
}
```

构造类
``` javascript
var Gadget = function (name) {
    this.name = name;
}.
method('getName', function () {
    return this.name;
}).
method('setName', function (name) {
    this.name = name;
    return this;
});

var i = new Gadget('iphone');
i.getName(); // =>'iphone'
i.setName('ipad').getName(); // =>'ipad'
```

这里使用了链式方法(Chaining methods)，这在jQuery和DOM库中相当常见

---

## 封装(Encapsulation)

在javascript中的对象体系中，并没有私有变量这一概念，对象中的所有信息都是公开的和可修改的，至少在ECMA3中是如此，在ECMA5中，对象的每个属性是可以配置成不可修改或不可枚举的，不过，因为现在主要使用闭包作为封装内部数据的方法，所以ECMA5中的这些新的权限相关特性还没有得到广泛的应用。

使用对象字面量时的封装，利用了闭包的概念，并使用了上边提到的immediate function
``` javascript
var iphone = (function () {
    var name = 'iphone';
    return {
        getName: function () {return name;}
    };
}());
iphone.getName(); //=> iphone
```

使用构造函数时的封装，相对完整的例子
``` javascript
var Gadget = (function () {
    //private static member
    var _counter = 0;

    //private instance member
    var _name;
    
    //constructor
    var Con = function (name) { 
        _name = name; 
        _counter++;
    };

    // public API 
    Con.prototype = {
        constructor: Con,
        getName: function () {return _name;},
        getLastId: function () {return _counter;}
    };

    return Con;
}());

var iphone = new Gadget('iphone');
iphone.getName(); //=> iphone
iphone.getLastId(); //=> 1
var ipad = new Gadget('ipad');
iphone.getName(); //=> ipad
iphone.getLastId(); //=> 2
```

需要注意的是，如果封装的内部变量是对象的话，外部代码还是可以对其进行修改的，因为javascript的对象传递的都是引用，为了避免这种情况，可以返回内部私有对象的部分属性或是拷贝

---

## 类继承(Class Inheritance)

先定义一下父类和子类的构造函数
``` javascript
var Parent = function () {/*...*/)};
var Child = function () {/*...*/)};
```
简单直接的想法
``` javascript
Child.prototype = new Parent();
```
这种方式的缺点是Parent的实例成员也被继承了，我们希望只继承prototype中的公用部分
``` javascript
Child.prototype = Parent.prototype;
```
这种共享原型的缺点是`Child`的原型被更改时全部父对象都被影响

可以引入一个代理构造器来回避这个缺点
``` javascript
var F = function () {};
F.prototype = Parent.prototype;
Child.prototype = new F();
```

用一个完整的例子说明一下，我们用`klass`模拟实现通常面向对象语言中的类

使用起来是这样的

``` javascript
var Gadget = klass(null, {
    initialize: function (name) {
        this.name = name;
    },
    getName: function () {
        return this.name;
    }
});

(new Gadget('iphone')).getName(); //=> iphone

var Phone = klass(Gadget, {
    initialize: function (name) {},
    getName: function () {
        var name = Phone.super.getName.call(this);
        return "Phone: " + name;
    }
});

(new Phone('iphone')).getName(); //=> Phone: iphone
```

`klass`的实现
``` javascript
var klass = function (Parent, props) {
    var Child, F, i;
    
    Child = function () {
        //这里自动调用父类的构造方法
        if (Child.super && Child.super.hasOwnProperty("initialize")) 
            Child.super.initialize.apply(this, arguments);
        if (Child.prototype.hasOwnProperty("initialize")) 
            Child.prototype.initialize.apply(this, arguments);
    };
    
    Parent = Parent || Object;
    F = function () {};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.super = Parent.prototype;
    Child.prototype.constructor = Child;

    //获取所有定义的属性和方法
    for (i in props) {
        if (props.hasOwnProperty(i)) 
            Child.prototype[i] = props[i];
    };
    
    return Child;
};
```

这种基于类思想的代码复用在很多库中都实现了，用起来可以让程序员完全忘记原型的存在，不过我更倾向于使用其他方法进行代码复用，这样更高效和简单，也更贴近js语言的本质

---

## 原型继承(Prototype Inheritance)

这里不牵涉到类的概念，只是一个对象复用另外一个对象

``` javascript
if (!Object.create) {
    Object.create = function (o) {
        var F = function () {};
        F.prototype = o;
        return new F();
    };
}

var parent = {name: 'parent'};
var child = Object.create(parent);
child.name; //=> parent
```

这也是ECMA5中`Object.create`方法的简化实现

---

## 绑定(Bind)

除了完整地复用其他对象外，也可以只使用对象的指定方法，把一个对象的方法借给另一个对象使用，这种行为称为绑定

``` javascript
if (!Function.prototype.bind) {
    Function.prototype.bind = function(o) {
        var self = this, _bound_args = arguments;
        return function() {
            var args = [], i;
            for(i = 1; i < _bound_args.length; i++) args.push(_bound_args[i]);
            for(i = 0; i < arguments.length; i++) args.push(arguments[i]);
            return self.apply(o, args);
        };
    };
}

var one = {
    name: 'one',
    say: function (greet) { return greet + ", " + this.name; }
};

var two = {
    name: 'two'
};

(one.say.bind(two))('hello'); //=> hello, two
```

这是ECMA5中`bind`方法的简化实现，在客户端大量的事件回调中`bind`的应用相当普遍

顺带一提，包括上边的`method`方法和`bind`方法，js中所有的对象，都是开放的，可以在运行时动态地修改和添加新的行为，这种特性一般称为*monkey patching*

---

## 混入(Mixin)

Mixin是很多语言都有的概念，有时候是以多重继承的面貌展现的

在JavaScript中虽然没有内置Mixin功能，但实现很容易

``` javascript
var extend = function (parent, child) {
    var i;
    child = child || {};
    for (i in parent) 
        if (parent.hasOwnProperty(i)) 
            child[i] = parent[i];
    return child;
};
```

`extend`可以将一个对象的属性拷贝到另外一个对象里面去

需要注意的是，这里的拷贝是浅拷贝(shallow copy)，即只拷贝引用，一旦子对象的属性(这个属性本身是对象)被更改的话，父对象也会被更改

解决方法是如果需要的话就进行深拷贝(deep copy)，遍历获取对象的属性进行clone

jQuery库的`$.extend`方法是一个比较完善的实现，支持浅拷贝，深拷贝，甚至多个对象的Mixin

---

## 函数化继承(Functional Inheritance)

利用js的动态特性，使用mixin类似的方法也可以实现继承，这里没有用到构造函数和原型，来自*javascript, the good parts*
``` javascript
var mammal = function(spec) {
    var self = {};
    self.get_name = function() {return spec.name;};
    self.says = function() {return spec.saying || '';};
    return self;
};

var myMammal = mammal({name: "Herb"});

var cat = function(spec) {
    spec.saying = spec.saying || 'meow';
    var self = mammal(spec);
    self.get_name = function() {
        return self.says() + ' ' + spec.name;
    };
    return self;
};

var myCat = cat({name: "Henri"});

Object.prototype.superior = function(name) {
    var self = this;
    return function() {
        return self[name].apply(self, arguments);
    };
};

var coolcat = function(spec) {
    var self = cat(spec);
    var super_get_name = self.superior('get_name');
    self.get_name = function() {
        return 'like' + super_get_name() + ' baby';
    };
    return self;
};

var myCoolCat = coolcat({name: 'Bix'});
myCoolCat.get_name();
```

---

## 模块(Module)

js中，通常使用一个全局对象作为namespace，简单直接，在内部，可以使用上边封装部分提到的方法构建对象

``` javascript
var MYAPP = {};

MYAPP.array = (function () {
    //dependencies
    var myobj = MYAPP.object,
        mystring = MYAPP.string;

    //private properties
    var array_string = "[object Array]",
        ops = Object.prototype.toString;

    //public API
    return {
        isArray: function (a) {
            return ops.call(a) === array_string;
        }
    };
}());
```

这种构建功能模块的方式相当普遍，也可以构建出构造函数
``` javascript
MYAPP.array = (function () {
    //constructor
    var Con = function (o) { this.elements = this.toArray(o); };

    // public API -- prototype
    Con.prototype = {
        constructor: MYAPP.Array,
        toArray: function (obj) {
            for (var i = 0, a = [], len = obj.length; i < len; i += 1) 
                a[i] = obj[i];
            return a;
        }
    };

    return Con;
}());
```
---

## 柯里化(Curry)

js中由于闭包的存在，很多函数式编程的模式也可以利用，比如实现函数的curry化功能，即生成一个绑定部分参数的新函数，这在函数式编程中很常见。

Curry这个名称来自于数学家Haskell Curry(著名的Haskell语言也来自于他)

``` javascript
if (!Function.prototype.curry) {
    Function.prototype.curry = function () {
        //arguments并不是真正的数组，所以没有concat方法，我们使用了slice将其转换成了数组
        var slice = Array.prototype.slice,
            args = slice.apply(arguments),
            self = this;
        return function () {
            return self.apply(null, args.concat(slice.apply(arguments)));
        };
    };
}

var add = function (x, y) {return x+y;}
var add1 = add.curry(1);
add1(2); //=> 3
```

---

## 结语

真正掌握一门语言，就需要深入这门语言去编程，这意味着去挖掘语言中最宝贵的特质，使用语言表达能力所限制之内的最简介优雅的方式进行逻辑表达，在js中，最好的部分是对象字面量，闭包，和原型，在允许的情况下，尽量使用这些精华去进行架构设计，而不是为了省事和更快上手，去模拟其他语言的特性

本文只是js编程的基础和起点，在之后会逐渐介绍*coffee script*，*jQuery*，*backbone*等预处理语言，库，与框架，很多时候，这些框架给出的实现，都是非常优雅和便利的，但是要更深入地了解原因，以至于编写自己的扩展和库，这些基础知识是必不可少的

---
