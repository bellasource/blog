---
title: ES6+常用语法
date: 2021/02/26
tags: ["ES6", "class", "Promise", "Iterator", "Generate"]
---

## 严格模式"use strict"

ES5 新增严格模式，使得 Javascript 在更严格的语法条件下运行，区别

- 变量必须先声明，才能使用
  - 在正常模式中，如果一个变量没有声明就直接赋值，默认为全局变量
- 全局作用域函数中的 this 指向 undefined，
- 创建 eval 作用域
- script 标签 type='module'默认开启严格模式

## JSON

ES5 新增 JSON,json 是一种轻量级数据交换格式，可以实现数组、对象深拷贝

- JSON.stringify()
  - 将 js 数据转为 JSON 字符串
- JSON.parse()
  - 将 JSON 字符串转为 js 数据类型

## let const 和块作用域

- 作用域

  - 函数作用域：ES5 中，作用域是通过函数来区分
  - 块作用域：使用{}括起来的区域，防止变量污染,if 和 for 循环内的{}也属于块作用域

- 变量声明的方式
  - var：变量值可以修改，有变量提升，浏览器兼容性好，声明的全局变量属于 window 的属性；
  - let：变量值可以修改，没有变量提升，声明的变量只在 let 所在代码块内有效(增加了块级作用域)
  - const：变量值不可以修改，没有变量提升，声明的变量只在 const 所在代码块内有效(增加了块级作用域)

```js
// var声明
var b = 1;
fn1();
function fn1() {
  console.log(b); //undefined
  if (true) {
    var b = 2; //变量提升
  }
  console.log(b, window.b); //2 1
}

//let声明

let b = 1;
fn1();
function fn1() {
  console.log(b); //1
  if (true) {
    //只在当前的块作用域中生效
    let b = 2;
  }
  console.log(b, window.b); //1 undefined
}

// let 申明的变量，只在当前作用域中生效
function fn() {
  for (let i = 0; i < 5; i++) {
    setTimeout(() => console.log(i), 1000); // 1 2 3 4 5
  }

  for (var i = 0; i < 5; i++) {
    setTimeout(() => console.log(i), 1000); // 5 5 5 5 5
  }
}
```

## 解构赋值及展开运算符

从数组和对象中提取值，对变量进行赋值

```js
//解构赋值
const obj = {
  name: "laoli",
  childobj: {
    name: "xiaoli",
    age: 18,
  },
};
const {
  childobj: { name = "unknow" }, //设置解构默认值
} = obj;
console.log(name); //xiaoli

//展开运算符
const newObj = { ...obj, name: "dali" };
```

## 运算符扩展

- 空合并运算符 ??
- 可选链操作符 ?
- 逻辑赋值运算符 ||= &&= ??=

```js
const fn = (name, age) => {
  // name = name || "xiaofang";
  // age = age || 18;
  // console.log(name, age); //xiaoming 18
  name = name ?? "xiaofang";
  age = age ?? 18;
  console.log(name, age); //xiaoming ""
};
fn("xiaoming", "");
//------------------------------------
const obj = {
  child: {},
};
console.log(obj.child?.big?.name); //无属性返回undefined
console.log(obj.child.big.name); //报错
//------------------------------------
const user = {};
user.id ||= 1; //等价 user.id || user.id=1
user.name &&= "xiaming"; //等价 user.name && user.name="xiaming"
```

## 数据结构 map set

### map

map 对象，传统的 Object 键名只能是字符串，map 对象的键值可以是任意值

```js
let obj1 = {
  id: 1,
};
let mp1 = new Map([
  [obj1, 1],
  ["b", 2],
  ["c", 3],
]);
console.log(mp1.get(obj1)); //1
// 可以通过for of遍历
for (let i of mp1) {
  console.log(i);
}
```

- 实用场景

```js
const status = 1;
let operation = new Map([
  [1, "启用"],
  [2, "停用"],
  [3, "注销"],
]);
text = operation.get(status);
```

