---
title: Vue3笔记（入门使用，setup，响应式数据，计算属性）
published: 2025-07-07
updated: 2025-07-07
description: 'Vue的入门使用，setup，响应式数据（ref、reactive），计算属性'
image: ''
tags: [Vue]
category: 'Frame'
draft: false 
---

# Vue3笔记

## 快速入门

通过如下指令快速创建vite项目

```
npm create vue@latest
```



通过如下指令安装npm指定的相关依赖

```
npm install
```



main.ts

```tsx
import './assets/main.css'

import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

createApp是创建应用

App是组件（根），写的后续vue组件都是安装到App上

mount在对应的id挂在，这个app在index.html中



以下是选项式编写的vue

App.vue

在App中导入并注册Person组件

```vue
<script lang="ts">
import Person from './components/Person.vue'
export default {
  name: 'App',
  components: { Person },
}
</script>

<template>
  <div class="app">
    <h1>Hello World</h1>
    <Person />
  </div>
</template>

<style scoped>
.person {
  width: 100px;
  height: 100px;
  background-color: #2e5aae;
}
</style>
```

Person组件

```vue
<script lang="ts">
export default {
  data() {
    return {
      name: '张三',
      age: 18,
      gender: '男',
    }
  },
  methods: {
    showSex() {
      alert(this.gender)
    },
    changeName() {
      this.name = '李四'
    },
    changeAge() {
      this.age = this.age + 1;
    },
  },
}
</script>

<template>
  <div class="person">
    <h2>姓名:{{ name }}</h2>
    <h2>年龄:{{ age }}</h2>
    <button @click="showSex">查看性别</button>
    <button @click="changeName">修改名字</button>
    <button @click="changeAge">修改年龄</button>
  </div>
</template>

<style scoped>
.person {
  width: 300px;
  height: 100px;
  background-color: #265bc3;
}
</style>

```



## setup

以下是组件式的vue编程

setup是Vue3中的一个新的配置项，值是一个函数，组件中用到的所有的数据、方法、计算属性、监视等等均配置在setup中

```vue
<script lang="ts">
export default {
  setup(){
    let name = '张三'
    let age = 18
    const sex = '男'
    //方法
    const showSex = () => {
      alert(sex)
    }
    const changeName = () => {
      console.log('修改姓名')
      name = '李四'
    }
    const changeAge = () => {
      age = 20
    }
    return {name,age,sex,showSex,changeName,changeAge}
  }
}
</script>

<template>
  <div class="person">
    <h2>姓名:{{ name }}</h2>
    <h2>年龄:{{ age }}</h2>
    <button @click="showSex">查看性别</button>
    <button @click="changeName">修改名字</button>
    <button @click="changeAge">修改年龄</button>
  </div>
</template>

<style scoped>
.person {
  width: 300px;
  height: 100px;
  background-color: #265bc3;
}
</style>
```

这里的变量不是响应式的，通过按钮更新的数据不会更改回页面的数据



setup的返回值就是将数据交出去，或者可以使用箭头函数将结果返回给页面，可以直接返回函数渲染

```vue
return ()=>'可以'
```



通过setup语法糖简化写法

```vue
<script setup lang="ts">
let name = '张三'
let age = 18
const sex = '男'
//方法
const showSex = () => {
  alert(sex)
}
const changeName = () => {
  console.log('修改姓名')
  name = '李四'
}
const changeAge = () => {
  age = 20
}
</script>

<template>
  <div>
    <h2>姓名:{{ name }}</h2>
    <h2>年龄:{{ age }}</h2>
    <button @click="showSex">查看性别</button>
    <button @click="changeName">修改名字</button>
    <button @click="changeAge">修改年龄</button>
  </div>
</template>

<style scoped></style>
```



## 响应式数据

### 基本类型

使用ref

```vue
<script setup lang="ts">
import { ref } from 'vue'

const name = ref('张三')
const age = ref(18)
const sex = '男'
//方法
const showSex = () => {
  alert(sex)
}
const changeName = () => {
  console.log('修改姓名')
  name.value = '李四'
}
const changeAge = () => {
  age.value = 20
}
</script>

