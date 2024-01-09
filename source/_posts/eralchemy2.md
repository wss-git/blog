---
title: 入坑eralchemy2
categories:
  - 项目问题积累
excerpt: 'mac 安装依赖eralchemy2遇到的问题'
description: ''
date: 2024-01-09 10:22:06
---

> mac 环境

## 简介

eralchemy2 从数据库或 SQLAlchemy 模型生成实体关系(ER)图。

## 安装

```bash
pip install eralchemy2
```

### 遇到错误

```log
pygraphviz/graphviz_wrap.c:3020:10: fatal error: 'graphviz/cgraph.h' file not found
      #include "graphviz/cgraph.h"
               ^~~~~~~~~~~~~~~~~~~
      1 error generated.
      error: command '/usr/bin/clang' failed with exit code 1
      [end of output]
  note: This error originates from a subprocess, and is likely not a problem with pip.
  ERROR: Failed building wheel for pygraphviz
Failed to build pygraphviz
ERROR: Could not build wheels for pygraphviz, which is required to install pyproject.toml-based projects
```

### 问题解决

使用 pip 和 venv 尝试了网络上大部分的教程，均失败告终。最后使用 conda 安装成功。（神奇的 python 包版本管理工具）

1. 初始 conda 环境，<a href="/笔记/conda" target="_blank">参考文档</a>

2. 安装命令

```bash
conda install --channel conda-forge eralchemy2
```

> 可能还需要安装 graphviz: `brew install graphviz`


## 使用

```python
from eralchemy2 import render_er

## Draw from database
render_er("postgresql+psycopg2://username:password@hostname:5432/databasename", 'er.png')
```

> 注意：postgresql+psycopg 可能会报错以下错误，修改为 postgresql+psycopg2 即可

```log
Traceback (most recent call last):
  File "/baseDir/lib/python3.12/site-packages/eralchemy2/main.py", line 298, in render_er
    tables, relationships = all_to_intermediary(input, schema=schema)
                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/baseDir/lib/python3.12/site-packages/eralchemy2/main.py", line 191, in all_to_intermediary
    return database_to_intermediary(filename_or_input, schema=schema)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/baseDir/lib/python3.12/site-packages/eralchemy2/sqla.py", line 102, in database_to_intermediary
    engine = create_engine(database_uri)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "<string>", line 2, in create_engine
  File "/baseDir/lib/python3.12/site-packages/sqlalchemy/util/deprecations.py", line 281, in warned
    return fn(*args, **kwargs)  # type: ignore[no-any-return]
           ^^^^^^^^^^^^^^^^^^^
  File "/baseDir/lib/python3.12/site-packages/sqlalchemy/engine/create.py", line 601, in create_engine
    dbapi = dbapi_meth(**dbapi_args)
            ^^^^^^^^^^^^^^^^^^^^^^^^
  File "/baseDir/lib/python3.12/site-packages/sqlalchemy/dialects/postgresql/psycopg.py", line 378, in import_dbapi
    import psycopg
ModuleNotFoundError: No module named 'psycopg'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/baseDir/pip/test.py", line 5, in <module>
    render_er("postgresql+psycopg://username:password@hostname:5432/databasename", 'erd.png')
  File "/baseDir/lib/python3.12/site-packages/eralchemy2/main.py", line 310, in render_er
    module_name = e.message.split()[-1]
                  ^^^^^^^^^
AttributeError: 'ModuleNotFoundError' object has no attribute 'message'
```