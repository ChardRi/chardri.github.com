title: Javascript继承：原型继承
date: 2013-08-25 22:45:34
tags: ['Javascript','基础']
description: 原型继承是Javascript最重要的语言特性之一，它使得Javascript拥有了丰富、多变且适用于动态语言的对象系统。本文主要讨论原型继承中的几种实现方式(部分内容转自'Javascript面向对象编程（二）：构造函数的继承')。
---
原型继承是Javascript最重要的语言特性之一，它使得Javascript拥有了丰富、多变且适用于动态语言的对象系统。本文主要讨论原型继承中的几种实现方式(部分内容转自[Javascript面向对象编程（二）：构造函数的继承](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html))。

### prototype模式

如果Dog的一个对象，指向了Animal的一个实例，那么所有Dog的实例，都能继承Animal了。

```
function Animal(){
	this.species = "动物";
	this.getSpecies = function(){
		console.log(this.species);
	}
}
function Dog(name,color){
	this.name = name;
	this.color = color;
}
Dog.prototype = new Animal();
Dog.prototype.constructor = Dog; // 重设构造函数
var dogA = new Dog("大黄","黄色");
console.log(dogA.species); // 动物
```

<!--more-->

### 直接继承prototype

由于Animal对象中，不变的属性都可以直接写入`Animal.prototype`。所以，我们也可以让`Dog()`跳过`Animal()`，直接继承`Animal.prototype`。 现在，我们先将Animal对象改写：

```
function Animal(){ }
Animal.prototype.species = "动物";
```

然后，将Dog的`prototype`对象，然后指向Animal的`prototype`对象，这样就完成了继承。

```
Dog.prototype = Animal.prototype;
Dog.prototype.constructor = Dog; // 重设构造函数
var dogA = new Dog("大黄","黄色");
console.log(dogA.species); // 动物
```

与前一种方法相比，这样做的优点是效率比较高（不用执行和建立Animal的实例了），比较省内存。缺点是`Dog.prototype`和`Animal.prototype`现在指向了同一个对象，那么任何对`Dog.prototype`的修改，都会反映到`Animal.prototype`。
所以，上面这一段代码其实是有问题的。请看第二行：

```
Dog.prototype.constructor = Dog;
```

这一句实际上把`Animal.prototype`对象的`constructor`属性也改掉了！

```
console.log(Animal.prototype.constructor); // Dog
```

并且，我们需要注意此处对于`Dog.prototype`的修改（具体原因参考博文：Javascript变量）:

```
Dog.prototype = {
	species:"植物"
}
console.log(Animal.prototype.species); //动物。此时Dog.prototype重新开辟了一个内存地址，与Animal.prototype指向不同。
Dog.prototype.species = "植物";
console.log(Animal.prototype.species); //植物
```

### 利用空对象作为中介

由于"直接继承prototype"存在上述的缺点，所以就有这种方法，利用一个空对象作为中介。

```
var F = function(){};
F.prototype = Animal.prototype;
Dog.prototype = new F();
Dog.prototype.constructor = Dog;
```

F是空对象，所以几乎不占内存。这时，修改Dog的`prototype`对象，就不会影响到Animal的`prototype`对象。

```
console.log(Animal.prototype.constructor); // Animal
```

我们将上面的方法，封装成一个函数，便于使用。

```
function extend(Child, Parent) {
	var F = function(){};
	F.prototype = Parent.prototype;
	Child.prototype = new F();
	Child.prototype.constructor = Child;
	Child.uber = Parent.prototype;
}
```

使用的时候，方法如下

```
extend(Dog,Animal);
var dogA = new Dog("大毛","黄色");
console.log(dogA.species); // 动物
```

这个`extend`函数，就是YUI库如何实现继承的方法。 另外，说明一点，函数体最后一行

```
Child.uber = Parent.prototype;
```

意思是为子对象设一个`uber`属性，这个属性直接指向父对象的`prototype`属性。（uber是一个德语词，意思是"向上"、"上一层"。）这等于在子对象上打开一条通道，可以直接调用父对象的方法。这一行放在这里，只是为了实现继承的完备性，纯属备用性质。

### 利用Object.create

`Object.create`是ECMAScript第五版中的方法。使用这个方法，"类"就是一个对象，不是函数。
这种方法和“利用空对象作为中介”一致，只是ECMAScript第五版原生自带这个方法。

```
Dog.prototype = Object.create(Animal.prototype);
```