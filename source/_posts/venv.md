---
title: 使用 venv 管理包
categories:
  - 笔记
excerpt: '使用 venv 管理包'
description: '使用 venv 管理包'
date: 2023-12-26 11:45:54
---

1. window 下如果安装依赖异常,可以尝试使用 venv 或者 conda 管理. venv 为例:  
2. python -m venv venv  
3. .\venv\Scripts\activate  
4. pip install -r requirements.txt  
5. python -u app\main.py  
6. deactivate # 退出虚拟环境