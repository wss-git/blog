---
title: 微信傀儡
categories:
  - 轮子应用
excerpt: '微信机器人（ wechaty ）使用'
description: '微信机器人（ wechaty ）使用'
date: 2024-04-05 10:57:47
---

## 地址文档

- [wechaty-1](https://wechaty.js.org/zh/docs)
- [Wechaty-2](https://wechaty.gitbook.io/wechaty/v/zh/example)
- [wechaty-puppet-padlocal](https://github.com/wechaty/puppet-padlocal/wiki/API-%E4%BD%BF%E7%94%A8%E6%96%87%E6%A1%A3-(TypeScript-JavaScript))

## 如何在没有 onMessage 的情况下发送消息？

### 向一个房间发送消息

```js
const room = await wechaty.Room.find('room name')
room.say('hello room')
```

### 向一个联系人发送消息

```js
const contact = await wechaty.Contact.find('contact name')
contact.say('hello room')
```

## onScan 的状态有哪些

```js
function getStatus(status) {
  const ScanStatus = require('wechaty').ScanStatus;
  const statusMap = {
    [ScanStatus.Waiting]: '等待扫描',
    [ScanStatus.Timeout]: '已过期，重新扫描',
    [ScanStatus.Scanned]: '扫描完成，请手机确认登录',
    [ScanStatus.Confirmed]: '已确认登录',
  }
  return statusMap[status] || '未知状态'
}
```

## wechaty-puppet-padlocal token 如何获取

[地址](http://pad-local.com/#/tokens)
> PS: 月租 ¥200


## 如何将文件存到本地

```js
const file = await msg.toFileBox()
const name = file.name
console.log('Save file to: ' + name)
file.toFile(name)
```


## 如何设置微信关联唯一

测试：
1. 获取联系人的ID `contact.id`。这个值是包生成并非真正的ID，下次登录时值会改变。
2. 设置别名 `contact.alias('value')`（有些包是`remark`）, 并不会真正的修改，类似ID的机制

可行性：
经测试，如果是手动添加好友之后，手动设置一个备注。


## 如何区分联系人和公众号的消息

```ts
contact.type()

enum ContactType {
  Unknown = 0,
  Individual = 1,
  Official = 2,
  Corporation = 3,
}
```

## 如何获取群二维码

```js
const qrString = await room.qrCode();
// 用二维码生成工具，将 qrString 生成为二维码即可
```