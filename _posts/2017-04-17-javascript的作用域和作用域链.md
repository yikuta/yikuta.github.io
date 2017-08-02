---
layout: post
section-type: post
title: 作用域和作用域链
category: js
#tags: [ 'tutorial' ]  #给帖子加标签
#<span>&nbsp;</span>
#<pre><code>

#</code></pre>
#<code></code>
---

##  前言

作用域是每种计算机语言最重要的基础之一，当然它也是JavaScript最重要的概念之一。要想真正的深入了解JavaScript，了解JavaScript的作用域链非常必要。

##  函数的作用域

<pre><code>
var scope="global";  
function t(){  
    console.log(scope);  
    var scope="local"  
    console.log(scope);  
}  
t(); 
</code></pre>
上诉代码，第一句打印的是：<code>undefined</code>，而不是<code>global</code>；第二句打印的是：<code>local</code>。
你可能会认为第一句会打印：<code>undefined</code>，因为代码还没执行<code>var scope="local"</code>,所以会打印<code>global</code>。
这想法完全没错，但是在javascript中，函数作用域与我们所熟知的其他面向对象语言的块级作用域要区分开来。
在其他面对对象语言，每段代码都具有各自的作用域，而变量在声明它们的代码段之外是不可访问的。而javascript是没有块级作用域的，而是函数作用域。

<br>

所谓函数作用域就是说：变量在声明它们的函数体以及函数体嵌套的任意函数体内都能访问到。所以，上诉代码重写如下：
<pre><code>
var scope="global";  
function t(){
    var scope;
    console.log(scope);  
    scope="local"  
    console.log(scope);  
}  
t(); 
</code></pre>
我们可以看到，由于函数作用域的特性，局部变量在整个函数体内都是能访问的，该变量将被变量声明提升到函数体顶部（详情看另一篇<a href="/js/2017/04/13/javascript%E7%9A%84%E5%8F%98%E9%87%8F%E6%8F%90%E5%8D%87.html">变量提升</a>），同时变量初始化还在原来的位置。

为啥说javascript没有块级作用域呢，看下面代码：
<pre><code>
var name="global";  
if(true){  
    var name="local";  
    console.log(name)  
}  
console.log(name);  
</code></pre>
打印的都是<code>local</code>，如果有块级作用域，明显if语句将创建局部变量<code>name</code>，并不会修改全局<code>name</code>，可是并没有这样，所以javascript没有块级作用域。
现在很好理解为什么会先打印<code>undefined</code>，然后再打印<code>local</code>。

###  ES6 let

用上面代码，把<code>var</code>变量声明换成es6的<code>let</code>声明呢，
<pre><code>
var name="global";  
if(true){  
    let name="local";  
    console.log(name)  
}  
console.log(name);  
</code></pre>
这时候就是先打印<code>local</code>再打印<code>global</code>。这就是ES6中新特性：局部变量声明<code>let</code>（详情看另一篇<a href="/es6/2017/03/30/ES6%E5%B8%B8%E7%94%A8%E6%96%B0%E7%89%B9%E6%80%A7.html">ES6常用新特性</a> ）。

##  变量作用域

<pre><code>
function t(flag){  
    if(flag){  
        s="ifscope";  
        for(var i=0;i<2;i++)   
            ;  
    }  
    console.log(i);  
}  
t(true);  
console.log(s);   
</code></pre>
上诉代码，if语句中变量<code>s</code>没有使用<code>var</code>进行变量声明。最后程序会打印<code>ifscope</code>。
这是因为js中没有用<code>var</code>声明的变量都是全局变量，而且是顶层对象的属性。所以你执行<code>console.log(window.s)</code>也会打印<code>ifscope</code>。

<br>

当使用<code>var</code>声明一个变量时，创建的这个属性时不可配置的，也就是说无法通过<code>delete</code>运算符删除。
<ul>
  <li><code>var name = 1</code> —— 不可删除</li>
  <li><code>sex = 'man'</code> —— 可删除</li>
  <li><code>this.age = 22</code> —— 可删除</li>
</ul>

##  作用域链

<pre><code>
name="tom";  
function test(){  
    var name="jerry";  
    function show(){  
        var name="selly";  
        console.log(name);  
    }  
    function say(){  
        console.log(name);  
    }  
    show();  
    say();  
}  
test();     
</code></pre>
当执行<code>show()</code>时，将创建函数show的执行环境（调用对象），并将该对象置于链表开头，然后将函数test的调用对象链接在之后，最后是全局对象。然后从链表开头寻找变量<code>name</code>，很明显<code>name</name>打印出<code>selly<code>。
但是执行<code>say()</code>,作用域链是：<code>say()->test()->window</code>，所以<code>name</code>打印<code>jerry</code>。

下面看一个容易犯错的例子：

<pre><code>
<html>  
<head>  
<script type="text/javascript">  
function buttonInit(){  
    for(var i=1;i<4;i++){  
        var b=document.getElementById("button"+i);  
        b.addEventListener("click",function(){ alert("Button"+i);},false);  
    }  
}  
window.onload=buttonInit;  
</script>  
</head>  
<body>  
<button id="button1">Button1</button>  
<button id="button2">Button2</button>  
<button id="button3">Button3</button>  
</body>  
</html>    
</code></pre>

当文档加载完毕，给按钮注册点击事件，当我们点击按钮时，会弹出什么呢？
结果是，无论你点击哪一个按钮都会弹出<code>Button4</code>。这是因为，当注册事件结束后，函数<code>buttonInit()</code>i的值已经为4了，当点击按钮时，事件触发<code>function(){ alert("Button"+i);}</code>这个匿名函数，根据作用域链，所以到函数<code>buttonInit()</code>中找。
如果要解决上诉问题，ES5中使用闭包：
<pre><code>
<script type="text/javascript">  
function buttonInit(){  
    function test(i) {
      var onclick = function(){
        alert("Button" + i);
      }
      return onclick;
    }
    var b=document.querySelectorAll("button");  
    for(var i=1;i<b.length + 1;i++){  
        b[i - 1].addEventListener("click", test(i), false);  
    }
}  
window.onload=buttonInit;  
</script> 
</code></pre>

在有了ES6的let局部变量声明后，我们只需将for的循环条件声明改成let：
<pre><code>
<script type="text/javascript">  
function buttonInit(){  
    for(let i=1;i<4;i++){  
        var b=document.getElementById("button"+i);  
        b.addEventListener("click",function(){ alert("Button"+i);},false);  
    }  
}  
window.onload=buttonInit;  
</script> 
</code></pre>

##  with语句

说到作用域链，不得不说with语句。with语句主要用来临时扩展作用域链，将语句中的对象添加到作用域的头部。
<pre><code>
person={name:"tom",age:22,height:175,wife:{name:"jerry",age:21}};  
with(person.wife){  
    console.log(name);  
}
</code></pre>
with语句将<code>person.wife</code>添加到当前作用域链的头部，所以输出的就是：<code>jerry</code>。
with语句结束后，作用域链恢复正常。
