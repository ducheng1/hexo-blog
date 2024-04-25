---
title: 使用Vue3+Vant实现移动端@功能
author: DuCheng
top: false
tags:
  - vue
  - vant
categories:
  - 前端
date: 2024-04-24 09:40:00
cover:
password:
---

# 基础DOM结构

为了方便进行DOM操作，实际上是使用原生的可编辑元素`contenteditable`实现一个简单的富文本框。

```vue
<script setup lang="ts">
const inputRef = shallowRef<HTMLDivElement>();
</script>

<template>
  <div contenteditable class="mention-input" ref="inputRef"></div>
</template>

<style lang="scss" scoped>
.mention-input {
  background-color: #f6f7fb;
  width: 100%;
  padding: 18px 24px;
  max-height: 172px;
  border-radius: 8px;
  overflow-x: hidden;
  font-size: 28px;
  line-height: 36px;

  &:focus-visible {
    outline: none;
  }

  // 这里使用:empty选择器代替placeholder
  &:empty::before {
    content: "请输入评论";
    color: #b1b2b6;
  }
}
</style>
```

以上就得到了一个简单的可编辑元素。

![image-20240424085618205](https://img.dcwedu.top/i/2024/04/24/662858b3045ea.png)

# 监听是否输入@并替换为所需内容

## 添加监听器

接下来就要给这个div添加一个监听器，监听用户的键入事件，用于判断是否输入了@并显示人员选择框。这里我选择监听`keyup`事件（`keydown`同理）。

```vue
<template>
  <div
    contenteditable
    class="mention-input"
    ref="inputRef"
    @keyup="onKeyup"
  ></div>
</template>
```

当触发`keyup`事件时，所要做的就是去判断是否键入了@，以此判断是否需要显示人员选择框。为了提升用户体验，并不能简单地去判断`e.code==='@'`，而是需要获取@对应的`TextNode`以及当前光标所在位置。对于这一点，请确保你知道在`contenteditable`的元素中输入后生成的是`TextNode`，也就是DOM中对应的`#text`元素。知道了这一点后，要做的就比较简单了。我们先定义两个ref，一个用于保存光标位置，一个用于保存@对应的`TextNode`。

```typescript
// 光标位置
const cursorIndex = ref<number>();
// @对应的TextNode
const textNode = ref<Selection["focusNode"]>();
```

## 获取光标位置和TextNode

接下来需要去获取这两个值。这里用到了[`window.getSelection()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getSelection)，官方文档对于该API的描述如下：

![image-20240424090448097](https://img.dcwedu.top/i/2024/04/24/66285aaf9094e.png)

兼容性还是不错的。

![image-20240424090548426](https://img.dcwedu.top/i/2024/04/24/66285aebcaf2d.png)

获取对应光标位置和`TextNode`的代码如下：

```typescript
function setCursorIndex() {
  cursorIndex.value = window.getSelection()?.focusOffset;
}

function setTextNode() {
  textNode.value = window.getSelection()?.focusNode;
}
```

## 判断是不是@

接下来就需要用`TextNode`去判断是不是@。

```typescript
// 是否键入@
function isAt() {
  // 判断是否获取到了@对应的TextNode
  if (!textNode.value || textNode.value?.nodeType !== Node.TEXT_NODE)
    return false;
  // 获取元素中的文本内容
  const content = textNode.value.textContent ?? "";
  // 获取文本内容中的@（正则结束断言）
  const match = /@$/.exec(content.slice(0, cursorIndex.value));
  // 是否匹配到@并且只有一个@字符
  return match && match.length === 1;
}
```

## Keyup监听器的具体逻辑

我这里用了[`vueuse`](https://vueuse.org/shared/useToggle/)，直接用ref肯定也是可以的。注意这里是移动端，用的是Vant的popup组件，具体逻辑自己实现。

```vue
<script setup lang="ts">
import { useToggle } from "@vueuse/core";

// 是否显示选择用户弹窗
const [showSelectUser, setShowSelectUser] = useToggle();

function onKeyup(e: KeyboardEvent) {
  setTextNode();
  setCursorIndex();
  // 判断是否在用键盘移动光标
  if (
    e.code === "ArrowUp" ||
    e.code === "ArrowDown" ||
    e.code === "ArrowLeft" ||
    e.code === "ArrowRight"
  ) {
    return;
  }
  // 判断是否键入@
  if (!isAt() || e.code === "Backspace" || e.code === "Delete") {
    return;
  }
  // 显示选择用户弹窗
  setShowSelectUser(true);
}
</script>

<template>
  <div
    contenteditable
    class="mention-input"
    ref="inputRef"
    @keyup="onKeyup"
  ></div>
  <UserSelect v-model:show="showSelectUser" @confirm="handleUserSelect" />
</template>
```

## 替换@节点逻辑

在上一个小节中，我们从`UserSelect`组件获取到了选中的用户列表，并通过`confirm`事件返回给父组件。接下来要实现的就是`handleUserSelect`这个方法，替换@节点。具体实现如下：

```typescript
// 选中用户后替换@
function handleUserSelect(userList: any[]) {
  // 代码聚焦可编辑div
  inputRef.value?.focus();

  if (!userList.length || !textNode.value) return;

  // 获取@内容
  const content = textNode.value!.textContent ?? "";

  // 获取父节点（可编辑div）和相邻节点
  const parentNode = textNode.value.parentNode;
  const nextNode = textNode.value.nextSibling;

  // 使用@分割文本并替换掉@
  const preSlice = content.slice(0, cursorIndex.value).replace(/@$/, "");
  const restSlice = content.slice(cursorIndex.value);

  // 使用上面分割出来的文本创建Text节点
  const prevTextNode = new Text(preSlice.slice(0, preSlice.length - 1));
  const nextTextNode = new Text(restSlice);

  // 创建mention元素
  const mentionEl = userList.map((user) => createMentionElement(user));

  mentionEl.forEach((el) => {
    // 将mention元素插入到文本框
    // 判断是否存在相邻节点，存在则在相邻节点之前插入，不存在则直接插入
    if (nextNode) {
      parentNode?.insertBefore(prevTextNode, nextNode);
      parentNode?.insertBefore(el, nextNode);
      parentNode?.insertBefore(nextTextNode, nextNode);
    } else {
      parentNode?.appendChild(prevTextNode);
      parentNode?.appendChild(el);
      parentNode?.appendChild(nextTextNode);
    }
  });

  // 移除前置@textNode
  parentNode?.removeChild(textNode.value);

  // 重置光标位置
  const range = new Range();
  const selection = window.getSelection();
  range.setStart(nextTextNode, 0);
  range.setEnd(nextTextNode, 0);
  selection?.removeAllRanges();
  selection?.addRange(range);

  setTextNode();

  // 移除末尾@textNode
  parentNode?.removeChild(textNode.value);
}
```

> 注意：这里就是需要移除两次TextNode，因为在多选情况下会生成多个TextNode。

## 创建mention元素

上一节中我们实现了替换@节点，这一节展示如何创建用于替换@节点的元素，即代码中的`createMentionElement`方法。

```typescript
function createMentionElement(user: any) {
  // 创建mention-node元素
  const el = document.createElement("span");
  el.style.display = "inline-block";
  // 给元素添加data-属性，后端需要什么就挂什么，这里只需要id
  el.dataset.userId = user ? user.id : "";
  // 类名，注意需要是不会重复的，后续需要获取元素伪数组
  el.className = "mention-node";
  // 元素内容
  el.textContent = user ? user.username : "";

  // 使用\u200b零宽字符占位，用于后续替换为约定格式
  // 创建前置占位符
  const spaceEl = document.createElement("span");
  spaceEl.style.whiteSpace = "pre";
  spaceEl.textContent = "@\u200b";

  // 创建后置占位符
  const spaceElAfter = document.createElement("span");
  spaceElAfter.style.whiteSpace = "pre";
  spaceElAfter.textContent = "\u200b ";
  // 可选，克隆节点
  // const spaceElAfter = document.cloneNode(spaceEl)

  // 创建父元素进行包裹
  const wrapper = document.createElement("span");
  wrapper.appendChild(spaceEl);
  wrapper.appendChild(el);
  wrapper.appendChild(spaceElAfter);
  return wrapper;
}
```

![image-20240424093202427](https://img.dcwedu.top/i/2024/04/24/66286111bc99a.png)

到了这里我们的@功能已经实现了，之后只需要实现获取文本框内容逻辑，实现的是业务所需的逻辑，仅供参考。

# 获取文本框内容（参考）

```typescript
function getContent() {
  // 获取可编辑div中的内容
  const content = inputRef.value?.textContent ?? "";
  const userIdList: string[] = [];
  // 获取所有mention元素中的数据，这里只需要id
  document.querySelectorAll(".mention-node").forEach((item) => {
    userIdList.push(item.getAttribute("data-user-id") ?? "");
  });
  return {
    // 我跟后端约定的格式为@#username# ，这里将\u200b零宽字符替换为#就是我们所约定的格式
    content: content.replaceAll("\u200b", "#"),
    userIdList: userIdList.filter(Boolean),
  };
}

// 清空可编辑div内容
function clear() {
  inputRef.value!.textContent = "";
}

defineExpose({
  getContent,
  clear,
});
```

# 在列表中替换展示@节点（参考）

## main.ts

```typescript
import VueDompurifyHtml from "vue-dompurify-html";

const app = createApp(App);

// 安全的v-html
app.use(VueDompurifyHtml);
```

## bubble.vue

```vue
<script setup lang="ts">
// 替换@为特殊样式
function handleContent(content: string) {
  return content.replaceAll(/@#.+#/g, (item) => {
    return `<span class="mention">${item.replaceAll("#", "")} </span>`;
  });
}
</script>

<template>
  <div class="content" v-dompurify-html="handleContent(content)"></div>
</template>

<style lang="scss" scoped>
...
:deep(.mention) {
  color: var(--van-primary-color);
}
...
</style>
```

![image-20240424093544378](https://img.dcwedu.top/i/2024/04/24/662861efa9f18.png)
