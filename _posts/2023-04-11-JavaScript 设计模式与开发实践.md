# 第一部分 基础知识
## 闭包
### 作用域
一个形象的描述，JavaScript 函数像一层半透明的玻璃，在函数里面可以看到外面的变量，而在函数外面则无法看到函数里面的变量。
### 生存周期
在闭包中，变量并不随着函数退出而销毁，这是因为外部还保存着对内部函数的引用；
```typescript
var func = function () {
	var a = 1;
    return function () {
        a++;
        alert(a);
    }
};

var f = func();
```
经典应用 - es6 的话可以直接使用 let
```typescript
for (var i = 0, len = nodes.length; i < len; i++) {
	(function(i) {
        nodes[i].onclick = function () {
            console.log(i);
        }
    })(i)
}
```
### 应用

- 封装变量
- 延续局部变量的寿命

案例：解决通过 img 标签请求丢失的问题
```typescript
var report = (function () {
	var imgs = [];
    return function (src) {
        var img = new Image();
        imgs.push(img);
        img.src = src;
    }
})();
```
### 内存管理
使用闭包和内存泄露有关系的地方：容易形成循环引用；
在基于引用计数的垃圾回收机制中，如果两个对象之间形成了循环引用，将导致两个对象都无法被回收，但本质上不是闭包的问题；
解决则将循环引用中的变量设置为 null 即可；

# 第二部分 设计模式
## 单例模式
### 定义
保证一个类仅有一个实例，并提供一个访问它的全局访问点
### 例子
```typescript
let CreateDiv = function (html) {
	this.html = html;
    this.init();
};

CreateDiv.prototype.init = function () {
    let div = document.createElement('div');
    div.innerHTML = this.html;
    document.body.appendChild(div);
};

let ProxySingletonCreateDiv = (function () {
    let instance;
    return function (html) {
        if (!instance) {
        	instance = new CreateDiv(html);
        }

        return instance;
    }
})();

let a = new ProxySingletonCreateDiv('sven1');
let b = new ProxySingletonCreateDiv('sven2');

alert(a === b);
```
### 小结
简单但非常实用的模式，经常出现在我们的代码结构中；

## 策略模式
### 定义
定义一系列的算法，把他们分别封装起来，并且使他们可以互相替换，目的是将算法的使用与算法的实现分离开来
### 例子
```typescript
let strategies = {
	's': function(salary) {
        return salary * 4;
    },
    'a': function(salary) {
        return salary * 3;
    },
    'b': function(salary) {
        return salary * 2;
    }
};

// Context 
let calculateBonus = function (level, salary) {
    return strategies[level](salary);
};

console.log(calculateBonous('s', 2000));
console.log(calculateBonous('a', 2000));
```
### 优缺点
#### 优点
策略模式利用组合、委托和多态等技术和思想，可以有效地避免多重条件选择语句。
策略模式提供了对开放—封闭原则的完美支持，将算法封装在独立的 strategy 中，使得它们易于切换，易于理解，易于扩展。
策略模式中的算法也可以复用在系统的其他地方，从而避免许多重复的复制粘贴工作。
在策略模式中利用组合和委托来让 Context 拥有执行算法的能力，这也是继承的一种更轻便的替代方案。
#### 缺点
增加了较多的策略类或策略对象
使用策略模式必须了解所有的 strategy，违反最少知识原则

## 代理模式
### 定义
代理模式是为一个对象提供一个代用品，以便控制对他的访问；遵循面向对象设计中的单一职责原则，同时也符合开放—封闭原则；在 JavaScript 中最常用的是虚拟代理（如图片预加载，合并 http 请求）和缓存代理；
### 例子
缓存代理
```typescript
let mult = function () {
	let a = 1;
    for (let i = 0, len = arguments.length; i < len; i++) {
        a = a * arguments[i];
    }
    return a;
};

let proxyMult = (function() {
    let cache = {};
    return function () {
        let args = Array.prototype.join.call(arguments, ',');
        if (args in cache) {
            return cache[args];
        }
        return cache[args] = mult.apply(this, arguments);
    }
})();

proxyMult(1, 2, 3, 4);
proxyMult(1, 2, 3, 4);
```

## 迭代器模式
### 定义
提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示；迭代器模式是一种相对简单的模式，简单到很多时候我们都不认为它是一种设计模式

