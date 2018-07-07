---
title: 你不知道的javascript(上)
date: 2017-07-01 22:06:04
tags: [javascript]
categories: [web ,技术]
---
前两天又看了一遍《你不知道的javascript（上）》，依旧收货颇多。发现书上将的一些内容，再次看的时候已经记不清楚了。一些理解有偏颇的概念，得到了重新的认识。最后讲的“行为委托”对象关联模型与面向“类”的编程思想进行了对比，作者阐述了对象关联模型在javascript中的优势，应用对象关联模型进行javascript编程更加自然、更加符合javascript语言特性。
下面我记载了一些，自己没有掌握的或者容易出错、容易遗忘的内容。
<!--more-->
# 作用域与闭包
在严格模式的程序中，`eval(..)`在运行时有其自己的词法作用域，意味着其中的声明无法
修改所在的作用域。
```js
function foo(str) {
"use strict";
eval( str );
console.log( a ); // ReferenceError: a is not defined
}
foo( "var a = 2");
```
`eval(..)`函数如果接受了含有一个或多个声明的代码，就会修改其所处的词法作用域，而`with`声明
实际上是根据你传递给它的对象凭空创建了一个全新的词法作用域。
>另外一个不推荐使用eval(..)和with的原因是会被严格模式所影响（限制）。with被完全
>禁止，而在保留核心功能的前提下，间接或非安全地使用eval(..)也被禁止了。

JavaScript引擎会在编译阶段进行数项的性能优化。其中有些优化依赖于能够根据代码的词法进
行静态分析，并预先确定所有变量和函数的定义位置，才能在执行过程中快速找到标识符。

但如果引擎在代码中发现了eval(..)或with，它只能简单地假设关于标识符位置的判断都是无效
的，因为无法在词法分析阶段明确知道eval(..)会接收到什么代码，这些代码会如何对作用域进行
修改，也无法知道传递给with用来创建新词法作用域的对象的内容到底是什么。

设计的时候要遵循`最小特权原则`

## 函数声明和表达式
>区分函数声明和表达式最简单的方法是看function关键字出现在声明中的位置（不仅仅
>是一行代码，而是整个声明中的位置）。如果function是声明中的第一个词，那么就是一个函数
>声明，否则就是一个函数表达式。函数声明会提升

```js
foo(); // 1
var foo;
function foo() {
  console.log( 1 );
}
foo = function() {
  console.log( 2 );
};
```
上面代码片段被引擎理解为：
```js
function foo() {
  console.log( 1 );
}
foo(); // 1
foo = function() {
  console.log( 2 );
};
```
> 注意，var foo尽管出现在function foo()...的声明之前，但它是重复的声明（因此被忽略了），因为
函数声明会被提升到普通变量之前。

一段代码及输出如下:
```js
b = c;

b();
console.log(a);    //1
console.log(b);    //2
console.log(c);    //3

function c() {
    a = 1, b = 2, c = 3;
};
```
将上述代码稍作修改:
```js
b = function c() {
    a = 1, b = 2, c = 3;
};

b();
console.log(a);    //1
console.log(b);    //2
console.log(c);    //Uncaught ReferenceError: c is not defined
```
再次将上述代码稍作修改:
```js
b = function c() {
    a = 1, b = 2, c = 3;
    console.log(a);    //1
    console.log(b);    //2
    console.log(c);    //fuction c(){...  这里c仍然是一个function 只读的。如果使用var c = 3;这里就是3了。
};
b();
```
Here’s how it works: when the interpreter at the code execution stage meets named FE, before creating FE, it creates auxiliary special object and adds it in front of the current scope chain. Then it creates FE itself at which stage the function gets the [[Scope]] property (as we know from the Chapter 4. Scope chain) — the scope chain of the context which created the function (i.e. in [[Scope]] there is that special object). After that, the name of FE is added to the special object as unique property; value of this property is the reference to the FE. And the last action is removing that special object from the parent scope chain. Let’s see this algorithm on the pseudo-code:

```js
specialObject = {};
  
Scope = specialObject + Scope;
  
foo = new FunctionExpression;
foo.[[Scope]] = Scope;
specialObject.foo = foo; // {DontDelete}, {ReadOnly}
  
delete Scope[0]; // remove specialObject from the front of scope chain
```

