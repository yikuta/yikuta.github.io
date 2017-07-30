---
layout: post
section-type: post
title: 原型对象与原型链
category: js
#tags: [ 'tutorial' ]  #给帖子加标签
#<span>&nbsp;</span>
#<pre><code>

#</code></pre>
---

##  前言

<span>&nbsp;&nbsp;</span>javascript作为一个弱语言，没有像其他面向对象语言中的类的继承模式，而是使用的prototype模式。虽然ES6中，把class关键字加到了其中，但是这个class只是一个语法糖，仅仅是建立在旧的原型系统上的。

##  经常会这么写
<pre><code>
  function Person () {
    this.name = 'John';
  }
  var person = new Person();
  Person.prototype.say = function() {
    console.log('Hello,' + this.name);
  };
  person.say(); //Hello,John
</code></pre>
<span>&nbsp;&nbsp;</span>上面代码很简单，给Person原型对象定义了一个公共的say方法，虽然是在实例化person之后，但是因为原型方法在调用之前已经声明，因此实例化出来的每个对象都有该方法。所以：<strong>原型对象的用途是为了每个实例化对象存储共享方法和属性，它仅是一个普通对象而已。并且所有的实例化对象是共享一个原型对象的，因此有别于实例化方法和属性，原型对象仅有一份。</strong>
<pre><code>
  person.say == new Person().say
</code></pre>

##  可能也会这样写
<pre><code>
  function Person () {
    this.name = 'John';
  }
  var person = new Person();
  Person.prototype = {
    say: function() {
      console.log('Hello,' + this.name);
    }
  }
  person.say(); //person.say is not a function
</code></pre>
<span>&nbsp;&nbsp;</span>很不幸，person.say方法没有找到，报错了。其实这样写的出发点是好的：因为如果想要在原型对象上添加更多的方法和属性，我不得不每次都写一行Person.prototype，还不如直接写成一个object。但是这个例子恰好在构造实例化对象操作是在添加原型方法之前，这样就会造成问题：
<span>&nbsp;&nbsp;</span>当<small>var person = new Person()</small>时，Person.prototype为<small>Person()</small>（当然，内部还有constructor属性），即Person.prototype指向一个空对象{}。而对于实例person而言，其内部有一个原型链指针__proto__，该指针指向了Person.prototype指向的对象，即{}。接下来重置了Person的原型对象，使其指向了另一个对象，即<small>Object {say function}</small>，这时person.__proto__的指向还没有发生改变，它指向的{}对象里没有say方法，所有报错。
所以：<strong>在js中，对象调用一个方法时，会首先在该对象自身里寻找是否有该方法，若没有，则去原型链上去寻找，依次层层递进，这里说的原型链就是实例对象__proto__属性</strong>。
若想上诉例子成功运行，最简单有效的方法就是交换构造对象和重置对象原型对象的顺序，即
<pre><code>
  function Person () {
     this.name = 'John';
  }
  Person.prototype = {
      say: function() {
          console.log('Hello,' + this.name);
      }
  };
  var person = new Person();
  person.say();//person.say is not a function
</code></pre>

##  原型对象结构

<pre><code>
Function.prototype = {
    constructor : Function,
    __proto__ : parent prototype,
    some prototype properties: ...
  };
</code></pre>

##  总结

<img src="/img/blog/prototype-chain.png" alt="原型链">
<span>&nbsp;&nbsp;</span><strong>函数的原型对象constructor默认指向函数本身，原型对象除了有原型属性外，为了实现继承，还有一个原型链指针__proto__，该指针指向上一层的原型对象，而上一层的原型对象结构依然类似，这样利用__proto__一直指向Object的原型对象上，而Object的原型对象用Object.prototype.__proto__ == null表示原型链的最顶端，如此便形成了js的原型链继承，同时也解释了为什么所有js对象都具有Object的基本方法。</strong>
