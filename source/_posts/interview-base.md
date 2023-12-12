---
title: 前端基础面试题
categories:
  - 面经
  - 基础知识
excerpt: '收敛的一些基础面经'
description: '收敛的一些基础面经'
date: 2023-12-12 15:05:59
---


## 浏览器输入 URL 发生了什么

1. DNS 解析:将域名解析成 IP 地址
2. TCP 连接：TCP 三次握手
3. 发送 HTTP 请求
4. 服务器处理请求并返回 HTTP 报文
5. 浏览器解析渲染页面
   > 根据 HTML 解析出 DOM 树
   > 根据 CSS 解析生成 CSS 规则树
   > 结合 DOM 树和 CSS 规则树，生成渲染树
   > 根据渲染树计算每一个节点的信息
   > 根据计算好的信息绘制页面
6. 断开连接：TCP 四次挥手

## 盒模型

盒模型分为四部分：content + padding + border + margin

IE 盒模型（box-sizing: border-box）： 总宽度 = width + margin
标准盒模型（box-sizing: content-box）： 总宽度 = width + padding + border + margin

## BFC

简单来说，BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。

解决的问题：浮动带来的高度塌陷、margin 边距重叠

BFC 触发条件：

- 根元素
- float 属性部位 none
- position 为 absolute 或 fixed
- oversion 属性不为 visible
- display 为 inline-block、table-cell、flex 等

## 浮动

会使元素向左或向右移动，其周围的元素也会重新排列。

清除浮动： clear: both、left、right

## 样式优先级的规则是什么

!important > 行内样式（1000） > ID 选择器（100） > 类选择器（10） > 标签 | 伪类 | 属性选择 （1）> 通配符 > 继承 > 浏览器默认属性

## CSS 尺寸设置的单位

px：像素单位，绝对尺寸单位。
em：相对于应用在当前元素的字体尺寸。一般浏览器默认为 16px，2em == 32px
rem：相对于根元素（html）的字体尺寸。
vm：视口尺寸，1vm == 100%。

## 未知宽高元素水平垂直居中方法

```html
<div class="container">
  <div class="element"></div>
</div>
```

1. flex 布局：

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

2. 定位(需要声明高度)

```CSS
.container {
  position: relative;
}
.element {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

3. Grid

```css
.container {
  display: grid;
  justify-items: center;
  align-items: center;
  height: 100vh; /* 设置高度为100%，以便看到居中效果 */
}
```

## 三栏布局的实现方案

```html
<div class="container">
  <div class="left"></div>
  <div class="center"></div>
  <div class="right"></div>
</div>
```

1. 浮动布局

```css
.left {
  float: left;
  width: 200px;
}
.right {
  float: right;
  width: 200px;
}
.center {
  float: left;
  margin: 0 200px;
}
```

2. 定位

```css
.container {
  position: relative;
}
.left {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 200px;
}
.left {
  position: absolute;
  top: 0;
  bottom: 0;
  right: 0;
  width: 200px;
}
.center {
  margin: 0 200px;
}
```

3. flex 布局

```css
.container {
  display: flex;
  /* flex-wrap: nowrap; */
  justify-content: space-between;
}
```

4. grid 布局

```css
.container {
  display: grid;
}
```

## 三列不定高元素怎么实现等高

```html
<div class="container">
  <div class="column">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
  </div>
  <div class="column">
    <div class="item">4</div>
    <div class="item">5</div>
    <div class="item">6</div>
  </div>
  <div class="column">
    <div class="item">7</div>
    <div class="item">8</div>
    <div class="item">9</div>
  </div>
</div>
```

```css
.container {
  display: flex;
  flex-wrap: wrap;
  margin: 0 auto;
  max-width: 1200px;
  padding: 10px;
}

.column {
  flex: 1;
  padding: 10px;
  box-sizing: border-box;
}

.item {
  padding: 20px;
  border: 1px solid #ccc;
  box-sizing: border-box;
}
```

在这个示例中，我们使用了以下 CSS 样式：

为.container 设置 display: flex，使其成为一个 flex 容器。
为.column 设置 flex: 1，使其占据剩余的可用空间。
为.item 设置 padding: 20px，设置项之间的间距。
为.container 设置 margin: 0 auto，使其在页面上居中显示。
为.column 设置 box-sizing: border-box，使其内部元素的大小包含边框和内边距。
这样，三列不定高元素就可以实现等高。当有一列元素高度较小时，其他两列元素会自动调整高度，以使整个容器的高度保持不变。

```css
.container {
  display: table;
  width: 100%;
  border-collapse: collapse;
}

