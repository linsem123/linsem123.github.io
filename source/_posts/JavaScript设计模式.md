---
title: JavaScript设计模式
date: 2019-10-21 15:10:47
categories:
- 编程
- 学习
tags:
- js
---

# 《JavaScript设计模式》

### 一、单例模式

```
/**
 * 定义： 保证一个类仅有一个实例，并提供访问此实例的全局访问点。
 * 用途：如果一个类负责连接数据库的线程池，日志记录逻辑等等，此时需要单例模式来保证对象不被重复创建，以达到降低开销的目的。
 */

const Singleton = function() {};
Singleton.getInstance = (function() {
  // 由于es6没有静态类型，使用闭包；函数外部无法访问 instacne
  let instance = null;
  return function() {
    if (!instance) {
      instance = new Singleton();
    }
    return instance;
  };
})();

let s1 = Singleton.getInstance();
let s2 = Singleton.getInstance();

console.log(s1 === s2);
```

### 二、工厂模式
```
/**
 * 定义：工厂模式的实质“定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例话推迟到子类中进行”，简单来说，就是把new 对象的操作包裹一层，对外提供一个可以根据不同参数创建不同对象的函数。
 * 优点：可以隐藏原始类，方便之后的代码迁移。调用者只需要记住类的代名词即可
 * 缺点：由于多了层封装，会造成类的数目过多，系统复杂度增加。
 */

class Dog {
  run() {
    console.log("dog");
  }
}

class Cat {
  run() {
    console.log("cat");
  }
}

class Animal {
  constructor(name) {
    name = name.toLocalLowerCase();
    switch (name) {
      case "dog":
        return new Dog();
      case "cat":
        return new Cat();
      default:
        throw TypeError("class name wrong");
    }
  }
}

const cat = new Animal("cat");
const dog = new Animal("dog");
cat.run();
dog.run();

```

### 三、抽象工厂模式
```
/**
 * 定义：抽象工厂模式就是，围绕一个超级工厂类，创建其他工厂类；在围绕工厂类，创建实体类
 * 相比较于传统的工厂模式，它多出了一个超级工厂类
 */

 // 动物实体类
 class Dog{
   run(){
     console.log('dog is running');
   }
 }

 class Cat{
   run(){
     console.log('cat is running');
   }
 }

// 人类实体类
class Male{
  run(){
    console.log('male is talking');
  }
}

class Female{
  run(){
    console.log('female is talking');
  }
}


/*********************/
// 工厂类
// 为了更好的约束工厂类，先实现一个接口类
class AbstractFactory{
  getPerson(){
    throw new Error('子类请实现接口')
  }

  getAnimal(){
    throw new Error('子类请实现接口')
  }
}

// 接下来，Animal 和 Person 继承这个接口类
class PersonFactory extends AbstractFactory{
  getPerson(person){
    person = person.toLocaleLowerCase();
    switch(person){
      case 'male':
        return new Male()
      case 'female':
        return new Female();
      default:
        break;
    }
  }

  getAnimal(){
    return null;
  }
}

class AnimalFactory extends AbstractFactory{
  getPerson(){
    return null;
  }

  getAnimal(animal){
    animal = animal.toLocaleLowerCase();
    switch(animal){
      case 'cat':
        return new Cat();
      case 'dog':
        return new Dog();
      default:
        break;
    }
  }
}

/**
 * 实现超级工厂
 */
class Factory{
  constructor(choice){
    switch(choice){
      case 'person':
        return new PersonFactory();
      case 'animal':
        return new AnimalFactory();
      default:
        break;
    }
  }
}

/**
 * 测试代码
 */

 const pf = new Factory('person');
 const male = pf.getPerson('male');
 const female = pf.getPerson('female');
 male.run();
 female.run();

 const af = new Factory('animal');
 const cat = af.getAnimal('cat');
 const dog = af.getAnimal('dog');
 cat.run();
 dog.run();
```

