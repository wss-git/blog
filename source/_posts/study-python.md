---
title: 学习python
categories:
  - 练习
excerpt: '好好看，好好学'
description: ''
date: 2023-12-20 10:54:28
---

## Day 1

### print

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

print('hello world')

print('''我
  是换行
  符号''')

# Boolean
print(True or False)
print(True and False)
print(not 1 > 2)

# 变量
a = 'ABC'
b = a
a = 'XYZ'
print(b)

# 字符串拼接
print('亲爱的%s你好！你%d月的话费是%f，余额是%f' % ('张三', 12, 10.23, 100.00))
print(f'亲爱的{"张三"}你好！你{12}月的话费是{10.23}，余额是{100.00}')
```

### list

```python
list1 = ['1', '2', '3']
print(f"第一个数据是{list1[0]};最后一个元素是{list1[-1]};长度是{len(list1)}")
# 追加
list1.append('4')
# 插入: 在1后面插入 0.5
list1.insert(1, '0.5')
print(f"转换后：{list1}")
# 删除最后一个数据
list1.pop()
# 删除第二个数据（下标0开始，所以1是第二个数据）
list1.pop(1)
print(f"转换后：{list1}")
```

### if elif

小明身高1.75，体重80.5kg。请根据BMI公式（体重除以身高的平方）帮小明计算他的BMI指数，并根据BMI指数：

低于18.5：过轻
18.5-25：正常
25-28：过重
28-32：肥胖
高于32：严重肥胖
用if-elif判断并打印结果

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

height = 1.75
weight = 80.5

bmi = weight / (height * height)

if bmi < 18.5:
  print('低于18.5：过轻')
elif 18.5 <= bmi < 25:
  print('18.5-25：正常')
elif 25 <= bmi < 28:
  print('25-28：过重')
elif 28 <= bmi < 32:
  print('28-32：肥胖')
else:
  print('高于32：严重肥胖')

```

### match case

```python
args = ['abc', 'name1', 'name2', 'name3'] # args = ['abc']
match args:
  case ['abc']:
    print('仅匹配 abc')
  case ['abc', n, *n2]:
    # 将 name1 赋值给 n，  ['name2', 'name3'] 赋值给 n2
    print('这是匹配了什么 n: %s; n2: %s' % (n, n2))
  case _:
    print('没有匹配')
```

### 循环

计算1-100的整数之和

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

sum = 0
for i in range(1, 101):
  sum += i
print(f'1-100整数之和: {sum}')
```

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
sum = 0
n = 100
while n > 0:
  sum += n
  n = n -1
print(f'1-100整数之和: {sum}')
```

请利用循环依次对list中的每个名字打印出Hello, xxx!

```python
L = ['Bart', 'Lisa', 'Adam']
for name in L:
  print(f"Hello, {name}!")
```
continue 和 break

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

n = 0
while n <= 100:
  n = n + 1
  if 10 < n < 15:
    print(f"当n（{n}）大于10小于15，即 11、12、13、14时，条件满足，执行 continue")
    continue
  if n > 20:
    print(f"当({n})n=21时，条件满足，执行break语句")
    break # break语句会结束当前循环
  print(f"运行到了下面n:{n}")

print('END')
```

### 字典

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

obj = {
  'name': 'John',
  'age': 30,
  'city': 'New York'
}

# 获取存在的值
print('name' in obj, obj['name'], obj.get('name'))
# 删除
obj.pop('name')
print('name' in obj)
# 获取不存在的值
print('name1' in obj, obj.get('name1', 234))
```

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# 无序和无重复元素的集合
s = set([1, 2, 3, 4, 5, 6, 5, 4, 3, 2, 1])

print(s)

s.add(1)
s.add(7)
s.remove(4)
print(s)
```

### 函数

> [学习链接](https://www.liaoxuefeng.com/wiki/1016959663602400/1017261630425888)

以下函数允许计算两个数的乘积，请稍加改造，变成可接收一个或多个数并计算乘积：

```python
def mul(x, y):
  return x * y

# 测试
print('mul(5) =', mul(5))
print('mul(5, 6) =', mul(5, 6))
print('mul(5, 6, 7) =', mul(5, 6, 7))
print('mul(5, 6, 7, 9) =', mul(5, 6, 7, 9))
if mul(5) != 5:
    print('测试失败!')
elif mul(5, 6) != 30:
    print('测试失败!')
elif mul(5, 6, 7) != 210:
    print('测试失败!')
elif mul(5, 6, 7, 9) != 1890:
    print('测试失败!')
else:
    try:
        mul()
        print('测试失败!')
    except TypeError:
        print('测试成功!')
```

解题:
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def mul(x, *y):
  sum = x
  for i in y:
    sum *= i
  return sum
```

## Day2

### 切片

利用切片操作，实现一个trim()函数，去除字符串首尾的空格，注意不要调用str的strip()方法：

```python
def trim(s):
  if s == "":
    return s
  while s[:1] == ' ':
    s = s[1:]
  while s[-1:] == ' ':
    s = s[:-1]
  return s

# 测试:
if trim('hello  ') != 'hello':
  print('测试失败!')
elif trim('  hello') != 'hello':
  print('测试失败!')
elif trim('  hello  ') != 'hello':
  print('测试失败!')
elif trim('  hello  world  ') != 'hello  world':
  print('测试失败!')
elif trim('') != '':
  print('测试失败!')
elif trim('    ') != '':
  print('测试失败!')
else:
  print('测试成功!')
```

### 迭代

迭代对象
```python
obj = {
  "key": "value",
  "k1": "v1",
  "k2": "v2",
}

for key in obj:
  print(f"key: {key}")
for value in obj.values():
  print(f"value: {value}")
for key, value in obj.items():
  print(f"key: {key}, value: {value}")
```

迭代数组
```python
arr = ['a', 'b', 'c']

for value in arr:
  print(f"value: {value}")
for index, value in enumerate(arr):
  print(f"value: {value}, index: {index}")
```

练习：请使用迭代查找一个list中最小和最大值，并返回一个tuple

```python
def findMinAndMax(L):
  if L == None or L == []:
    return (None, None)
  min = L[0]
  max = L[0]
  for num in L:
    if num < min:
      min = num
    if num > max:
      max = num
  return (min, max)

# 测试
if findMinAndMax([]) != (None, None):
  print('测试失败!')
elif findMinAndMax([7]) != (7, 7):
  print('测试失败!')
elif findMinAndMax([7, 1]) != (1, 7):
  print('测试失败!')
elif findMinAndMax([7, 1, 3, 9, 5]) != (1, 9):
  print('测试失败!')
else:
  print('测试成功!')
```

### 列表生成器

请修改L1，通过添加if语句保证列表生成式能正确地执行

```python
L1 = ['Hello', 'World', 18, 'Apple', None]

L2 = [s.lower() for s in L1 if isinstance(s, str)]

# 测试:
print(L2)
if L2 == ['hello', 'world', 'apple']:
  print('测试通过!')
else:
  print('测试失败!')
```


## Day3
