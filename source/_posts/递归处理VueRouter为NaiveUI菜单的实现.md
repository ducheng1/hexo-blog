---
title: 递归处理VueRouter为NaiveUI菜单的实现
author: DuCheng
top: false
tags:
  - naive-ui
  - vue
categories:
  - 前端
date: 2023-05-05 09:44:00
password:
---

# 主要方法

**handleRoute**

```TypeScript
const menuItems = ref<Array<MenuOption>>([])
const rawRoutes = ref<RouterOptions['routes']>(router.options.routes)

// 渲染图标至菜单
const renderIcon = (icon: Component) =>
    icon ? () => h(NIcon, null, { default: () => h(icon) }) : () => null

// 递归处理路由
const handleRoute = (routes: RouterOptions['routes'], pname?: RouteRecordName) => {
  // 处理子路由
  if (pname) {
    let children = []
    const pRoute = rawRoutes.value.filter(item => item.name === pname)
    // @ts-ignore
    for (const child of pRoute[0].children) {
      children.push({
        label: () => h(
            RouterLink,
            {
              to: {
                name: child.name
              }
            },
            {
              default: () => child.meta?.title
            }
        ),
        key: child.name as unknown as Key,
        icon: renderIcon(child.meta?.icon as Component)
      })
    }
    menuItems.value.push({
      label: pRoute[0].meta?.title,
      key: pRoute[0].name as unknown as Key,
      icon: renderIcon(pRoute[0].meta?.icon as Component),
      children
    })
  } else {
    for (const route of routes) {
      // 如果有子节点则递归
      if (route.children && route.children.length !== 0) {
        console.log(route.name)
        handleRoute(route.children, route.name)
        continue
      }
      // 隐藏菜单则不处理
      if (route.meta?.hideMenu) {
        continue
      }
      menuItems.value.push({
        label: () => h(
            RouterLink,
            {
              to: {
                name: route.name
              }
            },
            {
              default: () => route.meta?.title
            }
        ),
        key: route.name as unknown as Key,
        icon: renderIcon(route.meta?.icon as Component)
      })
    }
  }
}
```

# 其中一块路由定义

**multiNavRoutes.ts**

```TypeScript
import { RouteRecordRaw } from 'vue-router'
import { NavigateCircleOutline } from '@vicons/ionicons5'

const multiNavRoutes: RouteRecordRaw[] = [
  {
    path: '/multi-nav',
    name: 'multi-nav',
    redirect: { name: 'multi-nav-inner' },
    meta: {
      title: '多级导航',
      icon: NavigateCircleOutline
    },
    children: [
      {
        path: 'inner',
        name: 'multi-nav-inner',
        component: () => import('@/views/modules/multi-nav/inner-nav/index.vue'),
        meta: {
          title: '内联导航',
        }
      }
    ]
  }
]

export default multiNavRoutes
```

# 路由元数据类型

**types/index.d.ts**

```TypeScript
import { Component } from 'vue'

declare module 'vue-router' {
  interface RouteMeta {
    keepAlive?: boolean
    noLayout?: boolean
    title?: string
    needLogin?: boolean
    hideMenu?: boolean
    icon?: Component
  }
}
```