### 四、享元模式
```
/**
 * 享元模式：运用共享技术来减少创建的对象的数量，从而减少内存占用、提高性能
 * 1. 享元模式提醒我们将一个对象的属性划分为内部和外部状态。
 *  内部状态：可以被对象集共享，通常不会改变。
 *  外部状态；根据应用场景经常改变。
 * 2. 享元模式是利用时间换取空间的优化模式。
 * 
 * 应用场景：只要是需要大量创建重复的类的代码块，均可以使用享元模式抽离内部，外部状态，减少重复类的创建。
 */

//  对象池
class ObjectPool{
  constructor(){
    this.__pool = [];
  }

  // 创建对象
  create(Obj){
    return this.__pool.length === 0
    ? new Obj(this) // 对象池中没有空闲对象，则创建一个对象
    : this.__pool.shift(); // 对象池中有空闲对象，直接取出，无需再次创建
  }

  // 对象回收
  recover(obj){
    return this.__pool.push(obj);
  }

  // 对象池大小
  size(){
    return this.__pool.length;
  }  
}

// 模拟文件对象
class File{
  constructor(pool){
    this.pool = pool;
  }

  // 模拟下载操作
  download(){
    console.log(`+ 从${this.src} 开始下载 ${this.name}`);
    setTimeout(()=>{
      console.log(`- ${this.name} 下载完毕`); // 下载完毕后，将对象重新放入对象池
      this.pool.recover(this);
    },100) 
  }
}


/*** test ***/
let objPool = new ObjectPool(); 

let file1 = objPool.create(File);
file1.name = '文件1';
file1.src = 'https://download1.com';
file1.download();

let file2 = objPool.create(File);
file2.name = '文件2';
file2.src = 'www.baidu.com';
file2.download();

setTimeout(()=>{
  let file3 = objPool.create(File);
  file3.src = '文件3';
  file3.name = 'www.jianshu.com';
  file3.download();
},200)


setTimeout(()=>{
  console.log(`${'*'.repeat(50)}\n下载了3个文件，但是只创建了${objPool.size()}个对象`);
},1000)
```

### 五、代理模式
```
/**
 * 代理模式可以避免对一些对象直接的访问，以此为基础，常见的有保护代理和虚拟代理。保护代理可以在代理中直接绝对，对象的访问；虚拟代理可以延迟访问到真正需要的时候，以便节省程序开销。
 * 优点：代理模式有高度解耦，对象保护，易修改等优点。
 * 缺点：通过设置代理访问对象，因此开销会更大，时间也会慢。
 */

 const myImg = {
   setSrc(imgNode, src){
     imgNode.src = src;
   }
 }

//  利用代理模式实现图片的懒加载
const proxyImg = {
  setSrc(imgNode, src){
    // 1. 加载占位图片并且将图片放入<img>元素
    myImg.setSrc(imgNode, './image.png');
    // 2. 加载真正需要的图片
    let img = new Image();
    img.src = src;
    // 3. 加载完成后，更新<img>元素中的图片
    img.onload = () => {
      myImg.setSrc(imgNode, src)
    }
  }
}

// test
let imgNode = document.createElement('img'),
  img_src = 'https://upload-images.jianshu.io/upload_images/5486602-5cab95ba00b272bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp';

  document.body.appendChild(imgNode);
  proxyImg.setSrc(imgNode,img_src);
```

