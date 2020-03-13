---
title: Class中的私有属性
date: 2020-01-01 16:31:44
tags: [JavaScript, ES6, Class]
categories: 开发
index_img: https://upload-images.jianshu.io/upload_images/12319573-d161e20ae1b8c1d7.png
---

### 私有属性
私有属性是面向对象编程（OOP）中非常常见的一个特性，满足以下特点：
 - 能被 class 内部的不同方法访问，但不能在类外部被访问
 - 子类不能继承父类的私有属性

### 新的提案
在 class 中，用 '#' 开头的属性代表私有属性（该提案已经到Stage 3）
```
class Foo {
  #a; // 私有属性
  constructor(a, b) {
    this.#a = a;  
    this.b = b;
  }
}
```
*注：上面私有属性的声明，需要先经过Babel等编译器编译后才能正常使用*

### JS模拟实现私有属性
#### 约定俗成
变量前加下划线 '_' 前缀代表私有属性，实际还是公共属性
#### 闭包
在 constructor 作用域内定义局部变量，内部通过闭包的方式对外暴露
```
class Foo {
  constructor(a, b, c) {
    this.a = a;
    this.b = b;

    let _c = c;
    this.getC = function() {
      return _c;
    }
    this.setC = function(val) {
      _c = val;
    }
  }
}

const foo1 = new Foo('a1', 'b1', 'c1');
console.log(foo1.a, foo1._c, foo1.getC()); // a1 undefined c1
foo1.setC('c2')
console.log(foo1.getC()); // c2
```

#### Symbols & Getter
利用 Symbol 变量可以作为对象 key 的特点，可以模拟实现私有属性
```
const FoolSymbol = (() => {
  const _p = Symbol('p');
  class FoolSymbol {
    constructor(a, b, p) {
      this.a = a;
      this.b = b;
      this[_p] = p;
    }
    getP = (val) => {
      return this[_p];
    }
    setP = (val) => {
      this[_p] = val;
    }
  }
  return FoolSymbol;
})();

const foolSymbol = new FoolSymbol('a1', 'b1', 'p1');
console.log(foolSymbol.getP()); // p1
foolSymbol.setP('p2');
console.log(foolSymbol.getP()); // p2
```
*对比：第二种利用闭包的方式，该属性的get 和 set 方法只能在 Class 的 constructor 内部，外部访问不到该属性，而 Symbol 可以在外部获取到，所以get 和 set 方法可以写在 constructor 外部*