```html
<!-- 
  let dataList = new Map([['name':'xiaoli'],['age',18]])
 -->
<div v-for="[key, value] in dataList" :key="key">
  <span>{{ key }}:</span>
  <span>{{ value }}</span>
</div>
```

size:属性返回 Map 结构的成员总数。
set():设置键名 key 对应的键值为 value，返回 map 对象
get():读取 key 对应的键值，如果找不到 key，返回 undefined。
has():返回一个布尔值，表示某个键是否在当前 Map 对象之中。
delete():删除某个键，返回 true。如果删除失败，返回 false。
clear():清除所有成员，没有返回值。
keys()：返回键名的遍历器。
values()：返回键值的遍历器。
entries()：返回所有成员的遍历器。
forEach()：遍历 Map 的所有成员。

### set

类似于数组，但是成员的值都是唯一的，没有重复的值,可用于数组去重

```js
let st = new Set([1, 2, 3, 4, 4, 3, 3, 1, 5]);
[...new Set("ababbc")].join(""); // "abc"
```

size() :返回 Set 的长度
add(): 添加某个值，返回 Set 结构本身。
delete(): 删除某个值，返回一个布尔值，表示删除是否成功。
has(): 返回一个布尔值，表示该值是否为 Set 的成员
clear(): 清除所有成员，没有返回值。
keys()：返回键名的遍历器
values()：返回键值的遍历器
entries()：返回键值对的遍历器
forEach():遍历

## 数据类型 Symbol Bigint

### Symbol

用来表示独一无二的值，可以当做对象的 key，避免对象属性的冲突,

```js
let name = Symbol("name");
let obj = {
  [name]: "小王",
};
console.log(obj[name]);
//获取对象的所有Symbol键组成的数组
Object.getOwnPropertySymbols(obj);
```

### Bigint

js 的 number 精度为 53 个二进制位（16 个十进制位），大于的会返回 Infinity，造成计算不精确
Bigint 类型在数值后加 n

```js
// 超过 53 个二进制位的数值，无法保持精度
Math.pow(2, 53) === Math.pow(2, 53) + 1; // true

// 超过 2 的 1024 次方的数值，无法表示
Math.pow(2, 1024); // Infinity

const a = 2172141653n;
const b = 15346349309n;
// BigInt 可以保持精度
a * b; // 33334444555566667777n
```

## Proxy

构造函数，用于创建一个对象的代理，外界对该对象的访问，都必须先通过这层拦截，可以在代理中进行过滤和改写。类似于代理模式

### 语法

new Proxy(target, handler)

- target：需要使用 Proxy 包装的目标对象
- handler：以函数作为属性值的对象，对应不同的代理行为

  - receiver：原始的操作行为所在的对象，一般情况下是 proxy 实例本身

```js
const targetObj = {
  age: 18,
};
const obj = new Proxy(targetObj, {
  get: function (target, propKey, receiver) {
    console.log(target === targetObj, receiver === obj, receiver === target); //true,true,false
    return Reflect.get(target, propKey, receiver);
  },
  set: function (target, propKey, value, receiver) {
    return Reflect.set(target, propKey, value, receiver);
  },
});
// 要想拦截生效，必须通过操proxy实例，而不能直接操作被代理的目标对象

obj.name = "xiaoli"; //触发set方法
obj.name; //触发get方法
```

### get()

拦截某个属性的读取操作，get 方法可以继承。

- 参数
  - target:代理的目标对象;
  - propKey:获取或设置的属性名；
  - value：设置的属性值；

```js
const targetObj = {
  age: 18,
};
const obj = new Proxy(targetObj, {
  get: function (target, propKey, receiver) {
    if (target[propKey]) {
      return target[propKey];
    }
    throw new Error('Prop name "' + propKey + '" does not exist.');
  },
});
obj.name;
```

