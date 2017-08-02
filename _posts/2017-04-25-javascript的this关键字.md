---
layout: post
section-type: post
title: this关键字
category: js
#tags: [ 'tutorial' ]  #给帖子加标签
#<span>&nbsp;</span>
#<pre><code>

#</code></pre>
---

##  前言

<code>this</code>关键字是js中最令人头疼的机制之一。<code>this</code>是非常特殊的关键词标识符，在每个函数的作用域自动创建，但是它指向什么（对象），很容易出错。
当函数被调用时，一个activation record（即 execution context）被创建。这个record包含的信息：函数在哪儿被调用，函数怎么调用的，参数等。record的一个属性就是<code>this</code>，指向函数执行期间的<code>this</code>对象。
<ul>
  <li><code>this</code>不是author-time binding，而是 runtime binding。</li>
  <li><code>this</code>的上下文基于函数调用的情况。和函数在哪定义无关，而和函数怎么调用有关。</li>
</ul>

## <code>this</code>在具体情况下的分析

### Global context

在全局上下文（任何函数外），<code>this</code>指向全局对象。

<pre><code>
console.log(this === window); // true
</code></pre>

### Function context

在函数内部时，<code>this</code>由函数怎么调用来确定。

####  Simple call

简单调用，即独立函数调用。由于<code>this</code>没有通过<code>call</code>来指定，且<code>this</code>必须指向对象，那么默认就指向全局对象。
<pre><code>
function f1(){
  return this;
}
f1() === window; // global object
</code></pre>

在严格模式下，<code>this</code>保持进入execution context时被设置的值。如果没有设置，那么默认是<code>undefined</code>。它可以被设置为任意值**（包括<code>null/undefined/1</code>等等基础值，不会被转换成对象）**。
<pre><code>
function f2(){
  "use strict"; // see strict mode
  return this;
}
f2() === undefined;
</code></pre>

####  Arrow functions

在箭头函数中，<code>this</code>由词法/静态作用域设置（set lexically）。它被设置为包含它的execution context的<code>this</code>，并且不再被调用方式影响（call/apply/bind）。
<pre><code>
var globalObject = this;
var foo = (() => this);
console.log(foo() === globalObject); // true

// Call as a method of a object
var obj = {foo: foo};
console.log(obj.foo() === globalObject); // true

// Attempt to set this using call
console.log(foo.call(obj) === globalObject); // true

// Attempt to set this using bind
foo = foo.bind(obj);
console.log(foo() === globalObject); // true
</code></pre>

####  As an object method

当函数作为对象方法调用时，<code>this</code>指向该对象。
<pre><code>
var o = {
  prop: 37,
  f: function() {
    return this.prop;
  }
};
console.log(o.f()); // logs 37
</code></pre>
<strong>this on the object's prototype chain</strong>
原型链上的方法根对象方法一样，作为对象方法调用时<code>this</code>指向该对象。

####  构造函数

在构造函数（函数用<code>new</code>调用）中，<code>this</code>指向要被constructed的新对象。

####  call和apply

<code>Function.prototype</code>上的<code>call</code>和<code>apply</code>可以指定函数运行时的<code>this</code>。
<pre><code>
function add(c, d){
  return this.a + this.b + c + d;
}

var o = {a:1, b:3};
add.call(o, 5, 7); // 1 + 3 + 5 + 7 = 16
add.apply(o, [10, 20]); // 1 + 3 + 10 + 20 = 34
</code></pre>
注意，当用<code>call</code>和<code>apply</code>而传进去作为<code>this</code>的不是对象时，将会调用内置的<code>ToObject</code>操作转换成对象。所以<code>4</code>将会装换成<code>new Number(4)</code>，<strong>而<code>null/undefined</code>由于无法转换成对象，全局对象将作为<code>this</code></strong>。

####  bind

ES5引进了<code>Function.prototype.bind</code>。<code>f.bind(someObject)</code>会创建新的函数（函数体和作用域与原函数一致），但<code>this</code>被永久绑定到<code>someObject</code>，不论你怎么调用。

####  As a DOM event handler

<code>this</code>自动设置为触发事件的dom元素。
