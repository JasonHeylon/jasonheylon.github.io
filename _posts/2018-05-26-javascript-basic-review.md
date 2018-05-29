---
layout: post
title: "javascript 基础复习"
date: 2018-05-26 00:00:00
categories: javascript
comments: true
---

# 变量&作用域

### 类型检查

```javascript
var s = "Nicholas";
var b = true;
var i = 22;
var u;
var n = null;
var o = new Object();
alert(typeof s); //string
alert(typeof i); //number
alert(typeof b); //boolean
alert(typeof u); //undefined
alert(typeof n); //object
alert(typeof o); //object

```

### 引用类型
> 引用类型的值是保存在内存中的对象。与其他语言不同，JavaScript 不允许直接访问内存中的位置， 也就是说不能直接操作对象的内存空间。在操作对象时，实际上是在操作对象的引用而不是实际的对象。 为此，引用类型的值是按引用访问的

### 参数传递
> ECMAScript 中所有函数的参数都是按值传递的

```javascript
function setName(obj) {
  obj.name = "Nicholas";
  obj = new Object(); obj.name = "Greg";
}
var person = new Object();
setName(person);
alert(person.name);    //"Nicholas"
```

### 执行环境
> 执行环境(execution context，为简单起见，有时也称为“环境”)是 JavaScript 中最为重要的一个概 念。执行环境定义了变量或函数有权访问的其他数据，决定了它们各自的行为。每个执行环境都有一个 与之关联的变量对象(variable object)，环境中定义的所有变量和函数都保存在这个对象中。虽然我们 编写的代码无法访问这个对象，但解析器在处理数据时会在后台使用它。
>
> 全局执行环境是最外围的一个执行环境。根据 ECMAScript 实现所在的宿主环境不同，表示执行环 境的对象也不一样。在 Web 浏览器中，全局执行环境被认为是 window 对象(第 7 章将详细讨论)，因 此所有全局变量和函数都是作为 window 对象的属性和方法创建的。某个执行环境中的所有代码执行完 毕后，该环境被销毁，保存在其中的所有变量和函数定义也随之销毁(全局执行环境直到应用程序退 出——例如关闭网页或浏览器——时才会被销毁)。
>
> 每个函数都有自己的执行环境。当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。 而在函数执行之后，栈将其环境弹出，把控制权返回给之前的执行环境。ECMAScript 程序中的执行流 正是由这个方便的机制控制着。
>
> 当代码在一个环境中执行时，会创建变量对象的一个作用域链(scope chain)。作用域链的用途，是 保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所 在环境的变量对象。如果这个环境是函数，则将其活动对象(activation object)作为变量对象。活动对 6 象在最开始时只包含一个变量，即 arguments 对象(这个对象在全局环境中是不存在的)。作用域链中 的下一个变量对象来自包含(外部)环境，而再下一个变量对象则来自下一个包含环境。这样，一直延 续到全局执行环境;全局执行环境的变量对象始终都是作用域链中的最后一个对象。

### GC
> JavaScript 中最常用的垃圾收集方式是标记清除(mark-and-sweep)。当变量进入环境(例如，在函数中声明一个变量)时，就将这个变量标记为“进入环境”。从逻辑上讲，永远不能释放进入环境的变量所占用的内存，因为只要执行流进入相应的环境，就可能会用到它们。而当变量离开环境时，则将其 标记为“离开环境”。
>
> 可以使用任何方式来标记变量。比如，可以通过翻转某个特殊的位来记录一个变量何时进入环境， 或者使用一个“进入环境的”变量列表及一个“离开环境的”变量列表来跟踪哪个变量发生了变化。说 到底，如何标记变量其实并不重要，关键在于采取什么策略。
>
> 垃圾收集器在运行的时候会给存储在内存中的所有变量都加上标记(当然，可以使用任何标记方 式)。然后，它会去掉环境中的变量以及被环境中的变量引用的变量的标记。而在此之后再被加上标记 的变量将被视为准备删除的变量，原因是环境中的变量已经无法访问到这些变量了。最后，垃圾收集器 完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间。
>
> 到 2008 年为止，IE、Firefox、Opera、Chrome 和 Safari 的 JavaScript 实现使用的都是标记清除式的 垃圾收集策略(或类似的策略)，只不过垃圾收集的时间间隔互有不同。

# 对象

### Object类型

```javascript
var person = new Object();
person.name = "Nicholas";
person.age = 29;

var person = {
  name : "Nicholas",
  age : 29
};
```