将拦截操作定义在 prototype 对象上，当读取 sourceObj 对象的属性值，没有，则顺着**proto**查找，同样会触发 proxy 的 get 拦截

```js
let sourceObj = Object.create(obj);
sourceObj.age; //18
sourceObj.name; //报错
```

### set()

用于拦截某个属性的赋值操作
应当返回布尔值，true 表示设置成功，严格模式下返回 false 会报错

- 参数
  - target:代理的目标对象;
  - propKey:获取或设置的属性名；
  - value：设置的属性值；
  - receiver：最初被调用的对象，一般情况下是 proxy 实例本身。但是 handler 的 set 方法有可能在原型链上
    例如：一段代码 obj.name = 'lili',obj 不是一个 proxy，并且自身不含 name 属性，但是 obj 原型链上有一个 proxy，则 proxy 的 set（）拦截器会被调用，此时 receiver 为 obj

```js
const proxy = new Proxy(
  {},
  {
    set: function (obj, prop, value, receiver) {
      obj[prop] = receiver;
      return true;
    },
  }
);
const myObj = {};
Object.setPrototypeOf(myObj, proxy);

myObj.foo = "bar";
myObj.foo === myObj; // true
```

若目标属性是一个不可写及不可配置的数据属性，则不能改变它的值

```js
const targetObj = {};
Object.defineProperty(targetObj, "name", {
  value: "lili",
  writable: false,
});

const obj = new Proxy(targetObj, {
  set: function (target, propKey, value, receiver) {
    target[propKey] = value;
    return true;
  },
});
obj.name = "xiaoming"; //报错
```

有时，会在对象上添加可读不可修改的属性，通过 Proxy 可以很方便的实现

```js
const targetObj = {
  _id: 1,
  age: 18,
};
const obj = new Proxy(targetObj, {
  set: function (target, propKey, value, receiver) {
    // 设置的属性含_,则报错
    if (propKey[0] === "_") {
      throw new Error(`Invalid attempt to set private "${key}" property`);
    }
    target[propKey] = value;
    return true;
  },
});
obj.age = 20;
obj._id = 2;
```

### apply()

用于拦截函数的调用

- 参数
  - target:代理的目标对象、函数
  - thisArg：被调用时的上下文对象
  - args：被调用时的参数数组

```js
function sum(a, b) {
  return a + b;
}
const handler = {
  apply: function (target, thisArg, args) {
    return target(...args) * 10;
  },
};
const proxy = new Proxy(sum, handler);
console.log(sum(1, 2), proxy(1, 2)); //3 30
```

### has()

判断对象是否具有某个属性的方法，主要针对于 in 操作符

- 参数
  - target:代理的目标对象、函数
  - key：需要被判断的属性名

```js
const handler = {
  has(target, key) {
    if (key[0] === "_") {
      return false;
    }
    return key in target;
  },
};
const monster = {
  _secret: "easily scared",
  eyeCount: 4,
};
// 如果代理对象被禁止扩展，则使用has拦截会报错
// Object.preventExtensions(monster);
const proxy = new Proxy(monster, handler);
console.log("_secret" in proxy, "eyeCount" in proxy, "_secret" in monster); //false true true
```

### construct()

拦截 new 操作符的方法

```js
const Person = function (name, age) {
  this.name = name;
  this.age = age;
};
const handler = {
  construct: function (target, args, newTarget) {
    return new target(...args);
  },
};
const proxy = new Proxy(Person, handler);
new proxy("lili", 18);
```

### defineProperty()

拦截 Object.defineProperty(),defineProperties()方法，该方法必须返回一个 boolean，true 表示属性添加成功。

- 参数
  - target:代理的目标对象、函数
  - key：需要被定义的属性名
  - descriptor：属性配置项

```js
var handler = {
  defineProperty(target, key, descriptor) {
    return Reflect.defineProperty(target, key, descriptor);
  },
};
var target = {};
var proxy = new Proxy(target, handler);
proxy.bar = "bar"; // 不会生效
Object.defineProperty(proxy, "foo", {
  value: function () {},
  configurable: true,
  writable: true,
  enumerable: true,
});
```

