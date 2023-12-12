---
title: Joi
categories:
  - 轮子应用
excerpt: 'Joi 是 Node.js 中一款流行的数据验证库，它可以帮助开发者轻松地验证和规范数据格式，确保应用程序的数据输入不会出现问题。'
description: ''
date: 2023-12-12 09:30:01
---

## 基本概念

在使用 Joi 进行数据验证前，我们需要了解以下几个基本概念：

1. Schema

Schema 是 Joi 的核心概念，它定义了数据的结构和规则。我们可以通过一系列的 Joi 方法来创建一个 Schema，比如 string()、number()、object() 等。

2. Validation

Validation 是指对给定数据进行验证的过程，它基于 Schema 定义的规则，检查数据是否符合要求。

3. ValidationError

ValidationError 是指在验证数据时发现的错误，它会包含详细的错误信息，比如错误原因、位置等。我们可以通过捕获 ValidationError 来处理数据验证失败的情况。

## 常用方法

Joi 提供了很多常用的方法，用于创建和验证 Schema。下面是一些常用的方法：

- 验证字符串类型： string()

string() 方法用于创建一个字符串类型的 Schema。

示例代码：

```js
const schema = Joi.string().min(3).max(30).required();
const result = schema.validate('hello');
// result: { error: null, value: 'hello' }
```

- 验证数字类型：number()

number() 方法用于创建一个数字类型的 Schema。

示例代码：

```js
const schema = Joi.number().integer().min(1).max(100).required();
const result = schema.validate(50);
// result: { error: null, value: 50 }
```

- 验证布尔类型：boolean()

boolean() 方法用于创建一个布尔类型的 Schema。

示例代码：

```js
const schema = Joi.boolean().required();
const result = schema.validate(true);
// result: { error: null, value: true }
```

- 验证对象类型：object()

object() 方法用于创建一个对象类型的 Schema。

示例代码：

```js
const schema = Joi.object({
  name: Joi.string().required(),
  age: Joi.number().integer().min(18).max(100).required(),
});
const result = schema.validate({ name: 'John', age: 25 });
// result: { error: null, value: { name: 'John', age: 25 } }
```

- 验证数组类型：array()

array() 方法用于创建一个数组类型的 Schema。

示例代码：

```js
const schema = Joi.array().items(Joi.string()).required();
const result = schema.validate(['hello', 'world']);
// result: { error: null, value: ['hello', 'world'] }
```

- 验证日期类型：date()

```js
const schema = Joi.date().required();
const result = schema.validate(new Date());
// result: { error: null, value: <current date> }
```

- required()

required() 方法用于指定一个字段是必须的。

示例代码：

```js
const Joi = require('joi');

const schema = Joi.object({
  name: Joi.string().required(),
  age: Joi.number().required(),
});
```

上面的代码中，name 和 age 字段都是必须的，如果数据不含这些字段或这些字段的值为 null 或 undefined，数据验证将失败。

- optional()

optional() 方法用于指定一个字段是可选的。

示例代码：

```js
const Joi = require('joi');

const schema = Joi.object({
  name: Joi.string().optional(),
  age: Joi.number().optional(),
});
```

上面的代码中，name 和 age 字段都是可选的，如果数据不含这些字段或这些字段的值为 null 或 undefined，数据验证将通过。

- allow()

allow() 方法用于指定一个字段可以接受的值。

示例代码：

```js
const Joi = require('joi');

const schema = Joi.object({
  role: Joi.string().valid('admin', 'user', 'guest'),
});
```

上面的代码中，role 字段只能包含 admin、user、guest 中的一个值，如果数据中的 role 字段不是这三个值之一，数据验证将失败。

- email()

email() 方法用于验证电子邮件地址的格式是否正确。

示例代码：

```js
const Joi = require('joi');

const schema = Joi.object({
  email: Joi.string().email(),
});
```

上面的代码中，email 字段必须是一个有效的电子邮件地址，否则数据验证将失败。

## 示例代码

接下来，我们将演示一些实际的示例代码，帮助您更好地理解和使用 Joi。

1. 验证字符串长度

示例代码：

```js
const Joi = require('joi');

const schema = Joi.object({
  name: Joi.string().min(3).max(30),
});

const data = {
  name: 'John',
};

const result = schema.validate(data);

if (result.error) {
  console.log(result.error.details[0].message);
} else {
  console.log('Data is valid');
}
```

上面的代码中，我们定义了一个字符串类型的 Schema，规定了 name 字段的最小长度为 3，最大长度为 30。

2. 验证对象选填

```js
const Joi = require('joi');

const Schema = Joi.object({
  key: Joi.string().required(),
  config: Joi.object({
    bucket: Joi.string().required(),
    internal: Joi.boolean(),
  }).required(),
  credentials: Joi.object({
    key: Joi.string().required(),
  }),
}).unknown();

const result = schema.validate({
  key: 'key',
  config: {
    bucket: 'bucket',
    internal: false,
  },
});

if (result.error) {
  console.log(result.error.details[0].message);
} else {
  console.log('Data is valid');
}
```

上面的代码中，我们定义了一个对象类型的 Schema，规定了:
key 字段为字符串类型必填
config 字段为对象类型必填，并且 config.bucket 为字符串类型必填，config.internal 为布尔类型选填
credentials 字段为对象类型选填，如果填写 credentials.key 字段必填。
然后这个 Schema 允许存在未知键

> 补充：
> joi.validate 的第二个参数是一个可选的对象，用于设置一些额外的验证选项。这个参数包含以下选项：
>
> abortEarly：一个布尔值，表示当遇到第一个错误时是否停止验证。如果设置为 true，则只返回第一个错误；如果设置为 false，则返回所有错误。默认为 true。
> convert：一个布尔值，表示是否尝试将输入值转换为相应的类型。例如，将字符串转换为数字。默认为 true。
> allowUnknown：一个布尔值，表示是否允许未知的键。如果设置为 true，则不会验证未知的键；如果设置为 false，则会返回一个错误。默认为 false。
> stripUnknown：一个布尔值，表示是否从验证后的对象中删除未知的键。如果设置为 true，则未知键将被删除；如果设置为 false，则未知键将保留在对象中。默认为 false。
> skipFunctions：一个布尔值，表示是否跳过函数类型的键。如果设置为 true，则函数类型的键将被跳过；如果设置为 false，则会返回一个错误。默认为 false。