### 数据属性

数据属性包含一个数据值的位置。在这个位置可以读取和写入值。数据属性有 4 个描述其行为的
特性。
- `[[Configurable]]`: 表示能否通过 delete 删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。像前面例子中那样直接在对象上定义的属性，它们的这个特性默认值为 true。
- `[[Enumerable]]`: 表示能否通过 for-in 循环返回属性。像前面例子中那样直接在对象上定义的属性，它们的这个特性默认值为 true。
- `[[Writable]]`: 表示能否修改属性的值。像前面例子中那样直接在对象上定义的属性，它们的这个特性默认值为 true。
- `[[Value]]`: 包含这个属性的数据值。读取属性值的时候，从这个位置读;写入属性值的时候，
把新值保存在这个位置。这个特性的默认值为 undefined。

> 对于像前面例子中那样直接在对象上定义的属性，它们的[[Configurable]]、[[Enumerable]]
和[[Writable]]特性都被设置为 true，而[[Value]]特性被设置为指定的值

```javascript
var person = {};
Object.defineProperty(person, "name", {
  configurable: false,
  value: "Nicholas"
});
alert(person.name); //"Nicholas"
delete person.name;
alert(person.name); //"Nicholas"

```

### 访问器属性
> 访问器属性不包含数据值;它们包含一对儿 getter 和 setter 函数(不过，这两个函数都不是必需的)。 在读取访问器属性时，会调用 getter 函数，这个函数负责返回有效的值;在写入访问器属性时，会调用 setter 函数并传入新值，这个函数负责决定如何处理数据。访问器属性有如下 4 个特性。

- `[[Configurable]]`: 表示能否通过 delete 删除属性从而重新定义属性，能否修改属性的特 性，或者能否把属性修改为数据属性。对于直接在对象上定义的属性，这个特性的默认值为 true。
- `[[Enumerable]]`: 表示能否通过 for-in 循环返回属性。对于直接在对象上定义的属性，这 5 个特性的默认值为 true。
- `[[Get]]`: 在读取属性时调用的函数。默认值为 undefined。
- `[[Set]]`: 在写入属性时调用的函数。默认值为 undefined。

> 访问器属性不能直接定义，必须使用 Object.defineProperty()来定义。

```javascript
var book = {
  _year: 2004,
  edition: 1
};

Object.defineProperty(book, "year", {
  get: function() {
    return this._year;
  },
  set: function(newValue){
    if (newValue > 2004) {
      this._year = newValue;
      this.edition += newValue - 2004;
    }
  }
});

book.year = 2005; alert(book.edition); //2
```

### 定义多个属性
```javascript
var book = {};
Object.defineProperties(book, {
  _year: {
    value: 2004
  },
  edition: {
    value: 1
  },
  year: {
    get: function() {
      return this._year;
    },
    set: function(newValue){
      if (newValue > 2004) {
        this._year = newValue;
        this.edition += newValue - 2004;
      }
    }
  }
});

```

### 读取属性的特性

```javascript
var book = {};
Object.defineProperties(book, {
  _year: {
    value: 2004
  },
  edition: {
    value: 1
  },
  year: {
  get: function(){
    return this._year;
  },
  set: function(newValue){
    if (newValue > 2004) {
      this._year = newValue;
      this.edition += newValue - 2004;
    }
  }
});

var descriptor = Object.getOwnPropertyDescriptor(book, "_year");
alert(descriptor.value); //2004
alert(descriptor.configurable); //false
alert(typeof descriptor.get); //"undefined"

var descriptor = Object.getOwnPropertyDescriptor(book, "year"); alert(descriptor.value); //undefined alert(descriptor.enumerable); //false
alert(typeof descriptor.get); //"function"

```


# 创建对象

### 工厂模式

```javascript
function createPerson(name, age, job){
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function(){
    alert(this.name);
  };

  return o;
}

var person1 = createPerson("Nicholas", 29, "Software Engineer");
var person2 = createPerson("Greg", 27, "Doctor");
```

### 构造函数模式
```javascript
function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = new Function("alert(this.name)"); // 与声明函数在逻辑上是等价的
}


function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = sayName;
}
function sayName(){
  alert(this.name);
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

### 原型模式

```javascript
function Person(){
}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
  alert(this.name);
};
var person1 = new Person();
person1.sayName();   //"Nicholas"
var person2 = new Person();
person2.sayName(); //"Nicholas"
alert(person1.sayName == person2.sayName);  //true

