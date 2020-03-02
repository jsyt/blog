---
title: JavaScript 原型链与继承
date: 2018-04-23T21:53:54+08:00
categories: ["JavaScript"]
tags : ["JavaScript"]
toc: true
featured_image : ""
keywords: ["JavaScript", "原型链", "继承"]
description : "JavaScript 原型链与继承"
---

### 原型对象

无论什么时候，只要创建一个新函数，就会根据一组特定的规则为该函数创建一个 `prototype` 属性，这个属性指向函数的原型对象。默认情况下，所有原型对象都会自动获得一个 `constructor`（构造函数）属性，这个属性指向 `prototype` 属性所在的函数。


```js
function Person(){
}
```
<!--more-->

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/prototypeAndExtend-Person.png)




当我们用构造函数创建一个实例时，也会为这个实例创建一个 `__proto__` 属性，这个`__proto__` 属性是一个指针指向构造函数的原型对象

``` js
let person = new Person();
person.__proto__ === Person.prototype    // true
let person1 = new Person();
person1.__proto__ === Person.prototype    // true
```

由于同一个构造函数创建的所有实例对象的`__proto__` 属性都指向这个构造函数的原型对象，因此所有的实例对象都会共享构造函数的原型对象上所有的属性和方法，一旦原型对象上的属性或方法发生改变，所有的实例对象都会受到影响。

``` js
function Person(){
}
Person.prototype.name = "Luke";
Person.prototype.age = 18;
let person1 = new Person();
let person2 = new Person();
alert(person1.name)    // "Luke"
alert(person2.name)    // "Luke"
Person.prototype.name = "Jack";
alert(person1.name)    // "Jack"
alert(person2.name)    // "Jack"
```

### 重写原型对象

我们经常用一个包含所有属性和方法的对象字面量来重写整个原型对象，如下面的例子所示

``` js
function Person(){
}
Person.prototype = {
    name : "Luke",
    age : 18,
    job : "Software Engineer",
    sayName : function(){
        alert(this.name)
    }
}
```

在上面的代码中，我们将 `Person.prototype` 设置为一个新对象，而这个对象中没有`constructor`属性，这导致 `constructor` 属性不再指向 `Person`，而是指向 `Object`。

``` js
let friend = new Person();
alert(friend.constructor  === Person);    //false
alert(friend.constructor  === Object);    //true
```

如果 `constructor` 的值很重要，我们可以像下面这样特意将它设置回设置回适当的值

``` js
function Person(){
}
Person.prototype = {
    constructor : Person,
    name : "Luke",
    age : 18,
    job : "Software Engineer",
    sayName : function(){
        alert(this.name)
    }
}
```

### 原型链及原型链继承
每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针(`constructor`)，而实例都包含一个指向原型对象的内部指针(`__proto__`)。那么，假如我们让原型对象等于另一个类型的实例，结果会怎么样呢？显然，此时的原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个构造函数的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条。这就是所谓的原型链的基本概念。

```js
function Super(){
    this.property = true;
}

Super.prototype.getSuperValue = function(){
    return this.property;
}

function Sub(){
    this.subproperty = false;
}

Sub.prototype = new Super();    //继承了 Super

Sub.prototype.getSubValue = function (){
    return this.subproperty;
}

let instance = new Sub();
console.log(instance.getSuperValue());    //true

console.log(instance.__proto__ === Sub.prototype);    //true
console.log(Sub.prototype.__proto__ === Super.prototype);    //true

```
上面的代码中`Sub.prototype = new Super(); `通过创建Super的实例，并将该实例赋值给`Sub.prototype`来实现继承。此时存在于Super的实例和原型对象中的所有属性和方法，也都存在于Sub.prototype中。instanse的`__proto__`属性指向Sub的原型对象`Sub.prototype`，Sub原型对象的`__proto__`属性又指向Super的原型对象`Super.prototype`。

#### 原型链搜索机制
当访问一个实例的属性时，首先会在该实例中搜索该属性。如果没有找到该属性，则会继续搜索实例的原型。在通过原型链继承的情况下，搜索过程就得以沿着原型链继续向上查找，直到找到该属性为止，或者搜索到最高级的原型链`Object.prototype`中，任然没有找到则返回`undefined`。就拿上面的例子来说，调用`instance.getSuperValue()`会经历三个搜索步骤：1）搜索实例；2）搜索Sub.prototype;3）搜索Super.prototype，最后一步才会找到该方法。在找不到属性或方法的情况下，搜索过程总是要一环一环地前行到原型链的末端才会停下。

