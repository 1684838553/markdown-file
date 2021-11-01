### 类的数据类型是函数，类本身指向构造函数

- 类的定义

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`hello,${this.name}`);
  }
}
Animal.prototype.constructor === Animal; // true
let dog = new Animal("dog");
dog.sayHi(); // hello, dog
```

> constructor 方法用来创建和初始化对象，而且一个类中有且只能有一个 consctuctor 方法，默认为 constructor(){}。dog 就是 Animal 实例化的对象。

- 类的继承
  class 使用 extends 关键字来创建子类

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`hello, ${this.name}`);
  }
}

class Dog extends Animal {
  bark() {
    console.log(`喵喵喵`);
  }
}

let wangcai = new Dog("旺财");
wangcai.bark(); // 喵喵喵
wangcai.sayHi(); // hello, 旺财
```

> 但如果在子类中定义了 constructor 方法，必须先调用 super()才能使用 this，因为子类并没有 this 对象，而是继承父类的 this 对象，所以 super 必须在使用 this 关键字之前使用

class Animal {
constructor(name) {
this.name = name;
}
sayHi() {
console.log(`hello, ${this.name}`);
}
}

class Dog extends Animal {
constructor(name, sound) {
this.name = name;
this.sound = sound;
};
bark() {
console.log(this.sound);
}
}

let wangcai = new Dog('旺财', '喵喵喵');
wangcai.bark(); // referenceError

class Dog extends Animal {
constructor(name, sound) {
super(name);
this.sound = sound;
};
bark() {
console.log(this.sound);
}
}
let wangcai = new Dog('旺财', '喵喵喵');
wangcai.bark(); // 喵喵喵
super 不仅可以调用父类的 constructor 函数，还可以调用父类上的方法：

class Animal {
constructor(name) {
this.name = name;
}
sayHi() {
console.log(`hello, ${this.name}`);
}
}

class Dog extends Animal {
bark() {
super.sayHi();
}
}

let wangcai = new Dog('旺财');
wangcai.bark(); // hello, 旺财
静态方法
所有在类中定义的方法会被实例继承，但是有时候我们并不想所有实例都能继承某个方法，这时候，static 关键字就能达到你的目的，在声明方法前加上 static 关键字，这个方法就不会被实例继承，而是直接通过类来调用，它被叫做静态方法，如下：

class Animal {
constructor(name) {
this.name = name;
}
sayHi() {
console.log(`hello, ${this.name}`);
}
static bark() {
console.log('喵喵喵');
}
}
let dog = new Animal('dog');

dog.bark(); // TypeError
Animal.bark(); // 喵喵喵
静态方法虽然不能被实例继承，但是可以被子类继承，但是子类的实例依旧没有继承它：

class Animal {
static bark() {
console.log('喵喵喵');
}
}
class Dog extends Animal{

}
Dog.bark(); // 喵喵喵
let dog = new Dog();
dog.bark(); // TypeError
实例属性
实例属性必须定义在类方法中：

class Animal {
constructor(name) {
this.name = name; // 实例属性
}
}
静态属性和原型属性避必须在类的外面定义：

Animal.age = 18; // “静态属性”
Animal.prototype.sex = 'male'; // 原型属性
实例属性顾名思义，就是对象独有的方法/属性，静态属性就是位于 Class 本身的属性，但是 ES6 中明确说明 Class 只有静态方法，没有静态属性，原型属性也很容易理解也很容易看出，就是位于原型链上的属性/方法。


## 一、组件指南
[组件食用指南](https://www.cnblogs.com/danvic712/p/angular-components-guide.html)

### 组件与模板

1. 组件概念
2. 组件绑定语法
3. 数据绑定
4. 属性，样式绑定
5. 事件绑定

### 指令

1. 属性型指令
2. 结构型指令

### 管道

1. 模板表达式中的特殊运算符
2. 常用的管道函数

### 组件之间的通信

1. 输入属性与输出属性
2. 子组件获取父组件信息
3. 父组件获取子组件信息
4. 非父子组件之间的通信

### 组件的生命周期钩子函数

**angular生命周期钩子一共8个**

####  钩子函数执行顺序

1. 初始化页面

constructor -> ngOnChange()  (父组件给子组件传值)  -> ngOnInit() -> ngOnCheck() -> ngAfterContentChecked() -> ngAfterViewInit() -> ngAfterViewChecked()

2. 父组件给子组件传值改变

ngOnChange() -> ngOnCheck() -> ngAfterContentChecked() -> ngAfterViewChecked()

3. 卸载组件

ngOnDestroy()

#### 生命周期钩子函数

implements 实现，可以实现多个接口，用逗号分隔开

1. ngOnChanges()  父组件调用子组件，如果子组件绑定过输入属性，在ngOnInit()之前及属性值发生改变时，都会调用

2. ngOnInit()  初始化指令和组件，可在该生命周期钩子中请求服务端数据

3. ngDoCheck()

4. ngAfterContentInit()

5. ngAfterContentChecked()

6. ngAfterViewInit() 在该生命周期钩子中，能够直接操作dom

7. ngAfterViewChecked()

8. ngOnDestroy() 在 Angular 销毁指令或组件之前立即调用。例如，退出某个页面，保存该页面的数据


### 插槽

1. 单插槽
2. 多插槽
3. 有条件的插槽

```
<app-content>
    <p>你好，我是一个段落</p>

    <h6 header>你好，我是一个标题</h6>
</app-content>

<ng-content></ng-content>
<ng-content select="[header]"></ng-content>
```

### 二、问题：

1. 变量定义是，public和private的区别

### 三、处理异步
```
//在app.modules.js
import { HttpClientModule} from '@angular/common/http'
@NgModule({
  declarations: [
  ],
  imports: [
    HttpClientModule   
  ],
})
import { Component, OnInit } from '@angular/core';
import {HttpClient,HttpHeaders} from '@angular/common/http'

@Component({
  selector: 'app-content',
  templateUrl: './content.component.html',
  styleUrls: ['./content.component.less']
})
export class ContentComponent implements OnInit {

  constructor(public http:HttpClient) { }

  public list:Array<any> = []

  ngOnInit(): void {
    this.getData()
  }

  getData(){
    this.http.get('assets/data.json').subscribe((res:any)=>{
      console.log(res,'我是异步请求结果')
      this.list = res
    })
  }

  postData(){
    const httpOptions = {headers:new HttpHeaders({"content-Type":'application/json'}) }
    const api = ''
    this.http.post(api,{"userName":'张三'},httpOptions).subscribe((res:any)=>{
      console.log(res,'发送一个post请求')
    })
  }

}

import { Component, OnInit } from '@angular/core';
import {HttpClient,HttpHeaders} from '@angular/common/http'

@Component({
  selector: 'app-content',
  templateUrl: './content.component.html',
  styleUrls: ['./content.component.less']
})
export class ContentComponent implements OnInit {

  constructor(public http:HttpClient) { }

  public list:Array<any> = []

  ngOnInit(): void {
    this.getData()
  }

  getData(){
    this.http.get('assets/data.json').subscribe((res:any)=>{
      console.log(res,'我是异步请求结果')
      this.list = res
    })
  }

  postData(){
    const httpOptions = {headers:new HttpHeaders({"content-Type":'application/json'}) }
    const api = ''
    this.http.post(api,{"userName":'张三'},httpOptions).subscribe((res:any)=>{
      console.log(res,'发送一个post请求')
    })
  }

}
```



angular
前端安全
js原理
JSON必知必会
http
nodejs
小程序


