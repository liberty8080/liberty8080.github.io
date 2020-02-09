---
title: 使用gunicorn部署flask项目
date: 2020-02-09 20:05:10
tags:
- python
- linux
---

# 使用gunicorn部署flask项目

1. 安装gunicorn

2. 假设flask的app.p

   ```python
   from flask import Flask
   app = Flask(__name__)
   
   @app.route("/")
   def hello():
       return "Hello World!"
   ```

3. 在shell中输入

   ```python
   gunicorn -w 3 -b 127.0.0.1:5000 app:app
           #第一个为flask项目实例所在的包,第二个为app生成的flask项目
   ```

   

