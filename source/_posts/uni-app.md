---
title: uni-app
categories:
  - 练习
excerpt: '记录从零学习uni-app'
description: '记录从零学习uni-app'
date: 2024-07-18 15:53:20
---

> ⚠️ 陈旧文档废弃

# uni-app

## 学习背景

一直是写react PC 项目，忽然接到一个uni-app项目，就顺手学习一下。

本人技术栈：Nodejs、React、Python
常用的编辑器: VSCode

## 初始化项目

按照官网的教程初始化项目。

1. 下载 HBuildx （学习技术还要学一下新的编辑器，😂）
2. 新建项目：文件 -> 新建 -> 项目 -> uni-app (抛出问题：uni-app 和 uni-app x 区别在哪里?)
3. 运行项目：运行 -> 运行到浏览器 -> 选择自己的浏览器 (本人使用谷歌浏览器，所以选择谷歌浏览器，在浏览器输入 chrome://version/ 查看安装路径)

## 学习笔记

### 目录结构

> 文档链接: https://uniapp.dcloud.net.cn/tutorial/project.html

```
┌─uniCloud              云空间目录，支付宝小程序云为uniCloud-alipay，阿里云为uniCloud-aliyun，腾讯云为uniCloud-tcb（详见[uniCloud](https://doc.dcloud.net.cn/uniCloud/quickstart.html#structure)）
│─components            符合vue组件规范的uni-app组件目录
│  └─comp-a.vue         可复用的a组件
├─utssdk                存放uts文件
├─pages                 业务页面文件存放的目录
│  ├─index
│  │  └─index.vue       index页面
│  └─list
│     └─list.vue        list页面
├─static                存放应用引用的本地静态资源（如图片、视频等）的目录，注意：静态资源都应存放于此目录
├─uni_modules           存放uni_module
├─platforms             存放各平台专用页面的目录
├─nativeplugins         App原生语言插件
├─nativeResources       App端原生资源目录
│  ├─android            Android原生资源目录
|  └─ios                iOS原生资源目录
├─hybrid                App端存放本地html文件的目录，详见
├─wxcomponents          存放小程序组件的目录，详见
├─unpackage             非工程代码，一般存放运行或发行的编译结果
├─main.js               Vue初始化入口文件
├─App.vue               应用配置，用来配置App全局样式以及监听 应用生命周期
├─pages.json            配置页面路由、导航条、选项卡等页面类信息，详见
├─manifest.json         配置应用名称、appid、logo、版本等打包信息，详见
├─AndroidManifest.xml   Android原生应用清单文件
├─Info.plist            iOS原生应用配置文件
└─uni.scss              内置的常用样式变量
```

### 页面

> 文档链接: https://uniapp.dcloud.net.cn/tutorial/page.html

#### 页面管理

- uni-app中的页面，保存在工程根目录下的pages目录下。
- 每次操作页面，均需在 pages.json 中修改配置pages列表；未在pages.json -> pages 中注册的页面，uni-app会在编译阶段进行忽略。
- [pages.json](https://uniapp.dcloud.net.cn/collocation/pages.html)

#### 页面生命周期

> 更多声明周期详见：https://uniapp.dcloud.net.cn/tutorial/page.html#vue3-lifecycle-flow

```vue
<template>
	<view class="content">
		<button @click="buttonClick">{{title}}</button>
	</view>
</template>

<script>
  const TAB_OFFSET = 1; // 外层静态变量不会跟随页面关闭而回收

	export default {
		data() {
			return {
				title: "点我", // 定义绑定在页面上的data数据
			}
		},
		onLoad(option) {
			// 所以这里不能直接操作dom（可以修改data，因为vue框架会等待dom准备后再更新界面）；
      // 在 app-uvue 中获取当前的activity拿到的是老页面的activity，只能通过页面栈获取activity。
      // onLoad比较适合的操作是：接受上页的参数，联网取数据，更新data。但onLoad里不适合进行大量同步耗时运算，因为此时转场动画还没开始。
      console.log("页面加载", option)
		},
    onShow() {
      console.log("页面显示")
    },
    onHide() {
      console.log("页面隐藏")
    },
    onReachBottom() {
      console.log("页面到底了")
    },
    onPageScroll(e) {
      console.log("页面滚动了。滚动距离为：" + e.scrollTop)
    },
    onBackPress(options) {
      // onBackPress上不可使用async，会导致无法阻止默认返回
      //  Android 实体返回键 (from = backbutton)
      //  顶部导航栏左边的返回按钮 (from = backbutton)
      //  返回 API，即 uni.navigateBack() (from = navigateBack)
      /**
      注意事项：
        只有在该函数中返回值为 true 时，才表示不执行默认的返回，自行处理此时的业务逻辑。
        当不阻止页面返回却直接调用页面路由相关接口（如：uni.switchTab）时，可能会导致页面显示异常，可以通过延迟调用路由相关接口解决。
        不返回或返回其它值，均会执行默认的返回行为。
        H5 平台，顶部导航栏返回按钮支持 onBackPress()，浏览器默认返回按键及Android手机实体返回键不支持 onBackPress()
        暂不支持直接在自定义组件中配置该函数，目前只能是在页面中来处理。
      */
      console.log('from:' + options.from)
      if (options.from === 'navigateBack') {  
        return false;  
      }  
      this.back();  
      return true;  
    },
    onTabItemTap : function(e) {
      // e的返回格式为json对象： {"index":0,"text":"首页","pagePath":"pages/index/index"}
      console.log(e);
    },

		methods: {
			buttonClick: function () {
				console.log("按钮被点了")
        this.title = "被点了"
			},
       back() {  
          uni.navigateBack({  
            delta: 2  
          });  
        }  
		}
	}
</script>

<style>
	.content {
		width: 750rpx;
		background-color: white;
	}
</style>

```

#### 页面通讯

注意：

使用时，注意及时销毁事件监听，比如，页面 onLoad 里边 uni.$on 注册监听，onUnload 里边 uni.$off 移除，或者一次性的事件，直接使用 uni.$once 监听

```js
// 发送事件
uni.$emit('update', {msg:'页面更新'})
// 接受事件
uni.$on('update',function(data){
  console.log('监听到事件来自 update ，携带参数 msg 为：' + data.msg);
})
// 事件可以由 uni.$emit 触发，但是只触发一次
uni.$once('update',function(data){
  console.log('监听到事件来自 update ，携带参数 msg 为：' + data.msg);
})
// 移除全局自定义事件监听器。
  // 如果没有提供参数，则移除所有的事件监听器；
  // 如果只提供了事件，则移除该事件所有的监听器；
  // 如果同时提供了事件与回调，则只移除这个回调的监听器；
  // 提供的回调必须跟$on的回调为同一个才能移除这个回调的监听器
uni.$off('add', this.add)
```

#### 页面路由

跳转方式有两种：使用[navigator](https://uniapp.dcloud.net.cn/component/navigator.html)组件跳转、调用[API](https://uniapp.dcloud.net.cn/api/router.html)跳转。

**调用API**
```js
// 保留当前页面，跳转到应用内的某个页面，使用uni.navigateBack可以返回到原页面。
uni.navigateTo({
  url: 'path?key=value&key2=value2',
  events: {
    // 为指定事件添加一个监听器，获取被打开页面传送到当前页面的数据
  }, // 页面间通信接口，用于监听被打开页面发送到当前页面的数据。
  // success, fail, complete
})

// 关闭当前页面，返回上一页面或多级页面。可通过 getCurrentPages() 获取当前的页面栈，决定需要返回几层。
uni.navigateBack({
  delta: 1,
  animationType: 'pop-in',
  animationDuration,
  // success, fail, complete
})

// redirectTo: 关闭当前页面，跳转到应用内的某个页面。
// reLaunch: 关闭所有页面，打开到应用内的某个页面。
// switchTab: 跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面。(路径后不能带参数)
uni.redirectTo({
  url: 'path?key=value&key2=value2',
  // success, fail, complete
})

```

#### 页面动画

| 显示动画 | 关闭动画	 | 显示动画描述（关闭动画与之相反） |
| --- | --- | --- |
| slide-in-right |	slide-out-right	| 新窗体从右侧进入 |
| slide-in-left |	slide-out-left | 新窗体从左侧进入 |
| slide-in-top |	slide-out-top | 新窗体从顶部进入 |
| slide-in-bottom |	slide-out-bottom | 新窗体从底部进入 |
| pop-in | pop-out | 新窗体从左侧进入，且老窗体被挤压而出 |
| fade-in | fade-out | 新窗体从透明到不透明逐渐显示 |
| zoom-out | zoom-in | 新窗体从小到大缩放显示 |
| zoom-fade-out | zoom-fade-in | 新窗体从小到大逐渐放大并且从透明到不透明逐渐显示 |
| none | none | 无动画 |


### 引用

**组件**
```html
<template>
	<view>
		<!-- 2.使用组件 -->
		<uni-rate text="1"></uni-rate>
	</view>
</template>
<script setup>
	// 1. 导入组件
	import uniRate from '@/components/uni-rate/uni-rate.vue';
</script>
```

**js**: js 文件不支持使用/开头的方式引入

```js
// 绝对路径，@指向项目根目录，在cli项目中@指向src目录
import add from '@/common/add.js';
// 相对路径
import add from '../../common/add.js';
```

**npm**: 不推荐，优先从 uni-app插件市场 获取插件
```bash
# 根目录
npm init -y
npm i [packageName]

# 代码引入
# import package from 'packageName'
# const package = require('packageName')
```

**css**
```css
<style>
/* 用;表示语句结束 */
@import "../../common/uni.css";

/* 绝对路径 */
@import url('/common/uni.css');
@import url('@/common/uni.css');
/* 相对路径 */
@import url('../../common/uni.css');

.uni-card {
  box-shadow: none;

  /* 绝对路径 */
  background-image: url(/static/logo.png);
  background-image: url(@/static/logo.png);
  /* 相对路径 */
  background-image: url(../../static/logo.png);
}
</style>
```

**静态资源**

```html
<!-- 绝对路径，/static指根目录下的static目录，在cli项目中/static指src目录下的static目录 -->
<image class="logo" src="/static/logo.png"></image>
<image class="logo" src="@/static/logo.png"></image>
<!-- 相对路径 -->
<image class="logo" src="../../static/logo.png"></image>
```

**非 static 目录的静态资源**
```html
<!-- /pages/index/index.vue -->
<template>
	<view class="content">
        <image :src="src" />
	</view>
</template>

<script>
// 使用 import 引入静态资源，并在 data 中赋值引用
import icon_src from './icon.png'
export default { 
  data() {
    return { 
      src: icon_src
    }
  },
}
</script>
```


### 常用语法

**v-bind**
```html
<!-- 完整语法 -->
<image v-bind:src="imgUrl"></image>
<!-- 缩写 -->
<image :src="imgUrl"></image>

<button v-bind:disabled="isButtonDisabled">Button</button>
```

**v-on**
```html
	<!-- 完整语法 -->
	<view v-on:click="doSomething">点击</view>
	<!-- 缩写 -->
	<view @click="doSomething">点击</view>
```

**v-html**
```html
<template>
  <view>
    <view v-html="rawHtml"></view>
  </view>
</template>
<script>
  export default {
    data() {
      return {
        rawHtml: '<div style="text-align:center;background-color: #007AFF;"><div >我是内容</div><img src="https://qiniu-web-assets.dcloud.net.cn/unidoc/zh/uni@2x.png"/></div>'
      }
    }
  }
</script>
```

**data**
```js
//正确用法，使用函数返回对象
data() {
  return {
    title: 'Hello'
  }
}

//错误写法，会导致再次打开页面时，显示上次数据
data: {
  title: 'Hello'
}

//错误写法，同样会导致多个组件实例对象数据相互影响
const obj = {
  title: 'Hello'
}
data() {
  return {
    obj
  }
}
```

**class**
```html
<!-- 渲染结果  <view class="static active"></view> -->
<template>
  <view>
    <view class="static" :class="{ active: isActive}">hello uni-app</view>
    <view class="static" :class="{ active: isActive, 'text-danger': hasError }">hello uni-app</view>
  </view>
</template>
<script>
  export default {
    data() {
      return {
        isActive: true,
        hasError: false
      }
    }
  }
</script>
<style>
.static{
  color: #2C405A;
}
.active{
  background-color: #007AFF;
  font-size:30px;
}
.text-danger{
  color: #DD524D;
}
</style>
```
```html
<!--  -->
<template>
  <view>
    <view class="container" :class="computedClassStr">hello uni-app</view>
    <view class="container" :class="{active: isActive}">hello uni-app</view>
  </view>
</template>
<script>
  export default {
    data() {
      return {
        isActive: true
      }
    },
    computed: {
      computedClassStr() {
        return this.isActive ? 'active' : ''
      }
    }
  }
</script>
<style>
  .active {
    background-color: #007AFF;
    font-size:30px;
  }
</style>
```
```html
<template>
  <view>
    <view :style="[baseStyles, overridingStyles]">hello uni-app</view>
  </view>
</template>
<script>
  export default {
    data() {
      return {
        baseStyles: {
          color: 'green',
          fontSize: '30px'
        },
        overridingStyles: {
          'font-weight': 'bold'
        }
      }
    }
  }
</script>
```

**v-if / v-else-if / v-else**
```html
<template>
  <view>
    <view v-if="type === 'A'">
      A
    </view>
    <view v-else-if="type === 'B'">
      B
    </view>
    <view v-else-if="type === 'C'">
      C
    </view>
    <view v-else>
      Not A/B/C
    </view>
  </view>
</template>
<script>
  export default {
    data() {
      return {
        type:'B'
      }
    }
  }
</script>
```

**v-for**
```html
<template>
  <view>
    <view v-for="(item, index) in items">
      {{ parentMessage }} - {{ index }} - {{ item.message }}
    </view>
    <view v-for="(value, name, index) in object">
      {{ index }}. {{ name }}: {{ value }}
    </view>
    <!-- v-if 的优先级比 v-for 更高，这意味着 v-if 将没有权限访问 v-for 里的变量 -->
    <view v-for="item in items" v-if="!parentMessage">
      {{ todo }}
    </view>
  </view>
</template>
<script>
  export default {
    data() {
      return {
        parentMessage: 'Parent',
        items: [
          { message: 'Foo' },
          { message: 'Bar' }
        ],
        object: {
          title: 'How to do lists in Vue',
          author: 'Jane Doe',
          publishedAt: '2021-05-10'
        }
      }
    }
  }
</script>
```

### 事件

```html
<template>
  <view>
    <button @click="counter += 1">Add 1</button>
    <text>The button above has been clicked {{ counter }} times.</text>
  </view>
  <view>
    <!-- `greet` 是在下面定义的方法名 -->
    <button @click="greet">Greet</button>
  </view>
  <view>
    <!-- 内联语句处理 -->
    <button @click="say('hi')">Say hi</button>
    <!-- 访问原始的 DOM 事件。可以用特殊变量 $event 把它传入方法 -->
    <button @click="warn('Form cannot be submitted yet.', $event)">
      Submit
    </button>
  </view>
  <!-- .stop: 各平台均支持， 使用时会阻止事件冒泡，在非 H5 端同时也会阻止事件的默认行为 -->
	<view @click.stop="greet"></view>
</template>
<script>
  export default {
    data() {
      return {
        counter: 0
      }
    },
    // 在 `methods` 对象中定义方法
    methods: {
      greet(event){
        // `event` 是原生 DOM 事件
        console.log(event);
        uni.showToast({
          title: 'Hello ' + this.counter + '!'
        });
      },
      say(message) {
        uni.showToast({
          title: message
        });
      },
      warn(message, event) {
        // 现在我们可以访问原生事件对象
        if (event) {
          //可访问 event.target等原生事件对象
          console.log("event: ",event);
        }
        uni.showToast({
          title: message
        });
      },
    }
  }
</script>
```

事件映射表，左侧为 WEB 事件，右侧为 `uni-app` 对应事件
```js
{
  click: 'tap',
  touchstart: 'touchstart',
  touchmove: 'touchmove',
  touchcancel: 'touchcancel',
  touchend: 'touchend',
  tap: 'tap',
  longtap: 'longtap', //推荐使用longpress代替
  input: 'input',
  change: 'change',
  submit: 'submit',
  blur: 'blur',
  focus: 'focus',
  reset: 'reset',
  confirm: 'confirm',
  columnchange: 'columnchange',
  linechange: 'linechange',
  error: 'error',
  scrolltoupper: 'scrolltoupper',
  scrolltolower: 'scrolltolower',
  scroll: 'scroll'
}
```

### 表单

```html
<template>
  <view>
			<input v-model="message" placeholder="edit me">
			<text>Message is: {{ message }}</text>
		</view>
  <view>
    <!-- select 标签用 picker 组件进行代替 -->
    <picker @change="bindPickerChange" :value="index" :range="array">
      <view class="picker">
        当前选择：{{array[index]}}
      </view>
    </picker>
  </view>
  <view>
    <radio-group class="radio-group" @change="radioChange">
      <label class="radio" v-for="(item, index) in items" :key="item.name">
        <radio :value="item.name" :checked="item.checked" /> {{item.value}}
      </label>
    </radio-group>
  </view>
</template>
<script>
  export default {
    data() {
      return {
        message: "",
        index: 0,
        array: ['A', 'B', 'C'],
        items: [
          {
            name: 'USA',
            value: '美国'
          },
          {
            name: 'CHN',
            value: '中国',
            checked: 'true'
          },
          {
            name: 'BRA',
            value: '巴西'
          },
          {
            name: 'JPN',
            value: '日本'
          },
          {
            name: 'ENG',
            value: '英国'
          },
          {
            name: 'TUR',
            value: '法国'
          }
        ],
      }
    },
    methods: {
      bindPickerChange(e) {
        console.log(e)
        this.index = e.detail.value
      },
      radioChange(e) {
					console.log('radio发生change事件，携带value值为：', e.target.value)
				}
    }
  }
</script>
```

### 侦听器watch

```html
<template>
  <view>
    <input type="number" v-model="a" style="border: red solid 1px;" />
    <input type="number" v-model="b" style="border: red solid 1px;" />
    <input type="number" v-model="obj.a" style="border: red solid 1px;" />
    <input type="number" v-model="obj.b" style="border: red solid 1px;" />
    <view>总和：{{sum}}</view>
    <button type="default" @click="add">求和</button>
  </view>
</template>
<script>
  export default {
    data() {
      return {
        a:1,
        b:1,
        obj: {
          a: 1,
          b: 1,
        },
        sum: ""
      }
    },
    watch: {
      a: {
        handler(newVal, oldVal) {
          console.log("a------: ", newVal, oldVal);
        },
        immediate: true//初始化绑定时就会执行handler方法
      },
      b: {
        handler(newVal, oldVal) {
          console.log("b------: ", newVal, oldVal);
        },
        immediate: true, //初始化绑定时就会执行handler方法
      },
      obj: {
        handler(newVal, oldVal) {
          console.log('obj-newVal:' + JSON.stringify(newVal), 'obj-oldVal:' + JSON.stringify(oldVal), );
        },
        deep: true//对象中任一属性值发生变化，都会触发handler方法
      },
      "obj.a": {//监听obj对象中的单个属性值的变化
				handler(newVal, oldVal) {
					console.log('obj-newVal:' + newVal, 'obj-oldVal:' + oldVal);
				}
			}
    },
    methods: {
      add() {
        this.sum = parseInt(this.a) + parseInt(this.b)
      }
    }
  }
</script>
```

### 计算属性

```html
<template>
  <view>
    <view>{{ fullName }}</view>
    <view>{{ publishedBooksMessage }}</view>
  </view>
</template>
<script>
  export default {
    data() {
      return {
        author: {
          name: 'John Doe',
          books: [
            'Vue 2 - Advanced Guide',
            'Vue 3 - Basic Guide',
            'Vue 4 - The Mystery'
          ]
        },
        firstName: 'Foo',
        lastName: 'Bar'
      }
    },
    computed: {
      fullName: {
        // 计算属性的 getter
				publishedBooksMessage() {
					// `this` points to the vm instance
					return this.author.books.length > 0 ? 'Yes' : 'No'
				},
        // getter
        get(){
          return this.firstName + ' ' + this.lastName
        },
        // setter
        set(newValue){
          var names = newValue.split(' ')
          this.firstName = names[0]
          this.lastName = names[names.length - 1]
        }
      }
    }
  }
</script>
```