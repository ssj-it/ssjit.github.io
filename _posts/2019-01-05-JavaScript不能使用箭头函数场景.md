---
layout:     post
title:      JavaScript不能使用箭头函数场景
subtitle:   js中有时会有使用箭头函数异常情况 对此我们简单做一下分析
date:       2019-01-05
author:    Sunsj
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - JavaScript
---



定义方法的时候

 ① 定义字面量方法
```
const calculator = {
    array: [1, 2, 3],
    sum: () => {
        console.log(this === window); // => true
        return this.array.reduce((result, item) => result + item);
    }
};

console.log(this === window); // => true

// Throws "TypeError: Cannot read property 'reduce' of undefined"
calculator.sum();
```
calculator.sum 使用箭头函数来定义，但是调用的时候会抛出 TypeError，因为运行时 this.array 是未定义的，调用 calculator.sum 的时候，执行上下文里面的 this 仍然指向的是 window，原因是箭头函数把函数上下文绑定到了 window 上，this.array 等价于 window.array，显然后者是未定义的。

解决方法
```
const calculator = {
    array: [1, 2, 3],
    sum() {
        console.log(this === calculator); // => true
        return this.array.reduce((result, item) => result + item);
    }
};
calculator.sum(); // => 6
```

② 定义原型方法
```
function Cat(name) {
    this.name = name;
}

Cat.prototype.sayCatName = () => {
    console.log(this === window); // => true
    return this.name;
};

const cat = new Cat('Mew');
cat.sayCatName(); // => undefined

```
使用传统的函数表达式就能解决问题 JS Bin：

```
function Cat(name) {
    this.name = name;
}

Cat.prototype.sayCatName = function () {
    console.log(this === cat); // => true
    return this.name;
};

const cat = new Cat('Mew');
cat.sayCatName(); // => 'Mew'
```

sayCatName 变成普通函数之后，被调用时的执行上下文就会指向新创建的 cat 实例。



=== 定义事件回调函数

    箭头函数在声明的时候就绑定了执行上下文，要动态改变上下文是不可能的，在需要动态上下文的时候它的弊端就凸显出来。比如在客户端编程中常见的 DOM 事件回调函数（event listenner）绑定，触发回调函数时 this 指向当前发生事件的 DOM 节点，而动态上下文这个时候就非常有用，比如下面这段代码试图使用箭头函数来作事件回调函数 

```
const button = document.getElementById('myButton');
button.addEventListener('click', () => {
    console.log(this === window); // => true
    this.innerHTML = 'Clicked button';
});
```
在全局上下文下定义的箭头函数执行时 this 会指向 window，当单击事件发生时，浏览器会尝试用 button 作为上下文来执行事件回调函数，但是箭头函数预定义的上下文是不能被修改的，这样 this.innerHTML 就等价于 window.innerHTML，而后者是没有任何意义的。

使用函数表达式就可以在运行时动态的改变 this，修正后的代码 

```
const button = document.getElementById('myButton');
button.addEventListener('click', function() {
    console.log(this === button); // => true
    this.innerHTML = 'Clicked button';
});
```

定义构造函数

构造函数中的 this 指向新创建的对象，当执行 new Car() 的时候，构造函数 Car 的上下文就是新创建的对象，也就是说 this instanceof Car === true。显然，箭头函数是不能用来做构造函数， 实际上 JS 会禁止你这么做，如果你这么做了，它就会抛出异常。

换句话说，箭头构造函数的执行并没有任何意义，并且是有歧义的。比如，当我们运行下面的代码 JS Bin：

```
const Message = (text) => {
    this.text = text;
};
// Throws "TypeError: Message is not a constructor"
const helloMessage = new Message('Hello World!');
```
构造新的 Message 实例时，JS 引擎抛了错误，因为 Message 不是构造函数。在笔者看来，相比旧的 JS 引擎在出错时悄悄失败的设计，ES6 在出错时给出具体错误消息是非常不错的实践。可以通过使用函数表达式或者函数声明 来声明构造函数修复上面的例子 JS Bin：

```
const Message = function(text) {
    this.text = text;
};
const helloMessage = new Message('Hello World!');
console.log(helloMessage.text); // => 'Hello World!'

```