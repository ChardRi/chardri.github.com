title: 'Javascript类型判断&&相关知识'
date: 2013-08-18 22:29:04
tags: ['Javascript','基础']
description: typeof是一种快速但是作用有限的判断方式。注意typeof的结果是字符串。
---
### typeof

`typeof`是一种快速但是作用有限的判断方式。注意`typeof`的结果是字符串。

```
// Numbers
typeof 37 === 'number';
typeof Math.LN2 === 'number';
typeof Infinity === 'number';
typeof NaN === 'number'; // Despite being "Not-A-Number"
typeof Number(1) === 'number'; // but never use this form! 
// Strings
typeof "" === 'string';
typeof "bla" === 'string';
typeof (typeof 1) === 'string'; // typeof always return a string
typeof String("abc") === 'string'; // but never use this form! 
// Booleans
typeof true === 'boolean';
typeof false === 'boolean';
typeof Boolean(true) === 'boolean'; // but never use this form! 
// Undefined
typeof undefined === 'undefined';
typeof blabla === 'undefined'; // an undefined variable 
// Objects
typeof {a:1} === 'object';
typeof [1, 2, 4] === 'object';
// Array.isArray or Object.prototype.toString.call to differentiate regular objects from arrays
typeof new Date() === 'object'; 
typeof new Boolean(true) === 'object'; // this is confusing. Don't use!
typeof new Number(1) === 'object'; // this is confusing. Don't use!
typeof new String("abc") === 'object';  // this is confusing. Don't use! 
// Functions
typeof function(){} === 'function';
typeof Math.sin === 'function';
// null
typeof null === 'object';
// Regular expressions
typeof /s/ === 'function'; // Chrome 1-12 ... // Non-conform to ECMAScript 5.1
typeof /s/ === 'object'; // Firefox 5+ ...    // Conform to ECMAScript 5.1
```

特别需要注意`typeof`对于`String(' ')`和`new String(' ')`的判定。

### instanceof

`instanceof`能够检测目标是否是一个特殊类型的对象。
具体来说，它其实是检查对象的构造函数是否严格等于(===)给定的构造函数。`instanceof`检查`a.prototype`而不是`a`。
我们可以通过对象的`constructor`属性得到它的构造函数，所以可以说`obj.construtor`和`instanceof`是等价的。

```
var t = new Date(...); 
alert(a instanceof Date); // true
```

`instanceof`不适用于primitive values。

```
3 instanceof Number // false
'abc' instanceof String // false
true instanceof Boolean // false
```

当原型被重写时，`instanceof`会失效。

```
function C(){} // defining a constructor
var o = new C();

C.prototype = {};
var o2 = new C();

o2 instanceof C; // true
o instanceof C; // false, because C.prototype is nowhere in o's prototype chain anymore

o.__proto__ = C.prototype; // make C.prototype in o's prototype chain again
o instanceof C; // true
```

此外，如果在跨域环境中（frames，iframes），从其他窗口返回的值不能通过`instanceof`取得类型。
比如从另一个窗口接收到`Date d`，而`d instanceof Date`会返回`false`，因为它不是通过本窗口的`Date`创建的。
同理：`obj.constructor`也不适用于跨域情况。

### obj.constructor

```
var d = new Date();
alert(d instanceof Date);  // "true"
alert(d.constructor === Date);  // also "true"
```

不同于instanceof，obj.construtor适用于primitive values。

```
3..constructor === Number // true
//Why two dots for the 3? Because Javascript interprets the first dot as a decimal point ;)
'abc'.constructor === String // true
true.constructor === Boolean // true
```

如果你显式重定义了`obj.constructor`，那么`obj.constructor`将会变成你新定义的值。
因为当我们调用`a.constructor`的时候，内部操作其实是`ToObject(a).prototype.constructor`。但是`instanceof`还是会显示原来的值。

```
var d = new Date();
d.constructor = Number;
alert(d instanceof Date);  // "true"
alert(d.constructor);  // function Number() { [ native code ] }

var d = new Date();
Date.prototype.constructor = Number;
alert(d instanceof Date);  // "true"
alert(d.constructor);  // function Number() { [ native code ] }
```

如果通过原型继承，`constructor`会指向最上层的对象。（注意：此时没有闭合原型链，如果闭合原型链，那么`f.constructior == b`）

```
function a() { this.foo = 1;}
function b() { this.bar = 2; }
b.prototype = new a(); // b inherits from a

var f = new b(); // create new object with the b constructor
(f.constructor == b); // false
(f.constructor == a); // true
```

此外，IE9以及现代浏览器支持obj.constructor.name，能够直接返回构造函数的名字。特别需要注意匿名函数。

```
// using a named function:
function Foo() { this.a = 1; }
var obj = new Foo();
(obj instanceof Object);          // true
(obj instanceof Foo);             // true
(obj.constructor == Foo);         // true
(obj.constructor.name == "Foo");  // true

// let's add some prototypical inheritance
function Bar() { this.b = 2; }
Foo.prototype = new Bar();
obj = new Foo();
(obj instanceof Object);          // true
(obj instanceof Foo);             // true
(obj.constructor == Foo);         // false
(obj.constructor.name == "Foo");  // false

// using an anonymous function:
obj = new (function() { this.a = 1; })();
(obj instanceof Object);              // true
(obj.constructor == obj.constructor); // true
(obj.constructor); // function () { this.a = 1; }
(obj.constructor.name == "");         // true

// using an anonymous function assigned to a variable
var Foo = function() { this.a = 1; };
obj = new Foo();
(obj instanceof Object);      // true
(obj instanceof Foo);         // true
(obj.constructor == Foo);     // true
(obj.constructor.name == ""); // true

// using object literal syntax
obj = { foo : 1 };
(obj instanceof Object);            // true
(obj.constructor == Object);        // true
(obj.constructor.name == "Object"); // true
```

### Object.prototype.toString

`Object.prototype.toString`速度慢于`typeof`和`instanceof`，但是能够更准确区分对象的类型。不过它只能针对系统内置的javascript构造函数（比如`Date`和`String`），不能用于我们自己构造的函数。返回值是`[object *]`。

```
var what = Object.prototype.toString;  //等同于toString
alert(what.call(new Date()));   // "[object Date]" 
alert(what.call(function(){})); // "[object Function]" 
alert(what.call([]));           // "[object Array]"
alert(what.call(null)); // "[object Null]"
alert(what.call(undefined)); // "[obeject Undefined]"
```

相比`instanceof`，`Object.prototype.toString`可以取到其他窗口的对象的类型。
对于我们自己构造的函数，`Object.prototype.toString`返回值是`[object Object]`

```
function Foo(){};
var foo = new Foo();
var what = Object.prototype.toString;
alert(what.call(foo));  // "[object Object]"
```

一种Hack方法

```
Object.prototype.getName = function() { 
   var funcNameRegex = /function (.{1,}?)\(/;
   var results = (funcNameRegex).exec((this).constructor.toString());
   return (results && results.length > 1) ? results[1] : "";
};
```