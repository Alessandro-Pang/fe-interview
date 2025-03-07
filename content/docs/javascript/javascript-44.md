---
weight: 3042000
date: '2025-03-07T08:39:47.304Z'
draft: false
author: Kismet
title: apply,call,bind异同
icon: javascript
toc: true
description: >-
  请说明apply,call,bind这三个方法有什么异同？
tags:
  - JavaScript
  - 函数方法
  - this指向
  - call
  - apply
  - bind
---

## 考察点分析

本题主要考察以下几个方面：
1. 对 JavaScript 中函数上下文（this 指向）的理解
2. apply、call、bind 三个方法的使用方式和应用场景
3. 函数式编程中的函数借用和函数柯里化概念
4. JavaScript 中的隐式绑定和显式绑定机制

---

## 技术解析

### 关键知识点

1. **this 关键字**：在 JavaScript 中，this 的指向在函数被调用时动态确定(严格模式下未指定时为undefined)，其值取决于函数的调用方式。

2. **Function.prototype 方法**：apply、call 和 bind 都是 Function.prototype 上的方法，所有函数都可以使用这些方法。

3. **显式绑定**：这三个方法都是用来显式指定函数执行时的 this 值，属于显式绑定的范畴。

### 原理剖析

**1. call 方法**
- 语法：`function.call(thisArg, arg1, arg2, ...)`
- 作用：立即调用函数，并指定 this 值和参数列表
- 参数传递方式：逐个传递参数

**2. apply 方法**
- 语法：`function.apply(thisArg, [argsArray])`
- 作用：立即调用函数，并指定 this 值和参数数组
- 参数传递方式：通过数组传递参数

**3. bind 方法**
- 语法：`function.bind(thisArg, arg1, arg2, ...)`
- 作用：返回一个新函数，该函数的 this 值被永久绑定到指定值
- 执行时机：不会立即调用函数，而是返回一个绑定了 this 的新函数
- 特点：支持函数柯里化（部分参数预设）
- 补充说明：通过bind绑定的this值具有最高优先级，即使对返回函数使用new运算符创建实例时也不会改变

### 常见误区

1. **混淆三者的执行机制（如将bind的返回值当作执行结果使用）**：虽然三者都用于改变 this 指向，但在执行时机和返回值上有本质区别。

2. **忽略 bind 的永久绑定特性**：bind 返回的新函数的 this 值已被永久绑定，即使使用 call、apply 或 new 运算符也无法再次改变。

3. **混淆参数传递方式**：call 需要逐个传递参数，而 apply 需要将参数放在数组中传递。

4. **忽略返回值差异**：call 和 apply 返回函数执行结果，而 bind 返回一个新函数。

---

## 问题解答

apply、call 和 bind 都是 JavaScript 中用于改变函数执行上下文（this 指向）的方法，它们的异同点如下：

**相同点：**
1. 都是 Function.prototype 上的方法，可以被所有函数对象调用
2. 都可以指定函数运行时的 this 指向
3. 都可以传递参数给目标函数

**不同点：**
1. **执行时机**：
   - call 和 apply 会立即执行函数
   - bind 不会立即执行函数，而是返回一个绑定了 this 的新函数

2. **参数传递方式**：
   - call 接受一系列参数列表：`fn.call(thisArg, arg1, arg2, ...)`
   - apply 接受一个参数数组：`fn.apply(thisArg, [arg1, arg2, ...])`
   - bind 接受一系列参数列表，并支持部分参数预设：`fn.bind(thisArg, arg1, arg2, ...)`

3. **返回值**：
   - call 和 apply 返回函数执行的结果
   - bind 返回一个新的函数，该函数的 this 被永久绑定到指定值

4. **绑定持久性**：
   - call 和 apply 只在当次调用中改变 this 指向
   - bind 创建的新函数 this 值被永久绑定，后续无法再更改

5. **原型链影响**：
   - call/apply 不会改变原函数的原型链
   - bind 返回的新函数会丢失原函数的prototype属性