# this和对象原型

`forEach`:`[1,2,3].forEach(fn,obj)`第二个参数obj表示fn调用时的上下文，fn中this关键字指向obj。

使用`new`来调用函数，或者发生构造函数调用时，会自动执行下面的操作。
1. 构造一个全新的对象
2. 这个新对象会被执行[[原型]]连接
3. 这个新对象会绑定到函数调用的this
4. 如果函数没有返回其它对象，那么`new`表达式中的函数调用会自动返回这个新对象。

```js

Function.prototype.bind = function (oThis) {
      if (typeof this !== "function") {
        // 与 ECMAScript 5 最接近的
        // 内部 IsCallable 函数
        throw new TypeError(
          "Function.prototype.bind - what is trying " +
          "to be bound is not callable"
        );
      }
      debugger
      var aArgs = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP = function () { },
        fBound = function () {
          return fToBind.apply(
  //这里的this，如果是通过new调用，this会指向一个对象。通过this instanceof fNOP可以判断调用的方式是直接调用还是通过new。
  //如果bind(null)或者bind(undefined),则相当于执行fToBind.apply(null,args)，这时this指向window。这点和原生bind不一致。仅使用this //instanceof fNOP做判断，应该更合理。
  //如果bind(obj),则相当于执行fToBind.apply(this,args),这时this指向新创建的对象。
  //MDN上现在这里已经改为this instanceof fNOP? this: oThis || this。
            this instanceof fNOP &&
              oThis ? this : oThis  
            ,
            aArgs.concat(
              Array.prototype.slice.call(arguments)
            )
          );
        }
        ;
      fNOP.prototype = this.prototype;
      fBound.prototype = new fNOP();
      return fBound;
    };
    var foo = function (b) {
      this.a = b;
    }
    var bar = {};
    var baz = foo.bind(bar)
    baz(0);
    console.log(bar.a);
    var baw = new baz(1);
    console.log(baw.a);
```

## 判断this
现在我们可以根据优先级来判断函数在某个调用位置应用的是哪条规则。可以按照下面的顺序来
进行判断：
1. 函数是否在new中调用（new绑定）？如果是的话this绑定的是新创建的对象。
`var bar = new foo()`
2. 函数是否通过call、apply（显式绑定）或者硬绑定调用？如果是的话，this绑定的是指定的对象。
`var bar = foo.call(obj2)`
3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this绑定的是那个上下文对象。
`var bar = obj1.foo()`
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。
`var bar = foo()`
就是这样。对于正常的函数调用来说，理解了这些知识你就可以明白this的绑定原理了。

```js
function foo() {
console.log( this.a );
}
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };
o.foo(); // 3
(p.foo = o.foo)(); // 2
```

赋值表达式的返回值是目标函数的引用，因此调用位置是`foo()`而不是`p.foo()`或者`o.foo()`。  

**注意：对于默认绑定来说，决定this绑定对象的并不是调用位置是否处于严格模式，而是函数体是
否处于严格模式。如果函数体处于严格模式，this会被绑定到undefined，否则this会被绑定到全局
对象。**  

## 箭头函数
```js
function foo() {
  // 返回一个箭头函数
  return (a) => {
  //this继承自foo()
    console.log( this.a );
  };
}
var obj1 = {
  a:2
};
var obj2 = {
  a:3
};
var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 不是3！
```
foo()内部创建的箭头函数会捕获调用时foo()的this。由于foo()的this绑定到obj1，bar（引用箭头
函数）的this 也会绑定到obj1，箭头函数的绑定无法被修改。（new也不行！）.

## 对象
注意，简单基本类型（string、boolean、number、null和undefined）本身并不是对象。null有时会被当
作一种对象类型，但是这其实只是语言本身的一个bug，即对null执行typeof null时会返回字符
串"object"。1实际上，null本身是基本类型。
1原理是这样的，不同的对象在底层都表示为二进制，在JavaScript中二进制前三位都为0的话会被
判断为object类型，null的二进制表示是全0，自然前三位也是0，所以执行typeof时会返
回“object”