### deleteProperty()

拦截 delete 对象属性操作,如果目标对象的属性是不可配置的，那么该属性不能被删除

- 参数
  - target：目标对象
  - prop：需要删除的属性名

```js
var target = { name: "lili", age: 12 };
var proxy = new Proxy(target, {
  deleteProperty: function (target, prop) {
    return Reflect.deleteProperty(target, prop);
  },
});
delete proxy.name;
```

### getOwnPropertyDescriptor()

用于拦截 Object.getOwnPropertyDescriptor，返回属性的描述对象

- 参数
  - target：目标对象
  - prop：需要获取的属性名

```js
var target = {
  name: "lili",
};
var p = new Proxy(target, {
  getOwnPropertyDescriptor: function (target, prop) {
    return Reflect.getOwnPropertyDescriptor(target, prop);
  },
});
Object.getOwnPropertyDescriptor(p, "name"); //{configurable:true,enumerable:true,value:"lili"}
```

### ownKeys()

- 以下四种方式会触发拦截
  Object.getOwnPropertyNames()
  Object.getOwnPropertySymbols()
  Object.keys()
  for...in 循环

```js
var p = new Proxy(
  { name: "lili", age: "xiaoming" },
  {
    ownKeys: function (target) {
      console.log("called");
      return Reflect.ownKeys(target);
    },
  }
);
Object.getOwnPropertyNames(p); // [name,age]
```

### getPrototypeOf()

当读取代理对象原型时，会被触发

- 五种方式下会触发
  - Object.getPrototypeOf(p)
  - Reflect.getPrototypeOf(p)
  - p.**proto**
  - Array.prototype.isPrototypeOf(p)
  - p instanceof Array

```js
const monster1 = {
  eyeCount: 4,
};

const monsterPrototype = {
  eyeCount: 2,
};

const proxy1 = new Proxy(monster1, {
  getPrototypeOf(target) {
    return Reflect.getPrototypeOf(target);
  },
});
console.log(Object.getPrototypeOf(proxy1) === monsterPrototype); //true
```

### setPrototypeOf()

拦截 Object.setPrototypeOf()和 Reflect.setPrototypeOf()

## Reflect

内置对象，提供拦截 js 操作的方法，与 Proxy 类似，提供了 13 种静态方法

Reflect.apply(target, thisArg, args)
Reflect.construct(target, args)
Reflect.get(target, name, receiver)
Reflect.set(target, name, value, receiver)
Reflect.defineProperty(target, name, desc)
Reflect.deleteProperty(target, name)
Reflect.has(target, name)
Reflect.ownKeys(target)
Reflect.isExtensible(target)
Reflect.preventExtensions(target)
Reflect.getOwnPropertyDescriptor(target, name)
Reflect.getPrototypeOf(target)
Reflect.setPrototypeOf(target, prototype)

## 字符串扩展

### 模板字符串

模板字符串用反引号（`）标识。可以嵌套变量，可以换行，可以包含单引号和双引号。
模板字符串中嵌入变量，需要将变量名写在${}之中。
大括号内部可以放入任意的 JavaScript 表达式，可以进行运算，以及引用对象属性。

```js
const oApp = document.querySelector("#app");
const info = { name: "xiaoli", age: 18 };
const str1 = `${info.name}-${info.age}`;
oApp.innerHTML = str1;
```

## 数值的扩展

### 二进制和八进制

ES6 中明确了二进制（0b/0B\）和八进制（0o/0O）数值的新写法

```js
let num = 503;
0b111110111 === num;
0o767 === num;
parseInt(0b111110111, 2); //将任意进制的数转为10进制的数
parseInt(0x111110111);
parseInt(767, 8), //将八进制的767转为10进制的数
  num.toString(2); //"111110111",将num转为2进制的数的字符串
