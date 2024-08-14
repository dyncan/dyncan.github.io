---
layout: post
title: "TypeScript 中的三种特殊类型：any、unknown 和 never"
subtitle: ""
date: 2024-08-13 12:00:00
author: "Peter Dong"
header-img: "img/bg-post_three-special-types-in-typescript.jpg"
catalog: true
tags:
  - TypeScript
  - TypeSafety
  - WebDevelopment
---

TypeScript 的类型系统中，`any`、`unknown`和`never`这三种特殊类型各自扮演着独特的角色。本文将基于在 [Chrome Extension](https://chromewebstore.google.com/detail/salesforce-spotlight/kcnnhfdenihbihoikgjfapgphapdoggd){:target="_blank"} 和 [VS Code Extension](https://marketplace.visualstudio.com/items?itemName=dyncan.salesforce-soql-booster){:target="_blank"} 开发中的经验来探讨这些类型的概念、使用示例、优缺点等。

## 1. any 类型：双刃剑

### 概念

`any`类型是 TypeScript 类型系统中最灵活的类型，它允许赋予任何类型的值，并且可以对其进行任何操作。使用`any`类型实际上是在告诉编译器跳过类型检查。

#### 经典示例

```typescript
let flexibleVar: any;

flexibleVar = 42;
flexibleVar = "Hello, TypeScript";
flexibleVar = { name: "John", age: 30 };

// 以下操作不会在编译时报错，但可能在运行时出问题
console.log(flexibleVar.toUpperCase());
console.log(flexibleVar.nonExistentMethod());
```
![img](/img/in-post/post-three-special-types-in-typescript-01.png)

具体分析如下：

1. **编译时不会报错：**
   - 由于 `flexibleVar` 被定义为 `any` 类型，TypeScript 编译器允许对该变量进行任何类型的赋值操作及方法调用。因此，在编译时代码不会报错。

2. **`console.log(flexibleVar.toUpperCase());`**
   - **运行时错误：** 在这行代码中，`flexibleVar` 的当前值是一个对象 `{ name: "John", age: 30 }`。由于 `toUpperCase` 是字符串的方法，而不是对象的方法，因此尝试调用 `toUpperCase` 会导致错误。
   - **错误类型：** 运行时会抛出 `TypeError`，错误信息是 `flexibleVar.toUpperCase is not a function`。

3. **`console.log(flexibleVar.nonExistentMethod());`**
   - **运行时错误：** 在这行代码中，尝试调用一个不存在的方法 `nonExistentMethod`，但 `flexibleVar` 当前的值是一个对象 `{ name: "John", age: 30 }`，这个对象并不包含该方法。
   - **错误类型：** 运行时会抛出 `TypeError`，错误信息是 `flexibleVar.nonExistentMethod is not a function`。


### 容易犯错的场景

#### **类型污染**

```typescript
let pollutedArray: any[] = ["hello", "world"];
pollutedArray.push(42);  // 不会报错
let str: string = pollutedArray[2];  // 不会报错，但 str 实际上是 number 类型
console.log(str.toUpperCase());  // 运行时错误
```

![img](/img/in-post/post-three-special-types-in-typescript-02.png)

错误分析

1. **`pollutedArray.push(42);`**
   - **分析**: 这行代码将数字 `42` 添加到 `pollutedArray` 数组中，因为 `pollutedArray` 的类型是 `any[]`，它可以包含任何类型的元素，所以不会报错。

2. **`let str: string = pollutedArray[2];`**
   - **分析**: 这里将 `pollutedArray[2]` 的值（即 `42`）赋值给变量 `str`，并且 `str` 被声明为 `string` 类型。尽管编译器不会报错，但实际上 `str` 的值是一个 `number` 类型。

3. **`console.log(str.toUpperCase());`**
   - **分析**: `toUpperCase()` 是一个字符串方法，但由于 `str` 实际上是 `number` 类型（`42`），因此调用 `toUpperCase()` 方法会导致运行时错误。
   - **错误类型**: `TypeError`
   - **错误信息**: 运行时会抛出 `TypeError`，提示 `toUpperCase` 不是一个函数，因为 `number` 类型的变量没有 `toUpperCase` 方法。


#### **误用 any 绕过类型检查**

```typescript
interface User {
    name: string;
    age: number;
}

function processUser(user: any) {
    console.log(user.name.toUpperCase());
    console.log(user.age * 2); // 不报错，在 JavaScript 中，字符串 "30" 和数字相乘时，JavaScript 会尝试将字符串转换为数字
    console.log(user.age.toFixed(2)); // 报错，我们期望 age 是一个数字并要格式化输出
}

processUser({ name: "John", age: "30" });
```

![img](/img/in-post/post-three-special-types-in-typescript-03.png)

错误分析

在运行以下代码时可能会遇到一些错误：

1. **`user.name.toUpperCase()`**
   - **分析**: `user.name` 的值是 `"John"`，这是一个字符串类型。调用 `toUpperCase()` 方法会将字符串转换为大写。
   - **结果**: 该行代码不会报错，正常输出 `"JOHN"`。

2. **`user.age * 2`**
   - **分析**: `user.age` 的值是字符串 `"30"`。在 JavaScript 中，字符串和数字相乘时，JavaScript 会尝试将字符串转换为数字再进行运算。
   - **结果**: 该行代码不会报错，会输出 `60`。

3. **`user.age.toFixed(2)`**
   - **分析**: `toFixed()` 方法属于 `Number` 对象，用于格式化数字。由于 `user.age` 的值是字符串 `"30"`，而不是数字，调用 `toFixed()` 方法时会导致运行时错误。
   - **错误**: 在运行时，会抛出 `TypeError: user.age.toFixed is not a function`，因为字符串类型没有 `toFixed()` 方法。


### 最佳实践

- 尽量避免使用`any`，除非真的不确定类型。
- 如果必须使用`any`，请尽可能缩小其作用范围。
- 考虑使用`unknown`类型作为更安全的替代。

## 2. unknown 类型：安全的 any 替代品

### 概念

`unknown`类型是 TypeScript 3.0 引入的类型，它类似于`any`，但提供了更强的类型安全性。`unknown`类型的变量可以接受任何值，但在使用前必须进行类型检查或类型断言，可以视为严格版的 `any`。

#### 经典示例

```typescript
let safeVar: unknown;

safeVar = 42;
safeVar = "Hello, TypeScript";
safeVar = { name: "John", age: 30 };

// 以下操作会在编译时报错
// console.log(safeVar.toUpperCase());
// console.log(safeVar.nonExistentMethod());

// 正确的使用方式
if (typeof safeVar === "string") {
    console.log(safeVar.toUpperCase());  // 正确，因为已经检查了类型
}

// 或者使用类型断言
console.log((safeVar as string).toUpperCase());  //  使用类型断言，但要注意确保类型正确，运行时会出现错误，因为 safeVar 不是字符串
```

#### 改进第一部分的示例代码

```typescript
interface User {
    name: string;
    age: number;
}

function processUser(user: unknown) {
    if (typeof user === "object" && user !== null && "name" in user && "age" in user) {
        if (typeof user.name === "string" && typeof user.age === "number") {
            console.log(user.name.toUpperCase());
            console.log(user.age * 2);
        } else {
            console.log("Invalid user data types");
        }
    } else {
        console.log("Invalid user structure");
    }
}

processUser({ name: "John", age: "30" }); // 编译期会报错
```

### 最佳实践

- 当你不确定输入的类型时，使用`unknown`而不是`any`。
- 在使用`unknown`类型的值之前，始终进行类型检查或类型断言。
- 利用 TypeScript 的类型收窄功能来安全地使用`unknown`类型。

## 3. never 类型：表示不可能的类型

#### 概念

`never`类型表示永远不会出现的值的类型。它通常用于表示会抛出异常或永远不会返回的函数的返回类型，以及在某些高级类型操作中表示不可能的情况。

#### 经典示例

```typescript
// 总是抛出错误的函数
function throwError(message: string): never {
    throw new Error(message);
}

// 无限循环的函数
function infiniteLoop(): never {
    while (true) {}
}

// 在类型收窄中使用 never
type Fruit = "apple" | "banana" | "orange";

function assertNever(x: never): never {
    throw new Error("Unexpected object: " + x);
}

function getFruitColor(fruit: Fruit) {
    switch (fruit) {
        case "apple":
            return "red";
        case "banana":
            return "yellow";
        case "orange":
            return "orange";
        default:
            return assertNever(fruit);  // 确保涵盖了所有可能的情况
    }
}
```

#### 利用 never 进行完整性检查

```typescript
type Shape = Circle | Square | Triangle;

interface Circle {
    kind: "circle";
    radius: number;
}

interface Square {
    kind: "square";
    sideLength: number;
}

interface Triangle {
    kind: "triangle";
    sideLength: number;
}

function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.sideLength ** 2;
        case "triangle":
            return (Math.sqrt(3) / 4) * shape.sideLength ** 2;
        default:
            // 如果 Shape 联合类型新增了一个类型，这里会报错
            const _exhaustiveCheck: never = shape;
            return _exhaustiveCheck;
    }
}
```

**如何触发错误：**

可以在 Shape 联合类型中添加一个新的类型，例如 Rectangle，但在 getArea 函数的 switch 语句中不处理它。这样，default 分支的 never 类型检查将会报错。

```typescript
type Shape = Circle | Square | Triangle | Rectangle;

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}

interface Circle {
    kind: "circle";
    radius: number;
}

interface Square {
    kind: "square";
    sideLength: number;
}

interface Triangle {
    kind: "triangle";
    sideLength: number;
}

function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.sideLength ** 2;
        case "triangle":
            return (Math.sqrt(3) / 4) * shape.sideLength ** 2;
        default:
            const _exhaustiveCheck: never = shape; // 编译器会报错，由于 Rectangle 类型没有在 switch 语句中处理，TypeScript 编译器将会在 default 分支的 never 类型检查处报错
            return _exhaustiveCheck;
    }
}
```


### 最佳实践

- 使用`never`类型来表示那些永远不应该发生的情况。
- 在联合类型和 switch 语句中使用`never`来确保涵盖了所有可能的情况。
- 在泛型和高级类型操作中，`never`可以用来创建更精确的类型定义。

## 总结

通过这些经典示例和易错场景，我们可以看到`any`、`unknown`和`never`这三种特殊类型在 TypeScript 中的独特作用：

1. `any`类型提供了最大的灵活性，但应谨慎使用，因为它会失去 TypeScript 的类型安全性。
2. `unknown`类型是一种更安全的选择，它要求在使用前进行类型检查，非常适合处理不确定类型的数据。
3. `never`类型在表示不可能发生的情况时非常有用，尤其在进行完整性检查和高级类型操作时。

合理使用这些特殊类型可以显著提高代码的健壮性和可维护性。通过仔细选择合适的类型，我们可以充分利用 TypeScript 的类型系统，编写出更安全、更易于理解和维护的代码。记住，TypeScript 的强大之处不仅在于它的类型检查，还在于它如何帮助我们设计和实现更清晰、更可靠的程序结构。