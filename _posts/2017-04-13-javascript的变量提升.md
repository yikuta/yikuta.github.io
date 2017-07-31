---
layout: post
section-type: post
title: 变量提升
category: js
#tags: [ 'tutorial' ]  #给帖子加标签
#<span>&nbsp;</span>
#<pre><code>

#</code></pre>
#<code></code>
---

##  前言

大部分编程语言都是先声明变量再使用，但是在js中，有些不太一样：
<pre><code>
console.log(a); // undefined
var a = 1;
</code></pre>
上诉代码是合法的，正常输出<code>undefined</code>而非报错<code>Uncaught ReferenceError: a is not defined</code>。就是因为声明变量提升。

##  变量声明

####  语法：
<pre><code>
var varname1 [= value1 [, varname2 [, varname3 ... [, varnameN]]]];
</code></pre>
变量名可以是任意合法标识符；值可以是任意合法表达式。

####  重点：
<ul>
  <li>变量声明，不管在哪里发生（声明），都会在任意代码执行前处理。</li>
  <li>以<code>var</code>声明的变量的作用域就是当前执行上下文（execution context），即某个函数，或者全局作用域（声明在函数外）。</li>
  <li>赋值给未声明的变量，当执行时会隐式创建全局变量（成为global的属性）。</li>
</ul>

####  声明变量和未声明变量的区别：
<ul>
  <li>声明变量通常是局部的，未声明变量通常全局的。</li>
  <li>声明变量在任意代码执行前创建，未声明变量直到赋值时才存在。</li>
  <li>声明变量是execution context（function/global）的non-configurable 属性，未声明变量则是configurable。</li>
</ul>
在<code>es5 strict mode</code>，给未声明的变量赋值将报错。

##  定义函数（Defining functions）

定义一个函数有两种方式：函数声明（function definition/declaration/statement）和函数表达式（ function expression）。

####  函数声明

语法：<code>function name(arguments) {}</code>
对参数而言，primitive parameter是传值，对象是传引用。

####  函数表达式

语法：<code>var fun = function (arguments) {}</code>
函数表达式中函数可以不需要名字，即匿名函数。

####  其他

还可以用<code>Function</code>构造函数来创建函数。
在<strong>函数内部</strong>引用函数本身有三种方式。比如<code>var foo = function bar(){};</code>
<ul>
  <li>函数名字，即<code>bar()</code></li>
  <li><code>arguments.callee()</code></li>
  <li><code>foo()</code></li>
</ul>

##  声明提升

上面提到，<code>var</code>声明的变量会在任意代码之前处理，这意味着在任意地方声明变量都等同于在顶部声明————即声明提升。上面也同时强调了函数定义，因为声明提升中，需要综合考虑一般变量和函数。

在javascript中，一个变量名进入作用域的方式有4种：
<ol>
  <li><strong>Language-defined</strong>：所有的作用域默认都会给出 <code>this</code> 和 <code>arguments</code> 两个变量名（global没有<code>arguments</code>）;</li>
  <li><strong>Formal parameters</strong>（形参）：函数有形参，形参会添加到函数的作用域中;</li>
  <li><strong>Function declarations</strong>（函数声明）：如 <code>function foo() {}</code>;</li>
  <li><strong>Variable declarations</strong>（变量声明）：如 <code>var foo</code>，包括_函数表达式_。</li>
</ol>

<strong>函数声明和变量声明总是会被移动（即hoist）到它们所在的作用域的顶部</strong>（这对你是透明的）。
<strong>而变量的解析顺序（优先级），与变量进入作用域的4种方式的顺序一致。</strong>

下面一个详细的例子：
<pre><code>
function test(args) {
  console.log(args);  //args是形参，不会被重新定义
  console.log(a);   //因为函数声明比变量声明优先级高，所以这里是a函数
  var args = 'hello';  //var args;变量声明被忽略，args = 'hello'被执行
  var a = 10;   //var a;被忽略，a = 10被执行，a变成number
  function a() {
    console.log('fun');
  }   //被提升到作用域顶部
  console.log(a);   //输出10
  console.log(args);  //输出hello
}
test('hi');

/*输出：
hi
function a() {
  console.log('fun');
}
10
hello
*/
</code></pre>