### 六、桥接模式
```
/**
 * 桥接模式：将抽象部分与具体实现部分分离，两者可独立变化，也可以一起工作。
 * 应用场景：在封装开源库的时候，经常会用到这种设计模式。
 * 
 */

//  模拟forEach方法
const forEach = (arr, cb) => {
  if(!Array.isArray(arr)) return;

  const length = arr.length;
  for(let i = 0; i < length; ++i){
    cb && cb(arr[i], i);
  }
}

// test
let arr = [1,2,3,4,5];
forEach(arr, (el, index) => {
  console.log(`元素是：${el}, 位于第${index}个。`);
})
```
### 七、组合模式
```
/**
 * 将对象组合成树形结构，以用来表示“部分-整体结构”
 * 1. 用小的子对象构造更大的父对象，而这些子对象也由更小的子对象构成。
 * 2. 单个对象和组合对象用户暴露的接口具有一致性，而同种接口不同表现形式亦体现了多态性
 * 应用场景：组合模式可以在需要针对“树形结构”进行操作的应用中使用，例如扫描文件夹，渲染网站导航结构等等。
 * 
 */

//  文件类
class File{
  constructor(name){
    this.name = name || "File";
  }

  add(){
    throw new Error('文件不能添加文件');
  }
  scan(){
    console.log('扫描文件：' + this.name);
  }
}

// 文件夹类
class Folder{
  constructor(name){
    this.name = name || "Folder";
    this.files = [];
  }

  add(file) {
    this.files.push(file);
  }

  scan() {
    console.log('扫描文件夹：' + this.name);
    for(let file of this.files){
      file.scan();
    }
  }
}

// test
let home = new Folder('用户根目录');

let folder1 = new Folder('第一个文件夹');
let folder2 = new Folder('第二个文件夹');

let file1 = new File('1号文件');
let file2 = new File('2号文件');
let file3 = new File('3号文件');


folder1.add(file1);
folder2.add(file2);
folder2.add(file3);
home.add(folder1);
home.add(folder2);

home.scan();

```
### 八、装饰者模式
```
/**
 * 装饰者模式：在不改变对象自身的基础上，动态的添加功能代码
 * 应用场景：多用于一开始不确定的对象，或者对象功能经常变动的时候。尤其是在参数校验，参数拦截等场景。
 */


//  ES6 的装饰器语法规范只是在”提案阶段“，而且不能装饰普通的函数或箭头函数。
// 装饰器的触发可以在函数运行之前，也可以在函数运行之后
const addDecorator = (fn, before, after) => {
  let isFn = fn => typeof fn === 'function';

  if(!isFn(fn)) {
    return () => {};
  }

  return (...args) => {
    let result;
    // 按照顺序执行“装饰函数”
    isFn(before) && before(...args);
    // 保存返回函数结果
    isFn(fn) && (result = fn(...args));
    isFn(after) && after(...args);
    // 最后返回结果
    return result;
  }
}

// test
const beforeHi = (...args) => {
  console.log(`Before hello, args are ${args}`);
}

const hello = (name = 'user') => {
  console.log(`hello ${name}`);
  return name;
}

const afterHi = (...args) => {
  console.log(`After hello, args are ${args}`);
}

console.log(addDecorator(hello,beforeHi,afterHi)());
```
### 九、适配器模式
```
/**
 * 适配器模式为多个不兼容接口之前提供 ‘转化器’
 * 它的操作非常简单，检查接口的数据，进行过滤，重组等操作，使得另一接口可以使用数据
 * 应用场景：当数据不符合使用规则，就可以借助这种模式进行格式转化。
 */

const API = {
  qq: () => ({
    name: 'love',
    author: 'tylor swift',
    f: 1
  }),
  netease: () => ({
    n: 'love',
    a: 'tylor swift',
    f: false
  })
}

const adapter = (info = {}) => ({
  name: info.name || info.n,
  author: info.author|| info.a,
  free: !!info.f
})

console.log(adapter(API.qq()));
console.log(adapter(API.netease()));
```
### 十、命令模式
```
/**
 * 命令模式：是一种数据驱动的设计模式，它属于行为型模式。
 * 1. 请求以命令的形式包裹在对象中，并传给调用对象。
 * 2. 调用对象寻求可以处理该命令的合适的对象，并把该命令传给相应的对象。
 * 3. 该对象执行命令。
 * 
 * 在这三个步骤中，分别有3个主体，发送者，传递者，和执行者。
 * 应用场景：有时候需要向某些对象发送请求，但是又不知道请求的接受者是谁，更不知道被请求的操作是什么。此时，命令模式就是以一种松耦合的方式来设计程序。
 */

//  接收到命令，执行相关操作
const MenuBar = {
  refresh() {
    console.log("刷新菜单页面");
  }
}

// 命令对象，execute方法就是执行相关的命令
const RefreshMenuBarCommand = receiver => {
  return {
    execute() {
      receiver.refresh();
    }
  }
}

// 为按钮对象指定对应的对象
const setCommand = (button, command) => {
  button.onclick = () => {
    command.execute();
  }
}

let refreshMenuBarCommand = RefreshMenuBarCommand(MenuBar);
let button = document.querySelector('button');
setCommand(button, refreshMenuBarCommand)
```
### 十一、备忘录模式
```
/**
 * 备忘录模式：属于行为模式，保存某个状态，并且在需要的时候就直接获取，而不是重复计算
 * 注意：备忘录模式实现，不能破坏原始封装，也就是说，能拿到内部状态，将其保存在外部。
 * 应用场景：数据缓存呢
 */

 const fetchData = (() => {
  //  备忘录/缓存
  const cache = {};
   return page => 
    new Promise(resolve => {
      // 如果页面数据已经被缓存，直接推出
      if(page in cache){
        return resolve(cache[page])
      }

      // 否则，异步请求页面数据
      // 模拟异步数据
      setTimeout(() => {
        cache[page] = `内容是${page}`;
        resolve(cache[page])
      }, 1000);
    })
 })();

//  test
const run = async () => {
  let start = new Date().getTime(),
    now;
    // 第一次；没有缓存
    await fetchData(1).then(res => {
      console.log('res:' + res);
    });
    now = new Date().getTime();
    console.log(`没有缓存，耗时${now - start}ms`);

    // 第二次：有缓存，有备忘录
    start = now;
    await fetchData(1).then(res => {
      console.log('res:' + res);
    });
    now = new Date().getTime();
    console.log(`有缓存，耗时${now - start}ms`);
}

run();
```
### 十二、模板模式
```
/**
 * 模板模式：抽象父类定义了子类需要重写的相关方法，而这些方法，仍然是通过父类方法调用的。
 * 注意：父类定义接口方法，子类方法的调用受父类控制。
 * 应用场景：
 */

class Animal {
  constructor() {
    //  this 指向实例
    this.live = () => {
      this.eat();
      this.sleep();
    };
  }
  eat(){
    throw new Error('模板类方法必须被重写');
  }

  sleep() {
    throw new Error("模板类方法必须被重写");
  }
}


class Dog extends Animal{
  constructor(...args){
    super(...args)
  }

  eat(){
    console.log('dog eats food');
  }

  sleep(){
    console.log('dog sleeped');
  }
}

class Cat extends Animal{
  constructor(...args){
    super(...args)
  }

  eat(){
    console.log('cat eats fish');
  }

  sleep(){
    console.log('cat sleeped');
  }
}

// test
let dog = new Dog();
dog.live();

let cat = new Cat();
cat.live();
```
### 十三、状态模式
```
/**
 * 状态模式：对象的行为是基于状态来改变的。
 * 内部的状态转化，导致了行为表现形式不同。所以，用户在外面看来，好像是修改了行为。
 * 优点：封装了转化规则，对于大量分支语句来说，可以考虑使用状态类进一步封装。每个状态都是确定的，所以对象行为是可控的。
 * 缺点：状态模式的关键是将事物的状态都封装成为单独的类，这个类的各种方法就是“此种状态对应的表现行为”。因此，状态类会增加程序开销。
 * 
 */

//  FSM 有限状态机
const FSM = (()=>{
  let current_state = "download";
  return {
    download: {
      click: () => {
        console.log('暂停下载');
        current_state = 'pause';
      },
      del: () => {
        console.log('先暂停，再删除');
      },
    },
    pause: {
      click: () => {
        console.log('继续下载');
        current_state = 'download';
      },
      del: () => {
        console.log('删除任务');
        current_state = 'deleted'
      }
    },
    deleted: {
      click: () => {
        console.log('任务已删除，请重新开始');
      },
      del: () => {
        console.log('任务已删除');
      }
    },
    getState: () => current_state
  }
})();

class Download {
  constructor(fsm){
    this.fsm = fsm;
  }

  handleClick() {
    const { fsm } = this;
    fsm[fsm.getState()].click();
  }

  handleDel(){
    const {fsm} = this;
    fsm[fsm.getState()].del();
  }
}

// 开始下载
let download = new Download(FSM);

download.handleClick(); // 暂停下载
download.handleClick(); // 继续下载
download.handleDel(); // 下载中，无法删除
download.handleClick(); // 暂停下载
download.handleDel(); // 删除任务
download.handleClick(); // 任务已删除，请重新开始
download.handleDel();
```
### 十四、策略模式
```
/**
 * 策略模式：能够把一系列“可互换的”算法封装起来，并根据用户需求来选择其中一种。
 * 策略模式实现的核心就是将算法的使用和算法的实现分离。
 * 算法的实现交给策略类。
 * 算法的使用交给环境类，环境类会根据不同的情况选择合适的算法。
 * 在使用的时候，需要了解所有的策略之间的异同点，才能选择合适的策略进行调用。
 * 
 */

//  策略类
const strategies = {
  A(){
    console.log('This is strategy A');
  },
  B(){
    console.log('This is strategy B');
  }
}

// 环境类
const context = name => {
  return strategies[name]();
}

// test
// 调用策略A，B
context('A');
context('B');
```
### 十五、解释器模式
```
/**
 * 解释器模式：提供了评估语言的语法或表达式的方式
 */

class Context {
  constructor(){
    this.__list = []; // 存放 终结符表达式
    this.__sum = 0; // 
  }

  get sum() {
    return this.__sum;
  }

  set sum(newVale) {
    this.__sum = newVale
  }

  add(expression){
    this.__list.push(expression)
  }

  get list(){
    return [...this.__list]
  }
}

class PlusExpression {
  interpret(context) {
    if(!(context instanceof Context)){
      throw new Error('TypeError')
    }
    context.sum = ++context.sum;
  }
}

class MinusExpression {
  interpret(context) {
    if(!(context instanceof Context)){
      throw new Error('TypeError')
    }
    context.sum = --context.sum;
  }
}

// test
const context = new Context();
context.add(new PlusExpression());
context.add(new PlusExpression());
context.add(new MinusExpression());

context.list.forEach(expression => {
  expression.interpret(context);
});

console.log(context.sum);
```
### 十六、订阅-发布模式
```
/**
 * 订阅-发布模式：定义了对象之间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖他的对象都得到了通知。
 * 与观察者模式对比：订阅发布模式中多了一个中间层，一个被抽离出来的信息调度中心。
 */

// 


const Event = {
  clientList: {},

  // 绑定事件监听
  listen(key, fn){
    if(!this.clientList[key]){
      this.clientList[key] = [];
    }
    this.clientList[key].push(fn);
    return true;
  },

  // 触发对应事件
  trigger() {
    const key = Array.prototype.shift.apply(arguments);
    fns = this.clientList[key];

    if(!fns || fns.length === 0){
      return false;
    }

    for(let fn of fns){
      fn.apply(null, arguments)
    }
    return true;
  },

  // 移除相关事件
  remove(key, fn){
    let fns = this.clientList[key];
    if(!fns || !fn){
      return false;
    }

    // 反向遍历移除指定事件函数
    for(let l = fns.length - 1; l >= 0; l--){
      let _fn = fns[l];
      if(_fn == fn){
        fns.splice(l,1);
      }
    }
    return true;
  }
};

// 为对象动态安装 发布-订阅 功能
const installEvent = obj => {
  for (let key in Event){
    obj[key] = Event[key];
  }
}

let salesOffices = {};
installEvent(salesOffices);

// 绑定自定义事件和回掉函数
salesOffices.listen(
  'event01',
  (fn1 = price => {
    console.log('Price is ',price,'at evnet01');
  })
);

salesOffices.listen(
  'event02',
  (fn2 = price => {
    console.log('Price is ',price,'at evnet02');
  })
)

salesOffices.trigger('event01',1000);
salesOffices.trigger('event02',2000);

salesOffices.remove('event01',fn1)

console.log(fn1);

console.log(salesOffices.clientList);
console.log(salesOffices.trigger('event01'));
```
### 十七、责任链模式
```
/**
 * 责任链模式：多个对象均有机会处理请求，从而解除发送者和接受者之间的耦合关系，这些对象连接成为链式结构，每个节点转发请求，知道有对西那个处理为止。
 * 核心：请求者不必知道是谁的哪个节点处理的请求。如果当前不符合终止条件，那么就把请求转发给下一个节点处理。
 * 优点：可以根据需求变动，任意向责任链中添加，删除节点对象。
 * 没有固定的“开始节点”，可以从任意节点开始。
 * 代价：责任链对大的代价就是每个节点带来的多余消耗。当责任链过长，很多节点只有传递作用，而不是真正的处理逻辑。
 */

 class Handler {
   constructor(){
     this.next = null;
   }

   setNext(handler){
     this.next = handler;
     return this;
   }
 }

 class LogHandler extends Handler {
   constructor(...props){
     super(...props);
     this.name = 'log'
   }

   handler(level, msg){
     if(level === this.name){
       console.log(`Log: ${msg}`); 
       return;
     }
     this.next && this.next.handler(...arguments)
   }
 }

 class WarnHandler extends Handler{
   constructor(...props){
     super(...props);
     this.name = 'warn'
   }

   handler(level, msg){
    if(level === this.name){
      console.log(`Warn: ${msg}`); 
      return;
    }
    this.next && this.next.handler(...arguments)
  }
 }

 class ErrorHandler extends Handler{
   constructor(...props){
     super(...props);
     this.name = 'error'
   }

   handler(level, msg){
    if(level === this.name){
      console.log(`Error: ${msg}`); 
      return;
    }
    this.next && this.next.handler(...arguments)
  }
 }

//  test

let logHandler = new LogHandler();
let warnHandler = new WarnHandler();
let errorHandler = new ErrorHandler();

// 设置链
logHandler.setNext(
  warnHandler.setNext(
    errorHandler
  )
);

logHandler.handler('warn','some error occur')
```
### 十八、迭代器模式
```
/**
 * 迭代器模式：提供一种方法顺序访问一个集合对象的各个元素，使用者不需要了解集合对象的底层实现。
 * 内部迭代器：封装的方法完全接手迭代过程，外部只需要一次调用。
 * 外部迭代器：用户必须显式请求迭代下一代元素。
 */

//  实现一个外部迭代器。
const Iterator = obj => {
  let current = 0;
  let next = () => (current += 1);
  let end = () => current >= obj.length;
  let get = () => obj[current];

  return {next,end,get};
}

let myIter = Iterator(['hello','java','script']);

while(!myIter.end()){
  console.log(myIter.get());
  myIter.next();
}
```

### 参考
[GodBMW](https://xin-tan.com/)