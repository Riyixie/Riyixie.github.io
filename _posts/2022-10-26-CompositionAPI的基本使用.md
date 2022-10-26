---
layout: mypost
title: CompositionAPI的基本使用
categories: [CompositionAPI]
---

### Mixin

![](https://gitee.com/wenn0/picgo/raw/master/img/202210262023512.png)

即使在vue中没有定义，但是依旧可以使用js文件导入并混入vue文件使用

##### Mixin的合并规则

1. 如果是data函数的返回值对象
   - 返回值对象默认情况下会合并
   - 如果data返回值对象的属性发生了冲突，那么会**保留组件自身的数据**
2. 生命周期
   - 合并到一个数组，两个生命周期都会被调用
3. 值为对象，如methods，components和directives，那么将会合并为一个对象
   - 如果对象的key与混入的key相同，那么会取组件对象的键值对

### 全局混入

在app对象中使用mixin进行注册

```js
app.mixin({...demoMix})
```

### extends 类mixin，只能继承对象的属性

![](https://gitee.com/wenn0/picgo/raw/master/img/202210262038714.png)

### setup

![](https://gitee.com/wenn0/picgo/raw/master/img/202210262048447.png)

### **setup没有绑定this**   setup的调用发生在data、computed或methods之前，所以无法在setup中被获取

![](https://gitee.com/wenn0/picgo/raw/master/img/202210262104526.png)

 ### Reactive API

![](https://gitee.com/wenn0/picgo/raw/master/img/202210262118048.png)

### RefAPI

![](https://gitee.com/wenn0/picgo/raw/master/img/202210262121483.png)
