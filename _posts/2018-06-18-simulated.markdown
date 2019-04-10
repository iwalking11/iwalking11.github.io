---
layout:     post
title:      "js模拟实现"
subtitle:   " \"js模拟实现一些常用功能\""
date:       2018-06-18 20:00:00
author:     "iwalking11"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
	- js
    - 模拟实现
---

<font size=4>

### 1.请你实现一个深克隆
- 浅拷贝
```
let newObj = Object.assign({}, obj);
let newObj = {...obj};
```

- 深拷贝
```

let newObj = Json.parse(Json.stringfy(obj));
// 会忽略 undefined
// 会忽略 symbol
// 不能序列化函数
// 不能解决循环引用的对象


function deepCopy(obj) {
    let isBase = (obj) => typeof obj !== ('object' || 'function') || obj === null
    
    if (isBase(obj)) {
        // 基础类型数据直接返回
        return obj;
    }
    
    let newObj = typeof obj === 'object' ? {} :[];
    
    Reflect.ownKeys(obj).forEach(item => {
        newObj[item] = isBase(obj[item]) ? obj[item] : deepCopy(obj[item])
    });
    
    return newObj;
}
```

Date、RegExp、DOM等特殊对象以及原型链的处理
https://juejin.im/post/5abb55ee6fb9a028e33b7e0a

### 2.手写实现Function.prototype.call方法
- 分析
> fn.call(target)主要做了这么几件事：
> 1. 将fn方法赋值给target对象，target.fn = fn
> 2. 执行对象target.fn(),有参数，传递参数
> 3. 删除target.fn()
> 4. 返回执行结果

- 实现
```
Function.prototype.myCall = function(context) {
    if (typeof this !== 'function') {
        // 非函数类型调用myCall
        throw new TypeError('Error')
    }
    
    // 处理参数为空或者null的情况
    context = context || window;
    
    // 使用symbol防止污染context原属性
    let fn = Symbol('myCall');
    
    // 将方法赋值到目标对象，并调用
    context[fn] = this;
    let result = context[fn](...[...arguments].slice(1));
    
    // 删除调用过的方法
    delete context[fn];
    
    // 如果原方法有返回值，返回
    return result;
}

Function.prototype.myCall1 = function () {
    if (typeof this !== 'function') {
        throw new TypeError('typeError!');
    }
    let target = [...arguments].pop || window;
    let fn = Symbol('myCall1');
    target[fn] = this;
    let result = target[fn](...[...arguments].slice(1));
    delete target[fn];
    return result;
}
```

### 3.手写实现Function.prototype.apply方法
```
Function.prototype.myApply = function(context) {
    if (typeof this !== 'function') {
     // 非函数类型调用myApply
        throw new TypeError('Error')
    }
    
     // 处理参数为空或者null的情况
    context = context || window;
    
     // 使用symbol防止污染context原属性
    let fn = Symbol('myApply');
    
       // 将方法赋值到目标对象，并调用
    context[fn] = this;
    let reslut;
    if (arguments[1]) {
        // 方法有参数的情况
        result = context[fn](...arguments[1]);
    } else {
        result =  context[fn]();
    }
    
    delete context[fn];
    
    return result;
}
```

### 4.手写实现Function.prototype.bind方法
```
Function.prototype.myBind = function(context) {
    if (typeof this !== 'function') {
        throw new TypeError('Error')
    }
    context = context || window;
    let args = [...arguments].slice(1);
    let _this = this;
    return function f () {
        // 判断是否通过new 方式 被当作构造方法使用
        if (this instanceof f) {
            // 指向new 出来的实例this
            return _this.apply(this, args.concat([...arguments]))
        } else {
            return _this.apply(context, args.concat([...arguments]))
        }
    }
}
```

### 5. new的模拟实现
- 分析
> 创建了一个全新的对象。
> 这个对象会被执行[[Prototype]]（也就是__proto__）链接。
> 生成的新对象会绑定到函数调用的this。
> 通过new创建的每个对象将最终被[[Prototype]]链接到这个函数的prototype对象上。
> 如果函数没有返回对象类型Object(包含Functoin, Array, Date, RegExg, Error)，那么new表达式中的函数调用会自动返回这个新的对象。