```

### 数值分隔符

较长的数值允许每三位添加一个分隔符（\_），增加数值的可读性

```js
let num = 1_234_567;
console.log(num === 1234567);
```

## 对象的扩展

- 对象属性简写:当对象属性名和属性值变量名一致，可简写
- 属性名表达式：表达式作为对象的属性，需要用[]括起来

```js
// 简写
const name = "xiaoli";
const obj1 = { name }; //等价于 obj1 = {name:name}

// 表达式
obj1[name] = 18;
```

## 函数扩展

### 函数参数默认值

```js
// ES6之前
function fn1(a, b) {
  if (!b) {
    b = "world";
  }
  console.log(a, b);
}
// ES6
function fn2(a, b = "world") {
  console.log(a, b);
}
```

### rest 参数

用于获取函数的多余参数

```js
function fn(name, ...rest) {
  console.log(name, rest);
}
fn("xiaoming", 18, "nan"); //xiaoming [18, 'nan']
```

### 箭头函数

- 箭头函数没有自己的 this，内部的 this 指向定义时所处的对象，即定义时的执行上下文
- 箭头函数不能用于构造函数，也就是不能使用 new 关键字调用
- 箭头函数内部没有 argument 对象,可以通过...args 接收参数
- 箭头函数不能使用 yield 命令，不能当作 gennerator 函数

```js
const fn = () => {};
// 普通函数，谁调用内部this指向谁
document.onclick = function () {
  console.log(this); //document
  let f = () => {
    console.log(this); //document
  };
  let b = function () {
    console.log(this); //window
  };
  f();
  b();
};

let obj = {
  do: () => {
    console.log(this); // this指向window
  },
  show: function () {
    console.log(this); // this指向obj对象
  },
};
obj.do();
obj.show();
```

### 尾调用优化

尾调用：函数执行的最后一步是调用另一个函数，

尾调用是函数执行最后一步，不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以。
这将大大减少调用栈，节省内存

非尾调用造成原因：

- 每个函数在调用另一个函数的时候，并没有 return 该调用，所以 JS 引擎会认为还没有执行完，会保留你的调用帧

```js
// inner用到了外部函数的变量，不属于尾调用
function addOne(a) {
  var one = 1;
  function inner(b) {
    return b + one;
  }
  return inner(a);
}

// 不属于尾调用
function f(x) {
  let y = g(x);
  return y;
}

// 调用函数后还有运算，不属于尾调用
function f(x) {
  return g(x) + 1;
}

// 调用函数后 return undefined,不属于尾调用
function f(x) {
  g(x);
}
```

- 尾递归：在函数的末尾，调用自身
  尾递归不会保存之前的调用栈，因此不会存在内存溢出的风险

- 斐波那契数列进行尾递归优化

```js
function getFibo(n) {
  if (n <= 1) return 1;
  return getFibo(n - 1) + getFibo(n - 2);
}
console.log(getFibo(15));

function getFibo2(n, a1 = 1, a2 = 1) {
  if (n <= 1) return a2;
  return getFibo2(n - 1, a2, a1 + a2);
}
console.log(getFibo2(15));
```

- 手动将递归改为循环

- 函数内部改为循环的形式

```js
// 函数内部改为循环的形式
function getFibo3(n, a = 1, b = 1) {
  while (n--) {
    [a, b] = [b, a + b];
  }
  return a;
}
```

- 蹦床函数

```js
// 蹦床函数
function trampoline(f) {
  while (f && f instanceof Function) {
    f = f();
  }
  return f;
}
function getFibo4(n, a = 1, b = 1) {
  if (n > 0) {
    [a, b] = [b, a + b];
    return getFibo4.bind(null, n - 1, a, b);
  } else {
    return a;
  }
}
trampoline(getFibo4(15));
```

- 尾递归优化包装函数:此方法不需要改写原函数

```js
function tco(f) {
  var value;
  var active = false;
  var accumulated = [];

  return function accumulator() {
    accumulated.push(arguments);
    if (!active) {
      active = true;
      while (accumulated.length) {
        value = f.apply(this, accumulated.shift());
      }
      active = false;
      return value;
    }
  };
}
var sum = tco(function (n, a1 = 1, a2 = 1) {
  if (n <= 1) return a2;
  return sum(n - 1, a2, a1 + a2);
});

