---
layout: post
section-type: post
title: ES5实现ES6的extends
category: js
#tags: [ 'tutorial' ]  #给帖子加标签
#<span>&nbsp;</span>
#<pre><code>

#</code></pre>
---

## 构造函数、原型对象和实例之间的关系
<pre><code>
function F(){};
var f = new F();
// 实例原型链
f.__proto__ === F.prototype;    // true
F.prototype.__proto__ === Object.prototype;   // true
Object.prototype.__proto__ === null;    // true
</code></pre>

## <code>ES6 extends</code>继承做了什么操作
我们先看看这段包含静态方法的<code>ES6</code>继承代码：
<pre><code>
// ES6
class Parent{
  constructor(name){
    this.name = name;
  }
  static sayHello(){
    console.log('hello');
  }
  sayName(){
    console.log('my name is ' + this.name);
    return this.name;
  }
}
class Child extends Parent{
  constructor(name, age){
    super(name);
    this.age = age;
  }
  sayAge(){
    console.log('my age is ' + this.age);
    return this.age;
  }
}
let parent = new Parent('Parent');
let child = new Child('Child', 18);
console.log('parent: ', parent);  // parent:  Parent {name: "Parent"}
Parent.sayHello();  // hello
parent.sayName();   // my name is Parent
console.log('child: ', child);  // child:  Child {name: "Child", age: 18}
Child.sayHello();   // hello
child.sayName();  // my name is Child
child.sayAge();   // my age is 18
</code></pre>
其中这段代码里的原型链。
<pre><code>
// 实例原型链
child.__proto__ === Child.prototype;  // true
Child.prototype.__proto__ === Parent.prototype;   // true
Parent.prototype.__proto__ === Object.prototype;  // true
Object.prototype.__proto__ === null;  // true
</code></pre>
结合代码可以知道，<code>ES6 extends</code>继承，主要就是：
<ul>
  <li>把子类构造函数(<code>Child</code>)的原型(<code>__proto__</code>)指向了父类构造函数(<code>Parent</code>)，</li>
  <li>把子类实例<code>child</code>的原型对象(<code>Child.prototype</code>)的原型(<code>__proto__</code>)指向了父类<code>parent</code>的原型对象(<code>Parent.prototype</code>)。</li>
</ul>

## <code>new</code>、<code>Object.create</code>和<code>Object.setPrototypeOf</code>可以设置<code>__proto__</code>
#### <code>new</code>做了什么：
<ul>
  <li>创建了一个全新的对象。</li>
  <li>链接到原型: <code>obj.__proto__ = Con.prototype</code></li>
  <li>生成的新对象会绑定到函数调用的<code>this</code>: <code>apply</code></li>
  <li>返回新对象</li>
</ul>
#### <code>Object.create</code>ES5提供的
<code>Object.create(proto, [propertiesObject])</code>方法创建一个新对象，使用现有的对象来提供新创建的对象的<code>__proto__</code>。它接收两个参数，不过第二个可选参数是属性描述符（不常用，默认是<code>undefined</code>）。
<pre><code>
// 简版：也正是应用了new会设置__proto__链接的原理。
if(typeof Object.create !== 'function'){
  Object.create = function(proto){
    function F() {}
    F.prototype = proto;
    return new F();
  }
}
</code></pre>
#### <code>Object.setPrototypeOf</code>ES5提供的
<code>Object.setPrototypeOf()</code>方法设置一个指定的对象的原型（即, 内部<code>[[Prototype]]</code>属性）到另一个对象或<code>null</code>。<code>Object.setPrototypeOf(obj, prototype)</code>
<pre><code>
`ployfill`
// 仅适用于Chrome和FireFox，在IE中不工作：
Object.setPrototypeOf = Object.setPrototypeOf || function (obj, proto) {
  obj.__proto__ = proto;
  return obj; 
}
</code></pre>

## ES6的<code>extends</code>的ES5版本实现
知道了ES6<code>extends</code>继承做了什么操作和设置<code>__proto__</code>的知识点后，把上面ES6例子的用ES5就比较容易实现了，也就是说实现<stronger>寄生组合式继承</stronger>，简版代码就是：
<pre><code>
// ES5 实现ES6 extends的例子
function Parent(name){
  this.name = name;
}
Parent.sayHello = function(){
  console.log('hello');
}
Parent.prototype.sayName = function(){
  console.log('my name is ' + this.name);
  return this.name;
}

