---
layout: post
title: "常用的 Javascript 方法"
subtitle: ""
date: 2022-03-28 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-js-method.jpeg"
catalog: true
tags:
  - Javascript
  - Skills
  - Front-end
---

平时在工作中有些时候需要使用 JS 显示数据列表，并且希望对数据进行分类，过滤，搜索，修改或更新。或者你想进行快速计算，如求和，乘法等等。实现这一目标的最佳方式是什么？今天将会介绍几种常用的方法帮助提高数据处理效率.
#### 1. Spread operator

可以在函数调用/数组构造时，将数组表达式或者 string 在语法层面展开; 还可以在构造字面量对象时，将对象表达式按 key-value 的方式展开。

**示例：** 
你想在不创建循环函数的情况下显示一个最喜欢的食物列表。可以使用 Spread operator.

``` javascript
  const favoriteFood = ['Pizza', 'Fries', 'Apple'];
  console.log(...favoriteFood); // Pizza Fries Apple
```

#### 2. for…of

for…of 语句用于遍历可迭代对象的元素，并为你提供修改特定项目的能力。它取代了传统的 for-loop 方式。

**示例：**

``` javascript
  const toolBox = ['Hammer', 'Screwdrtver', 'Ruler']
  for(const item of toolBox){
    console.log(item)
  }
  // Hammer
  // Screwdrtver
  // Ruler
```

#### 3. Includes()

includes() 方法是用来检查一个特定的字符串是否存在于一个集合中，并返回 true 或 false. 请记住，它是区分大小写的：如果集合里面的项目是 APEX, 而你搜索的是 Apex, 它将返回 false.

**示例：**
由于某种原因，你不知道你的车库里有哪些汽车，你需要一个系统来检查你想要的汽车是否存在。

```javascript
const garage = ['BMW', 'AUDI', 'VOLVO']; 
 
const findCar = garage.includes('BMW'); 

console.log(findCar);
// true
```

#### 4. Some()

some() 方法检查一个数组中是否存在某些元素，并返回真或假。这有点类似于 includes() 方法，只不过参数是一个函数而不是一个字符串。

**示例：**

**ES5:**
```javascript
const age = [16, 14, 18];

age.some( function(person){

  return person >= 18;

});
// Output: true
```

**ES6:**
```javascript
const age = [16, 14, 18];

age.some((person) => person >= 18);
// true
```

#### 5. Every()

every() 方法在数组中循环检查每个项目，并返回真或假。与 some() 的概念相同。除了每个项目都必须满足条件语句，否则，它将返回 false.

**示例：**

**ES5:**
```javascript
const age = [15, 20, 19];

age.every( function(person){

  return person >= 18;

});
// Output: false
```

**ES6:**
```javascript
const age = [15, 20, 19];

age.every((person) => person >= 18);
// false
```

#### 6. Filter()

filter() 方法会用所有通过条件的元素创建一个新数组。

**示例：**

**ES5:**
```javascript
// array
const prices = [25, 30, 15, 55, 40, 10];

prices.filter(function(price){ 

  return price >= 30;

});
// Output:[ 30, 55, 40 ]
```

**ES6:**
```javascript
const prices = [25, 30, 15, 55 40, 10]; 

prices.filter((price) => price >= 30);
// [30, 55,40]
```

#### 7. Map()

map() 方法与 filter() 方法类似，都是返回一个新的数组。然而，唯一的区别是，它是用来修改项目的。

**示例：**

**ES5:**
```javascript
const productPriceList = [200, 350, 1500, 5000]; 

productPriceList.map(function(item){

  return item * 0.75;

});
// [ 150, 262.5, 1125, 3750 ]
```

**ES6:**
```javascript
const productPriceList = [200, 350, 1500, 5000]; 

productPriceList.map(( item ) => item * 0.75);
// [ 150, 262.5, 1125, 3750 ]
```

#### 8. Reduce()

reduce() 方法可以用来将一个数组转化为其他东西，可以是一个 integer, 一个 object, chain promises 等等。一个简单的用例是对一个整数列表求和。简而言之，它将整个数组 "还原 "为一个值。

**示例：**

**ES5:**
```javascript
const weeklyExpenses = [200, 350, 1500, 5000, 450, 680, 350];

weeklyExpenses.reduce(function(first, last){
  return first + last;
});
// 8530
```

**ES6:**
```javascript
const weeklyExpenses = [200, 350, 1500, 5000, 450, 680, 350];
weeklyExpenses.reduce((first, last) => first + last);
// 8530
```
