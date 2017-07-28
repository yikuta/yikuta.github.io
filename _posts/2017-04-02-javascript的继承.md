---
layout: post
section-type: post
title: 继承
category: js
#tags: [ 'tutorial' ]  #给帖子加标签
#<span>&nbsp;</span>
#<pre><code>

#</code></pre>
---

## 继承的实现方式

我们先定义一个父类：
<pre><code>
  // 定义一个动物类
  function Animal (name) {
    // 属性
    this.name = name || 'Animal';
    // 实例方法
    this.sleep = function(){
      console.log(this.name + '正在睡觉！');
    }
  }
  // 原型方法
  Animal.prototype.eat = function(food) {
    console.log(this.name + '正在吃：' + food);
  };
</code></pre>

### 原型链继承
将父类的实例作为子类的原型。
<pre><code>
  function Cat(){ 
  }
  Cat.prototype = new Animal();
  Cat.prototype.constructor = Cat;
  Cat.prototype.name = 'cat';
  //　Test Code
  var cat = new Cat();
  console.log(cat.name);
  console.log(cat.eat('fish'));
  console.log(cat.sleep());
  console.log(cat instanceof Animal); //true 
  console.log(cat instanceof Cat); //true
</code></pre>
该继承方式特点：父类新增的原型方法，子类能访问；原型对象引用的属性是所有实例共享的；创建子类无法向父类传参。

### 构造继承
使用父类的构造函数来增强子类实例，相当于是复制父类实例属性，但是子类拿不到父类原型。
<pre><code>
  function Cat(name){
    Animal.call(this);
    this.name = name || 'Tom';
  }
  // Test Code
  var cat = new Cat();
  console.log(cat.name);
  console.log(cat.sleep());
  cat.eat('fish');  //报错：cat.eat is not a function
  console.log(cat instanceof Animal); // false
  console.log(cat instanceof Cat); // true
</code></pre>
该继承方式特点：解决了子类实例共享父类引用属性的问题；创建子类可以传参；只能继承父类的实例属性不能继承父类的原型属性和方法。

### 拷贝继承
<pre><code>
  function Cat(name){
    var animal = new Animal();
    for(var p in animal){
      Cat.prototype[p] = animal[p];
    }
    Cat.prototype.name = name || 'Tom';
  }
  // Test Code
  var cat = new Cat();
  console.log(cat.name);
  console.log(cat.sleep());
  console.log(cat instanceof Animal); // false
  console.log(cat instanceof Cat); // true
</code></pre>
该继承方式特点：只能获取到能for in的父类属性和方法。

### 组合继承
通过调用父类构造，继承父类的属性并保留传参的优点，然后通过将父类实例作为子类原型
<pre><code>
  function Cat(name){
    Animal.call(this);
    this.name = name || 'Tom';
  }
  Cat.prototype = new Animal();
  // Test Code
  var cat = new Cat();
  console.log(cat.name);
  console.log(cat.sleep());
  cat.eat('fish');  //Tom正在吃：fish
  console.log(cat instanceof Animal); // true
  console.log(cat instanceof Cat); // true
</code></pre>
该继承方式特点：继承了父类实例属性方法也继承了原型属性方法；不存在引用属性共享。

### 寄生组合继承
通过寄生方式，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化两次实例方法/属性，避免的组合继承的缺点
<pre><code>
  function Cat(name){
    Animal.call(this);
    this.name = name || 'Tom';
  }
  (function(){
    // 创建一个没有实例方法的类
    var Super = function(){};
    Super.prototype = Animal.prototype;
    //将实例作为子类的原型
    Cat.prototype = new Super();
  })();
  // Test Code
  var cat = new Cat();
  console.log(cat.name);
  console.log(cat.sleep());
  console.log(cat instanceof Animal); // true
  console.log(cat instanceof Cat); //true
</code></pre>
该继承方式特点：堪称完美，比较复杂。