---

## 解决方案

### 编码示例

```javascript
// 定义一个简单的函数
function greet(greeting, punctuation) {
  return greeting + ', ' + this.name + punctuation;
}

// 定义一个对象
const person = { name: '张三' };

// 使用 call
const result1 = greet.call(person, '你好', '!');
console.log(result1); // 输出: "你好, 张三!"

// 使用 apply
const result2 = greet.apply(person, ['Hello', '!!']);
console.log(result2); // 输出: "Hello, 张三!!"

// 使用 bind
const boundGreet = greet.bind(person, '嗨');
console.log(boundGreet('?')); // 输出: "嗨, 张三?"

// bind 的永久绑定特性
const person2 = { name: '李四' };
console.log(boundGreet.call(person2, '~')); // 输出: "嗨, 张三~"（this仍然指向原始绑定的person对象）

// 实际应用场景：借用数组方法
function convertArgs() {
  // arguments 是类数组对象，没有数组的方法
  const args = Array.prototype.slice.call(arguments);
  // 或使用 apply: const args = Array.prototype.slice.apply(arguments);

  /* ES6写法
    const args = Array.from(arguments);
    // 或
    const args = [...arguments];
  */

  return args.map(x => x * 2);
}

console.log(convertArgs(1, 2, 3)); // 输出: [2, 4, 6]

// 实际应用场景：函数柯里化
function multiply(x, y) {
  return x * y;
}

const double = multiply.bind(null, 2);
console.log(double(5)); // 输出: 10

// 箭头函数特性说明
const arrowFunc = () => console.log(this);
const boundArrow = arrowFunc.bind({name: 'test'});
boundArrow(); // 箭头函数的this无法被bind改变

```

---

## 深度追问
### 1. 如何实现 call、apply 和 bind 方法的 polyfill？

  ```javascript
    // call 方法的 polyfill 实现
    Function.prototype.myCall = function(context, ...args) {
      // 处理 null 或 undefined 的情况
      context = context || window;

      // 为上下文对象创建一个唯一的属性
      const uniqueKey = Symbol('uniqueKey');

      // 将当前函数设置为上下文对象的方法
      context[uniqueKey] = this;

      // 调用该方法并获取结果
      const result = context[uniqueKey](...args);

      // 删除临时属性
      delete context[uniqueKey];

      // 返回函数执行结果
      return result;
    };

    // apply 方法的 polyfill 实现
    Function.prototype.myApply = function(context, argsArray = []) {
      // 处理 null 或 undefined 的情况
      context = context || window;

      // 为上下文对象创建一个唯一的属性
      const uniqueKey = Symbol('uniqueKey');

      // 将当前函数设置为上下文对象的方法
      context[uniqueKey] = this;

      // 调用该方法并获取结果
      const result = context[uniqueKey](...argsArray);

      // 删除临时属性
      delete context[uniqueKey];

      // 返回函数执行结果
      return result;
    };

    // bind 方法的 polyfill 实现
    Function.prototype.myBind = function(context, ...args1) {
      // 保存原始函数的引用
      const originalFunction = this;

      // 返回一个新函数
      return function(...args2) {
        // 合并参数并使用 apply 调用原始函数
        return originalFunction.apply(context, [...args1, ...args2]);
      };
    };

    // 测试示例
    function greet(greeting, punctuation) {
      return greeting + ', ' + this.name + punctuation;
    }

    const person = { name: '张三' };

    console.log(greet.myCall(person, '你好', '!')); // 输出: "你好, 张三!"
    console.log(greet.myApply(person, ['Hello', '!!'])); // 输出: "Hello, 张三!!"

    const boundGreet = greet.myBind(person, '嗨');
    console.log(boundGreet('?')); // 输出: "嗨, 张三?"
  ```

### 2. 在什么情况下，使用 apply 比 call 更合适？反之亦然？

