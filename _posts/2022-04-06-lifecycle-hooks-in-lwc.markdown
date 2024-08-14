---
layout: post
title: "LWC 的生命周期函数"
subtitle: ""
date: 2022-04-06 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-lifecycle-hooks-in-lwc.jpeg"
catalog: true
tags:
  - LWC
  - Lifecycle
  - Front-end
---

LWC 的生命周期负责创建，插入组件到 DOM 中，并将其渲染和从 DOM 中删除。它还监控已渲染的组件的任何属性变化，Lifecycle 钩子其实是一种回调方法，在组件实例的生命周期的特定阶段被触发。下面为大家介绍 LWC 的几个生命周期 Hooks.

### 1. Constructor() 

当一个组件的实例被创建时，constructor() 被调用。在这个阶段，我们不能访问组件的公共属性 (@api 修饰), 因为它们还不存在，如果我们在组件中包含了一个子组件，那么我们也不能访问子组件的元素，因为 LWC 的生命周期流程是从父组件到子组件。也就是说，子组件将在父组件被渲染后被渲染。下面的例子显示了我们如何在 LWC 文件中使用 constructor 函数。

```javascript
import { LightningElement } from "lwc";

export default class ParentChildDemo extends LightningElement {
  constructor() {
    super(); // calling constructor of LightningElement
  }
}
```

作为一个必要的步骤，我们需要从构造函数 () 中调用 super() 关键字。因为每个 LWC 组件都扩展了 LightningElement，而 LightningElement 有它的构造函数，我们不应该绕过调用父类的构造函数。

我们可以从构造函数中设置值，但最好的做法是使用 getter 和 setter 方法来获取或设置变量的值。请参考下面的例子来更好地理解它。

```javascript
import { api, LightningElement } from "lwc";

export default class ParentChildDemo extends LightningElement {
  variable1;
  variable2;

  constructor() {
    super();
    this.variable1 = "value";
  }

  @api
  get item() {
    return this.variable2;
  }

  set item(value) {
    this.variable2 = value.toUpperCase();
  }
}
```

对于构造函数，有一些注意事项。

**DO's**

  - 设置属性的值，也可以定义变量.
  - 可以调用 Apex Method
  - 可以调用 UI APIs, 比如 uiRecordApi

**DON'T**

  - 不要试图访问元素的属性，因为它们还不存在。
  - 不能从构造函数中 create 和 dispatch custom events, 比如 show toast message


### 2. ConnectedCallback() 

当组件被插入到 DOM 中时，它被调用。当这个方法被执行的时候，所有的公共属性都已经从它们的父组件那里得到了值。在这个方法中，我们可以调用需要将公共属性作为参数的 Apex 方法，因为现在公共属性已经存在了。

如果我们想在组件被加载到 DOM 后执行任何种类的逻辑，我们可以使用 connectedCallback() 生命周期方法。

```javascript
import { LightningElement, api } from "lwc";

export default class ParentChildDemo extends LightningElement {
  @api publicProperty;

  constructor() {
    super();
  }

  connectedCallback() {
    console.log("Now We can access public property: " + this.publicProperty);
  }
}
```

要检查组件是否连接到 DOM，我们可以使用 isConnected 属性，如果组件连接了，则返回真，否则返回假。

```javascript
import { LightningElement } from "lwc";

export default class ParentChildDemo extends LightningElement {
  constructor() {
    super();
    let elmt = this.template;
    console.log("From constructor: " + elmt.isConnected); // output: false
  }

  connectedCallback() {
    let elmt = this.template;
    console.log("From connectedCallback: " + elmt.isConnected); // output: true
  }
}
```

对于 connectedCallback 函数，有一些注意事项。

**DO's**

  - 可以触发一个 Custom Event.
  - 可以调用 UI APIs & navigation service
  - 订阅/取消订阅一个 message channel
  - 可以访问组件的元素

**DON'T**

  - 还不能访问任何子组件的元素，因为这时候子组件还不存在.

### 3. renderedCallback() 

renderedCallback 是 LWC 特有的，其他的都是 HTML custom elements 规范。由于这个钩子在组件的每次渲染后都会被调用，所以应该对它进行保护，以避免触发无限的渲染循环。当一个属性的值发生变化时，一个组件会被 rendered.

下面的示例展示了 renderedCallback 是如何工作的

我们现在有两个组件：parentLwc 和 childLwc.

**parentLwc.html**
```html
<template>
  <c-child-lwc></c-child-lwc>
</template>
```

**parentLwc.js**
```javascript
import { LightningElement } from "lwc";

export default class ParentLwc extends LightningElement {
  constructor() {
    super();
  }

  connectedCallback() {}

  renderedCallback() {
    console.log("renderedCallback() Called From Parent component.");
  }
}
```

**childLwc.html**
```html
<template>
  <h1>I'm child component</h1>
</template>
```

**childLwc.js**
```javascript
import { LightningElement } from "lwc";

export default class ChildLwc extends LightningElement {
  constructor() {
    super();
  }

  connectedCallback() {}

  renderedCallback() {
    console.log("renderedCallback() Called From Child component.");
  }
}
```

**Output logs:**
```javascript
// renderedCallback() Called From Child component.
// renderedCallback() Called From Parent component.
```

renderedCallback() 方法被多次调用，导致页面的无用渲染，为了确保 renderedCallback() 只在需要的时候被调用，我们可以使用私有布尔属性来判断。

**DO's**

  - 在一个组件完成渲染阶段后执行业务逻辑.
  - 访问组件元素
  - 调用 Apex, UI APIs, navigation service
  - 创建 和 Dispatch custom events


**DON'T**

  - 不要使用 renderedCallback() 来改变一个正在设置属性值的组件的状态，可以使用 getter 和 setter.
  - 不能在 renderedCallback() 中更新一个 wire adapter 配置的对象的属性，因为这会导致无限循环。
  - 不要在 renderedCallback() 中更新一个 reactive 属性或字段，因为这可能会导致无限循环。

### 4. disconnectedCallback

当一个组件被从 DOM 中移除时，它就会被执行。它是编写组件被销毁或从 DOM 中移除后需要实现的逻辑的最佳位置。

```javascript
import { LightningElement } from "lwc";

export default class ParentChildDemo extends LightningElement {
  constructor() {
    super();
  }

  disconnectedCallback() {
    console.error("disconnectedCallback() Called From Parent component.");
  }
}
```


**DO's**

  - 移除缓存
  - 移除事件监听器
  - 取消订阅 LMS channels

### 5. errorCallback(error, stack)

在生命周期阶段，当所有子组件出现故障或错误时，它将被执行。它是 LWC 独有的，其余的都来自 HTML 规范。通过这个钩子，我们可以给我们的组件设置一个错误边界。它可以帮助我们记录堆栈信息，并在子组件中遇到错误时渲染一个替代模板。这样，用户就可以知道到底发生了什么，接下来该怎么做。

它类似于 javaScript 的 catch{}块，用于在子组件生命周期钩子中抛出错误的组件。是的，错误边界组件只抓取来自子组件生命周期钩子的错误，而不是来自它自己。

它需要两个参数：error 和 stack。error 是一个 javaScript 本地错误对象，stack 是一个字符串。

```javascript
import { LightningElement } from "lwc";

export default class ParentChildDemo extends LightningElement {
  error;
  stack;

  constructor() {
    super();
  }

  errorCallback(error, stack) {
    console.error("errorCallback() Called From Parent component.")
    this.error = error;
    this.stack = stack;
  }
}
```