.column {
  display: table-cell;
  padding: 10px;
  border: 1px solid #ccc;
}

.item {
  padding: 20px;
  border: 1px solid #ccc;
  box-sizing: border-box;
}
```

在这个示例中，我们使用了以下 CSS 样式：

为.container 设置 display: table，使其成为一个表格容器。
为.column 设置 display: table-cell，使其成为一个表格单元格。
为.item 设置 padding: 20px，设置项之间的间距。
为.container 设置 border-collapse: collapse，使其边框合并。
这样，三列不定高元素就可以实现等高。当有一列元素高度较小时，其他两列元素会自动调整高度，以使整个容器的高度保持不变。

## rgba 和 opacity 的透明效果有什么不同？

主要区别有两点：

1、opacity 作用于元素，以及元素内的所有内容的透明度，设置的透明度会被子级元素继承；rgba()只作用于元素的颜色或其背景色。（设置 rgba 透明的元素的子元素不会继承透明效果！）
2、rgba 可以设置 background-color , color , border-color , text-shadow , box-shadow

## display: none 与 visibility: hidden 的区别是什么？

display: none 不显示对应的元素，在文档布局中不在分配空间。
visibility: hidden 是隐藏对应元素，在文档布局中仍然保留原来的空间。

## position 详解

static: 默认值。没有定位，元素出现在正常的流中。
fixed: 元素相对浏览器的窗口位置定位。(脱离文档流)
relative: 元素相对正常位置定位。
absolute: 元素相对第一个非 static 定位。(脱离文档流)
sticky: 基于用户的滚动位置定位，实现吸顶效果。

## 实现单行和多行溢出隐藏

单行溢出

```css
.container {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

多行溢出

```css
.container {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
```

## HTML 语义化

使用正确的标签做正确的事。是页面内容结构化，便于浏览器、搜索引擎解析；即使没有 css 样式也会以一种文档格式显示，并且便于阅读。有利于 SEO。

## href 与 src

href 属性定义了超链接的目标 URL，目的不是为了引用资源，而是建立链接关系，比如 a 标签

src（source）指向外部资源的位置，指定的内容将会应用到当前标签的位置。比如 img 标签

区别：

作用结果不同： herf 为了建立链接，src 会将资源下载并应用到文档中

浏览器解析结果不同：当浏览器解析到 src（没有其他配置：defer async）会暂停其他资源的下载和处理，知道资源下载和解析执行完毕。

## 标签 title 和 alt 的属性

title 是元素的注释信息，主要给用户解读。鼠标放到文字或者图片有 title 文字显示。
alt 是对图片的描述，主要用于图片无法显示时，给用户提示。

## HTML 中的 script 元素定义了 6 个属性

async： 可选，表示立即下载脚本，但是不应该妨碍页面中的其他操作，比如下载其他资源或者等待加载其他脚本，只对外部脚本文件有效。

chartset：可选，src 属性指定的代码的字符集。

defer：可选，表示脚本可以延迟到文档完全被解析显示后执行。只对外部脚本文件有效。

language：已废弃，目前没有浏览器使用这个属性。

src：可选，表示要导入的外部脚本文件的 URL。

type：可选，可以看成 language 的替代属性。表示编写代码使用脚本语言的内容类型，默认为 text/javascript

## defer 和 async 区别

defer 和 async 都是仅仅适用于外部脚本，异步下载。

defer 会在 html 解析完成之后执行，async 则是下载完成之后执行。

defer 是按照加载顺序执行，async 是按照那个文件先记载完成哪个先运行。

## iframe 的优缺点

优点：

- 安全沙盒

缺点:

- 阻塞主页面的 onload 事件
- 即使页面为空，加载也需要时间
- 没有语义

## cookie sessionStorage localStorage 区别

cookie 数据始终在同源的 http 请求中携带，即 cookie 会在浏览器和服务端之间来回传递。
sessionStorage 和 localStorage 不会自动把数据发给服务器，仅在本地保存。

cookie 数据还有路径（path）的概念，服务器可以针对不同的路径发送不同的 cookie。
sessionStorage 和 localStorage 只能保存字符串，想要保存对象需要先将其转换为 JSON 字符串，再保存。

cookie 存储比较小，一般在 4k 左右。而 sessionStorage 和 localStorage 存储比较大。

cookie 数据还有失效期，过期之前一直有效，而 localStorage 数据没有失效时间，除非主动删除，sessionStorage 数据在窗口关闭时自动删除。

## 如何实现可过期的 localStorage 数据

在存储数据的时候增加一个失效时间，每次读取的时候对比时间。

## token 能放在 cookie 中吗

可以，但是有跨域限制。也可以存在 localStorage/sessionStorage 里面。

## 浏览器如何渲染页面的

1. 浏览器从服务器获取 HTML 文档构建成文件对象模型 DOM。
2. 将样式载入和解析，构建成层叠样式表模式 CSSOM
3. 在 DOM 和 CSSOM 的基础上，构建渲染树。

## 重绘、重排区别如何避免

重绘：当页面样式但是不影响文档流中的位置时，例如修改颜色、背景色等。浏览器只会将样式赋予元素进行重绘操作。

回流：改变文档内容、结构或者元素位置时，回流操作就会被触发，一般有以下几种情况：

- 添加或者删除元素
- 内容变化，包括表单区域内的文本修改
- CSS 属性的更改和重新计算
- 伪类元素的激活（:houver 等）

## 懒加载

当页面需要加载一个资源时，会等到用户触发滚动事件时，才会加载资源。为什么要用懒加载？页面图片或者外部资源较多，比如商城。如果一次性加载资源很多，就会导致页面加载速度比较慢。

优点：页面加载速度快，减轻服务器压力，节约流量。

实现原理：页面的 img 元素，如果没有 src 属性。浏览器就不会去发送请求下载资源。

实现步骤：

1. 不将图片地址放在 img 标签的 src 中，而是记录在其他属性中
2. 监听滚动事件，当滚动到该图片附近时，将图片的地址赋值给 src 属性，实现加载。

```js
<!DOCTYPE html>
<html lang="en">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>懒加载示例</title>
</head>
<body>
  <div id="list-container"></div>
  <script>
    class LazyLoad {
        constructor(container, itemCls) {
          this.container = container;
          this.itemCls = itemCls;
          this.items = [];
        }

        // 添加需要懒加载的项
        addItem(url) {
          this.items.push(url);
        }

        // 加载指定索引的项
        loadItem(index) {
          const item = document.createElement(this.itemCls);
          item.innerHTML = `<img src="${this.items[index]}">`;
          this.container.appendChild(item);
        }

        // 加载所有项
        loadAll() {
          this.items.forEach((url, index) => {
            this.loadItem(index);
          });
        }
      }

    const listContainer = document.getElementById('list-container');
    const lazyLoad = new LazyLoad(listContainer, 'li');

    lazyLoad.addItem('https://example.com/image1.jpg');
    lazyLoad.addItem('https://example.com/image2.jpg');


    let isLoading = false;

    function loadItem() {
      if (isLoading) return;
      isLoading = true;

      const scrollTop = window.scrollY;
      const listHeight = listContainer.offsetHeight;
      const pageHeight = window.innerHeight;

      if (scrollTop + pageHeight > listHeight - window.innerHeight) {
        const index = lazyLoad.items.length - 1;
        lazyLoad.loadItem(index);
        lazyLoad.items.splice(index, 1);
      }

      isLoading = false;
    }

    window.addEventListener('scroll', loadItem);
  </script>
</body>
</html>
```

## 预加载

提前加载图片，当用户需要查看时可直接从本地缓存中渲染。为什么要用预加载？图片预先加载到浏览器中，保证了图片快速的无缝的查看，给用户更好的体验。

实现方法：

1. CSS 和 javascript 实现预加载
2. 使用请求实现预加载

## JS 数据类型有哪些

基本类型:

null：变量值为空值，
undefined：表示变量未赋值
boolean：true、 false
number：数字，整数或者浮点数
string：字符串
symbol：es6 新增。一种实例唯一且不可改变的
bigint：es2020 新增。一种任意精度的数字

引用类型：
object：对象。包括 array、function、map 等

## null 和 undefined

null 表示一个对象被定义了，但是为空；typeof null === 'object'
undefined 表示对象未被赋值

## Javascript 有几种方法判断变量的类型？

typeof 运算符，判断类型不精确；但是 undefined、 string、 number、boolean 是可以的

instanceof 运算符，通过原型链判断的（但是不能判断 null），比如：`const str = new String(); str instanceof Object === true`

Object.prototype.toString.call() 方法，可以判断对象的类型

## 判断数组的方式

Object.prototype.toString.call(value) === '[object Array]'

Array.isArray(value) === true

value instanceof Array === true

value.**proto** === Array.prototype

Array.prototype.isPrototypeOf(value) === true

## 数组去重都有哪些方法？

```js
const arr = [
  1,
  2,
  2,
  55,
  88,
  88,
  1,
  'a',
  'b',
  'b',
  null,
  null,
  NaN,
  true,
  true,
  NaN,
  undefined,
  {},
  {},
  { a: 1 },
  ,
  { a: 1 },
];
```

1. Set + Array.from

> NaN !== NaN === true 但是这里会将 NaN 去重

```js
Array.from(new Set(arr));
// [1, 2, 55, 88, 'a', 'b', null, NaN, true, undefined, {}, {}, { a: 1 }, , { a: 1 }]
```

2. 遍历的方式

```js
function delectDuplicate(arr) {
  const result = [];
  for (const item of arr) {
    // [NaN].inculudes(NaN) === true, 所以此时 NaN 也会被去重
    // 这里如果不去重 NaN，可以使用 result.indexof(item) !== -1
    if (result.includes(item)) {
      result.push(item);
    }
  }
  return result;
}
```

## map 和 forEach 的区别

map 会返回一个全新的数组

## es6 中箭头函数

箭头函数不绑定 this，如果外层有普通函数的话，this 指向的是外层函数的 this，如果没有则指向全局变量。

```js
const obj = {
  fn: function () {
    console.log('fn', this);
    return () => console.log('fn => ', this);
  },
  fn1: () => console.log('fn1', this),
};
obj.fn()(); // obj
obj.fn1(); // window
```

箭头函数没有原型，所以不能使用 new 命令，否则会抛出错误； call，apply，bind 也不能改变 this 的指向。

箭头函数自身没有 arguments，可以使用拓展符接受参数

## 闭包的理解

闭包是指有权访问另一个函数作用域的变量的函数。

函数特性：

1. 函数嵌套函数
2. 函数内部可以引用外部的参数和变量
3. 参数和变量不会被垃圾回收机制回收

应用场景，设置私有变量的方法
不适用场景：返回闭包的函数是个非常大的函数

## JS 变量提升

函数和变量声明会被提升到函数的最顶端，赋值不会提升。

es6 的 const 和 let 不会提升，并且不能重复声明和引用（暂时性死区）。

## this 指向（普通函数、箭头函数）

每个函数的 this 都有指向。

### 普通函数

1. 默认绑定规则

```JS
console.log(this); // window


function test() {
  console.log(this);
}
// 全局作用域，独立调用函数，this 指向 window
test(); // window
```

2. 隐式绑定规则

```JS
const obj = {
  name: 'obj',
  foo: function () {
    console.log(this); // obj
    function test () {
      console.log(this); // window, 因为是独立调用
    }
    test();
  },
}
obj.foo();
```

3. 显示绑定，通过 call、apply、bind 改变 this

```JS
var name = 'window';
const obj = {
  name: 'obj',
  foo: function (parme) {
    console.log(`this 指向：${this.name}；参数：${parme}`);
  },
}

obj.foo(); // this 指向：obj

// 赋值之后会导致 obj.foo 直接调用，this 指向改变
let foo = obj.foo;
foo(); // this 指向：window

// apply 强制修改 foo 的 this 的指向
// 参数通过数组传入
foo.apply(obj, ['apply']); // this 指向：obj；参数：apply

// apply 强制修改 foo 的 this 的指向
// 参数列表的形式传入参数
foo.call(obj, 'call'); // this 指向：obj；参数：apply

// 调用指向还是 this，说明 call 和 apply 仅仅作用一次
foo(); // this 指向：window；

foo = foo.bind(obj);
foo(); // this 指向：obj
```

> 补充，手动实现 bind ([参考](https://zhuanlan.zhihu.com/p/82340026#:~:text=apply%EF%BC%8Ccall%EF%BC%8Cbind%E4%B8%89%E8%80%85%E7%9A%84%E5%8C%BA%E5%88%AB%201%20%E4%B8%89%E8%80%85%E9%83%BD%E5%8F%AF%E4%BB%A5%E6%94%B9%E5%8F%98%E5%87%BD%E6%95%B0%E7%9A%84this%E5%AF%B9%E8%B1%A1%E6%8C%87%E5%90%91%E3%80%82%202%20%E4%B8%89%E8%80%85%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%8F%82%E6%95%B0%E9%83%BD%E6%98%AFthis%E8%A6%81%E6%8C%87%E5%90%91%E7%9A%84%E5%AF%B9%E8%B1%A1%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%9C%89%E8%BF%99%E4%B8%AA%E5%8F%82%E6%95%B0%E6%88%96%E5%8F%82%E6%95%B0%E4%B8%BAundefined%E6%88%96null%EF%BC%8C%E5%88%99%E9%BB%98%E8%AE%A4%E6%8C%87%E5%90%91%E5%85%A8%E5%B1%80window%E3%80%82%203,%E4%B8%89%E8%80%85%E9%83%BD%E5%8F%AF%E4%BB%A5%E4%BC%A0%E5%8F%82%EF%BC%8C%E4%BD%86%E6%98%AFapply%E6%98%AF%E6%95%B0%E7%BB%84%EF%BC%8C%E8%80%8Ccall%E6%98%AF%E5%8F%82%E6%95%B0%E5%88%97%E8%A1%A8%EF%BC%8C%E4%B8%94apply%E5%92%8Ccall%E6%98%AF%E4%B8%80%E6%AC%A1%E6%80%A7%E4%BC%A0%E5%85%A5%E5%8F%82%E6%95%B0%EF%BC%8C%E8%80%8Cbind%E5%8F%AF%E4%BB%A5%E5%88%86%E4%B8%BA%E5%A4%9A%E6%AC%A1%E4%BC%A0%E5%85%A5%E3%80%82%204%20bind%20%E6%98%AF%E8%BF%94%E5%9B%9E%E7%BB%91%E5%AE%9Athis%E4%B9%8B%E5%90%8E%E7%9A%84%E5%87%BD%E6%95%B0%EF%BC%8C%E4%BE%BF%E4%BA%8E%E7%A8%8D%E5%90%8E%E8%B0%83%E7%94%A8%EF%BC%9Bapply%20%E3%80%81call%20%E5%88%99%E6%98%AF%E7%AB%8B%E5%8D%B3%E6%89%A7%E8%A1%8C%20%E3%80%82))：

```js
//实现bind方法
Function.prototype.bind = function (oThis) {
  if (typeof this !== 'function') {
    // closest thing possible to the ECMAScript 5
    // internal IsCallable function
    throw new TypeError(
      'Function.prototype.bind - what is trying to be bound is not callable',
    );
  }

  var aArgs = Array.prototype.slice.call(arguments, 1),
    fToBind = this,
    fNOP = function () {},
    fBound = function () {
      // this instanceof fBound === true时,说明返回的fBound被当做new的构造函数调用
      return fToBind.apply(
        this instanceof fBound ? this : oThis,
        // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
        aArgs.concat(Array.prototype.slice.call(arguments)),
      );
    };

  // 维护原型关系
  if (this.prototype) {
    // 当执行Function.prototype.bind()时, this为Function.prototype
    // this.prototype(即Function.prototype.prototype)为undefined
    fNOP.prototype = this.prototype;
  }

  // 下行的代码使fBound.prototype是fNOP的实例,因此
  // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
  fBound.prototype = new fNOP();
  return fBound;
};
var arr = [1, 11, 5, 8, 12];
var max = Math.max.bind(null, arr[0], arr[1], arr[2], arr[3]);
console.log(max(arr[4])); //12
```

4. new 绑定，构造函数的 this 指向实例

## call apply bind 的作用和区别

三者都可以改变函数的 this 对象指向。
三者第一个参数都是 this 要指向的对象，如果如果没有这个参数或参数为 undefined 或 null，则默认指向全局 window。
三者都可以传参，但是 apply 是数组，而 call 是参数列表，且 apply 和 call 是一次性传入参数，而 bind 可以分为多次传入。
bind 是返回绑定 this 之后的函数，便于稍后调用；apply 、call 则是立即执行。

## new 会发生什么

> [参考链接](https://juejin.cn/post/7062663423410044964)

1. 创建一个新的空对象
2. 将空对象的 **proto** 指向构造函数的 prototype
3. 执行构造函数, 并将新创建的空对象绑定为构造函数的 this 对象
4. 如果构造函数有返回一个对象, 则返回这个对象, 否则返回新创建的那个对象

### 模拟实现

```js
function Custom(Fn) {
  // 创建一个空对象
  var obj = {};
  // 将空对象的 __proto__ 执行构造函数的 prototype
  obj.__proto__ = Fn.prototype;
  // 执行构造函数，并且将新创建的空对象绑定为构造函数的 this 指向
  var args = [].slice.call(arguments);
  args.shift();
  var res = Fn.apply(obj, args);
  // 如果构造函数有一个返回对象，则返回这个对象，否则返回创建的对象
  return typeof res === 'object' ? res : obj;
}

// 定义构造函数
function Person(name) {
  console.log('constructor');
  // 将构造函数的this指向新对象
  this.name = name;
}

// 定义类的属性
Person.prototype.say = function () {
  console.log('My name is', this.name);
};

// // 创建新对象
var p1 = Custom(Person, 'wss');
p1.say();
```