1. apply 更合适的场景：

    参数已存在于数组中 ：当函数参数已经以数组形式存在时，使用 apply 可以直接传入数组，无需解构。
    ```javascript
    // 使用 apply 将数组元素作为参数传递
    const numbers = [5, 6, 2, 3, 7];
    const max = Math.max.apply(null, numbers); // 等同于 Math.max(5, 6, 2, 3, 7)
    console.log(max); // 7

    // 如果使用 call，则需要解构数组
    const maxWithCall = Math.max.call(null, ...numbers);
    ```
    参数数量不确定 ：当处理可变参数函数时，特别是从其他地方接收参数数组的情况。
    ```javascript
      function doOperation(operation, values) {
        // 动态调用不同的数学函数
        return Math[operation].apply(null, values);
      }

      console.log(doOperation('min', [3, 1, 4, 1, 5])); // 1
    ```
2. call 更合适的场景：

   参数数量固定且较少 ：当参数数量确定且较少时，call 的语法更直观。
   ```javascript
     function Person(name, age) {
       this.name = name;
       this.age = age;
     }

     function Student(name, age, grade) {
       // 复用 Person 构造函数
       Person.call(this, name, age);
       this.grade = grade;
     }

     const student = new Student('李四', 18, '高三');
     console.log(student); // Student {name: '李四', age: 18, grade: '高三'}
   ```
   代码可读性 ：当参数较少且明确时，call 的语法更清晰，不需要创建临时数组。
   ```javascript
     // 使用 call 更直观
     function greet() {
       const fullName = this.firstName + ' ' + this.lastName;
       console.log(`Hello, ${fullName}!`);
     }

     const person = { firstName: '张', lastName: '三' };
     greet.call(person); // 输出: "Hello, 张 三!"
   ```
### 3. bind 方法如何实现函数柯里化？请举例说明

  函数柯里化是一种将接受多个参数的函数转换为一系列使用一个参数的函数的技术。bind 方法通过预设部分参数，可以很方便地实现函数柯里化。

  柯里化实现要点：
 - 参数预设：通过 bind 提前绑定部分参数
 - 延迟执行：返回新函数等待剩余参数
 - 组合调用：支持多次 bind 实现多级柯里化

  ```javascript
  // 动态柯里化函数
  function curry(fn, ...args) {
    return fn.length <= args.length
      ? fn(...args)
      : curry.bind(null, fn, ...args);
  }
  // 使用示例
  const add = curry((a, b, c) => a + b + c);
  console.log(add(1)(2)(3)); // 6 (1 + 2 + 3)

  // 使用 bind 进行柯里化
  const add5 = add.bind(null, 5); // 预设第一个参数为 5
  const add5and10 = add5.bind(null, 10); // 预设第二个参数为 10

  console.log(add5(10, 20)); // 输出: 35 (5 + 10 + 20)
  console.log(add5and10(15)); // 输出: 30 (5 + 10 + 15)

  // 实际应用：创建特定配置的函数
  function request(baseURL, endpoint, data) {
    console.log(`Sending ${data} to ${baseURL}${endpoint}`);
    // 实际请求逻辑...
  }

  // 为特定 API 创建预配置的请求函数
  const apiRequest = request.bind(null, 'https://api.example.com');
  const userApiRequest = apiRequest.bind(null, '/users');

  // 使用预配置的函数
  userApiRequest('{"id": 123}'); // 输出: "Sending {"id": 123} to https://api.example.com/users"

  // 实际应用：事件处理器
  function handleEvent(userId, eventType, event) {
    console.log(`处理用户 ${userId} 的 ${eventType} 事件`);
    // 事件处理逻辑...
  }

  // 为特定用户创建事件处理器
  const user123Handler = handleEvent.bind(null, '123');
  const user123ClickHandler = user123Handler.bind(null, 'click');

  // 添加事件监听器
  document.getElementById('button').addEventListener('click', user123ClickHandler);
  // 当点击按钮时，输出: "处理用户 123 的 click 事件"
  ```

