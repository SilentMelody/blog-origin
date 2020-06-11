---
title: 理解ES6中的Class（摘抄）
date: 2019-04-02 21:12:40
tags: [JavaScript, ES6, Class]
categories: 开发
---

**！！本文摘抄自 [https://www.jianshu.com/p/86267fab4878](https://www.jianshu.com/p/86267fab4878)，转载请注明原出处**

### 传统方法实现类
传统的javascript中只有对象，没有类的概念。它是基于原型的面向对象语言。原型对象特点就是将自身的属性共享给新对象。这样的写法相对于其它传统面向对象语言来讲，很有一种独树一帜的感脚！非常容易让人困惑！
如果要生成一个对象实例，需要先定义一个构造函数，然后通过new操作符来完成。构造函数示例：
```
//函数名和实例化构造名相同且大写（非强制，但这么写有助于区分构造函数和普通函数）
function Person(name,age) {
  this.name = name;
  this.age = age;
}
Person.prototype.say = function() {
  return "我的名字叫" + this.name+"今年" + this.age + "岁了";
}
var obj=new Person("laotie",88); // 通过构造函数创建对象，必须使用 new 运算符
console.log(obj.say()); // 我的名字叫laotie今年88岁了
```
#### 构造函数生成实例的执行过程（new 关键字） 
本小节摘自 [https://www.jianshu.com/p/15ac7393bc1f](https://www.jianshu.com/p/15ac7393bc1f)
```
1.声明一个中间对象；
2.将该中间对象的原型指向构造函数的原型；
3.将构造函数的this，指向该中间对象；
4.返回该中间对象，即返回实例对象。
```
模拟 new 的执行过程
```
// 先一本正经的创建一个构造函数，其实该函数与普通函数并无区别
var Person = function(name, age) {
    this.name = name;
    this.age = age;
    this.getName = function() {
        return this.name;
    }
}
// 将构造函数以参数形式传入
function New(func) {
    // 声明一个中间对象，该对象为最终返回的实例
    var res = {};
    if (func.prototype !== null) {
        // 将实例的原型指向构造函数的原型
        res.__proto__ = func.prototype;
    }
    // ret为构造函数执行的结果，这里通过apply，将构造函数内部的this指向修改为指向res，即为实例对象
    var ret = func.apply(res, Array.prototype.slice.call(arguments, 1));
    // 当我们在构造函数中明确指定了返回对象时，那么new的执行结果就是该返回对象
    if ((typeof ret === "object" || typeof ret === "function") && ret !== null) {
        return ret;
    }
    // 如果没有明确指定返回对象，则默认返回res，这个res就是实例对象
    return res;
}
// 通过new声明创建实例，这里的p1，实际接收的正是new中返回的res
var p1 = New(Person, 'tom', 20);
console.log(p1.getName());
// 当然，这里也可以判断出实例的类型了
console.log(p1 instanceof Person); // true
```
### ES6的类
将之前的Person代码改为ES6的写法就会是这个样子
```
class Person{//定义了一个名字为Person的类
  constructor(name,age){//constructor是一个构造方法，用来接收参数
    this.name = name;//this代表的是实例对象
    this.age=age;
  }
  say(){//这是一个类的方法，注意千万不要加上function
    return "我的名字叫" + this.name+"今年"+this.age+"岁了";
  }
}
var obj=new Person("laotie",88);
console.log(obj.say());//我的名字叫laotie今年88岁了
```
由下面代码可以看出类实质上就是一个函数。类自身指向的就是构造函数。所以可以认为ES6中的类其实就是构造函数的另外一种写法！
```
console.log(typeof Person);//function
console.log(Person===Person.prototype.constructor);//true
```
实际上类的所有方法都定义在类的prototype属性上

**实例的隐式原型(`person1.__proto__`)指向构造该实例的类的原型(`Person.prototype`) ，都指向类的原型对象`{constructor: f, say: f}`，而该原型对象的构造函数(`constructor`) 又反过来指向该类，即 `person1.__proto__.constructor === Person.prototype.constructor === Person`**
```
Person.prototype.say=function(){//定义与类中相同名字的方法。成功实现了覆盖！
    return "我是来证明的，你叫" + this.name+"今年"+this.age+"岁了";
}
var obj=new Person("laotie",88);
console.log(obj.say());//我是来证明的，你叫laotie今年88岁了
```
constructor方法是类的构造函数的默认方法，通过new命令生成对象实例时，自动调用该方法
```
class Box{
  constructor(){
    console.log("啦啦啦，今天天气好晴朗");//当实例化对象时该行代码会执行。
  }
}
var obj=new Box();
```
constructor方法如果没有显式定义，会隐式生成一个constructor方法。所以即使你没有添加构造函数，构造函数也是存在的。constructor方法默认返回实例对象this，但是也可以指定constructor方法返回一个全新的对象，让返回的实例对象不是该类的实例
```
class Desk{
  constructor(){
    this.xixi="我是一只小小小小鸟！哦";
  }
}
class Box{
  constructor(){
    return new Desk();// 这里没有用this哦，直接返回一个全新的对象
  }
}
var obj=new Box();
console.log(obj.xixi);//我是一只小小小小鸟！哦
```
constructor中定义的属性可以称为实例属性（即定义在this对象上），constructor外声明的属性都是定义在原型上的，可以称为原型属性（即定义在class上)。hasOwnProperty()函数用于判断属性是否是实例属性。其结果是一个布尔值， true说明是实例属性，false说明不是实例属性。in操作符会在通过对象能够访问给定属性时返回true,无论该属性存在于实例中还是原型中
```
class Box{
  constructor(num1,num2){
    this.num1 = num1;
    this.num2=num2;
  }
  sum(){
    return num1+num2;
  }
}
var box=new Box(12,88);
box.say = () => {};
console.log(box.hasOwnProperty("num1"));//true
console.log(box.hasOwnProperty("num2"));//true
console.log(box.hasOwnProperty("sum"));//false
console.log("num1" in box);//true
console.log("num2" in box);//true
console.log("sum" in box);//true
```
类的所有实例共享一个原型对象，它们的原型都是Person.prototype，所以proto属性是相等的
```
class Box{
  constructor(num1,num2){
    this.num1 = num1;
    this.num2=num2;
  }
  sum(){
    return num1+num2;
  }
}
//box1与box2都是Box的实例。它们的__proto__都指向Box的prototype(实例的隐式原型(`__proto__`)指向构造该实例的类的原型(`prototype`) )
var box1=new Box(12,88);
var box2=new Box(40,60);
console.log(box1.__proto__===box2.__proto__);//true
console.log(box1.__proto__===Box.prototype);//true
```
由此，也可以通过proto来为类增加方法。使用实例的proto属性改写原型，会改变Class的原始定义，影响到所有实例，所以不推荐使用
```
class Box{
  constructor(num1,num2){
    this.num1 = num1;
    this.num2=num2;
  }
  sum(){
    return num1+num2;
  }
}
var box1=new Box(12,88);
var box2=new Box(40,60);
box1.__proto__.sub=function(){
  return this.num2-this.num1;
}
console.log(box1.sub());//76
console.log(box2.sub());//20
```
class不存在变量提升，所以需要先定义再使用。因为ES6不会把类的声明提升到代码头部，但是ES5就不一样,ES5存在变量提升,可以先使用，然后再定义
```
//ES5可以先使用再定义,存在变量提升
new A();
function A(){

}
//ES6不能先使用再定义,不存在变量提升 会报错
new B();//B is not defined
class B{

}
```
**文末再次声明：本文摘抄自 [https://www.jianshu.com/p/86267fab4878](https://www.jianshu.com/p/86267fab4878)，转载请注明原出处**