### 属性描述符
1. Writable
`writable`:决定是否可以修改属性的值。
```js
var myObject = {};
Object.defineProperty( myObject, "a", {
  value: 2,
  writable: false, // not writable!
  configurable: true,
  enumerable: true
} );
myObject.a = 3;
myObject.a; // 2 严格模式下会报错TypeError
```
2. Configurable
只要属性是可配置的，就可以使用`defineProperty(..)`来修改属性描述符。
```js
var myObject = {
  a:2
};
myObject.a = 3;
myObject.a; // 3
Object.defineProperty( myObject, "a", {
  value: 4,
  writable: true,
  configurable: false, // 不可配置！
  enumerable: true
} );
myObject.a; // 4
myObject.a = 5;
myObject.a; // 5
Object.defineProperty( myObject, "a", {
  value: 6,
  writable: true,
  configurable: true,
  enumerable: true
} ); // TypeError
Object.defineProperty( myObject, "a", {
  value: 6,
  writable: false, // 将writable由true改为false，不会报错
  configurable: false,
  enumerable: true
} ); 
```
`configurable:false`会禁止删除这个属性
```js
var myObject = {
  a:2 
};
myObject.a; // 2
delete myObject.a;
myObject.a; // undefined
Object.defineProperty( myObject, "a", {
  value: 2,
  writable: true,
  configurable: false,
  enumerable: true
} );
myObject.a; // 2
delete myObject.a;
myObject.a; // 2
```
3. Enumerable
控制对象的属性是否会出现在对象的属性枚举中，比如说`for...in`循环。如果为false，这个属性不会出现在枚举中，虽然仍然可以正常访问它。

### 不变性
1. 对象常量
不可修改、重定义或者删除。
```js
var myObject = {};
Object.defineProperty( myObject, "FAVORITE_NUMBER", {
  value: 42,
  writable: false,
  configurable: false
} );
```
2. 禁止扩展
禁止一个对象添加新属性并且保留已有属性，`Object.preventExtensions(..)`
```js
var myObject = {
  a:2
};
Object.preventExtensions( myObject );
myObject.b = 3;
myObject.b; // undefined严格模式会TypeError
```
3. 密封
`Object.seal(..)`会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调
用`Object.preventExtensions(..)`并把所有现有属性标记为`configurable:false`。
所以，密封之后不仅不能添加新属性，也不能重新配置或者删除任何现有属性（虽然可以修改属性
的值）。
4. 冻结
`Object.freeze(..)`会创建一个冻结对象，这个方法实际上会在一个现有对象上调
用`Object.seal(..)`并把所有“数据访问”属性标记为`writable:false`，这样就无法修改它们的值。

### `[[Get]]`
```js
var myObject = {
  a: 2
};
myObject.a; // 2
```
`myObject.a`是一次属性访问，但是这条语句并不仅仅是在`myObjet`中查找名字为`a`的属性，虽然看起
来好像是这样。
在语言规范中，`myObject.a`在`myObject`上实际上是实现了`[[Get]]`操作（有点像函数调
用：`[[Get]]()`）。对象默认的内置`[[Get]]`操作首先在对象中查找是否有名称相同的属性，如果找到
就会返回这个属性的值。
然而，如果没有找到名称相同的属性，按照`[[Get]]`算法的定义会执行另外一种非常重要的行为。就是遍历可能存在的`[[Prototype]]`链，也就是原型链。
### [[Put]]
如果已经存在这个属性，`[[Put]]`算法大致会检查下面这些内容。
1. 属性是否是访问描述符,如果是并且存在`setter`就调用`setter`。
2. 属性的数据描述符中`writable`是否是`false`,如果是，在非严格模式下静默失败，在严格模式下
抛出`TypeError`异常。
3. 如果都不是，将该值设置为属性的值。

### Getter和 Setter
对象默认的`[[Put]]`和`[[Get]]`操作分别可以控制属性值的设置和获取。

