---
title: uni-app实践
categories:
  - 练习
excerpt: '记录开发uni-app中遇到的问题'
description: '记录开发uni-app中遇到的问题'
date: 2024-10-25 10:09:39
---


## 使 view(div) 能够获取焦点

### 源码
```vue
<template>
  <text class="test-v">11111111111</text>
  <text class="test-v" tabindex="0">0000000000000000000000</text>
</template>
<style>
.test-v:focus {
  border: 5px solid red;
}
</style>
```

### 知识点

1. 
```text
An element can have focus if the tabIndex property is set to any valid negative or positive integer.
Elements that receive focus can fire the onblur and onfocus events as of Internet Explorer 4.0, and the onkeydown, onkeypress, and onkeyup events as of Internet Explorer 5.
只要元素的tabIndex属性设置成任何有效的整数那么该元素就能取得焦点。元素在取得焦点后就能触发onblur，onfocus，onkeydown, onkeypress和onkeyup事件。

不同tabIndex值在tab order（Tabbing navigation）中的情况：
Objects with a positive tabIndex are selected in increasing iIndex order and in source order to resolve duplicates.
Objects with an tabIndex of zero are selected in source order. 
Objects with a negative tabIndex are omitted from the tabbing order.
tabIndex值是正数的对象根据递增的值顺序和代码中的位置顺序来被选择
tabIndex值是0的对象根据在代码中的位置顺序被选择
tabIndex值是负数的对象会被忽略
```

2. 采用vue时， @blur事件可能无效时，采用 @blur.native.capture，获取焦点同理。




## uni-app 拖拽

### 源码
```vue
<template>
  <movable-area class="w-300 h-300" class="panel">
    <image :src="curDigitalPerson.cover_url" mode="scaleToFill" />
    <movable-view direction="all" scale>
      <image class="w-30 h-30" src="/static/images/icon/tip.png" mode="scaleToFill" tabindex="0" />
    </movable-view>
    <movable-view direction="all" out-of-bounds><text class="test-v"
        tabindex="0">0000000000000000000000</text></movable-view>
    <movable-view direction="all"><text class="test-v" tabindex="0">111111111111111111</text></movable-view>
  </movable-area>
</template>
<style>
.test-v:focus {
  border: 5px solid red;
}
</style>
```

### 知识点

1. movable-area 的宽高需要手动指定，默认为 10px
2. movable-view的移动方向 direction 需要手动设置，默认为 None。属性值有all、vertical、horizontal、none



## 实现某一区域高亮,并存在圆角

> 类似 antd Tour 的 gap.round 属性

### 源码

```vue
<template>
<view @click.stop="" class="fixed" :style="{ ...middle, boxShadow: '0 0 0 100vmax rgba(0, 0, 0, 0.8)' }">
</template>
<script setup lang="ts">
const middle = ref({})
onMounted(() => {
  uni.$on("guide:visible", ({ instance, id }) => {
    const query = uni.createSelectorQuery().in(instance.proxy);
    query
      .select(`#${id}`)
      .fields({
        computedStyle: ['borderRadius'],
        rect: true,
        size: true,
      }, (data: any) => {
        console.log("得到布局位置信息", data);

        middle.value.left = data.left + 'px'
        middle.value.top = data.top + 'px'
        middle.value.width = data.width + 'px'
        middle.value.height = data.height + 'px'
        middle.value.borderRadius = data.borderRadius
      })
      .exec();
  })
})
</script>
```

### 知识点

1. vmax：vmax表示浏览器窗口的可视区域的最大尺寸。如果浏览器窗口的宽度是 600px，高度是 800px，那么 1vmax 等于 80px（因为高度的 1% 是 8px，宽度的 1% 是 6px，800px 的 1% 大于 600px 的 1%，所以 1vmax 等于视口高度的 1%，即 80px）。

`boxShadow: '0 0 0 100vmax rgba(0, 0, 0, 0.8)'` : 这里的100vmax是作为spread（阴影扩展半径）的值。并且由于颜色是rgba(0, 0, 0, 0.8)（半透明黑色），所以可以产生一种类似于模态框背后的深色遮罩的效果。

2. 用于获取节点信息的对象: nodesRef.fields(object,callback), 可以通过第一个参数的 computedStyle 属性来获取节点的样式信息，包括背景颜色、边框颜色、字体大小等。