console.log(sum(1000000));
```

### 对象、数组、字符串等的新增方法

ES6 模块化在另一篇文章{% post_link 模块化及模块化规范 模块化及模块化规范 %}中有详细说明。

## class 类

ES5 及以前，生成实例对象的传统方法是通过构造函数

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}
Person.prototype.info = function () {
  console.log(`${this.name}-${this.age}`);
};
const person = new Person("xiaofang", 12);
person.info();
```

ES6 引入 class 类的概念，用于定义模板对象

- 类必须有 constructor，如果没有显式定义，会默认添加一个空的
- static 关键字，表示为构造函数的静态方法，实例对象无法访问
- #修饰符，实例无法直接获取的属性

```js
class Person {
  // name = 'xiaoming'; //实例对象的属性
  #sex = "nv"; //实例私有属性，无法通过实例获取
  constructor(name) {
    this.name = name;
  }
  // Person.prototype的属性
  getSex() {
    // 通过共有方法获取私有属性
    console.log(this.#sex);
  }
  setName(name) {
    this.name = name;
  }
  static do() {
    console.log("do");
  }
}
const person = new Person("xiaofang");
person.getSex(); // nv
console.log(person.#sex); //语法错误
```

- static 的应用扩展
  需求：一个类只能实例一次，如果有实例，则返回第一次的实例对象

```js
class Person {
  static init;
  constructor(name) {
    if (Person.init) {
      return Person.init;
    }
    Person.init = this;
    this.name = name;
  }
}
new Person("xiaofang"); //{name:'xiaofang'}
new Person("xiaoli"); //{name:'xiaofang'}
```

- extends 继承

```js
// 定义父类，基类
class Person {
  constructor(name) {
    if (new.target === "Person") {
      throw new Error("基类不允许被new");
    }
    this.name = name;
  }
  do() {
    console.log("do");
  }
}
class Child extends Person {
  constructor(name, age) {
    super(name); //super用于调用父类constructor,即Person.prototype，继承父类的this对象
    this.age = age;
  }
  showdo() {
    super.do(); //Person.prototype
  }
}
new Child("xiaoming", 12).showdo();
```

## ES6 模块化

ES6 模块化在另一篇文章{% post_link 模块化及模块化规范 模块化及模块化规范 %}中有详细说明。

## Promise

- Promise 是实现异步编程的一种解决方案，将异步操作以同步流程表达，解决回调函数层层嵌套的问题。

- Promise 是一个构造函数，自己身上有 all,resolve,reject,race 等方法，其 prototype 上有 then，catch，finally 方法

- Promise 对象有三种状态，pedding,resolved,rejected,状态只能修改一次

```js
const promise = new Promise((resolve, reject) => {
  //Promise的回调函数同步执行
  resolve("a数据");
  // 调用resolve,reject改变状态后，后续代码会继续执行
  console.log("后续代码");
  return { name: "xiao" }; //new Promise返回值与return值无关
});
```

### Promise.prototype 的 then，catch

- 用于捕获 promise 成功或者失败状态，其回调接收一个参数
- then/catch 方法返回一个新的 promise 对象，所以可以链式调用 then/catch

  - 新 promise 对象默认是成功状态
  - 如果内部函数执行报错，返回失败状态的 promise
  - 如果返回 promise 对象，则 then/catch 的返回值为这个 promise 对象
  - 如果返回其他类型的值，则会把该值包装为 promise 对象

  ```js
  //then的两种写法
  promise
    .then(
      (success) => {
        console.log("成功的then");
        //return{age:18}
        //return Promise.resolve()
      },
      (error) => {
        console.log("失败的then");
      }
    )
    .catch((error) => {
      console.log("失败的catch");
    })
    .then((success) => {
      console.log(success, "成功的then");
    });
  ```

