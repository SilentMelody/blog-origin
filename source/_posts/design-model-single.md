---
title: 设计模式系列笔记-单例模式
date: 2019-09-12 21:36:43
tags: [JS, JavaScript, 设计模式]
categories: 开发
---


*写在前面：本系列文章内容为《JavaScript设计模式与开发实践》一书学习笔记，感谢作者曾探*
## 单例模式
定义：保证一个类仅有一个实例，并可以全局访问该实例
举例：线程池、全局缓存、window对象等，或者全局的弹框组件、登录组件等

### 1.实现单例模式
思路：用一个变量来标志当前是否已经为某个类创建过对象，如果是，则在下一次获取该类的实例时，直接返回之前创建的对象
示例：
```
const Singleton = function(name) {
  this.name = name;
  this.instance = null;
};
Singleton.prototype.getName = function() {
  alert(this.name);
};
Singleton.getInstance = function(name) {
  if (!this.instance) {
    this.instance = new Singleton(name);
  }
  return this.instance;
};
const a = Singleton.getInstance('sven1');
const b = Singleton.getInstance('sven2');
alert(a === b); // true
```

### 2.透明的单例模式
透明的单例模式: 用户从这个单例类中创建对象的时候，可以像使用其他任何普通类一样
```
// 透明的单例模式
const CreateDiv = (function() {
  let instance;
  const CreateDiv = function(html) {
    if (instance) {
      return instance;
    }
    this.html = html;
    this.init();
    return instance = this;
  };
  CreateDiv.prototype.init = function() {
    const div = document.createElement('div');
    div.innerHTML = this.html;
    document.body.appendChild(div);
  };
  return CreateDiv;
})();
const a = new CreateDiv('sven1');
const b = new CreateDiv('sven2');
alert(a === b); // true
```
但是这种写法也有缺点：CreateDiv 的构造函数实际上负责了两件事情。第一是创建对象和执行初始化init 方法，第二是保证只有一个对象，这种多重职责使这个构造函数看起来很奇怪

### 3.用代理实现单例模式
通过引入代理的方式，我们改造上面的例子，跟之前不同的是，现在我们把负责管理单例的逻辑移到了代理类proxySingletonCreateDiv 中。这样一来，CreateDiv 就变成了一个普通的类，它跟proxySingletonCreateDiv 组合起来可以达到单例模式的效果。
```
// 代理实现单例模式
class CreateDiv {
  constructor(html) {
    this.html = html;
    this.init();
  }
};
CreateDiv.prototype.init = function() {
  const div = document.createElement('div');
  div.innerHTML = this.html;
  document.body.appendChild(div);
};
// 引入代理方法proxySingletonCreateDiv：
const ProxySingletonCreateDiv = (function() {
  let instance;
  return (html) => {
    if (!instance) {
      instance = new CreateDiv(html);
    }
    return instance;
  }
})();
const a = ProxySingletonCreateDiv('sven1');
const b = ProxySingletonCreateDiv('sven2');
alert(a === b);
```
### 4.惰性单例
惰性单例指的是在需要的时候才创建对象实例
以网页QQ的登录框举例
```
// 惰性单例
var createLoginLayer = (function() {
  var div;
  return function() {
    if (!div) {
      div = document.createElement('div');
      div.innerHTML = '我是登录浮窗';
      div.style.display = 'none';
      document.body.appendChild(div);
    }
    return div;
  }
})();
document.getElementById('loginBtn').onclick = function() {
  var loginLayer = createLoginLayer();
  loginLayer.style.display = 'block';
};
```
这种方案存在的问题：
- 这段代码仍然是违反单一职责原则的，创建对象和管理单例的逻辑都放在 createLoginLayer对象内部
- 如果我们下次需要创建页面中唯一的 iframe，或者 script 标签，用来跨域请求数据，就必须得如法炮制，把 createLoginLayer 函数几乎照抄一遍

### 5.通用的惰性单例
管理单例的逻辑其实是完全可以抽象出来的，这个逻辑始终是一样的：用一个变量来标志是否创建过对象，如果是，则在下次直接返回这个已经创建好的对象
现在我们就把如何管理单例的逻辑从原来的代码中抽离出来，这些逻辑被封装在 getSingle
函数内部，创建对象的方法 fn 被当成参数动态传入 getSingle 函数
```
var getSingle = function(fn) {
  var result;
  return function() {
    return result || (result = fn.apply(this, arguments));
  }
};
```
接下来将用于创建登录浮窗的方法用参数 fn 的形式传入 getSingle，我们不仅可以传入createLoginLayer，还能传入 createScript、createIframe、createXhr 等。之后再让 getSingle 返回一个新的函数，并且用一个变量 result 来保存 fn 的计算结果。result 变量因为身在闭包中，它永远不会被销毁。在将来的请求中，如果 result 已经被赋值，那么它将返回这个值。代码如下：
```
var createLoginLayer = function() {
  var div = document.createElement('div');
  div.innerHTML = '我是登录浮窗';
  div.style.display = 'none';
  document.body.appendChild(div);
  return div;
};
var createSingleLoginLayer = getSingle(createLoginLayer);
document.getElementById('loginBtn').onclick = function() {
  var loginLayer = createSingleLoginLayer();
  loginLayer.style.display = 'block';
};
```
我们再试试创建唯一的 iframe 用于动态加载第三方页面：
```
var createSingleIframe = getSingle(function() {
  var iframe = document.createElement('iframe');
  document.body.appendChild(iframe);
  return iframe;
});
document.getElementById('loginBtn').onclick = function() {
  var loginLayer = createSingleIframe();
  loginLayer.src = 'http://baidu.com';
};
```
在这个例子中，我们把创建实例对象的职责和管理单例的职责分别放置在两个方法里，这两个方法可以独立变化而互不影响，当它们连接在一起的时候，就完成了创建唯一实例对象的功能

### 6.小节
在 getSinge 函数中，实际上也提到了闭包和高阶函数的概念。单例模式是一种简单但非常实用的模式，特别是惰性单例技术，在合适的时候才创建对象，并且只创建唯一的一个，使用了代理模式和单一职责原则，创建对象和管理单例的职责被分布在两个不同的方法中
