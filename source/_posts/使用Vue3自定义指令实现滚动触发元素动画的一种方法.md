---
title: 使用Vue3自定义指令实现滚动触发元素动画的一种方法
author: DuCheng
top: false
tags:
  - vue
categories:
  - 前端
date: 2023-04-04 16:54:00
---

主要实现原理是使用 IntersectionObserver 对元素进行监听，以元素是否与该观察者相交来验证元素是否进入视图范围；使用 threshold 来控制元素与观察者相交多少百分比后触发事件。

```TypeScript
import { Directive } from 'vue'

const anime: Directive = {
  mounted: (el: HTMLElement, binding: { value: string }) => {
    if (binding.value.endsWith('Big')) {
      console.error(el, '避免使用Big类型的动画，这可能会引起未知错误')
      return
    }
    const name = binding.value ? binding.value : 'fadeIn'

    el.classList.add('animate__animated')
    // 创建IntersectionObserver实例
    const observer = new IntersectionObserver(
      (entries: IntersectionObserverEntry[]) => {
        const entry = entries[0]
        // 若该元素已经触发过动画则不再触发
        if (el.classList.contains('animate__done')) return
        // 监听元素是否与Observer相交
        if (entry.isIntersecting) {
          el.style.opacity = '100%'
          el.classList.add(`animate__${name}`)
          // 添加animate__done类 声明该元素已经触发过动画
          el.classList.add('animate__done')
        } else {
          el.style.opacity = '0%'
          el.classList.remove(`animate__${name}`)
        }
      },
      {
        // 元素与Observer相交90%时再触发事件
        threshold: [0.9]
      }
    )
    // 监听当前元素
    observer.observe(el)
  }
}

export default anime
```
