---
title: 在 Next.js13 中集成 antd 和 layout 的使用
author: DuCheng
tags:
  - react
  - nextjs
categories:
  - 前端
date: 2023-02-14 09:26:00
---

# 在 Next.js13 中集成 antd 和 layout 的使用

nextjs 官方文档：https://nextjs.org/docs/basic-features/layouts
antd 官方文档：https://ant-design.gitee.io/

## 引入 antd

在 Next 中引入 antd 与在常规 React 项目中引入 antd 是一样的，直接安装 antd 作为组件使用即可。

`npm i antd`

## 创建组件

在`/pages`下分别创建`/components/layout.tsx`、`/components/navbar.tsx`和`/components/footer.tsx`，并创建对应 React 函数组件。

```tsx
// components/layout.tsx
import React from 'react'
import Navbar from '@/pages/components/navbar'
import { Layout as AntdLayout } from 'antd'
import Footer from '@/pages/components/footer'
import style from '@/styles/Layout.module.scss'

function Layout({ children }: React.PropsWithChildren) {
  return (
    <>
      <AntdLayout>
        <Navbar />
        <AntdLayout.Content className={style.content}>{children}</AntdLayout.Content>
        <AntdLayout.Footer>
          <Footer />
        </AntdLayout.Footer>
      </AntdLayout>
    </>
  )
}

export default Layout
```

`Navbar.tsx`和`Footer.tsx`使用 antd 官方 demo，不放代码。

## 根节点包裹

在`/pages/_app.tsx`中包裹 Component，即完成对全局 Layout 的配置。

```tsx
import '@/styles/globals.scss'
import type { AppProps } from 'next/app'
import Layout from '@/pages/components/layout'
import { App as AntdApp } from 'antd'
import Head from 'next/head'

export default function App({ Component, pageProps }: AppProps) {
  return (
    <>
      <Head>
        <title>Next.js</title>
      </Head>
      <AntdApp>
        <Layout>
          <Component {...pageProps} />
        </Layout>
      </AntdApp>
    </>
  )
}
```
