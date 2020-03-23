---
title: Class中的super简析
date: 2020-01-02 21:02:44
tags: [JavaScript, ES6, Class]
categories: 开发
---

### super当作函数使用
super()执行父类的构造函数
```
class A {}
class B extends A {
  constructor() {
    super(); //ES6 要求，子类的构造函数必须执行一次super函数
  }
}
```
super() 返回的是子类的实例，即 super 内部的 this 指向的是B，此时 super() 相当于 A.prototype.constructor.call(this)
```
class A {
  constructor() {
    console.log(new.target.name); //new.target指向当前正在执行的函数
  }
}
class B extends A {
  constructor() {
    super();
  }
}
new A() // A
new B() // B
```
在上面的例子中，super() 执行时，它指向的是子类的构造函数
### 当作对象使用
在普通方法中，指向父类的原型对象；在静态方法中，指向父类
```
class A {
  c() {
    return 2;
  }
}
class B extends A {
  constructor() {
    super();
    console.log(super.c()); // 2
  }
}
let b = new B();
```
上面代码中，子类B当中的super.c()，就是将super当作一个对象使用。这时，super在普通方法之中，指向A.prototype，所以super.c()就相当于A.prototype.c()
通过super调用父类的方法时，super会绑定子类的this
```
class A {
  constructor() {
    this.x = 1;
  }
  s() {
    console.log(this.x);
  }
}
class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.s();
  }
}
let b = new B();
b.m() // 2
```
上面代码中，super.s()虽然调用的是A.prototype.s()，但是A.prototype.s()会绑定子类B的this，导致输出的是2，而不是1。也就是说，实际上执行的是super.s.call(this)
由于绑定子类的this，所以如果通过super对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性
```
class A {
  constructor() {
    this.x = 1;
  }
}
class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}
let b = new B();
```
上面代码中，super.x赋值为3，这时等同于对this.x赋值为3。而当读取super.x的时候，读的是A.prototype.x，所以返回undefined。
注意，使用super的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错
```
class A {}
class B extends A {
  constructor() {
    super();
    console.log(super); // 报错 Error: 'super' keyword unexpected here
  }
}
```
上面代码中，console.log(super)当中的super，无法看出是作为函数使用，还是作为对象使用，所以 JavaScript 引擎解析代码的时候就会报错。这时，如果能清晰地表明super的数据类型，就不会报错
