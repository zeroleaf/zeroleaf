---
title: js_动态调用
date: 2018-06-13 23:17:59
tags: [Javascript, Function]
---

Javascript 里面, 动态调用(绑定 this)的操作主要有如下3个函数

- `call`
- `apply`
- `bind`

下面分别对这3个函数进行详细的说明.

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
只不过起第一个参数是要绑定的 this 值而已.

值得注意的是, `call` 调用中关键的一个点是动态的改变 this 值, 但如果一个函数中没有用到 this 值,
传递任何给 this 值其执行结果都是相同的 (假设 function 是一个纯函数). 示例如下:

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

`apply` 方法的语法如下:

`function.apply(thisArg, [argsArray])`

在理解了 `call` 方法的基础上, `apply` 就相对简单了. 其与 `call` 基本类似, 区别在于,
`call` 的函数参数是逐个传递的, 而 `apply` 则统一为一个数组或者类数组对象. 举例来说:

```javascript
// 对于第一个代码的例子

zeroleaf.greeting.apply(sakura, ['Hello', 'And you?']);
// 输出: Hello, i am sakura, And you?
```

## bind

// TBD