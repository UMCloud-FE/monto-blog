---
layout: post
title: 微前端中常见的几种CSS隔离实现方案
date: 2023-06-12 20:21:23
tags:
---

微前端中，让人头疼的一个问题，CSS样式覆盖和冲突问题。

如何能让子应用的样式保持独立，业界采用了各种不同的方案，这里做一下汇总，每种方案都有其可取之处，但也都是在特定场景下的解决方案，了解这些实现方案，能更好的处理问题。

## 一、BEM

CSS 命名规范，根据子应用提前约定CSS前缀。

原理：通过制定一套命名规范，避免样式之间的相互干扰，提高样式表的可维护性。

*   优点是实现简单、清楚。
*   不足：1、容易突破规范。2、人肉维护成本高

具体用法：常见的 CSS 命名规范有 BEM、OOCSS、SMACSS 等。例如，使用 BEM 命名规范，可以将一个按钮的 HTML 和 CSS 代码写成这样：

BEM规范：Block、Element、Modifier

*   `__`：双下划线用来连接块和块的子元素，左右的元素可以单独作为类
*   `-` ：仅作为连字符使用，连接块或元素或修饰符的多个单词（也可以直接写成驼峰式），不能拆分，两个属性描述一个类
*   `--`：双中划线用来连接块或元素的状态（也可使用‘\_’单下划线表示,本文以'--'方式介绍）：不同的属性值

```html
<div class="search-box search-box--hover">
    <div class="search-box__item"></div>
    <div class="search-content__item"></div>
    <button class="search-box__button"></button>
</div>
```

对应的CSS文件

```css
.search-box {
    display: inline-block;
    border-radius: 5px;
    font-size: 16px;
}
item {
    padding: 10px;
}
.search-box--hover {
    cursor: pointer;
}
.search-box__item {
    display: inline-block;
    border-radius: 5px;
    font-size: 16px;
    padding: 10px;
}
.button {
    background-color: blue;
    color: white;
}
```

## 二、CSS Module

通过import导入css变量，引用css，结合webpack，css-loader支持

优点是：实现简单、引用不符合使用习惯

**index.module.css**

```css
.color-red {
    color: red;
}
```

**app.tsx**

```js
// 不使用 import 
// './index.module.css';

// 使用 CSS Modules 
import styles from './index.module.css'; 

function App() {
  return (
    <div className='color-red'>
      <div className={styles['color-red']}>
        站点头部颜色红色
      </div>
      <Dashboard />
    </div>
  );
}
```

参考：[阮一峰CSS Modules 用法教程](https://www.ruanyifeng.com/blog/2016/06/css_modules.html)

## 三、CSS in JS

原理：将 CSS 样式表编写成 JavaScript 对象，以避免样式之间的相互干扰。

- 优点：实现了CSS的独立
- 不足：CSS嵌入JS耦合太深，维护不方便

具体用法：常见的 CSS in JS 库有 `styled-components`、`Emotion`、`JSS` 等。

例如，使用 styled-components，可以将一个按钮的样式写成这样：

`button.js`

```js
import styled from 'styled-components';

const Button = styled.button`
  display: inline-block;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 16px;

  &.large {
    font-size: 20px;
  }

  &.primary {
    background-color: blue;
    color: white;
  }
`;
```

`demo.js`

```js
import Button from './button.js'
function Demo(){
    return <Button></Button>
}
```

## 四、Shadow DOM

CSS Shadow DOM 是一种将样式隔离到 Web 组件内部的技术，它可以让组件内部的样式不会影响到外部的样式，从而实现前端微服务中的 CSS 样式隔离。

具体来说，Shadow DOM 允许开发人员创建一个封装了 HTML、CSS 和 JavaScript 的组件，这个组件内部的样式只会影响这个组件内部的元素，不会影响到其他组件或页面上的元素。这样可以避免样式之间的相互干扰，提高样式表的可维护性和可重用性。

使用 CSS Shadow DOM 可以通过以下步骤实现：

1.  创建一个 Shadow DOM：

```js
const shadowRoot = element.attachShadow({ mode: 'open' });
```

2.  在 Shadow DOM 中定义 HTML、CSS 和 JavaScript：

```js
const template = document.createElement('template');
template.innerHTML = `

  <style>
    /* Shadow DOM styles */
  </style>

  <div class="my-component">
    <!-- Shadow DOM HTML -->
  </div>
`;
shadowRoot.appendChild(template.content.cloneNode(true));
```

3.  在外部使用组件时，只需要引入组件的 JavaScript 文件即可：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>My App</title>
</head>
<body>
  <my-component></my-component>
  <script src="my-component.js"></script>
</body>
</html>
```

全局的样式还是会被主应用的样式覆盖，如果主应用的CSS样式权重更高的话。

## 五、CSS预处理器

原理：通过使用预处理器，将 CSS 编写成更加易于维护的代码，避免样式之间的相互干扰。

具体用法：常见的 CSS 预处理器有 Sass、Less、Stylus 等。例如，使用 Sass，可以将一个按钮的样式写成这样：

```css
$button-padding: 10px 20px;
$button-border-radius: 5px;

.button {
    display: inline-block;
    padding: $button-padding;
      border-radius: $button-border-radius;
    font-size: 16px;

&--large {
    font-size: 20px;
}

&--primary {
    background-color: blue;
    color: white;
}
```

## 六、CSS后处理器

### 1、postcss、autoprefixer

```css
/* 输入样式 */ 
.box { display: flex; } 
/* 经过处理后的样式 */ 
.box { 
    display: -webkit-box; 
    display: -ms-flexbox; 
    display: flex; 
}
```

### 2、postcss-modules

```css
/* 输入样式 */ 
.box { color: red; } 
/* 经过处理后的样式 */ 
.box_1e9v8z6 { color: red; }
```

webpack

```js
module.exports = {
    plugins: [
        require('autoprefixer')，
        new webpack.LoaderOptionsPlugin({ 
            options: { 
            postcss: [ 
                require('postcss-modules')() 
            ] 
        }})
    ]
}
```

### 3、在Vue中，只需要在style上添加`scoped`属性

```jsx
<template>
    <div class="container"> 
        <h1>这是一个组件</h1> 
        <p>这是组件的内容</p> 
    </div> 
</template> 
<style scoped> 
    .container { 
        background-color: #f5f5f5; 
        padding: 20px; 
    } 
    h1 { color: #333; } 
    p { color: #666; } 
</style>
```