### 4. 如何处理 call、apply 和 bind 方法在严格模式和非严格模式下的行为差异？

   严格模式和非严格模式下，当传入 null 或 undefined 作为 this 值时，这三个方法的行为有显著差异：
   ```javascript
    // 非严格模式下的行为
    function nonStrictFunc() {
      console.log(this);
    }

    // 在非严格模式下，null 和 undefined 会被替换为全局对象（浏览器中为 window）
    nonStrictFunc.call(null); // 输出: Window {...}
    nonStrictFunc.apply(undefined); // 输出: Window {...}
    const boundNonStrict = nonStrictFunc.bind(null);
    boundNonStrict(); // 输出: Window {...}

    // 严格模式下的行为
    function strictFunc() {
      'use strict';
      console.log(this);
    }

    // 在严格模式下，this 值就是传入的值，不会被替换
    strictFunc.call(null); // 输出: null
    strictFunc.apply(undefined); // 输出: undefined
    const boundStrict = strictFunc.bind(null);
    boundStrict(); // 输出: null

    // 处理这种差异的安全方式
    function safelyBindThis(func, thisArg, ...args) {
      // 如果 thisArg 为 null 或 undefined，使用空对象代替
      const safeThis = (thisArg === null || thisArg === undefined) ? Object.create(null) : thisArg;
      return func.bind(safeThis, ...args);
    }

    // 使用安全绑定函数
    const safeBound = safelyBindThis(nonStrictFunc);
    safeBound(); // 输出: {} (空对象，而不是全局对象)
   ```
### 5. 在实际开发中，这三个方法的常见应用场景有哪些？
  ```javascript
    // 1. 借用数组方法处理类数组对象
    function printArguments() {
      // 将类数组对象 arguments 转换为真正的数组
      const args = Array.prototype.slice.call(arguments);
      args.forEach(arg => console.log(arg));
    }

    printArguments(1, 2, 3); // 输出: 1, 2, 3

    // 2. 实现继承
    function Animal(name) {
      this.name = name;
    }

    Animal.prototype.speak = function() {
      return `${this.name} makes a noise.`;
    };

    function Dog(name, breed) {
      // 调用父类构造函数
      Animal.call(this, name);
      this.breed = breed;
    }

    // 设置原型链
    Dog.prototype = Object.create(Animal.prototype);
    Dog.prototype.constructor = Dog;

    // 重写方法
    Dog.prototype.speak = function() {
      return `${this.name} barks.`;
    };

    const dog = new Dog('旺财', '哈士奇');
    console.log(dog.speak()); // 输出: "旺财 barks."

    // 3. 函数上下文控制
    const user = {
      name: '张三',
      greet() {
        console.log(`你好，我是 ${this.name}`);
      }
    };

    // 在事件处理中保持正确的 this 指向
    const button = document.createElement('button');
    button.addEventListener('click', user.greet.bind(user));
    // 点击按钮时输出: "你好，我是 张三"

    // 4. 部分应用函数（Partial Application）
    function fetchData(baseURL, endpoint, callback) {
      const url = `${baseURL}${endpoint}`;
      // 模拟异步请求
      setTimeout(() => {
        callback(`Data from ${url}`);
      }, 100);
    }

    // 创建预配置的 API 请求函数
    const fetchFromAPI = fetchData.bind(null, 'https://api.example.com');

    // 使用预配置的函数
    fetchFromAPI('/users', data => console.log(data)); // 输出: "Data from https://api.example.com/users"
    fetchFromAPI('/posts', data => console.log(data)); // 输出: "Data from https://api.example.com/posts"

    // 5. 函数节流（Throttling）
    function throttle(func, delay) {
      let lastCall = 0;
      return function(...args) {
        const now = Date.now();
        if (now - lastCall >= delay) {
          lastCall = now;
          return func.apply(this, args);
        }
      };
    }

    // 创建节流函数
    const throttledScroll = throttle(function() {
      console.log('Scroll event handled', this.scrollY);
    }, 300);

    // 添加事件监听器
    window.addEventListener('scroll', throttledScroll);
  ```