<template>
  <div>
    <h2>姓名:{{ name }}</h2>
    <h2>年龄:{{ age }}</h2>
    <button @click="showSex">查看性别</button>
    <button @click="changeName">修改名字</button>
    <button @click="changeAge">修改年龄</button>
  </div>
</template>

<style scoped></style>
```

### 对象类型

使用reactive

```vue
<script setup lang="ts">
import { reactive } from 'vue'
const car = reactive({
  brand: '奔驰',
  price: 1000000,
})
const games = reactive([
  { id: 1, name: '英雄联盟' },
  { id: 2, name: '绝地求生' },
])
function changePrice() {
  car.price += 100000
}
function changeName() {
  games[0].name = '王者荣耀'
}
</script>

<template>
  <div>
    <h1>汽车品牌：{{ car.brand }}</h1>
    <h1>汽车价格：{{ car.price }}</h1>
  </div>
  <button @click="changePrice">修改汽车价格</button>
  <ul>
    <li v-for="game in games" :key="game.name">{{ game.name }}</li>
  </ul>
  <button @click="changeName">修改名字</button>
</template>

<style scoped></style>
```

事实上ref也可以定义响应式对象数据

```javascript
<script setup lang="ts">
import { reactive, ref } from 'vue'
const car = reactive({
  brand: '奔驰',
  price: 1000000,
})
const games = ref([
  { id: 1, name: '英雄联盟' },
  { id: 2, name: '绝地求生' },
])
function changePrice() {
  car.price += 100000
}
function changeName() {
  games.value[0].name = '王者荣耀'
}
</script>
```

方法实现上games取出value使用即可



如果需要通过json格式的数据修改对象内容完成响应式

需要使用到Object.assign脚本

```javascript
const car = reactive({
  brand: '奔驰',
  price: 1000000,
})
function changeCar() {
  // car.brand = '宝马'
  // car.price = 2000000
  Object.assign(car, { brand: '宝马', price: 2000000 })
}
```

如果是ref响应，可以使用value直接替换

```javascript
function changeCar() {
  // car.brand = '宝马'
  // car.price = 2000000
  //
  car.value = {
    brand: '宝马',
    price: 2000000,
  }
}
```



### toRefs使用

通过toRefs使得新建的变量为ref响应式

```javascript
const car = reactive({
  brand: '奔驰',
  price: 1000000,
})
const {brand, price} = toRefs(car)

function changeCar() {
  brand.value = '宝马'
  price.value = 2000000
}
```



### toRef

```javascript
const name = toRef(car, 'brand')

function changeName() {
  name.value += "~"
}
```



## 计算属性

相比较于方法调用而言，计算属性存在缓存，效率更高

这里的计算属性fullName只读，不可以修改

```javascript
<script setup lang="ts">
import { ref, computed} from 'vue'
const firstName = ref('张')
const lastName = ref('三')

const fullName = computed(()=>{
  return firstName.value + lastName.value
  //在这里可以写具体的逻辑代码
  //计算属性中属性发生变化就重新计算
})

</script>

<template>
  <div>
    姓：<input type="text" v-model="firstName" /><br />
    名：<input type="text" v-model="lastName" /><br />
    全名：<span>{{fullName}}</span>
  </div>
</template>

<style scoped></style>
```

要修改为可写可以使用如下写法

```javascript
const fullName = computed({
  get(){
    return firstName.value + lastName.value
  },
  set(val){
    console.log("set",val)
  }
})

function changeFullName() {
  fullName.value = "testName"
}
```

要是想要让全名更新两个前名后名可以使用set中val接收修改的参数

```vue
<script setup lang="ts">
import { ref, computed} from 'vue'
const firstName = ref('张')
const lastName = ref('三')
const fullName = computed({
  get(){
    return firstName.value + "-" + lastName.value
  },
  set(val){
    const nameArr = val.split("-")
    firstName.value = nameArr[0]
    lastName.value = nameArr[1]
  }
})

function changeFullName() {
  fullName.value = "li-si"
}

</script>

<template>
  <div>
    姓：<input type="text" v-model="firstName" /><br />
    名：<input type="text" v-model="lastName" /><br />
    全名：<span>{{fullName}}</span>
    <button @click="changeFullName">修改全名为li-si</button>
  </div>
</template>

<style scoped></style>
```