function Child(name, age){
  // 相当于super
  Parent.call(this, name);
  this.age = age;
}
// new
function object(){
  function F() {}
  F.prototype = proto;
  return new F();
}
function _inherits(Child, Parent){
  // Object.create
  Child.prototype = Object.create(Parent.prototype);
  // __proto__
  // Child.prototype.__proto__ = Parent.prototype;
  Child.prototype.constructor = Child;
  // ES6
  // Object.setPrototypeOf(Child, Parent);
  // __proto__
  Child.__proto__ = Parent;
}
_inherits(Child,  Parent);
Child.prototype.sayAge = function(){
  console.log('my age is ' + this.age);
  return this.age;
}
var parent = new Parent('Parent');
var child = new Child('Child', 18);
console.log('parent: ', parent); // parent:  Parent {name: "Parent"}
Parent.sayHello(); // hello
parent.sayName(); // my name is Parent
console.log('child: ', child); // child:  Child {name: "Child", age: 18}
Child.sayHello(); // hello
child.sayName(); // my name is Child
child.sayAge(); // my age is 18
</code></pre>
我们完全可以把上述ES6的例子通过<code>babeljs</code>转码成ES5来查看，更严谨的实现。
<code><pre>
"use strict";
// 主要是对当前环境支持Symbol和不支持Symbol的typeof处理
function _typeof(obj) {
  if (typeof Symbol === "function" && typeof Symbol.iterator === "symbol") {
    _typeof = function _typeof(obj) {
      return typeof obj;
    };
  } else {
    _typeof = function _typeof(obj) {
      return obj && typeof Symbol === "function" && obj.constructor === Symbol && obj !== Symbol.prototype ? "symbol" : typeof obj;
    };
  }
  return _typeof(obj);
}
// _possibleConstructorReturn 判断Parent。call(this, name)函数返回值 是否为null或者函数或者对象。
function _possibleConstructorReturn(self, call) {
  if (call && (_typeof(call) === "object" || typeof call === "function")) {
    return call;
  }
  return _assertThisInitialized(self);
}
// 如何 self 是void 0 （undefined） 则报错
function _assertThisInitialized(self) {
  if (self === void 0) {
    throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
  }
  return self;
}
// 获取__proto__
function _getPrototypeOf(o) {
  _getPrototypeOf = Object.setPrototypeOf ? Object.getPrototypeOf : function _getPrototypeOf(o) {
    return o.__proto__ || Object.getPrototypeOf(o);
  };
  return _getPrototypeOf(o);
}
// 寄生组合式继承的核心
function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function");
  }
  // Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。 
  // 也就是说执行后 subClass.prototype.__proto__ === superClass.prototype; 这条语句为true
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass,
      writable: true,
      configurable: true
    }
  });
  if (superClass) _setPrototypeOf(subClass, superClass);
}
// 设置__proto__
function _setPrototypeOf(o, p) {
  _setPrototypeOf = Object.setPrototypeOf || function _setPrototypeOf(o, p) {
    o.__proto__ = p;
    return o;
  };
  return _setPrototypeOf(o, p);
}
// instanceof操作符包含对Symbol的处理
function _instanceof(left, right) {
  if (right != null && typeof Symbol !== "undefined" && right[Symbol.hasInstance]) {
    return right[Symbol.hasInstance](left);
  } else {
    return left instanceof right;
  }
}

function _classCallCheck(instance, Constructor) {
  if (!_instanceof(instance, Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}
// 按照它们的属性描述符 把方法和静态属性赋值到构造函数的prototype和构造器函数上
function _defineProperties(target, props) {
  for (var i = 0; i < props.length; i++) {
    var descriptor = props[i];
    descriptor.enumerable = descriptor.enumerable || false;
    descriptor.configurable = true;
    if ("value" in descriptor) descriptor.writable = true;
    Object.defineProperty(target, descriptor.key, descriptor);
  }
}
// 把方法和静态属性赋值到构造函数的prototype和构造器函数上
function _createClass(Constructor, protoProps, staticProps) {
  if (protoProps) _defineProperties(Constructor.prototype, protoProps);
  if (staticProps) _defineProperties(Constructor, staticProps);
  return Constructor;
}

// ES6
var Parent = function () {
  function Parent(name) {
    _classCallCheck(this, Parent);
    this.name = name;
  }
  _createClass(Parent, [{
    key: "sayName",
    value: function sayName() {
      console.log('my name is ' + this.name);
      return this.name;
    }
  }], [{
    key: "sayHello",
    value: function sayHello() {
      console.log('hello');
    }
  }]);
  return Parent;
}();

var Child = function (_Parent) {
  _inherits(Child, _Parent);
  function Child(name, age) {
    var _this;
    _classCallCheck(this, Child);
    // Child.__proto__ => Parent
    // 所以也就是相当于Parent.call(this, name); 是super(name)的一种转换
    // _possibleConstructorReturn 判断Parent.call(this, name)函数返回值 是否为null或者函数或者对象。
    _this = _possibleConstructorReturn(this, _getPrototypeOf(Child).call(this, name));
    _this.age = age;
    return _this;
  }
  _createClass(Child, [{
    key: "sayAge",
    value: function sayAge() {
      console.log('my age is ' + this.age);
      return this.age;
    }
  }]);
  return Child;
}(Parent);

var parent = new Parent('Parent');
var child = new Child('Child', 18);
console.log('parent: ', parent); // parent:  Parent {name: "Parent"}
Parent.sayHello(); // hello
parent.sayName(); // my name is Parent
console.log('child: ', child); // child:  Child {name: "Child", age: 18}
Child.sayHello(); // hello
child.sayName(); // my name is Child
child.sayAge(); // my age is 18
</code></pre>