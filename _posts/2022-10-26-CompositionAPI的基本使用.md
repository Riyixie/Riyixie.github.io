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

![](https://gitee.com/wenn0/picgo/raw/master/img/202210270847407.png)

在`setup`中存在两种方式，分别为`reactive`和`ref`，但是二者的使用方式不同，如上

ref的解包只能是浅层解包

### readonly

本质为返回一个受劫持的代理对象，在set中不允许修改对象的值

![](https://gitee.com/wenn0/picgo/raw/master/img/202210270902463.png)

正常对于如果不想自身对象被其他组件修改，则返回一个只读对象；但是当需要进行修改时，只需要将数据本身传递给要修改的，之后的只读对象会自动同步

### Reactive判断API

![](https://gitee.com/wenn0/picgo/raw/master/img/202210270916430.png)

### toRefs toRef

`toRefs`适用于多个属性的性能开销，`toRef`适用于单个属性，减小性能开销；二者传入的必须是响应式对象

```js
import {toRefs,toRef} from 'vue'

const obj = reactive({name:'xi',age:20})
const {age,name} = toRefs(obj) //此时进行双向链接，任何一方进行修改都会进行修改
//const increment=()=>{obj.age++}
const increment = ()=>{age.value++}  //二者都会改变双方的数据
//const {name,age} = obj //如果为这样，则不会建立链接，不会进行修改
const name = toRef(obj,"name")   //第一个参数为目标对象，第二个参数为目标属性
```

### Ref其它API

![](https://gitee.com/wenn0/picgo/raw/master/img/202210270944759.png)

### customRef

简单节流Ref实现

```js
import {customRef} from 'vue'
export default function(value，delay=1000){
    let timer = null
    return customRef((track,trigger)=>{
        get(){
            track()  //收集依赖
            return value
        }
        set(newValue){
            timer = setTimeout(()=>{
                value = newValue
                trigger()
            },delay)
        }
    })
}
```

### computed watchEffect

computed返回的是一个ref对象

```js
//1.传入一个getter函数
import {ref,computed} from 'vue'
export default{
    setup(){
        const firstName = ref('Kobe')
        const lastName = ref('Bryant')
        //1.const fullName = computed(()=>firstName.value+' '+lastName.value)
        //2.传入一个实现getter/setter的对象
        const fullName = computed({
            get:()=>firstName.value+' '+lastName.value
            set:(newValue)=>{
            const names = newValue.split(' ')
            firstName.value = names[0]
            lastName.value = names[1]
        }
        
        const changeName=()=>{
            fullName.value = 'xxxx yyyy'  //fullName是一个ref对象
        } 
        })
    return {
    fullName,
    changeName
}
    }
}
```

```js
import {ref,watchEffect} from 'vue'
export default {
    setup(){
        const name = ref('why')
        const age = ref(20)
        //watchEffect返回一个函数，如果在达到某些条件时不在监听，则可以使其失效
        const stop = watchEffect((onInvalidate)=>{
            const timer = setTimout(()=>{
                console.log('网络请求成功')
            },200)
            //不论挂载、更新、销毁都会首先执行一次
            //如果发送网络请求，但是其值还没有返回，此时值更新，可以取消前一次的请求
            onInvalidate(()=>{
                //request.cancel()
                clearTimeout(timer)
            })
            //之后在执行后续代码
            console.log('name',name.value,'age',age.value)
            //在watchEffect中自动收集使用到的依赖，挂载时首先执行一次，添加到deps
            //如果不在watchEffect中使用ref对象，则不会添加依赖，后续即便数值更新也不会执行该处代码
        })
        
        const changeAge =()=>{
            age.value++
            if(age.value>25){  //当age的值大于25时，使watchEffect失效
                stop()
            }
        }
        
        return {
            name,
            age,
            changeAge
        }
    }
}
```

#### setup中使用ref

```vue
<template>
	<h2 ref='title'>  //绑定setup中的ref对象
        haha
    </h2>
</template>
<script>
import {ref,watchEfect} from 'vue' 
export default {
    setup(){
        const title = ref(null)  //设置一个ref对象用来专与ref元素进行绑定
        //watchEffect的第二个参数用于在合适的时机进行执行
        watchEffect(()=>{
            cconsole.log(title.value)
        },{
            flush:'pre' //默认值，提前的
            fulsh:'post' //挂载之后执行
            fulsh:'sync'  //二者同步，低效，很少使用
        })
        
        return {
            title
        }
    }
}
</script>
```

### watch 惰性的，只有值发生改变才会执行

```js
import {reactive,watch} from 'vue'
export default {
    setup(){
        const info = reactive({name:'xi'})
        
        //1.侦听watch时，传入一个getter函数
        watch(()=>info.name,(newValue,oldValue)=>{
            console.log(newValue,oldValue)
        })
        
        const changeName = ()=>{
            info.name = 'why'
        }
        
        return {
            info,
            changeName
        }
    }
}
```

##### 2.传入一个可响应式对象

```js
import {reactive,watch} from 'vue'
export default {
    setup(){
        const info = reactive({name:'xi'})
    //情况一，传入reactive对象，返回的也是一个reactive对象，此时新旧value一致
        watch(info,(newValue,oldValue)=>{
            console.log(newValue,oldValue)  //返回同一个reactive对象
            console.log(newValue === oldValue)  //true
        })
    //如果希望newValue和oldValue是一个普通的对象，则需要解构
        watch(()=>({...info}),(newValue,oldValue)=>{
            console.log(newValue,oldValue)  //此时为两个普通的对象
        })
    }
}
```

```js
import {ref,watch} from 'vue'
export default {
    setup(){
        const info = ref({name:'xi'})
        const name = ref('xo')
        //如果传入的是一个ref复杂对象，则监听的二者也是代理对象，如果是ref普通对象，则直接返回一对应的值
        watch(info.value,(newValue,oldValue)=>{
            console.log(newValue,oldValue)  //此时返回的是一个Proxy对象
        })
        watch(name,(newValue,oldValue)=>{
            console.log(newValue,oldValue)   //返回值
        })
    }
}
```

```js
//侦听多个数据源使用数组，返回的也是一个数组，按位
import {ref,watch} from 'vue'
export default {
    setup(){
        const name = ref('hi')
        const age = ref(20)
        watch([name,age],(newValue,oldValue)=>{
            console.log(newValue,oldValue)  //['hahah',30] ['hi',20]
        })
    }
}
```

```js
//使用深度侦听
import { watch,reactive,res} from 'vue'
export default {
    setup(){
        const obj = reactive({name:'xi',firiends:{name:'hi'}})
        //如果传入的是一个可响应式对象，则默认开启深度侦听，不论是ref还是reactive
        watch(obj,(newValue,oldValue)=>{
            console.log(newValue,oldValue)
        })
    }
}
```

```js
//使用深度侦听
import { watch,reactive,res} from 'vue'
export default {
    setup(){
        const obj = reactive({name:'xi',firiends:{name:'hi'}})
        //如果传入的是一个普通对象，则需要手动开启
        watch(()=>({...obj}),(newValue,oldValue)=>{
            console.log(newValue,oldValue)
        },{
            deep:true,   //深度监听
            immediate:true   //挂载也执行一次
        })
    }
}
```

### setup中的生命周期以onX监听即可

### Provide函数 inject

与vue2的使用一致，不过依旧为从vue中导入对应的函数即可

尽量遵循单向数据流，在子组件中不要修改父组件的值，如果要修改，则emit事件，使父组件进行修改

```js
import {provide,ref,readonly} from 'vue'
export default {
    setup(){
        let counter = ref(100)
        //不在自身使用readonly，传递时修改为只读，不会影响服父组件的使用
        provide('counter',readonly(counter))
        
        const increment=()=>counter.value++
    }
}
```

使用composition有利于抽取出hook，这样可以使代码整洁

![](https://gitee.com/wenn0/picgo/raw/master/img/202210271449049.png)

如果想添加对应的，则只需要继续抽离即可，使用则调用对应的函数

**修改title的hook**

![](https://gitee.com/wenn0/picgo/raw/master/img/202210271503809.png)

**实时显示滚动的x与y**

![](https://gitee.com/wenn0/picgo/raw/master/img/202210271507783.png)

**实时显示鼠标的移动位置**

![](https://gitee.com/wenn0/picgo/raw/master/img/202210271515930.png)

**本地存储hook**

```js
export default function (key,value) {
  const data = ref(value)
  if (value) {
    window.localStorage.setItem(key,JSON.stringify(value))
  } else {
    data.value = JSON.parse(window.localStorage.getItem(key))
  }

  watch(data, (newValue) => {
    window.localStorage.setItem(key,JSON.stringify(newValue))
  })

  return data
}
```

### setpup的顶层编写方式 

```vue
<template>
  <div>
    <h2>{{counter}}</h2>
    <button @click="increment">阿牛</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const counter = ref(0)
const increment=()=>counter.value++
</script>
```

### h函数

h函数的简单编写方式

```js
<script>
import {h} from 'vue';

export default {
  render () {
    return h('h2',{class:'title'},'Hello Render')
  }
}
</script>
```

简单计数器案例实现

```vue
<script>
import { h } from 'vue'

export default {
	data() {
		return {
			counter: 0,
		}
	},
	render() {
		return h('div', { class: 'app' }, [
			h('h2', null, `当前计数${this.counter}`),
			h(
				'button',
				{
					onClick: () => {
						this.counter++
					},
				},
				'+1'
			),
			h(
				'button',
				{
					onClick: () => {
						this.counter--
					},
				},
				'-1'
			),
		])
	},
}
</script>

```

setup实现

```vue
<script>
import { h, ref } from 'vue'

export default {
	setup() {
		const counter = ref(0)

		return () => {
			return h('div', { class: 'app' }, [
				h('h2', null, `当前计数${counter.value}`),
				h(
					'button',
					{
						onClick: () => {
							counter.value++
						},
					},
					'+1'
				),
				h(
					'button',
					{
						onClick: () => {
							counter.value--
						},
					},
					'-1'
				),
			])
		}
	},
}
</script>
```

### 在vue中编写jsx的方式

```js
<script>
export default {
	data() {
		return {
			counter: 0,
		}
	},

	render() {
		const increment = () => {
			this.counter++
		}
		return (
			<div>
				<h2>当前计数</h2>
				<h2>{this.counter}</h2>
				<button onClick={increment}>按钮</button>
			</div>
		)
	},
}
</script>
```

