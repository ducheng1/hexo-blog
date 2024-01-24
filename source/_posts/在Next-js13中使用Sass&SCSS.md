---
title: 在Next.js13中使用Sass/SCSS
author: DuCheng
tags:
  - react
  - nextjs
categories:
  - 前端
  - ''
date: 2023-02-14 08:49:00
---

官方文档见这一章：https://nextjs.org/docs/basic-features/built-in-css-support

1. 安装 Sass
   `npm i sass -D`
2. 在 next.config.js 中配置

```javascript
import path from 'path'

/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  sassOptions: {
    includePaths: [path.join(__dirname, 'styles')]
  }
}

module.exports = nextConfig
```

上述配置只会对 styles 文件夹下的.scss 文件进行预处理