#### 原型链问题
原型链继承最大的问题是来自包含引用类型值的原型。引用类型值的原型属性会被所有实例共享。而这正是为什么要在构造函数中，而不是原型对象中定义属性的原因。在通过原型来实现继承时，原型实际上会另一个类型的实例。于是，原先的实例属性也就顺理成章地变成了现在的原型属性了。

```js
function Super(){
    this.colors = ["red", "blue", "green"];
}
function Sub(){

}
Sub.prototype = new Super();    // 继承了Super

let instance1 = new Sub();

instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"

let instance2 = new Sub();
alert(instance2.colors);    //"red, blue, green, black"
```

上面的代码中，Super 构造函数定义了一个colors 属性，该属性是一个数组。Super 的每个实例都会有各自包含自己数组的colors 属性。当Sub 通过原型链继承了Super之后，Sub.prototype 就变成了Super 的一个实例，因此它也拥有了一个它自己的colors 属性。结果是所有的Sub 实例都会共享这一个colors 属性。
原型链的第二个问题是没有办法在不影响所有对象实例的情况下，给超类的构造函数传递参数。

### 构造函数继承（经典继承）
即在子类构造函数的中调用父类构造函数，此时当构建一个子类实例时，此实例也会拥有父类实例的属性和方法。

```js
function Super(){
    this.colors = ["red", "blue", "green"];
}
function Sub(){
    Super.call(this, name);    //继承了Super
}

let instance1 = new Sub();

instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"

let instance2 = new Sub();
alert(instance2.colors);    //"red, blue, green"
```
上面的代码，当构建Sub的实例时，也会调用Super 的构造函数，这样就会在新Sub对象上执行Super()函数中定义的所有对象初始化代码。结果，Sub 的每个实例就都会具有自己的colors 属性的副本了。

#### 构造函数继承问题
如果仅仅是借用构造函数，那么也将无法避免构造函数模式存在的问题————方法都在构造函数中定义，因此函数服用就无从谈起。而且，在超类原型中定义的方法，对子类而已也是不可见的。

### 组合继承
是指将原型链和构造函数的相结合，发挥二者之长的一种继承模式。其思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，即通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。

```js
function Super(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

Super.prototype.sayName = function (){
    alert(this.name);
};

function Sub(name, age){
    Super.call(this, name);    //继承了Super 属性 (第二次调用Sup构造函数)
    this.age = age;
}

Sub.prototype = new Super();    // 继承了Super 原型链上的方法 (第一次调用Sup构造函数)
Sub.prototype.constructor = Sub;
Sub.prototype.sayAge = function (){
    alert(this.age);
};

var instance1 = new Sub("Luke", 18);
instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"
instance1.sayName();    //"Luke"
instance1.sayAge()    //18

var instance2 = new Sub("Jack", 20);
alert(instance2.colors);    //"red, blue, green"
instance2.sayName();    //"Jack"
instance2.sayAge()    //20

```
在上面的例子中，Sup构造函数定义了两个属性：name和colors。Sup的原型定义了一个方法sayName()。Sub构造函数在调用Sup构造函数时传入了name参数，紧接着又定义了它自己的属性age。然后，将Sup的实例赋值给Sub的原型，然后又在该新原型上定义了sayAge()方法。这样就可以让两个不同的Sub 实例即分别拥有自己的属性————包括colors 属性，又可以使用相同的方法了。
组合继承避免了原型链和构造函数的缺陷，融合了它们的优点，是JavaScript中最常用的继承模式。但是美中不足的是，上面的代码中调用了两次父类构造函数。Sub.prototype = new Super(); 第一次调用父类构造函数时，将Sup父类构造函数的实例赋值给了Sub子类的原型对象Sub.prototype。此时也会将父类构造函数实例上的属性赋值给子类的原型对象Sub.prototype。而第二次是在子类的构造函数中调用父类的构造函数 Super.call(this)，此时会将父类构造函数实例上的属性赋值给子类的构造函数的实例。根据原型链搜索原则，实例上的属性会屏蔽原型链上的属性。因此我们没有必要将父类构造函数实例的属性赋值给子类的原型对象，这是浪费资源而又没有意义的行为。


### 优化后的组合继承


```js
function Super(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

Super.prototype.sayName = function (){
    alert(this.name);
};

function Sub(name, age){
    Super.call(this, name);    //继承了Super 属性
    this.age = age;
}

function F(){
}
F.prototype = Super.prototype;
Sub.prototype = new F();    // 继承了Super 原型链上的方法

Sub.prototype.constructor = Sub;
Sub.prototype.sayAge = function (){
    alert(this.age);
};

var instance1 = new Sub("Luke", 18);
instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"
instance1.sayName();    //"Luke"
instance1.sayAge()    //18

var instance2 = new Sub("Jack", 20);
alert(instance2.colors);    //"red, blue, green"
instance2.sayName();    //"Jack"
instance2.sayAge()    //20

```