## 发布-订阅模式
### 定义
发布—订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。在 JavaScript 开发中，我们一般用事件模型来替代传统的发布—订阅模式。
### 基础例子
```typescript
let event = {
	clientList: [],
    listen: function (key, fn) {
        if (!this.clientList[key]) {
            this.clientList[key] = [];
        }
        this.clientList[key].push(fn);
    },
    trigger: function () {
        let key = Array.prototype.shift.call(arguments);
        let fns = this.clientList[key];

        if (!fns || fns.length === 0) {
            return false;
        }

        for (let i = 0, len = fns.length; i < len; i++) {
            let fn = fns[i];
            fn.apply(this, arguments);
        }
    },
    remove: function (key, fn) {
        let fns = this.clientList[key];

        if (!fns) {
            return false;
        }

        if (!fn) {
            fns && (fns.length = 0);
        }
        else {
            for (let len = fns.length - 1; len >= 0; len--) {
                let _fn = fns[len];
                if (_fn === fn) {
                    fns.splice(len, 1);
                }
            }
        }
    }
};

let installEvent = function (obj) {
    for (let i in event) {
        obj[i] = event[i];
    }
};
```
### 例子
中介公司的例子，常见的 event-bus 事件机制也是基于类似的原则实现；es6 的语法下，通常使用 class 进行封装；
```typescript
let Event = (function() {
    let clientList = {};

    let listen = function (key, fn) {
        if (!clientList[key]) {
            clientList[key] = [];
        }
        clientList[key].push(fn);
    };

    let trigger = function () {
        let key = Array.prototype.shift.call(arguments);
        let fns = clientList[key];

        if (!fns || fns.length === 0) {
            return false;
        }

        for (let i = 0, len = fns.length; i < len; i++) {
            let fn = fns[i];
            fn.apply(this, arguments);
        }
    };

    let remove = function (key, fn) {
        let fns = clientList[key];

        if (!fns) {
            return false;
        }

        if (!fn) {
            fns && (fns.length = 0);
        }
        else {
            for (let len = fns.length - 1; len >= 0; len--) {
                let _fn = fns[len];
                if (_fn === fn) {
                    fns.splice(len, 1);
                }
            }
        }
    };

    return {
        listen,
        trigger,
        remove
    };
})();
```
### 优缺点
#### 优点
一为时间上的解耦，二为对象之间的解耦。
#### 缺点
创建订阅者本身要消耗一定的时间和内存，而且当你订阅一个消息后，也许此消息最后都未发生，但这个订阅者会始终存在于内存中。

## 命令模式
### 定义
命令模式是最简单和优雅的模式之一，命令模式中的命令（command）指的是一个执行某些特定事情的指令。
最常见的应用场景：需要向某些对象发送请求，但是并不知道请求的接受者，也不知道被请求后续操作；
撤销重做功能，本质上是将命令存放到队列中，通过存取队列来控制；
### 例子
```typescript
const Ryu = {
    attack: function() {
        console.log('attack');
    },
    defence: function() {
        console.log('defence');
    },
    jump: function() {
        console.log('jump');
    },
    crouch: function() {
        console.log('crouch');
    }
};

const makeCommand = function (receiver, state) {
    return function() {
        receiver[state]();
    }
};

const commands = {
    '119': 'jump',
    '115': 'crouch',
    '97': 'defence',
    '100': 'attack'
};

const commandStack = [];

document.onkeypress = function (ev) {
    let keyCode = env.keyCode;
    let command = makeCommand(Ryu, commands[keyCode]);

    if (command) {
        command();
        commandStack.push(command);
    }
};

document.getElementById('replay').onClick = function () {
    let command;
    while (command = commandStack.shift()) {
        command();
    }
};

```

