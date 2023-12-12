---
title: js 继承的方法和优缺点
categories:
  - 基础
excerpt: 'js 继承的方法和优缺点'
description: 'js 继承的方法和优缺点'
date: 2023-12-12 15:18:29
---


## 原型链继承

- 优点：写法简单、容易理解。
- 缺点：
  1. 引用类型的值会被所有实例共享；
  2. 在子类实例对象创建时，不能向父类传参；

```js
function Parent() {
  this.parentPrototypeString = 'parentPrototype string';

  // 引用类型值会被所有实例共享
  this.parentPrototypeObject = {
    info: 'parentPrototype object',
  };
}

function Child() {}
// 原型链继承
Child.prototype = new Parent();

const child_1 = new Child();
const child_2 = new Child();

child_1.parentPrototypeString = 'child_1';
console.log('parentPrototypeString: ', child_1.parentPrototypeString); // parentPrototypeString:  child_1
console.log('parentPrototypeString: ', child_2.parentPrototypeString); // parentPrototypeString:  parentPrototype string

// 缺点：修改引用类型其他实例也会共享修改值
child_1.parentPrototypeObject.info = 'child_1';
console.log('parentPrototypeObject: ', child_1.parentPrototypeObject.info); // parentPrototypeObject:  child_1
console.log('parentPrototypeObject: ', child_2.parentPrototypeObject.info); // parentPrototypeObject:  child_1
```

## 借用构造函数继承

- 优点：
  1. 避免了引用类型的值会被所有实例共享；
  2. 在子类实例对象创建时，可以向父类传参；
- 缺点：方法在构造函数中，每次创建实例对象时都会重新创建一遍方法；（因为是使用的 call / apply，均在构造函数中实现）

```js
function Parent(name, id) {
  this.parentPrototypeString = 'parent prototype';
  this.parentPrototypeObject = {
    info: 'parent obj',
  };
  this.fn = function () {
    console.log('parent function');
  };
  console.log(name, id);
}

function Child(name, id) {
  // 可以传递参数
  Parent.call(this, name, id); // this作为函数上下文对象，apply、call 方法都会使函数立即执行
}

const child_1 = new Child();
const child_2 = new Child(); // 缺点 此时Parent()会再次创建一个fn函数，这个是没有必要的
child_1.parentPrototypeObject.info = 'child_1';
console.log('parentPrototypeObject: ', child_2.parentPrototypeObject.info); // parentPrototypeObject:  parent obj
```

## 组合继承

- 优点：

  1. 可继承父类原型上的属性，且可传参；
  2. 每个新实例引入的构造函数是私有的

- 缺点：
  1. 无论什么情况下，父类构造函数都会被调用两次，一是创建子类原型对象时，二是子类构造函数内部。

```js
function Parent(name, id) {
  this.id = id;
  this.name = name;
  this.list = ['1'];
  this.printName = function () {
    console.log(this.name);
  };
}
Parent.prototype.sayName = function () {
  console.log(this.name);
};

function Child(name, id) {
  Parent.call(this, name, id);
  // Parent.apply(this, arguments);
}
Child.prototype = new Parent();

const child = new Child('jin', '1');
child.printName(); // jin
child.sayName(); // jin

const child_1 = new Child();
const child_2 = new Child();
child_1.list.push('2');
console.log(child_2.list); // ['1']
```

## 原型式继承

- 优点：不需要单独创建构造函数；
- 缺点：引用类型的值会被所有实例共享。

```js
const parent = {
  names: ['a'],
};
function copy(object) {
  function F() {}
  F.prototype = object;
  return new F();
}

const child_1 = copy(parent);
const child_2 = copy(parent);
child_1.names.push('b');
console.log(child_2.names); // ['a', 'b']
```

> 以上例子解释：
> 创建一个函数 copy，内部定义一个构造函数 F
> 将 F 的原型对象设置为参数，参数是一个对象，完成继承
> 将 F 实例化后返回，即返回的是一个实例化对象

## 寄生式继承

- 优点：

  1. 不需要单独创建构造函数；
  2. 可添加新的属性和方法

- 缺点：方法在构造函数中，每次创建实例对象时都会重新创建一遍。

```js
const parent = {
  names: ['a'],
};
function copy(object) {
  function F() {}
  F.prototype = object;
  return new F();
}
function createObject(obj) {
  const o = copy(obj);
  o.getNames = function () {
    console.log(this.names);
    return this.names;
  };
  return o;
}

const child_1 = createObject(parent);
const child_2 = createObject(parent);
child_1.names.push('b');
console.log(child_2.getNames()); // ['a', 'b']
console.log(child_1.getNames == child_2.getNames); // false
```

> 或者使用 Object.create 实现

```js
function createObject(obj) {
  const o = Object.create(obj);
  o.getNames = function () {
    console.log(this.names);
    return this.names;
  };
  return o;
}
const parent = {
  names: ['a'],
};
const child_1 = createObject(parent);
const child_2 = createObject(parent);
child_1.names.push('b');
console.log(child_2.getNames()); // ['a', 'b']
console.log(child_1.getNames == child_2.getNames); // false
```

## 寄生组合继承

- 优点：高效率只调用一次父类构造函数，并且避免了子类原型对象上不必要、多余的属性，同时，还能将原型链保持不变，因此能使用 instanceof 和 isPrototypeOf。

- 缺点：代码复杂

```js
function copy(object) {
  function F() {}
  F.prototype = object;
  return new F();
}

function inheritPrototype(subClass, superClass) {
  // 复制一份父类的原型
  const p = copy(superClass.prototype);
  // 修正构造函数
  p.constructor = subClass;
  // 设置子类原型
  subClass.prototype = p;
}

function Parent(name, id) {
  this.id = id;
  this.name = name;
  this.list = ['a'];
  this.printName = function () {
    console.log(this.name);
  };
}

Parent.prototype.sayName = function () {
  console.log(this.name);
};

function Child() {
  Parent.apply(this, arguments);
}

inheritPrototype(Child, Parent);
```