在ES5中可以使用`getter`和`setter`部分改写默认操作，但是只能应用在单个属性上，无法应用在整
个对象上。`getter`是一个隐藏函数，会在获取属性值时调用。`setter`也是一个隐藏函数，会在设置属
性值时调用。
当你给一个属性定义`getter`、`setter`或者两者都有时，这个属性会被定义为“访问描述符”（和“数据描
述符”相对）。
对于访问描述符来说，JavaScript会忽略它们的`value`和`writable`特性，取而代之的是
关心`set`和`get`（还有`configurable`和`enumerable`）特性。
```js
var myObject = {
//给a定义一个getter
get a() {
  return 2;
}
};
Object.defineProperty(
  myObject, // 目标对象
  "b", // 属性名
  { // 描述符
  // 给b设置一个getter
  get: function(){ return this.a * 2 },
  // 确保b会出现在对象的属性列表中
  enumerable: true
  }
);
myObject.a; // 2
myObject.b; // 4
//也可以使用字面量形式
var myObject2 = {
  // 给 a 定义一个getter
  get a() {
  return this._a_;
  },
  // 给 a 定义一个setter
  set a(val) {
  this._a_ = val * 2;
}
};
myObject2.a = 2;
myObject2.a; // 4
```

## 遍历

>遍历数组下标时采用的是数字顺序（for循环或者其他迭代器），但是遍历对象属性时的顺
序是不确定的，在不同的JavaScript引擎中可能不一样。因此，在不同的环境中需要保证一致性
时，一定不要相信任何观察到的顺序，它们是不可靠的。

es6 增加的`for...of`可以直接遍历数组值。（如果对象定义了迭代器的话也可以遍历对象）
数组有内置的@@iterator，因此for..of可以直接应用在数组上。

```js
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();
it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```
> @@iterator本身并不是一个迭代器对象，而是一个返回迭代器对象的函数——这点非常精妙并且非常重要。

可以给想遍历的对象定义@@iterator
```js
var myObject = {
  a: 2,
  b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
  enumerable: false,
  writable: false,
  configurable: true,
  value: function(){
    var o = this;
    var idx = 0;
    var ks = Object.keys( o );
    return {
      next: function() {
        return {
          value: o[ks[idx++]];
          done: (idx > ks.length)
        }
      }
    }
  }
});
// 手动遍历myObject
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }
// 用for..of遍历myObject
for (var v of myObject) {
  console.log( v );
} // 2
// 3
```
随机数

```js
var randoms = {
  [Symbol.iterator]: function() {
    return {
      next: function() {
        return { value: Math.random() };
      }
    };
  }
};
var randoms_pool = [];
for (var n of randoms) {
  randoms_pool.push( n );
  if(randoms_pool.length === 100) break;
}
```

## 混合对象“类”

## 原型
`Object.prototype`
所有普通的[[Protorype]]链最终都会执行内置的`Object.prototype`.

### 属性设置和屏蔽
`myObject.foo='bar'`;
1. 如果在[[Prototype]]链上层存在名为foo的普通数据访问属性（参见第3章）并且没有被标记为
只读（writable:false），那就会直接在myObject中添加一个名为foo的新属性，它是屏蔽属性。
2. 如果在[[Prototype]]链上层存在foo，但是它被标记为只读（writable:false），那么无法修改已
有属性或者在myObject上创建屏蔽属性。如果运行在严格模式下，代码会抛出一个错误。否则，这条
赋值语句会被忽略。总之，不会发生屏蔽。
3. 如果在[[Prototype]]链上层存在foo并且它是一个setter（参见第3章），那就一定会调用这个
setter。foo不会被添加到（或者说屏蔽于）myObject，也不会重新定义foo这个setter。
如果你希望在第二种和第三种情况下也屏蔽foo，那就不能使用=操作符来赋值，而是使
用Object.defineProperty(..)来向myObject添加foo。

>这看起来有点奇怪，myObject对象竟然会因为其他对象中有一个只读foo就不能包含foo属性。更奇怪的是，这个
限制只存在于=赋值中，使用Object.defineProperty(..)并不会受到影响。

