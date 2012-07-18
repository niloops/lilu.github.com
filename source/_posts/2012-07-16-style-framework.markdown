---
layout: post
title: "Twitter Bootstrap及其他样式框架"
date: 2012-07-16 12:22
comments: true
categories: 
---

[Twitter Bootstrap](http://twitter.github.com/bootstrap/)(以下简称*Bootstrap*)是一个web开发的前端工具库，由Twitter的*Mark Otto*和*Jacob Thornton*创建，在很短的时间内，就成为了github上最热门的项目，不过，目前web开发已经进入相对成熟的后期了，可以找到很多出色的前端样式框架，虽然很方便，不过对于一个新项目来说，不可避免地会陷入选择性障碍，今天就简单总结一下我所知的主流现状：

## SASS and LESS

不管是[SASS](http://sass-lang.com/)，还是[LESS](http://lesscss.org)，都可以视为一种基于CSS之上的高级语言，其目的是使得CSS开发更灵活和更强大，这两者我的感觉是对于程序员来说，SASS的功能要远比LESS强大，基本可以说是一种真正的编程语言了，而对于设计师，LESS则相对清晰明了，[这里](http://css-tricks.com/sass-vs-less/)是Chris Coyier写的一篇关于SASS和LESS的背靠背对比，可以说是相当中肯的(评论也相当有料喔)。当然，如果使用Rails之类的框架，基于SASS是会来的更方便一些。

---

## Compass and Blueprint

SASS和[Compass](http://compass-style.org/)的关系，在很多人来看类似于ruby和rails，compass基于SASS，是一个真正意义上的编程框架，提供了大量的mixin(可理解为函数库)，无论是对css3繁杂的多浏览器写法的简化支持，还是实现各种常见功能的helper，都是强大而丰富的。另外，包括Scott Davis和Eric Meyer的核心团队，也可以说是全明星组合。

[Blueprint](http://www.blueprintcss.org/)是一套预定义的样式，包括对大部分常用web交互组件的渲染，并且有一个强大的栅格系统(grid system)，即使不懂设计的程序员，也可以使用blueprint的默认样式做出很漂亮的页面。

Blueprint和Compass，是一个分工很明确的组合，前者负责样式渲染，后者则是基础框架和模块，可以说，在bootstrap诞生之前，是web开发首选的黄金组合。

---

## HTML5 Boilerplate

[HTML5 Boilerplate](http://html5boilerplate.com/)项目(以下简称h5bp)则如同名字一样，实现的是一个web页面的标准模板，尤其针对html5进行了全面优化，同时也对老浏览器向后兼容，基本上来说，h5bp与样式相关的主要部分，是compass的一个子集，不过h5bp并不只限于css，还默认引入了很多很好的js开发库，包括[Modernizr](http://modernizr.com/)和Jquery，再加上一个标准化的index.html模板

h5bp是这里提到的所有框架中使用起来最方便的，当然受功能限制，她最适用的目标是单页web app或者静态页面，对于复杂的项目来说，需要和其他框架做互补。

---

## Twitter Bootstrap

新兴而野心十足的Bootstrap跟上述又都不同，她是基于LESS的一套前端工具库，意图非常明显，想以一个项目，整合Compass，Blueprint，h5bp的目标功能，成为web前端的一站式解决方案。

* 一套完整的基础css模块，但不如compass丰富和强大
* 一套预定义样式表，也包括一个栅格系统，和blueprint提供的差不多，只是设计风格不一样
* 一组基于Jquery的js交互插件，这是Bootstrap真正强大的地方，也是她严格意义上可以取代Blueprint的原因所在，这些非常不错的小插件，包括对话框，下拉导航等等，不但功能完善，而且十分精致，正在成为众多jquery项目的默认设计标准。

特别提一下，Bootstrap使用[Normalize.css](http://necolas.github.com/normalize.css/)来进行Reset CSS，这一项目已经成为了事实标准(超过Compass的Eric meyer 2.0)，强烈推荐使用，另外前边说的h5bp也使用Normalize，因此，如果你在项目中同时使用了h5bp和Bootstrap， 请注意，**没有必要再引入h5bp的初始样式表style.css**

---

## So What?

说了一大堆，该来点结论了，目前对于web开发，尤其是由程序员进行的full stack开发，最好的组合是：

**SASS+Compass+Bootstrap**

这样既可以利用SASS强大的编程能力，Compass强大的底层函数，又可以获取Bootstrap丰富的UI组件支持。

只是，Bootstrap是基于LESS的，要让她们协同工作，需要一个SASS的Bootstrap移植版本，幸亏github上从来不缺这类项目，当前最好的一个是[bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass)

---