### Promise.prototype 的 finally

- 不管 promise 的最终状态是成功还是失败，都会执行 finally。
- 其回调不接收任何参数
- finally 总会返回传入的 promise，与内部 return 的值无关

```js
promise
  .then((success) => {
    console.log("成功的then", success);
    return 18;
  })
  .finally(() => {
    // finally的返回值与return的值无关
    return Promise.resolve("12");
  });
```

### Promise.resolve .reject

- Promise.resolve() 用于创建成功状态的 promise
- Promise.reject() 用于创建失败状态的 promise
  - 若内部参数为 promise 对象，则返回该参数
  - 若内部参数为非 promise 值，则返回成功状态的 promise
  - 若内部未传递参数，则 promise 对象的值为 undefined

```js
Promise.resolve({ age: 18 });
// 等价于
new Promise((res, rej) => {
  res({ age: 18 });
});
```

### Promise.race .all .allSettled .any

用于将多个 promise 对象包装为一个新的 promise 对象

- Promise.all() 只有所有 promise 都成功，才返回成功状态,有一个失败，则返回失败的值

  - 通常用于发送多个互相无关联的请求时可用 all 方法，统一管理响应值

- Promise.race() 第一个改变状态的 promise 即为 Promise.race 的值

  - 可用于当请求超时，改变 promise 的状态

- Promise.allSettled() 只有当所有 promise 状态都发生变化，才返回成功状态的 promise

- Promise.any() 有一个成功则返回成功的 promise，否则返回失败的 promise

```js
const promise1 = new Promise((res, rej) =>
  setTimeout(() => res("promise1"), 2000)
);
const promise2 = new Promise((res, rej) =>
  setTimeout(() => rej("promise2"), 4000)
);
Promise.all([promise1, promise2])
  .then((res) => {
    console.log(res, "成功res");
  })
  .catch((err) => {
    // err为失败的promise的值
    console.log(err, "err");
  });
Promise.race([promise1, promise2])
  .then((res) => {
    console.log(res, "成功res");
  })
  .catch((err) => {
    console.log(err, "err");
  });
Promise.allSettled([promise1, promise2]).then((res) => {
  console.log(res, "res");
  /* 
      [
        {"status": "fulfilled","value": "promise1"},
        {"status": "rejected","reason": "promise2"}
      ] 
    */
});
Promise.any([promise1, promise2])
  .then((res) => console.log(res, "成功then")) //promise1
  .catch((err) => console.log(err, "err")); //如果全部失败，err为All promises were rejected "err"
```

## Iterator 遍历器和 for of

- 遍历器为各种不同的数据结构提供统一个访问机制 for of
- 原生具有 Iterator 接口的数据结构有:Array,Map,Set,String,函数的 arguments 对象，nodeList
- Object 类型无法确定哪个属性应该先被遍历，哪个属性应该后被遍历，所以无 iterator 接口
- 作用：为各种数据结构，提供统一个访问接口；使得数据结构的成员能按某种次序排列；有 iterator 接口的数据都可以通过 for of 来遍历

- 应用场景：解构赋值、扩展运算符

实现 iterator

```js
let it = makeIterator(["a", "b"]);
it.next();
it.next();
it.next();
function makeIterator(array) {
  // 闭包，保存下标
  let currentIndex = 0;
  return {
    next() {
      if (currentIndex < array.length) {
        return { value: array[currentIndex++], done: false };
      } else {
        return { value: undefined, done: true };
      }
    },
  };
}

let arr = [1, 2, 3, 4];
for (let value of arr) {
  console.log(value, "value");
}
```

## Generate 函数 yeild