alert(Person.prototype.isPrototypeOf(person1));  //true
alert(Person.prototype.isPrototypeOf(person2));  //true

alert(Object.getPrototypeOf(person1) == Person.prototype); //true
alert(Object.getPrototypeOf(person1).name); //"Nicholas"



function Person(){
}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
}

var person1 = new Person();
var person2 = new Person();
alert(person1.hasOwnProperty("name"));  //false
person1.name = "Greg";
alert(person1.name); //"Greg"——来自实例 alert(person1.hasOwnProperty("name")); //true
alert(person2.name); //"Nicholas"——来自原型 alert(person2.hasOwnProperty("name")); //false
delete person1.name;
alert(person1.name); //"Nicholas"——来自原型 alert(person1.hasOwnProperty("name")); //false



function Person(){
}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
  alert(this.name);
};
var person1 = new Person();
var person2 = new Person();
alert(person1.hasOwnProperty("name"));  //false
alert("name" in person1);  //true
person1.name = "Greg";
alert(person1.name); //"Greg" ——来自实例 alert(person1.hasOwnProperty("name")); //true alert("name" in person1); //true
alert(person2.name); //"Nicholas" ——来自原型 alert(person2.hasOwnProperty("name")); //false alert("name" in person2); //true
delete person1.name;
alert(person1.name); //"Nicholas" ——来自原型 alert(person1.hasOwnProperty("name")); //false alert("name" in person1); //true


function hasPrototypeProperty(object, name){
  return !object.hasOwnProperty(name) && (name in object);
}

```

### 组合使用构造函数模式和原型模式

```javascript
function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
  this.friends = ["Shelby", "Court"];
}
Person.prototype = {
  constructor : Person,
  sayName : function() {
    alert(this.name);
  }
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");

person1.friends.push("Van");
alert(person1.friends);    //"Shelby,Count,Van"
alert(person2.friends);    //"Shelby,Count"
alert(person1.friends === person2.friends); //false
alert(person1.sayName === person2.sayName); //true

```

### 动态原型模式

```javascript
function Person(name, age, job){
  //属性
  this.name = name;
  this.age = age;
  this.job = job;
  //方法
  if (typeof this.sayName != "function"){
    Person.prototype.sayName = function(){
      alert(this.name);
    };
  }
}
var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();

```

> 使用动态原型模式时，不能使用对象字面量重写原型。前面已经解释过了，如果 在已经创建了实例的情况下重写原型，那么就会切断现有实例与新原型之间的联系。


### 寄生构造函数模式

```javascript

function Person(name, age, job){
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function(){
    alert(this.name);
  };
  return o;
}

var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();  //"Nicholas"



function SpecialArray(){
  //创建数组
  var values = new Array();
  //添加值
  values.push.apply(values, arguments);
  //添加方法
  values.toPipedString = function(){
    return this.join("|");
  };
  //返回数组
  return values;
}

var colors = new SpecialArray("red", "blue", "green");
alert(colors.toPipedString()); //"red|blue|green"

```

### 稳妥构造函数模式
```javascript
function Person(name, age, job){
  //创建要返回的对象
  var o = new Object();
  //可以在这里定义私有变量和函数
  //添加方法
  o.sayName = function(){
    alert(name);
  };
  //返回对象
  return o;
}
```
> 与寄生构造函数模式类似，使用稳妥构造函数模式创建的对象与构造函数之间也 没有什么关系，因此 instanceof 操作符对这种对象也没有意义。

# 继承实现

### 原型链
> 实现原型链有一种基本模式，其代码大致如下

```javascript
function SuperType(){
  this.property = true;
}
SuperType.prototype.getSuperValue = function(){
  return this.property;
};
function SubType(){
  this.subproperty = false;
}

//继承了 SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function (){
  return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue()); //true

```

### 借用构造函数

```javascript
function SuperType(){
  this.colors = ["red", "blue", "green"];
}
function SubType(){
  //继承了 SuperType
  SuperType.call(this);
}
var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors);    //"red,blue,green,black"
var instance2 = new SubType();
alert(instance2.colors);    //"red,blue,green"
```

### 组合继承

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
  alert(this.name);
};

function SubType(name, age) {
  //继承属性 SuperType.call(this, name);
  this.age = age;
}

//继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function(){
    alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors); // "red,blue,green,black"
instance1.sayName(); // "Nicholas";
instance1.sayAge(); // 29

var instance2 = new SubType("Greg", 27);
alert(instance2.colors); //"red,blue,green"
instance2.sayName(); // "Greg";
instance2.sayAge(); // 27

```