- 代码
```
function newfn() {
    let args = [...arguments];
    let constructor = args.pop();
    if(typeof constructor !== 'function'){
      throw 'newOperator function the first param must be a function';
    }
    
    let obj = {};
    Object.setPrototypeof(obj, constructor.prototype);
    // 等同于 let obj = Object.create(constructor.prototype);
    // 也等同于 obj.__proto__ = constructor.prototype; (__proto__是浏览器环境的属性，不属于es标准)
    
    let reslut = constructor.apply(obj, args);
    return (/^(object|function)$/.test(typeof result) && result !== null) 
    ? reulst : obj; 
    
}

ES6 简化版
function objectFactory(Constructor, ...rest) {
  const instance = Object.create(Constructor.prototype);
  const result = Constructor.apply(instance, rest);
  return (typeof result === 'object' && result) || instance;
}

/**
 * 模拟实现 new 操作符
 * 作者：若川
 * 链接：https://juejin.im/post/5bde7c926fb9a049f66b8b52
 * @param  {Function} ctor [构造函数]
 * @return {Object|Function|Regex|Date|Error}      [返回结果]
 */
function newOperator(ctor){
    if(typeof ctor !== 'function'){
      throw 'newOperator function the first param must be a function';
    }
    // ES6 new.target 是指向构造函数
    newOperator.target = ctor;
    // 1.创建一个全新的对象，
    // 2.并且执行[[Prototype]]链接
    // 4.通过`new`创建的每个对象将最终被`[[Prototype]]`链接到这个函数的`prototype`对象上。
    var newObj = Object.create(ctor.prototype);
    // ES5 arguments转成数组 当然也可以用ES6 [...arguments], Aarry.from(arguments);
    // 除去ctor构造函数的其余参数
    var argsArr = [].slice.call(arguments, 1);
    // 3.生成的新对象会绑定到函数调用的`this`。
    // 获取到ctor函数返回结果
    var ctorReturnResult = ctor.apply(newObj, argsArr);
    // 小结4 中这些类型中合并起来只有Object和Function两种类型 typeof null 也是'object'所以要不等于null，排除null
    var isObject = typeof ctorReturnResult === 'object' && ctorReturnResult !== null;
    var isFunction = typeof ctorReturnResult === 'function';
    if(isObject || isFunction){
        return ctorReturnResult;
    }
    // 5.如果函数没有返回对象类型`Object`(包含`Functoin`, `Array`, `Date`, `RegExg`, `Error`)，那么`new`表达式中的函数调用会自动返回这个新的对象。
    return newObj;
}


```
### 6. 防抖节流实现
- 分析  
防抖： 频繁触发的事件，停止出发一段时间后再执行事件；
节流： 事件按照一定的频率触发

- 防抖代码
```
funciotn debounce(fn, wait) {
    let timer= null;
    return function() {
        let context = this;
        let args = arguments;
        if (timer) {
            clearTimeout(timer); 
            timer = null;
        }
        timer = setTimeout(()=>{
            fn.apply(context, args);
        }, wait);
        
    }
}

// 优化版
function debounce(func, wait, immediate) {

    var timeout, result;

    var debounced = function () {
        var context = this;
        var args = arguments;

        if (timeout) clearTimeout(timeout);
        if (immediate) {
            // 如果已经执行过，不再执行
            var callNow = !timeout;
            timeout = setTimeout(function(){
                timeout = null;
            }, wait)
            if (callNow) result = func.apply(context, args)
        }
        else {
            timeout = setTimeout(function(){
                func.apply(context, args)
            }, wait);
        }
        return result;
    };

    debounced.cancel = function() {
        clearTimeout(timeout);
        timeout = null;
    };

    return debounced;
}
```

- 节流代码
```
function throttle(fn, wait) {
    let timer = null;
    let previous = 0;
    return functtion() {
        let remaining = wait - ( +new Date() - previous);
        let args= arguments;
        let context = this;
        
        // 第一次小于零， 后面都是===0时候调用
        if (remaining <= 0) {
            fn.apply(context, args) 
        } else {
            // 最后一次wait时间后调用fn
            timer = setTimeout(function() {
                fun.apply(context, args);
                timer = null;
            }, wait)
        }
    };
}
```

### 7.instanceof实现
- 分析
instanceof 主要的实现原理就是只要右边变量的 prototype 在左边变量的原型链上即可。  
因此，instanceof 在查找的过程中会遍历左边变量的原型链，直到找到右边变量的 prototype，如果一直查到null，表明查找失败，返回 false。

- 代码
```
function instanceof(obj, Obj) {
    let proto = Object.getPrototypeOf(obj);
    while (true) {
        if (proto === null ){
            return false;
        }
        if (proto === Obj.prototype) {
            return true;
        }
        proto = Object.getPrototypeOf(proto);
    }
}
```

### 8.继承（ES3、ES5、ES6）
- ES3继承  
利用 Parent.call(this) 执行“方法借用”，获取 Parent 的属性，继承实例属性和方法  
利用一个空函数将 Person.prototype 加入原型链，继承原型属性和方法
```
function Child() {
  Parent.call(this, "Bob");
  this.hobby = "Histroy";
}
function prototype(Child, parent) {
    function F() {}
    F.prototype = parent.prototype;
    Child.prototype = new F();
     prototype.constructor = Child;
}
// 当我们使用的时候：
prototype(Child, Parent);
```
- ES5继承  
利用 Parent.call(this) 执行“方法借用”，获取 Parent 的属性  
利用 ES5 增加的 Object.create 方法将 Parent.prototype 加入原型链
```
function Child() {
  Parent.call(this, "Bob");
  this.hobby = "Histroy";
}

// Child.prototype = new Parent();
// 调用了两次父类的构造方法，生成两次实例

// Child.prototype = Parent.prototype;
// 子类和父类的构造方法是同一个对象，无法辨别生成的实例是通哪个创建的

//不存在引用属性共享问题
Child.prototype  = Object.create(Parent.prototype, {
  constructor: {
    value: Child,
    enumerable: false,
    configurable: true,
    writable: true
  }
});

```
- ES6继承  
1.利用 ES6 增加的 class 和 extends 实现比以前更完善的继承;  
2.通过Object.setPrototypeOf()来实现类的静态方法继承
```
function Sub(name, age){
    Super.call(this, name);    //继承了Super 属性
    this.age = age;
}
//Sub.prototype.__proto__ = Super.prototype
Object.setPrototypeOf(Sub.prototype, Super.prototype)
Object.setPrototypeOf(Sub, Super)    // 继承父类的静态属性或方法
```


</font>