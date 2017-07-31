---
layout: post
section-type: post
title: 闭包（closure）
category: js
#tags: [ 'tutorial' ]  #给帖子加标签
#<span>&nbsp;</span>
#<pre><code>

#</code></pre>
---

闭包（closure）是Javascript语言的一个难点，也是它的特色，很多高级应用都要依靠闭包实现。
<br>
当function里嵌套function时，内部的function可以访问到外部function的变量。
<pre><code>
  function foo(x) {
    var tmp = 3;
    function bar(y) {
      console.log(x + y + (++tmp));
    }
    bar(10);
  }
  foo(2);
</code></pre>
无论执行多少次都会打印16，因为bar能访问到foo的参数x和tmp。但是这还不是闭包。当你return内部这个function时，就是一个闭包了。内部function会close-over外部function的变量直到内部function结束。
<br>
<pre><code>
  function foo(x) {
    var tmp = 3;
    return function(y) {
      console.log(x + y + (++tmp));
    }
  }
  var bar = foo(2); //此时的bar就是一个闭包
  bar(10);
</code></pre>
上诉代码最后也会打印16，虽然bar不直接处于foo内部作用域，但是bar还是能访问到x和tmp。并且，由于tmp存在于bar闭包内，没调用依次bar，tmp都会自加1。
<br>
上面的x是一个字面值，和js中其他字面值一样，当调用foo时，实参x的值被复制了一份，复制的那份作为foo的参数x。下面我们调用foo传递一个object，foo函数return也会引用最初的那个object。
<pre><code>
  function foo(x) {
  var tmp = 3;
  return function (y) {
      console.log(x + y + tmp);
      x.memb = x.memb ? x.memb + 1 : 1;
      alconsole.logert(x.memb);
      }
  }
  var age = new Number(2);
  var bar = foo(age); // bar 现在是一个引用了age的闭包
  bar(10);
</code></pre>

##  闭包的概念
这里借用<a href="http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html">阮一峰的理解</a>，就是：闭包是能够读取其他函数内部变量的函数。由于在Javascript语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成"定义在一个函数内部的函数"。所以，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。
<br>
代码段一：
<pre><code>
　　var name = "The Window";
　　var object = {
　　　　name : "My Object",
　　　　getNameFunc : function(){
　　　　　　return function(){
　　　　　　　　return this.name;
　　　　　　};
　　　　}
　　};
　　alert(object.getNameFunc()());
</code></pre>
代码段二：
<pre><code>
　　var name = "The Window";
　　var object = {
　　　　name : "My Object",
　　　　getNameFunc : function(){
　　　　　　var that = this;
　　　　　　return function(){
　　　　　　　　return that.name;
　　　　　　};
　　　　}
　　};
　　alert(object.getNameFunc()());
</code></pre>
结合上面两段代码，好好理解一下这句话：<strong>JavaScript中的函数运行在它们被定义的作用域里，而不是它们被执行的作用域里。而this对象的指向则是看被执行的作用域的。</strong>
