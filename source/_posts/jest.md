---
title: jest
categories:
  - 轮子应用
excerpt: 'jest 简单使用'
description: ''
date: 2023-12-12 09:28:25
---


## describe 描述

会形成一个作用域

```javascript
describe('作用域', () => {
  const a = 2;
  it('it', () => {
    expect(a).toBe(2);
  });

  test('test', () => {
    const a = 3;
    expect(a).toBe(3);
  });
});
```

## 常用的钩子

- beforeAll：在所有测试用例执行之前执行
- beforeEach：每个测试用例执行前执行，可让每个测试用例中使用的变量互不影响，因为分别为每个测试用例实例化了一个对象
- afterAll：等所有测试用例都执行之后执行
- afterEach：每个测试用例执行结束之后都会执行

```javascript
describe('钩子测试', () => {
  beforeAll(() => {
    console.log('beforeAll');
  });
  beforeEach(() => {
    console.log('beforeEach');
  });
  afterAll(() => {
    console.log('afterAll');
  });
  afterEach(() => {
    console.log('afterEach');
  });
  test('测试', () => {
    console.log('测试执行');
  });
  test('测试2', () => {
    console.log('测试执行2');
  });
});
```

输出顺序：`beforeAll`、`beforeEach`、`测试执行`、`afterEach`、`beforeEach`、`测试执行2`、`afterEach`、`afterAll`

## only 只对指定测试用例进行调试

```javascript
test.only('测试', () => {
  console.log('测试执行');
});
test('测试2', () => {
  console.log('测试执行2');
});
```

仅输出 `测试执行`

## toThrow 处理异步异常

```javascript
const request = require('supertest');

test('rejects', async () => {
  await expect(async () => {
    await request('/abc/test');
  }).rejects.toThrow('Request failed with status code 404');
});
```

## 测试 express 接口（通过 supertest）

需要先将 express 程序启动，然后再运行 jest

```javascript
const request = require("supertest");
const url = 'http://0.0.0.0:9000';

test("GET /", async () => {
  await request(url).get("/").expect(200);
});

test('POST /abc', , async () => {
  const res = await request(url).post('/abc')
    .set('Cookie', [`jwt={jwt}`])
    .send({ abc: 'xxx' })
    .expect(200);
  const body = res.body; // 接口返回
});
```

## 匹配器

### toBe 精确匹配

```javascript
test('精确匹配', () => {
  expect(1).toBe(1);
});
```

### toEqual 对象相等

```javascript
test('对象相等', () => {
  expect({ a: 1, b: 2 }).toEqual({ a: 1, b: 2 });
});
```

### toMatch 正则表达式验证字符串

```javascript
test('对象不相等', () => {
  expect('77abc66').toMatch(/abc/);
});
```

### not 取反

```javascript
test('对象不相等', () => {
  expect({ a: 1, b: 1 }).not.toEqual({ a: 1, b: 2 });
});
```

### toHaveLength 判断长度

```javascript
test('length', () => {
  expect('1234').toHaveLength(4);
  expect([1, 2, 3, 4]).toHaveLength(4);
});
```

### toContain / toContainEqual 数组包含子项

```javascript
test('数组包含子项', () => {
  const strArr = ['a', 'b', 'c'];
  expect(strArr).toContain('a');
  const objArr = [{ a: 'a' }, { b: 'b' }, { c: 'c' }];
  expect(objArr).toBeFalsy({ a: 'a' });
});
```

### toHaveProperty 指定路径是否有这个属性

```javascript
test('指定路径是否有这个属性', () => {
  const obj = {
    a: 2,
    objArr: [{ a: 'a' }, { b: 'b' }, { c: 'c' }],
  };
  expect(obj).toHaveProperty('a');
  expect(obj).toHaveProperty('a', 2);
  expect(obj).toHaveProperty(['objArr', 1], { b: 'b' });
  expect(obj).toHaveProperty('objArr.0', { a: 'a' });
});
```

### toBeTruthy / toBeFalsy 布尔值

```javascript
test('布尔值', () => {
  expect(true).toBeTruthy();
  expect(false).toBeFalsy();
});
```

### toBeNull / toBeUndefined / toBeDefined

```javascript
test('空', () => {
  expect(null).toBeNull(); // 只匹配 null
  expect(undefined).toBeUndefined(); // 只匹配 undefined
  expect(null).toBeDefined(); // 与 toBeUndefined 相反
});
```
