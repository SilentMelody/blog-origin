---
title: 设计模式系列笔记-策略模式
date: 2019-09-14 21:29:07
tags: [JS, JavaScript, 设计模式]
categories: 开发
---


*写在前面：本系列文章内容为《JavaScript设计模式与开发实践》一书学习笔记，感谢作者曾探*
## 策略模式
在现实中，很多时候可以选择多种途径到达同一个目的地，程序设计中，要实现某一个功能有多种方案可以选择，可以根据具体情况选择其中一种方案，这就是策略模式
定义：策略模式就是定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换

### 1.策略模式实现
举例：年终奖的计算（绩效为 S 的人年
终奖有 4 倍工资，绩效为 A 的人年终奖有 3 倍工资，而绩效为 B 的人年终奖是 2 倍工资）
一般最容易想到的办法就是使用组合函数，代码如下：
```
const performanceS = (salary) => {
  return salary * 4;
};
const performanceA = (salary) => {
  return salary * 3;
};
const performanceB = (salary) => {
  return salary * 2;
};
const calculateBonus = (performanceLevel, salary) => {
  if (performanceLevel === 'S') {
    return performanceS(salary);
  }
  if (performanceLevel === 'A') {
    return performanceA(salary);
  }
  if (performanceLevel === 'B') {
    return performanceB(salary);
  }
};
calculateBonus('A', 10000); // 输出：30000
```
缺陷：
calculateBonus 函数有可能越来越庞大，而且在系统变化的时候缺乏弹性

### 2.JavaScript 的策略模式实现
```
// 策略模式
const strategies = {
  "S": (salary) => {
    return salary * 4;
  },
  "A": (salary) => {
    return salary * 3;
  },
  "B": (salary) => {
    return salary * 2;
  }
};
const calculateBonus = (level, salary) => {
  return strategies[level](salary);
};
console.log(calculateBonus('S', 20000)); // 输出：80000
console.log(calculateBonus('A', 10000)); // 输出：30000
```

### 3.策略模式实现表单校验
假设我们正在编写一个注册的页面，在点击注册按钮之前，有如下几条校验逻辑
 - 用户名不能为空
 - 密码长度不能少于 6 位
 - 手机号码必须符合格式

HTML部分
```
<form action="http:// xxx.com/register" id="registerForm" method="post">
  请输入用户名：<input type="text" name="userName"/>
  请输入密码：<input type="text" name="password"/>
  请输入手机号码：<input type="text" name="phoneNumber"/>
  <button>提交</button>
</form>
```
1. 我们先把校验逻辑都封装成策略对象
```
var strategies = {
  isNonEmpty: function(value, errorMsg) { // 不为空
    if (value === '') {
      return errorMsg;
    }
  },
  minLength: function(value, length, errorMsg) { // 限制最小长度
    if (value.length < length) {
      return errorMsg;
    }
  },
  isMobile: function(value, errorMsg) { // 手机号码格式
    if (!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
      return errorMsg;
    }
  }
};
```
2. 接下来我们期望的使用方法
```
var validataFunc = function() {
  var validator = new Validator(); // 创建一个 validator 对象
  /***************添加一些校验规则****************/
  validator.add(registerForm.userName, 'isNonEmpty', '用户名不能为空');
  validator.add(registerForm.password, 'minLength:6', '密码长度不能少于 6 位');
  validator.add(registerForm.phoneNumber, 'isMobile', '手机号码格式不正确');
  var errorMsg = validator.start(); // 获得校验结果
  return errorMsg; // 返回校验结果
}
var registerForm = document.getElementById('registerForm');
registerForm.onsubmit = function() {
  var errorMsg = validataFunc(); // 如果 errorMsg 有确切的返回值，说明未通过校验
  if (errorMsg) {
    alert(errorMsg);
    return false; // 阻止表单提交
  }
};
```
3. 实现我们需要的 Validator 类
```
var Validator = function() {
  this.cache = []; // 保存校验规则
};
Validator.prototype.add = function(dom, rule, errorMsg) {
  var ary = rule.split(':'); // 把 strategy 和参数分开
  this.cache.push(function() { // 把校验的步骤用空函数包装起来，并且放入 cache
    var strategy = ary.shift(); // 用户挑选的 strategy
    ary.unshift(dom.value); // 把 input 的 value 添加进参数列表
    ary.push(errorMsg); // 把 errorMsg 添加进参数列表
    return strategies[strategy].apply(dom, ary);
  });
};
Validator.prototype.start = function() {
  for (var i = 0, validatorFunc; validatorFunc = this.cache[i++];) {
    var msg = validatorFunc(); // 开始校验，并取得校验后的返回信息
    if (msg) { // 如果有确切的返回值，说明校验没有通过
      return msg;
    }
  }
};
```
使用策略模式重构代码之后，我们仅仅通过“配置”的方式就可以完成一个表单的校验，这些校验规则也可以复用在程序的任何地方，还能作为插件的形式，方便地被移植到其他项目中

### 4.策略模式优缺点

优点：
- 策略模式利用组合、委托和多态等技术和思想，可以有效地避免多重条件选择语句
- 策略模式提供了对开放—封闭原则的完美支持，易于切换，易于理解，易于扩展

缺点：
- 会在程序中增加许多策略类或者策略对象
- 要使用策略模式，必须了解所有的 strategy，必须了解各个 strategy 之间的不同点，这样才能选择一个合适的 strategy，一定程度上违背了最少知识原则

### 5.一等函数对象与策略模式
**在函数作为一等对象的语言中，策略模式是隐形的**
在 JavaScript 中，除了使用类来封装算法和行为之外，使用函数当然也是一种选择。这些“算法”可以被封装到函数中并且四处传递，也就是我们常说的"高阶函数”

在 JavaScript 这种将函数作为一等对象的语言里，策略模式已经融入到了语言本身当中，我们经常用高阶函数来封装不同的行为，并且把它传递到另一个函数中，例如：
```
var S = function(salary) {
  return salary * 4;
};
var A = function(salary) {
  return salary * 3;
};
var B = function(salary) {
  return salary * 2;
};
var calculateBonus = function(func, salary) {
  return func(salary);
};
calculateBonus(S, 10000); // 输出：40000
```
在 JavaScript 语言的策略模式中，策略类往往被函数所代替，这时策略模式就成为一种“隐形”的模式