上面的例子通过将父类的原型对象直接赋值给一个中间构造函数的原型对象，然后将这个中间构造函数的实例赋值给子类的原型对象Sub.prototype，从而完成原型链继承。它的高效性体现在只调用了一个父类构造函数Super，并且原型链保持不变。还有一种简便的写法是采用ES5的Object.create()方法来替代中间构造函数，其实原理都是一样的


```js
function Super(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

Super.prototype.sayName = function (){
    alert(this.name);
};

function Sub(name, age){
    Super.call(this, name);    //继承了Super 属性
    this.age = age;
}
/*
function F(){
}
F.prototype = Super.prototype;
Sub.prototype = new F();    // 继承了Super 原型链上的方法

Sub.prototype.constructor = Sub;
*/
//这行代码的原理与上面注释的代码是一样的
Sub.prototype = Object.create(Super.prototype, {constructor: {value: Sub}})

Sub.prototype.sayAge = function (){
    alert(this.age);
};

var instance1 = new Sub("Luke", 18);
instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"
instance1.sayName();    //"Luke"
instance1.sayAge()    //18

var instance2 = new Sub("Jack", 20);
alert(instance2.colors);    //"red, blue, green"
instance2.sayName();    //"Jack"
instance2.sayAge()    //20
```

### 更简单的继承方式
还有一种更简单的继承方法，就是直接将子类的原型对象(prototype)上的`__proto__`指向父类的的原型对象(prototype)，这种方式没有改变子类的原型对象，所以子类原型对象上的`constructor`属性还是指向子类的构造函数，而且当子类的实例在子类的原型对象上没有搜索到对应的属性或方法时，它会通过子类原型对象上的`__proto__`属性，继续在父类的原型对象上搜索对应的属性或方法

```js
function Super(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

Super.prototype.sayName = function (){
    alert(this.name);
};

function Sub(name, age){
    Super.call(this, name);    //继承了Super 属性
    this.age = age;
}

Sub.prototype.__proto__ = Super.prototype
Sub.prototype.sayAge = function (){
    alert(this.age);
};
var instance1 = new Sub("Luke", 18);
instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"
instance1.sayName();    //"Luke"
instance1.sayAge()    //18

var instance2 = new Sub("Jack", 20);
alert(instance2.colors);    //"red, blue, green"
instance2.sayName();    //"Jack"
instance2.sayAge()    //20
```

### Object.setPrototypeOf()
Object.setPrototypeOf()是ECMAScript 6最新草案中的方法，相对于 Object.prototype.__proto__ ，它被认为是修改对象原型更合适的方法

```js
function Super(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

Super.prototype.sayName = function (){
    alert(this.name);
};

function Sub(name, age){
    Super.call(this, name);    //继承了Super 属性
    this.age = age;
}

//Sub.prototype.__proto__ = Super.prototype
Object.setPrototypeOf(Sub.prototype, Super.prototype)

Sub.prototype.sayAge = function (){
    alert(this.age);
};
var instance1 = new Sub("Luke", 18);
instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"
instance1.sayName();    //"Luke"
instance1.sayAge()    //18

var instance2 = new Sub("Jack", 20);
alert(instance2.colors);    //"red, blue, green"
instance2.sayName();    //"Jack"
instance2.sayAge()    //20
```

### 类的静态方法继承
上面所有的继承方法都没有实现类的静态方法继承，而在ES6的`class`继承中，子类是可以继承父类的静态方法的。我们可通过`Object.setPrototypeOf()`来实现类的静态方法继承，非常简单
```js
Object.setPrototypeOf(Sub, Super)
```

```js
function Super(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

Super.prototype.sayName = function (){
    alert(this.name);
};

Super.staticFn = function(){
    alert('Super.staticFn')
}

function Sub(name, age){
    Super.call(this, name);    //继承了Super 属性
    this.age = age;
}

//Sub.prototype.__proto__ = Super.prototype
Object.setPrototypeOf(Sub.prototype, Super.prototype)
Object.setPrototypeOf(Sub, Super)    // 继承父类的静态属性或方法
Sub.staticFn()    // "Super.staticFn"

Sub.prototype.sayAge = function (){
    alert(this.age);
};
var instance1 = new Sub("Luke", 18);
instance1.colors.push("black");
alert(instance1.colors);    //"red, blue, green, black"
instance1.sayName();    //"Luke"
instance1.sayAge()    //18

var instance2 = new Sub("Jack", 20);
alert(instance2.colors);    //"red, blue, green"
instance2.sayName();    //"Jack"
instance2.sayAge()    //20
```

这大概就是最终的理想继承方式吧。