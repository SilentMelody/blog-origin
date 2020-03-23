---
title: 设计模式系列笔记-代理模式
date: 2019-09-17 23:24:11
tags: [JS, JavaScript, 设计模式]
categories: 开发
---

*写在前面：本系列文章内容为《JavaScript设计模式与开发实践》一书学习笔记，感谢作者曾探*
## 代理模式
代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问

代理模式的关键是，当客户不方便直接访问一个对象或者不满足需要的时候，提供一个替身对象来控制对这个对象的访问，客户实际上访问的是替身对象。替身对象对请求做出一些处理之后，再把请求转交给本体对象

例如 A 访问 B
非代理模式：A —> B
用代理模式：A —> C —> B

### 1.代理模式实现
举例：小明追求 A，B 为两人共同好友，小明让 B 帮他向 A 送花
```
var Flower = function() {};
var xiaoming = {
  sendFlower: function(target) {
    var flower = new Flower();
    target.receiveFlower(flower);
  }
};
var B = {
  receiveFlower: function(flower) {
    A.receiveFlower(flower);
  }
};
var A = {
  receiveFlower: function(flower) {
    console.log('收到花 ' + flower);
  }
};
xiaoming.sendFlower(B);
```
可以发现此处的代理没有任何作用，这时候我们添加设定，小明希望在 A 心情好的时候送花表白，但是不了解 A 的心情变化，所以让比较了解 A 的 B 帮忙送花
```
// 心情好的时候再送花
var Flower = function() {};
var xiaoming = {
  sendFlower: function(target) {
    var flower = new Flower();
    target.receiveFlower(flower);
  }
};
var B = {
  receiveFlower: function(flower) {
    A.listenGoodMood(function() { // 监听 A 的好心情
      A.receiveFlower(flower);
    });
  }
};
var A = {
  receiveFlower: function(flower) {
    console.log('收到花 ' + flower);
  },
  listenGoodMood: function(fn) {
    setTimeout(function() { // 假设 10 秒之后 A 的心情变好
      fn();
    }, 10000);
  }
};
xiaoming.sendFlower(B);
```

### 2.保护代理和虚拟代理
保护代理：代理 B 可以帮助 A 过滤掉一些请求，一些请求就可以直接在代理 B处被拒绝掉。这种代理叫作保护代理
虚拟代理：虚拟代理把一些开销很大的对象，延迟到真正需要它的时候才去创建，以送花举例，使用虚拟代理可以将买花这一操作放到需要的时候再进行，防止过早的买花，等 A 心情好的时候已经不新鲜了
```
var B = {
  receiveFlower: function(flower) {
    A.listenGoodMood(function() { // 监听 A 的好心情
      var flower = new Flower(); // 延迟创建 flower 对象
      A.receiveFlower(flower);
    });
  }
};
```
护代理用于控制不同权限的对象对目标对象的访问，但在 JavaScript 并不容易实现保护代理，因为我们无法判断谁访问了某个对象。而虚拟代理是最常用的一种代理模式

### 3.虚拟代理实现图片预加载
引入代理对象 proxyImage，在图片被真正加载好之前，页面中出现 loading 的 gif 动画, 来提示用户图片正在加载
```
var myImage = (function() {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  return {
    setSrc: function(src) {
      imgNode.src = src;
    }
  }
})();
var proxyImage = (function() {
  var img = new Image;
  img.onload = function() {
    myImage.setSrc(this.src);
  }
  return {
    setSrc: function(src) {
      myImage.setSrc('/loading.gif');
      img.src = src;
    }
  }
})();
proxyImage.setSrc('http://xxxx.jpg');
```
上面这个程序，我们并没有改变或者增加 MyImage 的接口，但是通过代理对象，实际上给系统添加了新的行为，符合开放—封闭原则的
给 img 节点设置 src 和图片预加载这两个功能，被隔离在两个对象里，它们可以各自变化而不影响对方

### 4.代理和本体接口的一致性
在客户看来，代理对象和本体是一致的， 代理接手请求的过程对于用户来说是透明的，用户并不清楚代理和本体的区别
作用：
- 用户可以放心地请求代理，他只关心是否能得到想要的结果
- 在任何使用本体的地方都可以替换成使用代理

如果代理对象和本体对象都为一个函数（函数也是对象），函数必然都能被执行，则可以认为它们也具有一致的“接口”
例如：
```
var myImage = (function() {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  return function(src) {
    imgNode.src = src;
  }
})();
var proxyImage = (function() {
  var img = new Image;
  img.onload = function() {
    myImage(this.src);
  }
  return function(src) {
    myImage('/loading.gif');
    img.src = src;
  }
})();
proxyImage('http://xxx.jpg');
```

### 5.代理模式其他应用
##### 1. 虚拟代理合并HTTP请求
可以通过一个代理函数来收集一段时间之内的请求， 最后一次性发送给服务器
##### 2. 缓存代理
编写一个简单的求乘积的程序
```
var mult = function(){
  console.log( '开始计算乘积' );
  var a = 1;
  for ( var i = 0, l = arguments.length; i < l; i++ ){
    a = a * arguments[i];
  }
  return a;
};
mult( 2, 3 ); // 输出：6
mult( 2, 3, 4 ); // 输出：24 
```

现在加入缓存代理函数：
```
var proxyMult = (function(){
  var cache = {};
  return function(){
    var args = Array.prototype.join.call( arguments, ',' );
    if ( args in cache ){
      return cache[ args ];
    }
    return cache[ args ] = mult.apply( this, arguments );
  }
})();
proxyMult( 1, 2, 3, 4 ); // 输出：24
proxyMult( 1, 2, 3, 4 ); // 输出：24 
```
当我们第二次调用 proxyMult( 1, 2, 3, 4 )的时候，本体 mult 函数并没有被计算，proxyMult直接返回了之前缓存好的计算结果
通过增加缓存代理的方式，mult 函数可以继续专注于自身的职责——计算乘积，缓存的功能是由代理对象实现的

##### 3.用高阶函数动态创建代理
通过传入高阶函数这种更加灵活的方式，可以为各种计算方法创建缓存代理
```
/**************** 计算乘积 *****************/
var mult = function(){
  var a = 1;
  for ( var i = 0, l = arguments.length; i < l; i++ ){
    a = a * arguments[i];
  }
  return a;
};
/**************** 计算加和 *****************/
var plus = function(){
  var a = 0;
  for ( var i = 0, l = arguments.length; i < l; i++ ){
    a = a + arguments[i];
  }
  return a;
};
/**************** 创建缓存代理的工厂 *****************/
var createProxyFactory = function( fn ){
  var cache = {};
  return function(){
    var args = Array.prototype.join.call( arguments, ',' );
    if ( args in cache ){
      return cache[ args ];
    }
    return cache[ args ] = fn.apply( this, arguments );
  }
};
var proxyMult = createProxyFactory( mult ),
  proxyPlus = createProxyFactory( plus );
alert ( proxyMult( 1, 2, 3, 4 ) ); // 输出：24
alert ( proxyMult( 1, 2, 3, 4 ) ); // 输出：24
alert ( proxyPlus( 1, 2, 3, 4 ) ); // 输出：10
alert ( proxyPlus( 1, 2, 3, 4 ) ); // 输出：10 
```
这些计算方法被当作参数传入一个专门用于创建缓存代理的工厂中， 这样一来，我们就可以为乘法、加法、减法等创建缓存代理