- 异步编程解决方案,执行 Generate 函数会返回一个遍历器对象,但 generator 代码内部不会马上执行，需要调用 next 方法的时候才会执行
- yield 语句返回结果通常为 undefined， 当调用 next 方法时传参内容会作为启动时 yield 语句的返回值。
- 在 generator 函数中，遇到 yield 就会停止，直到运行 next

```js
function* fn() {
  yield setTimeout(() => {
    console.log("a数据成功了");
    iteratorObj.next(); // 执行下一段代码
  }, 1000);

  yield setTimeout(() => {
    console.log("b数据成功了");
    iteratorObj.next(); // 执行下一段代码
  }, 2000);

  yield setTimeout(() => {
    console.log("c数据成功了");
    iteratorObj.next(); // 执行下一段代码
  }, 3000);

  console.log("全部数据请求回来~");
}

const iteratorObj = fn();
iteratorObj.next();
```

### yield 表达式

- 遇到 yield 表达式，就暂停执行后面的操作，并将紧跟在 yield 后面的那个表达式的值，作为返回的对象的 value 属性值。
- 下一次调用 next 方法时，再继续往下执行，直到遇到下一个 yield 表达式。
- 如果没有遇到新的 yield 表达式，就一直运行到函数结束，直到 return 语句为止，并将 return 的值，作为返回的对象的 value 属性值。

### next()参数

yield 表达式本身没有返回值，默认为 undefined；next 传递的参数会当做上一次 yeild 表达式的返回值

```js
function* f() {
  for (var i = 0; true; i++) {
    var reset = yield i; //默认reset为undefined
    if (reset) {
      i = -1;
    }
  }
}
var g = f(); //返回迭代器对象
console.log(g.next()); // { value: 0, done: false }
console.log(g.next()); // { value: 1, done: false }
console.log(g.next(true)); // { value: 0, done: false }
```

### for of

for...of 循环可以自动遍历 Generator 函数运行时生成的 Iterator 对象，且此时不再需要调用 next 方法

```js
function* gen() {
  yield "状态1";
  yield "状态2";
  yield "状态3";
  yield "状态4";
}
let it = gen();
for (let i of it) {
  console.log(i, "i");
}
```

### 实现对象的 iterator 接口

```js
function* objectEntries() {
  let propKeys = Object.keys(this);
  for (let propKey of propKeys) {
    yield [propKey, this[propKey]];
  }
}
let jane = { first: "Jane", last: "Doe" };

jane[Symbol.iterator] = objectEntries;

for (let [key, value] of jane) {
  console.log(`${key}: ${value}`);
}
```

## async 函数 await

- Generator 函数的语法糖
- async 与 generate 的区别

  - \*变为 async，yeild 变为 await
  - async 函数返回 promise 对象，generate 函数返回迭代器对象
  - async 内置执行器，不需要自己调用 next()
    **常规使用**

- await 的返回值

  - await 后不是 promise 对象，不会阻塞程序执行
  - await 后为 promise 对象，会阻塞后面的代码，
    - 等 promise 对象最终状态改为成功，得到成功的值，程序继续执行；
    - 如果是失败的状态，则 await 无返回值，且退出当前 async 函数

- async 函数的返回值

  - 如果在函数中 return 一个直接量，async 会把这个直接量通过 Promise.resolve() 封装成 Promise 对象
  - 如果在函数中 return promise 对象，则 async 返回该对象的拷贝
  - 如果 async 函数内部出错（1.正常报错，2.await 等到失败的 promise 对象），则 async 返回失败的 promise

- try...catch()接收错误

```js
async function fn() {
  // await后为失败的promise，函数不会继续向下执行
  try{
    await new Promise((res, rej) => {
    res("数据1");
  }).then(() => {
    throw new Error();
  });
  await new Promise((res, rej) => {
    res("数据2");
  });
  }.catch(err){

  }
}
```

## 动态 import 方法

- 参数指定要加载模块的位置
- import()返回一个 Promise 对象
- 代码运行时才会执行，是资源异步加载的解决方案之一