### 原型式继承

```javascript
function object(o){
  function F() {}
  F.prototype = o;
  return new F();
}
```
> 在 object()函数内部，先创建了一个临时性的构造函数，然后将传入的对象作为这个构造函数的 原型，最后返回了这个临时类型的一个新实例。从本质上讲，object()对传入其中的对象执行了一次浅复制。

```javascript
var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");
alert(person.friends);   //"Shelby,Court,Van,Rob,Barbie"



var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = Object.create(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = Object.create(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");
alert(person.friends); //"Shelby,Court,Van,Rob,Barbie"



var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = Object.create(person, {
  name: {
    value: "Greg"
  }
});

alert(anotherPerson.name); //"Greg"

```

### 寄生式继承

```javascript
function createAnother(original) {
  var clone=object(original); //通过调用函数创建一个新对象

  clone.sayHi = function() { //以某种方式来增强这个对象
    alert("hi");
  };
  return clone; //返回这个对象
}


var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};
var anotherPerson = createAnother(person);
anotherPerson.sayHi(); //"hi"
```

> 使用寄生式继承来为对象添加函数，会由于不能做到函数复用而降低效率;这一 点与构造函数模式类似。

### 寄生组合式继承

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
  alert(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name); //第二次调用SuperType()
  this.age = age;
}

SubType.prototype = new SuperType(); //第一次调用SuperType()
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function(){
    alert(this.age);
};


function inheritPrototype(subType, superType) {
  var prototype = object(superType.prototype); //创建对象
  prototype.constructor = subType; //增强对象
  subType.prototype = prototype; //指定对象
}




function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
  alert(this.name);
};

function SubType(name, age){
  SuperType.call(this, name);
  this.age = age;
}

inheritPrototype(SubType, SuperType);
SubType.prototype.sayAge = function(){
  alert(this.age);
};
```



# 函数&递归

> 函数表达式不同于函数声明。函数声明要求有名字，但函数表达式不需要。没有名字的函数表 达式也叫做匿名函数。
>
> 在无法确定如何引用函数的情况下，递归函数就会变得比较复杂;
>
> 递归函数应该始终使用 arguments.callee 来递归地调用自身，不要使用函数名——函数名可
能会发生变化。


# 闭包

> 闭包是指有权访问另一个 函数作用域中的变量的函数。
>
> 在后台执行环境中，闭包的作用域链包含着它自己的作用域、包含函数的作用域和全局作用域。
>
> 通常，函数的作用域及其所有变量都会在函数执行结束后被销毁。
>
> 但是，当函数返回了一个闭包时，这个函数的作用域将会一直在内存中保存到闭包不存在为止。使用闭包可以在 JavaScript 中模仿块级作用域(JavaScript 本身没有块级作用域的概念)，要点如下。
>
> 创建并立即调用一个函数，这样既可以执行其中的代码，又不会在内存中留下对该函数的引用。
>
> 结果就是函数内部的所有变量都会被立即销毁——除非将某些变量赋值给了包含作用域(即外部作用域)中的变量。
>
> 即使 JavaScript 中没有正式的私有对象属性的概念，但可以使用闭包来实现公有方法，而通过公 有方法可以访问在包含作用域中定义的变量。
>
> 有权访问私有变量的公有方法叫做特权方法。
>
> 可以使用构造函数模式、原型模式来实现自定义类型的特权方法，也可以使用模块模式、增强
的模块模式来实现单例的特权方法。


```javascript
function createComparisonFunction(propertyName) {
  return function(object1, object2){
    var value1 = object1[propertyName];
    var value2 = object2[propertyName];
    if (value1 < value2){
      return -1;
    } else if (value1 > value2){
      return 1;
    } else {
      return 0;
    }
  };
}
```


> 由于闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存。过 度使用闭包可能会导致内存占用过多，我们建议读者只在绝对必要时再考虑使用闭 包。虽然像 V8 等优化后的 JavaScript 引擎会尝试回收被闭包占用的内存，但请大家 还是要慎重使用闭包。


- 模仿块级作用域

``` javascript
(function(){
  var now = new Date();
  if (now.getMonth() == 0 && now.getDate() == 1){
    alert("Happy new year!");
  }
})();
```
> 这种做法可以减少闭包占用的内存问题，因为没有指向匿名函数的引用。只要函 数执行完毕，就可以立即销毁其作用域链了