## 中介者模式
### 定义
解除对象与对象之间的紧耦合关系。增加一个中介者对象后，所有的相关对象都通过中介者对象来通信，而不是互相引用，所以当一个对象发生改变时，只需要通知中介者对象即可。中介者使各对象之间耦合松散，而且可以独立地改变它们之间的交互。中介者模式使网状的多对多关系变成了相对简单的一对多关系
![image.png](https://cdn.nlark.com/yuque/0/2023/png/542898/1679817937492-78bb3037-bcaf-4d11-b6cd-56501be9bbb4.png#averageHue=%23eeeeee&clientId=u3eb381b6-c967-4&from=paste&height=436&id=ua76e8e0f&name=image.png&originHeight=316&originWidth=351&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31877&status=done&style=none&taskId=u97cbd995-3f01-4414-a539-6fc290e8405&title=&width=484)
### 例子
```typescript
const playDirector = (function () {
    let players = {};
	let operations = {};

    const receiveMessage = function () {
        let message = Array.prototype.shift.call(arguments);
        operations[messgae].apply(this, arguments);
    };

    return {
        receiveMessage
    }
})();
```
### 优缺点
#### 优点
中介者模式是迎合迪米特法则的一种实现。迪米特法则也叫最少知识原则，是指一个对象应该尽可能少地了解另外的对象（类似不和陌生人说话）。中介者模式使各个对象之间得以解耦，以中介者和对象之间的一对多关系取代了对象之间的网状多对多关系。各个对象只需关注自身功能的实现，对象之间的交互关系交给了中介者对象来实现和维护。
#### 缺点
因 为对象之间交互的复杂性，转移成了中介者对象的复杂性，使得中介者对象经常是巨大的。中介者对象自身往往就是一个难以维护的对象。
### 小结
如果对象之间的复杂耦合确实导致调用和维护出现了困难，而且这些耦合度随项目的变化呈指数增长曲线，那我们就可以考虑用中介者模式来重构代码。

## 装饰者模式
### 定义
给对象动态地增加职责，相比于继承更加轻便灵活；
### 例子
```typescript
var Plane = function(){}
Plane.prototype.fire = function(){
    console.log( '发射普通子弹' );
}
// 接下来增加两个装饰类，分别是导弹和原子弹：
var MissileDecorator = function( plane ){
    this.plane = plane;
}
MissileDecorator.prototype.fire = function(){
    this.plane.fire();
    console.log( '发射导弹' );
}
var AtomDecorator = function( plane ){
    this.plane = plane;
}
AtomDecorator.prototype.fire = function(){
    this.plane.fire();
    console.log( '发射原子弹' );
} 
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/542898/1679835453766-d7f0bb6a-9246-4c45-bd79-c79f033bcacd.png#averageHue=%23ececec&clientId=ud466ac6d-3d33-4&from=paste&height=329&id=ub03406b0&name=image.png&originHeight=329&originWidth=463&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36508&status=done&style=none&taskId=u67671367-6b40-4a9d-a650-547ca02a8ed&title=&width=463)
### 思考
修改某个对象中的某个单一职责的函数，可以使用装饰者模式；不然还是用继承稳妥一些；
例子：增加统计上报、动态改变函数参数
## 状态模式

## 适配器模式
解决已有接口不匹配的问题，通常是一种亡羊补牢的方法；

# 第三部分 设计原则和编程技巧
## 单一职责原则
一个对象（方法）只做一件事。
## 最少知识原则
减少对象之间的联系。
## 开放-封闭原则
软件实体（类、模块、函数）等应该是可以扩展的，但是不可修改。
开放-封闭原则是编写一个好程序的目标，其他设计原则都是达到这个目标的过程。
接受第一次愚弄：在最初编写代码的时候，先假设变化永远不会发生，这有利于我们迅速完成需求。当变化发生并且对我们接下来的工作造成影响的时候，可以再回过头来封装这些变化的地方。然后确保我们不 会掉进同一个坑里，这有点像星矢说的：“圣斗士不会被同样的招数击倒第二次。”
## 接口和面向接口编程
TypeScript 编写基于 interface 的命令模式
## 代码重构
从某种角度来看，设计模式的目的就是为许多重构行为提供目标；

- 提炼函数

有助于代码复用；拥有良好的命名则会起到注释作用；

- 提前让函数退出代替嵌套条件分支

提前 return 有助于减少层级，增加代码可读性；

- 传递对象参数代替过长的参数列表

一般函数参数超过两个，我都倾向于传入对象，并在函数内解构；传入对象的好处是方便新增或减少参数，增加了代码的可扩展性；

- 不大规模使用三目运算符

之前陷入一个怪区，总是强行用三目运算符去代替 if else，但现在发现有时候 if else 在可读性和可维护性上会更加清晰，因此要斟酌使用三目运算符和 if else；

- 合理使用链式调用

不多说了；

- 分解大型类

面向对象鼓励将行为分布在合理数量的更小对象之中；

- 用 return 退出多重循环

比记录 flag 然后通过 break 跳出要好；
