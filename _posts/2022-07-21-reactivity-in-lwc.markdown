---
layout: post
title: "浅谈 LWC 中的数据渲染"
subtitle: ""
date: 2022-07-21 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-reactivity-lwc.jpeg"
catalog: true
tags:
  - Reactivity
  - LWC
  - Salesforce
---



在 LWC 中, 如果一个属性的值发生了变化, 并且该属性在 tempalte 中有使用, 或者声明在一个属性的 getter 方法中, 该组件会重新显示新的值. 如果一个属性被分配给一个对象或一个数组, 框架会观察该对象或数组内部的一些变化, 比如当你分配了一个新值.

当一个组件重新渲染时, 模板中使用的表达式被重新评估, `renderedCallback()` 生命周期钩子会被执行.

我们来看接下来的一个例子:

当你在 FirstName/LastName 输入一个值时, 该组件将其转换为大写字母并渲染出来.

![gif](https://media.giphy.com/media/WpqAPDGqVKhFuVzxEG/giphy.gif)

```html
<template>
    <lightning-card title="HelloExpressions" icon-name="custom:custom14">
        <div class="slds-m-around_medium">
            <lightning-input
                name="firstName"
                label="First Name"
                onchange={handleChange}
            ></lightning-input>
            <lightning-input
                name="lastName"
                label="Last Name"
                onchange={handleChange}
            ></lightning-input>
            <p class="slds-m-top_medium">
                Uppercased Full Name: {uppercasedFullName}
            </p>
        </div>
    </lightning-card>
</template>
```

```javascript
import { LightningElement } from 'lwc';

export default class TrackExample extends LightningElement {
    firstName = '';
    lastName = '';

    handleChange(event) {
        const field = event.target.name;
        if (field === 'firstName') {
            this.firstName = event.target.value;
        } else if (field === 'lastName') {
            this.lastName = event.target.value;
        }
    }

    get uppercasedFullName() {
        return `${this.firstName} ${this.lastName}`.trim().toUpperCase();
    }
}
```

在 `Spring'20`之前,为了让组件在用户输入时重新渲染,你必须用 `@track` 来修饰这些属性.

```javascript
@track firstName = '';
@track lastName = '';
```

在 lwc 中, Fields 是 reactivity. `Expandos` 是在运行时添加到对象的属性, 不是Reactivity.

Expandos 是什么意思?  在javascript中,任何对象都是一个 `expando object`(可扩展对象).它的意思是, 只要你试图访问一个属性1, 它就会被自动创建.

```javascript
var myObj = {}; // 空的对象
myObj.myProp = 'value';
```

当你给 `myProp` 赋值的时候, `myProp` 这个属性就被动态地创建了, 尽管它之前并不存在的. 所以 `expando` 的能力是写, 而不是访问. Javascript对象允许你向一个对象写入新的属性, 而不需要像其他一些语言那样预先定义该属性.

### Reactivity 的一些考量

尽管 fields 是反应式的, 但LWC引擎以一种比较浅层的方式跟踪属性值的变化. 当一个新的值被分配给属性时, 通过使用 `===` 来比较值的身份来检测变化. 这对于像数字或布尔这样的原始类型很有效.

```javascript
import { LightningElement } from 'lwc';

export default class ReactivityExample extends LightningElement {
  bool = true;
  number = 42;
  obj = { name: 'John' };

  checkMutation() {
    this.bool = false;   //  检测到变化
    
    this.number = 42; // 没有检测到变化: 之前的值等于新分配的值
    this.number = 43; // 检测到变化
    
    this.obj.name = 'Bob'; // 检测到变化
    this.obj = { name: 'John' }; // 检测到变化 - 用相同的值重新定义对象,创建一个新的对象.
    this.obj = { ...this.obj, title: 'CEO' } // 检测到变化
  }  
}
```

当操作复杂的类型如对象和数组时, 你必须创建一个新的对象, 并将其分配给property, 以便检测到值的变化.

为了避免在处理复杂对象时出现这样的问题,可以使用 `@track` 来深入追踪对属性值的监测.

#### 追踪对象和数组内部的变化

为了观察一个对象的属性或一个数组的元素的变化,用 `@track` 来修饰这参数.

当一个 property 用 @track 修饰时,Lightning Web Components 会跟踪其内部值的变化.

LWC 引擎以递归方式观察对普通对象和数组的检测,包括嵌套对象,嵌套数组以及对象和数组的混合.循环引用也会被处理.

然而,LWC 引擎并不监测对复杂对象变化,例如 继承自 Object 的对象, 类实例, Date, Set 或 Map.

#### 观察一个对象的属性

为了告诉框架观察一个对象的属性变化, 用 @track 来修饰这个属性. 如前面所讲, 在不使用 `@track` 的情况下, 框架会观察为字段分配新值的更改. 如果新值不是 === 之前的值, 组件将重新渲染.

例如, 让我们稍微改变一下代码, 声明fullName字段, 它包含一个有两个属性的对象, firstName和lastName. 框架观察到的变化是给fullName分配一个新的值.

```javascript
fullName = { firstName : '', lastName : ''};
```

这段代码为 fullName 属性分配了一个新的值,因此该组件重新渲染.

```javascript
// rerenders.
this.fullName = { firstName : 'Peter', lastName : 'Dong'};
```

然而,如果我们给对象的一个属性分配一个新的值,这个组件就不会重新渲染,因为这些属性没有被监测到.

```javascript
// doesn't rerender.
this.fullName.firstName = 'Leo';
```

为了告诉框架监测对象属性的变化,用 `@track` 来修饰 `fullName` 字段.现在,如果我们改变任何一个属性,该组件就会重新渲染.

```javascript
// rerenders.
@track fullName = { firstName : '', lastName : ''};
this.fullName.firstName = 'John';
```

### 监测一个数组的元素

@track的另一个用例是告诉框架监测一个数组元素的变化. 如果你不使用 `@track`,框架会监测为字段分配新值的变化.

```javascript
arr = ['a','b'];
```

当你给arr分配一个新的值时,该组件会重新渲染.

```javascript
this.arr = ['x','y','z'];
```

然而,如果我们更新或添加数组中的一个元素,该组件就不会重新渲染.

```javascript
this.arr[0] = 'x';
this.arr.push('c');
```

要告诉框架监测数组元素的变化,可以用 `@track` 来修饰 arr 字段.此外,框架不会为数组元素的更新自动转换为字符串.要返回更新的字符串,请使用getter将数组的元素用join()转换为字符串.

```javascript
@track arr = ['a','b'];

get computedArray() { return this.arr.join(','); }

update() {
    this.arr[0] = 'x';
    this.arr.push('c');
}
```

### 监测复杂对象

让我们看看一个有日期类型属性x, template上有几个按钮可以改变x的内部状态. 这个例子中 new Date()创建了一个对象, 因为它不是一个普通的JavaScript对象, 所以内部状态的变化不会被LWC引擎监测到,即使代码使用了`@track`.

```javascript
import { LightningElement, track } from 'lwc';
export default class TrackDate extends LightningElement {
    @track x = new Date();

    initDate() {
        this.x = new Date();
    }

    updateDate() {
        this.x.setHours(7); // No mutation detected
    }
}
```

与我们之前的例子类似，template有几个按钮可以改变 x 的内部状态。

```html
<template>
    <p>Date: {x}</p>
    <button onclick={initDate}>Init</button>
    <button onclick={updateDate}>Update</button>
</template>
```

当你点击 Init 按钮时，变化被监测到，模板被重新渲染。Lightning Web Components 监测到x正指向一个新的Date对象。然而，当你点击更新时，模板并没有重新渲染。Lightning Web Components没有监测到Date对象的值的变化。

为了确保模板在值发生变化时被重新渲染，克隆现有的日期并更新其值。

```javascript
updateDate() {
    const cloned = new Date(this.x.getTime());
    cloned.setHours(7);

    this.x = cloned;
}
```