```js
var anotherObject = {
  a:2
};
var myObject = Object.create( anotherObject );
anotherObject.a; // 2
myObject.a; // 2
anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false
myObject.a++; // 隐式屏蔽！
anotherObject.a; // 2
myObject.a; // 3
myObject.hasOwnProperty( "a" ); // true
```
在普通的函数调用前加上`new`关键字之后，就会把这个函数调用变成一个“构造函数调用”。实际上，new会劫持所有普通函数并用构造对象的形式调用它。
```js
function NothingSpecial() {
console.log( "Don't mind me!" );
}
var a = new NothingSpecial();
// "Don't mind me!"
a; // {}
```
`NothingSpecial`只是一个普通的函数，但是使用`new`调用时，它就会构造一个对象并赋值给a，这看
起来像是`new`的一个副作用（无论如何都会构造一个对象）。这个调用是一个构造函数调用，但
是`NothingSpecial`本身并不是一个构造函数。
换句话说，在JavaScript中对于“构造函数”最准确的解释是，所有带`new`的函数调用。
函数不是构造函数，但是当且仅当使用`new`时，函数调用会变成“构造函数调用”。

`constructor`:`Foo.prototype`的`.constructor`属性只是`Foo`函数在声明时的默认属性。可以被更改，不可枚举。


## （原型）继承

```js
function Foo(name) {
  this.name = name;
}

Foo.prototype.myName = function() {
  return this.name;
}

function Bar(name,label) {
  Foo.call( this, name );
  this.label = label;
}
//创建一个新的Bar.prototype对象并关联到Foo.prototype
Bar.prototype = Object.create( Foo.prototype );
// 注意！现在没有Bar.prototype.constructor了
// 如果你需要这个属性的话可能需要手动修复一下它
Bar.prototype.myLabel = function() {
  return this.label;
}

var a = new Bar( "a", "Obj a" );
a.myName();//"a"
a.myLabel();//"obj a"

```
`Object.create(..)`会创建一个‘新’对象并把新对象内部的`[[prototype]]`关联到你指定的对象。
在声明`function Bar(){..}`的时候，和其他函数一样，`Bar`会有一个`.prototype`关联到默认的对象，但是这个对象并不是我们想要的`Foo.prototype`。因此创建了一个新的对象并把它关联到我们希望的对象上，直接把原始的关联对象抛弃掉。
注意，下面这两种方式是常见的错误做法，实际上他们都存在一些问题：
```js

Bar.prototype = Foo.prototype;
//基本满足需求，但是可能会产生一些副作用。
Bar.prototype = new Foo();
```
`Bar.prototype = Foo.prototype`并不会创建一个关联到`Bar.prototype`的新对象，它只是
让`Bar.prototype`直接引用`Foo.prototype`对象。因此当你执行类似`Bar.prototype.myLabel = ...`的赋
值语句时会直接修改`Foo.prototype`对象本身。显然这不是你想要的结果，否则你根本不需要`Bar`对
象，直接使用`Foo`就可以了，这样代码也会更简单一些。

`Bar.prototype = new Foo()`的确会创建一个关联到`Bar.prototype`的新对象。但是它使用了`Foo(..)`的‘构造函数调用’，如果函数`Foo`有一些副作用（比如写日志、修改状态、注册到其它对象、给`this`添加数据属性，等等）的话，就会影响到`Bar()`的后代，后果不堪设想。

使用`Object.create(..)`的唯一的缺点就是需要创建一个新的对象，然后把旧对象抛弃掉，不能直接修改已有的默认对象。

在ES6之前，我们只能通过设置`.__proto__`属性来实现，但是这个方法并不是标准并且无法兼容所有浏览器。ES6添加了
辅助函数`Object.setPrototypeOf(..)`，可以用标准并且可靠的方法来修改关联。
```js
Bar.prototype = Object.create(Foo.prototype);
//ES6 开始可以直接修改现有的Bar.prototype
Object.setPrototypeof( Bar.prototype, Foo.prototype);
```
如果忽略掉`Object.create(..)`方法带来的轻微性能损失（抛弃的对象需要进行垃圾回收），它实际
上比ES6及其之后的方法更短而且可读性更高。不过无论如何，这是两种完全不同的语法。

`instanceof`
```js
function Foo{

}
Foo.prototype.blah = ...;
var a = new Foo();
a instanceof Foo
```
`instanceof`:判断在对象`a`的整条`[[prototype]]`链中是否有指向`Foo.prototype`的对象。

