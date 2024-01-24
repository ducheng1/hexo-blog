---
title: 解决gin-swagger找不到doc.json问题
author: DuCheng
top: false
tags:
  - golang
  - gin
categories:
  - golang
date: 2023-02-27 16:47:00
cover:
password:
---

使用 gin-swagger 时出现找不到 doc.json 的问题，只需在 main.go 中使用`docs.SwaggerInfo.ReadDoc()`即可。

官方注释：`ReadDoc`会把`SwaggerTemplate`转换成为 swagger 文档，即把 docs 文件夹中的内容转换成 doc.json。
**注意：引入的`docs`应为工程内 swag-cmd 生成的文件夹**
