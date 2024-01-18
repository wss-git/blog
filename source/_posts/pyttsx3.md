---
title: 语音转文字:pyttsx3
categories:
  - 轮子应用
excerpt: ''
description: ''
date: 2024-01-18 16:37:10
---


## 安装依赖

> pip install pyttsx3 -i https://pypi.tuna.tsinghua.edu.cn/simple

> mac 下需要安装pyobjc, 否则报错 name 'objc' is not defined. Did you mean: 'object'?。 8.+ 版本安装不成功，10.+ 版本安装成功依旧报错
> pip install pyobjc==9.0.1 -i https://pypi.tuna.tsinghua.edu.cn/simple 


## 编写代码

- 简单模式

```python
import pyttsx3

engine = pyttsx3.init() #pytttsx初始化配置

# 保存
# engine.save_to_file('Hello World' , 'test.mp3')

# 把文本存入
engine.say('我是谁？') 
engine.say('我能为你做什么？')
# 发出语音，并在发出语音时阻塞程序
engine.runAndWait()
```

- 修改语速和音量

```python
import pyttsx3

engine = pyttsx3.init() #pytttsx初始化配置

#调节语速，范围一般在0~500之间
print(f'获取当前语速: {engine.getProperty("rate")}')
rate = 160
engine.setProperty('rate', 160) #设置新属性值
engine.say(f'调节语速为{rate}') 
engine.runAndWait()


#调节声量，范围在0~1之间
print(f'获取当前声量: {engine.getProperty("volume")}')
volume = 0.5
engine.setProperty('volume', volume)
engine.say('调节声量 为 0.5') 

engine.runAndWait()
```

- 修改音色

```python
import random
import pyttsx3

engine = pyttsx3.init() #pytttsx初始化配置

# 获取所有可用的音色
  # mac pyttsx3 运行 getProperty('voices') 异常 KeyError: 'VoiceAge'
  # The - quite rough - suggested solution I found online was to just edit the nsss.py file of pyttsx3 and remove the attr['VoiceAge'] part. So it can be solved by editing your /pyttsx3/drivers/nsss.py at the 64 line and remove the attr['VoiceAge'] at this location
voices = engine.getProperty('voices')

# 打印所有音色的信息
for voice in voices:
  print("Voice:")
  print(" - ID: %s" % voice.id)
  print(" - Name: %s" % voice.name)
  print(" - Languages: %s" % voice.languages)
  print(" - Gender: %s" % voice.gender)
  print(" - Age: %s" % voice.age)

# 随机选择一个音色
random_number = random.randint(0, len(voices) - 1)
# 设置选定的音色
engine.setProperty('voice', voices[random_number].id)
```