可惜，这个方法只能处理对象（`a`）和函数（带`.prototype`引用的`Foo`）之间的关系。如果你想判断两
个对象（比如`a`和`b`）之间是否通过`[[Prototype]]`链关联，只用`instanceof`无法实现。

```js
Foo.prototype.isPrototypeOf(a);//true
```
`isPrototype`:判断在`a`的整条`[[Prototype]]`链中是否出现过`Foo.prototype`.
```js
// 非常简单：b是否出现在c的[[Prototype]]链中？
b.isPrototypeOf( c );
```

`Object.getPrototypeOf(a)`:获取`a`的`[[Prototype]]`链。

`a.__proto__ === Foo.prototype; // true`,`.__proto__`（在ES6之前并不是标准！）引用对象内部的`[[Prototype]]`对象。
## 对象关联
`[[Prototype]]`机制就是存在于对象中的一个内部链接，它会引用其他对象。
通常来说，这个链接的作用是：如果在对象上没有找到需要的属性或者方法引用，引擎就会继续
在`[[Prototype]]`关联的对象上进行查找。同理，如果在后者中也没有找到需要的引用就会继续查
找它的`[[Prototype]]`，以此类推。这一系列对象的链接被称为“原型链”。
### 创建关联
```js
var anotherObject = {
a:2
};
var myObject = Object.create( anotherObject, {
  b: {
    enumerable: false,
    writable: true,
    configurable: false,
    value: 3
  },
  c: {
    enumerable: true,
    writable: false,
    configurable: false,
    value: 4
  }
});
myObject.hasOwnProperty( "a" ); // false
myObject.hasOwnProperty( "b" ); // true
myObject.hasOwnProperty( "c" ); // true
myObject.a; // 2
myObject.b; // 3
myObject.c; // 4

```
### 关联关系是备用

看起来对象之间的关联关系是处理“缺失”属性或者方法时的一种备用选项。这个说法有点道理，但
是我认为这并不是`[[Prototype]]`的本质。
```js
  var anotherObject = {
  cool: function() {
  console.log( "cool!" );
}
};
var myObject = Object.create( anotherObject );
myObject.cool(); // "cool!"
```
由于存在`[[Prototype]]`机制，这段代码可以正常工作。但是如果你这样写只是为了让`myObject`在无
法处理属性或者方法时可以使用备用的`anotherObject`，那么你的软件就会变得有点“神奇”，而且很
难理解和维护。
但是你可以让你的API设计不那么“神奇”，同时仍然能发挥`[[Prototype]]`关联的威力：
```js
var anotherObject = {
  cool: function() {
  console.log( "cool!" );
}
};
var myObject = Object.create( anotherObject );
myObject.doCool = function() {
  this.cool(); // 内部委托！
};
myObject.doCool(); // "cool!"
```
内部委托比起直接委托可以让API接口设计更加清晰。

# 行为委托

“类”和“委托”这两种设计模式，在思维模型方面的区别。
典型的面向对象风格：
```js
function Foo(who) {
  this.me = who;
}
Foo.prototype.identify =function() {
  return "I am " + this.me;
};
function Bar(who) {
  Foo.call( this, who );
} 
Bar.prototype =Object.create( Foo.prototype );
Bar.prototype.speak = function() {
  alert( "Hello, " + this.identify() + "." );
};
var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );
b1.speak();
b2.speak();
```
对象关联风格：
```js
Foo = {
init: function(who) {
this.me = who;
},
identify: function() {
return "I am " + this.me;
}
};
Bar = Object.create( Foo );
Bar.speak = function() {
alert( "Hello, " + this.identify() + "." );
};
var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );
b2.init( "b2" );
b1.speak();
b2.speak();
```
## 总结
总问言之，javascript中对象关联比类风格的代码更加简洁（而且功能相同）
行为委托认为对象之间是兄弟关系，互相委托，而不是父类和子类的关系。JavaScript
的`[[Prototype]]`机制本质上就是行为委托机制。也就是说，我们可以选择在JavaScript中努力实现
类机制，也可以拥抱更自然的`[[Prototype]]`委托机制。
对象关联（对象之间相互关联）是一种编码风格，它倡导的是直接创建和关联对象，不把他们抽象成类。对象关联可以用基于`[[prototype]]`的行为委托自然的实现。
