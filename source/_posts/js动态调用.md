---
title: Javascript 动态调用
date: 2018-06-13 23:17:59
tags: [Javascript, Function]
---

Javascript 里面, 动态调用(绑定 this)的操作主要有如下3个函数

- `call`
- `apply`
- `bind`

下面分别对这3个函数进行详细的说明.

<!-- more -->

## call

`call` 函数的语法为:

`function.call(thisArg, arg1, arg2, ...)`

`call` 方法主要以指定的 this 值以及逐个提供的参数来调用某个函数.
举例来说, 

```javascript
let zeroleaf = {
    name: 'zeroleaf',

    greeting(greet, hint) {
        console.log(`${greet}, i am ${this.name}, ${hint}`)
    }
};

let sakura = {
    name: 'sakura'
};

zeroleaf.greeting('Hello', 'And you?');
// 输出: Hello, i am zeroleaf, And you?

zeroleaf.greeting.call(sakura, 'Hello', 'And you?');
// 输出: Hello, i am sakura, And you?
```

可以看到, 通过 `call` 方法, 可以动态的改变函数中的 this 值. `call` 调用与直接的函数调用类似,
只不过其第一个参数是要绑定的 this 值而已.

值得注意的是, `call` 调用中关键的一点是动态的改变 this 值, 但如果一个函数中没有用到 this 值,
则传递任意值作为 this 值其执行结果都是相同的 (假设 function 是一个纯函数). 示例如下:

```javascript
let calculator = {
    sum(a, b) {
        return a + b
    }
};

let pc = {};

console.log(calculator.sum(1, 2));

console.log(calculator.sum.call(pc, 1, 2));
console.log(calculator.sum.call(null, 1, 2));
console.log(calculator.sum.call(false, 1, 2));
// 输出: 上述所有的输出都为 3
```

## apply

`apply` 函数的语法如下:

`function.apply(thisArg, [argsArray])`

在理解了 `call` 方法的基础上, `apply` 就相对简单了. 其与 `call` 基本类似, 区别在于,
`call` 的函数参数是逐个传递的, 而 `apply` 则统一为一个数组或者类数组对象. 举例来说:

```javascript
// 对于第一个代码的例子

zeroleaf.greeting.apply(sakura, ['Hello', 'And you?']);
// 输出: Hello, i am sakura, And you?
```

## bind

`bind` 函数的语法如下:

`fun.bind(thisArg[, arg1[, arg2[, ...]]])`

与 `call`, `apply` 不同的是, 对 `bind` 函数的调用返回的是原函数基于指定的 this 值以及初始化参数改造的
原函数的拷贝, 而不是以不同的 this 值调用原函数.

举例来说:

```javascript
let zeroleaf = {
    name: 'zeroleaf',

    greeting(greet, hint) {
        console.log(`${greet}, i am ${this.name}, ${hint}`)
    }
};

let sakura = {
    name: 'sakura'
};

let helloFromSakura = zeroleaf.greeting.bind(sakura, 'Hello');

helloFromSakura('Nice to meet you!');
// 输出: Hello, i am sakura, Nice to meet you!
```

可以看到, 对 `bind` 函数的调用返回了原函数的拷贝, 除 this 值之外也将 greet 参数初始化为 'Hello',
这样, 返回的拷贝函数则是一个单参数 (hint) 的函数了, 也即偏函数的用法.

结合 `bind` 与 `apply` 或 `call`, 可以创建函数的 "快捷方式". 如:

```javascript
let arrayLikeObject = {
    0: 'zero',
    1: 'one',
    2: 'two',
    length: 3
};

let numbers = Array.prototype.slice.apply(arrayLikeObject);
console.log('numbers is Array?', Array.isArray(numbers));
// 输出: numbers is Array? true
console.log(numbers);
// 输出: [ 'zero', 'one', 'two' ]

let slice = Function.prototype.apply.bind(Array.prototype.slice);

numbers = slice(arrayLikeObject);
console.log('numbers is Array?', Array.isArray(numbers));
// 输出: numbers is Array? true
console.log(numbers);
// 输出: [ 'zero', 'one', 'two' ]
```

对于, 类数组对象, 我们通常会使用 `Array.prototype.slice.apply` 将其转换为一个数组(这里用 `call` 效果一样),
但是如果这样的用法很多的情况时, 可以考虑一个简化的 slice 函数, 具体为即:

`let slice = Function.prototype.apply.bind(Array.prototype.slice);`

相比较 `Array.prototype.slice.apply`, 上面的例子还是比较绕的, 因此这里展开如下:

1. 数组函数 `slice` 的原型为 `arr.slice(begin, end)`, 其中 begin, end 是可选的,
   都省略的情况下, 则 begin = 0, end = 数组末尾, 即浅拷贝这个数组
2. 理想情况, 调用 `arrayLikeObject.slice()` 即可, 但是类数组对象是不支持 slice 函数的, 因此采用迂回的方式,
   将类数组对象作为 this 值调用 slice: `Array.prototype.slice.apply(arrayLikeObject)`
3. 简化, 去掉 prototype 之后为 `slice.apply(arrayLikeObject)`, 对这个函数调用来说, 
   this 值为 `slice`, 调用函数为 `apply`, 参数为 arrayLikeObject
4. 由于调用 `bind` 函数可以返回一个函数重新绑定了 this 值(或者参数值)之后的拷贝函数, 
   因此可以使用 `bind` 固化 this 值, 即 `apply.bind(slice)`
   
## 箭头函数

由于箭头函数没有 this 值(其 this 值是作用域的上一级 this, 因此在函数声明的时候相当于 this 值已经固定), 
因此也就不能重新绑定起 this 值, 因此 `call`, `aplly`, `bind` 中的 this 值绑定对其无效:

```javascript
let zeroleaf = {
    name: 'zeroleaf',

    greetingCallback() {
        return (greet, hint) => {
            return console.log(`${greet}, i am ${this.name}, ${hint}`)
        }
    }
};

let sakura = {
    name: 'sakura'
};

let zeroleafGreeting = zeroleaf.greetingCallback();

zeroleafGreeting('Hello', 'Nice to meet you!');
// 输出: Hello, i am zeroleaf, Nice to meet you!
zeroleafGreeting.apply(sakura, ['Hello', 'Nice to meet you!']);
// 输出: Hello, i am zeroleaf, Nice to meet you!
zeroleafGreeting.call(sakura, 'Hello', 'Nice to meet you!');
// 输出: Hello, i am zeroleaf, Nice to meet you!

let helloFromSakura = zeroleafGreeting.bind(sakura, 'Hello');
helloFromSakura('Nice to meet you!');
// 输出: Hello, i am zeroleaf, Nice to meet you!
```

## 总结

可以重新绑定 this 值, 给予了 JS 更大的灵活性(当然同时也更复杂了), 当前能想到的作用有:

- **this 对象的契约设计**, 满足指定条件的对象(类似面向对象中的接口)都可以作为参数调用给定的方法
- **程序的模块化**, 将整体中其中一块功能独立出来实现, 在调用的时候使用 this 绑定, 从而避免整体的实现太过于复杂. 类似于 PHP 中的 `trait`.
  
  举例来说, [plyr](https://github.com/sampotts/plyr) 的主体 plyr 播放器中包含有 **captions**(字幕??)子组件, 
  而其在 plyr.js 中对 captions 部分的调用方式为:
  
  `captions.set.call(this, input);` 
  
  即将 captions 的功能独立出来而在调用的时候再绑定 this 值以简化 plyr 的实现.