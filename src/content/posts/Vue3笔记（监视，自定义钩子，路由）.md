---
title: Vue3笔记（监视，自定义钩子，路由）
published: 2025-07-08
updated: 2025-07-08
description: '监视，ref属性，接口，props，自定义钩子，路由配置'
image: ''
tags: [Vue]
category: 'Frame'
draft: false 
---

# Vue3笔记

## watch

用于监视数据，在特定条件下执行逻辑

可以监视四种情况

:::note

1.ref定义的数据

2.reactive定义的数据

3.函数返回一个值（getter函数）

4.一个包含上述内容的数组

:::

### 情况一

watch监视ref定义的基本类型数据

定义变量存储watch再次调用可以终止监听

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'
const sum = ref(0)
function culSum() {
  sum.value += 1
}
const myWatch = watch(sum, (newVal, oldVal) => {
  console.log(newVal, oldVal)
  if (newVal >= 8) {
    myWatch()
  }
})
</script>

<template>
  <div>
    <h2>当前sum值：{{ sum }}</h2>
    <button @click="culSum">加一</button>
  </div>
</template>

<style scoped></style>
```



### 情况二

监视ref定义的对象类型数据

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'
const person = ref({
  name: '张三',
  age: 25,
})
function addAge() {
  person.value.age += 1
}
function changeName() {
  person.value.name += "~"
}
function change() {
  person.value = {
    name: '李四',
    age: 20,
  }
}
//监视的是person对象，属性变化不会被监视，地址变化才会触发
watch(person, (newVal, oldVal) => {
  console.log("监视对象变化",newVal, oldVal)
})

</script>

<template>
  <div>
    <h2>姓名：{{ person.name }}</h2>
    <h2>姓名：{{ person.age }}</h2>
    <button @click="addAge">年龄加一</button>
    <button @click="changeName">更改名字</button>
    <button @click="change">换人</button>

  </div>
</template>

<style scoped></style>
```

如果需要监视内部属性的变化可以加一个参数

```
//监视的是person对象，以及属性的变化
watch(person, (newVal, oldVal) => {
  console.log("监视对象变化",newVal, oldVal)
},{deep:true})
```



### 情况三

watch reactive定义的对象数据

默认开启deep:true深度监视，而且关不掉

```vue
<script setup lang="ts">
import { reactive, watch } from 'vue'
const person = reactive({
  name: '张三',
  age: 25,
})
function addAge() {
  person.age += 1
}
function changeName() {
  person.name += "~"
}
function change() {
  Object.assign(person,{
    name: '李四',
    age: 20,
  })
}
//监视的是person对象，属性变化不会被监视，地址变化才会触发
watch(person, (newVal, oldVal) => {
  console.log("监视对象变化",newVal, oldVal)
})

</script>

<template>
  <div>
    <h2>姓名：{{ person.name }}</h2>
    <h2>姓名：{{ person.age }}</h2>
    <button @click="addAge">年龄加一</button>
    <button @click="changeName">更改名字</button>
    <button @click="change">换人</button>

  </div>
</template>

<style scoped></style>
```



### 情况四

监视对象中的单个对象

传入getter函数即可

```javascript
const person = reactive({
  name: '张三',
  age: 25,
})
function addAge() {
  person.age += 1
}
function changeName() {
  person.name += "~"
}
function change() {
  Object.assign(person,{
    name: '李四',
    age: 20,
  })
}


//监视对象中的单个属性变化
function getPersonName() {
  return person.name
}

watch(getPersonName, (newName) => {
  console.log("监视对象发生改变",newName)
})
```

也可以写为箭头函数

```javascript
watch(()=>{return person.name}, (newName) => {
  console.log("监视对象发生改变",newName)
})
```



### 情况五

监视上述多个数据

```javascript
//监视对象中的多个属性变化，返回一个getter方法数组
watch([()=>person.name,()=>person.age], (newName) => {
  console.log("监视对象发生改变",newName)
})
```



### watchEffect

他会立即运行一个函数，同时响应式的追踪其依赖，并在依赖更改的时候重新执行该函数

一个实际使用场景

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'
const temp = ref(10)
const height = ref(0)
function addTemp() {
  temp.value += 10
}
function addHeight(){
  height.value += 10
}
const warnWatch = watch([temp,height], (value) => {
  console.log(value)
  const [newTemp, newHeight] = value
  if(newTemp >= 60 || newHeight >= 80) {
    console.log("发送请求")
    warnWatch();
  }
})
</script>

<template>
  <div>
    <h2>当前水温{{temp}} 水位{{height}}</h2>
    <button @click="addTemp">点我温度加一</button>
    <button @click="addHeight">点我高度加一</button>
  </div>
</template>

<style scoped></style>
```

用watchEffect优化

```javascript
watchEffect(()=>{
  if(temp.value >= 60 || height.value >= 80){
    console.log(temp.value, height.value)
  }
})
```



## 标签的ref属性

作用是用于注册模板引用

用在普通的标签上获取到的是dom节点

用在组件标签上，获取的是组建的实例对象



使用场景

通过ref的容器可以区分不同页面中的同名容器

```vue
<script setup lang="ts">
import { ref } from 'vue'
function showLog(){
  console.log(h2.value);
}
//创建一个h2，用于存储ref标记的内容
const h2 = ref()
</script>

<template>
  <div>
    <h1>中国</h1>
    <h2 ref="h2">北京</h2>
    <h3>海淀区</h3>
    <button @click="showLog">点我输出h2元素</button>
  </div>
</template>

<style scoped></style>
```



## 接口

在和后端通信的时候需要使用到

```tsx
export interface PersonInter {
  id: string;
  name: string;
  age: number;
}
```

```javascript
import { type PersonInter } from '@/types'

const person: PersonInter = {
  id: 'ast_sdf',
  name: '李四',
  age: 10,
}
```

泛型对数组中的每一个元素进行接口约束

```javascript
const personList: Array<PersonInter> = [
  {
    id: 'ast_sdf',
    name: '李四',
    age: 10,
  },
  {
    id: 'ast_sdf',
    name: '李四',
    age: 10,
  },
  {
    id: 'ast_sdf',
    name: '李四',
    age: 10,
  },
]
```



通过自定义类型也可以实现数组的约束

```tsx
export type Persons = Array<PersonInter>
```

这样也行

```javascript
import { type Persons } from '@/types'

const personList: Persons = [
  {
    id: 'ast_sdf',
    name: '李四',
    age: 10,
  },
  {
    id: 'ast_sdf',
    name: '李四',
    age: 10,
  },
  {
    id: 'ast_sdf',
    name: '李四',
    age: 10,
  },
]
```



## props

通过props可以进行组件之间参数的传递

```vue
<script setup lang="ts">
import TestComponent from '@/components/TestComponent.vue'
import { reactive } from 'vue'
import { type Persons } from '@/types'
const personList = reactive<Persons>([
  { id: 'asd', name: '李四', age: 19 },
  { id: 'as', name: '张三', age: 19 },
  { id: 'a', name: '王五', age: 19 },
])
</script>

<template>
  <div>
    <TestComponent a="App参数" :list="personList" />
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

被使用的模块可以使用传递的参数

```vue
<script setup lang="ts">
import { defineProps } from 'vue'

const props = defineProps(['a', 'list'])
</script>

<template>
  <div>
    <h2>{{ props.a }}</h2>
    <ul>
      <li v-for="person in list" :key="person.id">{{ person.name }}</li>
    </ul>
  </div>
</template>

<style scoped></style>
```



## 自定义Hooks

以下组件完成了一个按钮实现sum自增，另一个访问网站提供图片打印

```vue
<script setup lang="ts">
import axios from 'axios'
import { reactive, ref } from 'vue'
const sum = ref(0)
const url = 'https://picsum.photos/200/300'
const photos = reactive([url])


function add() {
  sum.value += 1
}
async function getPhoto(){
  try {
    const result = await axios.get(url)
    photos.push(url)
  } catch (error) {
    alert(error)
  }
}
</script>

<template>
  <div>
    <h2>{{ sum }}</h2>
    <button @click="add">点我加一</button>
    <hr />
    <img v-for="(photo, index) in photos" :src="photo" :key="index" />
    <button @click="getPhoto">再来一张</button>
  </div>
</template>

<style scoped>
img {
  height: 150px;
  margin-right: 10px;
}
</style>
```

缺点是变量和方法没有按照功能区分，可以通过自定义Hooks增加可读性

```tsx
import { ref } from 'vue'

export default function(){
  const sum = ref(0)
  function add() {
    sum.value += 1
  }
  return {sum,add}
}
```

```tsx
import axios from 'axios'
import { reactive } from 'vue'
const url = 'https://picsum.photos/200/300'
const photos = reactive([url])

export default function() {
  async function getPhoto(){
    try {
      const result = await axios.get(url)
      photos.push(url)
    } catch (error) {
      alert(error)
    }
  }
  //向外提供对象
  return{photos,getPhoto}
}
```

在需要用到钩子的组件中解构取出钩子返回对象以及方法即可使用

```vue
<script setup lang="ts">
import useSum from '@/hooks/useSum.ts'
import usePhotos from '@/hooks/usePhotos.ts'
const {sum, add} = useSum()
const {photos, getPhoto} = usePhotos()
</script>

<template>
  <div>
    <h2>{{ sum }}</h2>
    <button @click="add">点我加一</button>
    <hr />
    <img v-for="(photo, index) in photos" :src="photo" :key="index" />
    <button @click="getPhoto">再来一张</button>
  </div>
</template>

<style scoped>
img {
  height: 150px;
  margin-right: 10px;
}
</style>
```



## 路由

router

### 基本使用

1.确认导航区，展示区

```vue
<script setup lang="ts">

</script>

<template>
  <div class="app">
    <h2>Vue路由测试</h2>
    <div class="navigate">
      <a href="/">首页</a>
      <a href="/">新闻</a>
      <a href="/">关于</a>
    </div>
    <div class="main-context">
      展示区域
    </div>
  </div>
</template>

<style scoped>
    <!--省略css样式-->
</style>
```

2.安装路由器

```
npm i vue-router
```

3.指定路由规则

```tsx
//创建路由器，并暴露其
//引入createRouter
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/components/Home.vue'
import About from '@/components/About.vue'
import News from '@/components/News.vue'
//创建路由器
const router = createRouter({
  history: createWebHistory(),
  routes:[
    {
      path:'/home',
      component:Home
    },
    {
      path:'/about',
      component:About
    },
    {
      path:'/news',
      component:News
    },
  ]
})
//暴露路由
export default router
```

4.形成对应的组件

省略组件

在App.vue中引入RouterView，在展示区域通过RouterView进行组建的替换

```
<RouterView />
```

```vue
<script setup lang="ts">
  import {RouterView} from 'vue-router'
</script>

<template>
  <div class="app">
    <h2>Vue路由测试</h2>
    <div class="navigate">
<!--      <a href="/home">首页</a>-->
<!--      <a href="/news">新闻</a>-->
<!--      <a href="/about">关于</a>-->
      <RouterLink to="/home">首页</RouterLink>
      <RouterLink to="/news">新闻</RouterLink>
      <RouterLink to="/about">关于</RouterLink>
    </div>
    <div class="main-context">
      <RouterView />
    </div>
  </div>
</template>

<style scoped>
</style>
```



### 嵌套路由

有的时候我们需要在一个组件的展示区中嵌套一个展示区，这个时候我们就会需要使用到嵌套路由

```
{
  path:'/news',
  component:News,
  children:[
    {
      path:'detail',
      component:Detail
    },
  ]
},
```

注意在使用RouterLink的时候to指定news下的detail

```vue
<template>
  <div>
    <div>News</div>
    <ul>
      <li v-for="news in newsList" :key="news.id">
        <RouterLink to="/news/detail">{{news.title}}</RouterLink>
      </li>
    </ul>
    <div class="news-content">
      <RouterView></RouterView>
    </div>
  </div>
</template>
```

这个时候，我们没有传入新闻的id，还不能把具体的内容加载到页面中，需要从News页面点击传入的的参数加载不同的内容



### Query参数传递

通过query进行参数传递，方式类似于get请求

```vue
<template>
  <div>
    <div>News</div>
    <ul>
      <li v-for="news in newsList" :key="news.id">
        <RouterLink :to="`/news/detail?id=${news.id}&title=${news.title}&content=${news.content}`">{{news.title}}</RouterLink>
      </li>
    </ul>
    <div class="news-content">
      <RouterView></RouterView>
    </div>
  </div>
</template>
```

以下这样的写法也是可以的

```vue
<template>
  <div>
    <div>News</div>
    <ul>
      <li v-for="news in newsList" :key="news.id">
<!--        <RouterLink :to="`/news/detail?id=${news.id}&title=${news.title}&content=${news.content}`">{{news.title}}</RouterLink>-->
        <RouterLink
          :to="{
          path:'/news/detail',
          query:{
            id:news.id,
            title:news.title,
            content:news.content
          }
          }"
        >
          {{news.title}}
        </RouterLink>
      </li>
    </ul>
    <div class="news-content">
      <RouterView></RouterView>
    </div>
  </div>
</template>
```

Detai页面通过引入useRoute进行参数的接收

```vue
<script setup lang="ts">
import { useRoute } from 'vue-router'
const route = useRoute();
console.log(route)
</script>

<template>
  <ul class="news-context">
    <li>编号：{{route.query.id}}</li>
    <li>标题：{{route.query.title}}</li>
    <li>内容：{{route.query.content}}</li>
  </ul>
</template>

<style scoped>

</style>
```



### params参数

在路由中参数提前占位

```
{
  path:'/news',
  component:News,
  children:[
    {
      path:'detail/:id/:title/:content',
      component:Detail
    },
  ]
},
```

news页面传递参数写为

```vue
<template>
  <div>
    <div>News</div>
    <ul>
      <li v-for="news in newsList" :key="news.id">
        <RouterLink :to='`/news/detail/${news.id}/${news.title}/${news.content}`'>
          {{ news.title }}
        </RouterLink>
      </li>
    </ul>
    <div class="news-content">
      <RouterView></RouterView>
    </div>
  </div>
</template>
```

detail页面route对象取params对象，获取params参数

```vue
<script setup lang="ts">
import { useRoute } from "vue-router";
const route = useRoute();
console.log(route);
</script>

<template>
  <ul class="news-context">
    <li>编号：{{route.params.id}}</li>
    <li>标题：{{route.params.title}}</li>
    <li>内容：{{route.params.content}}</li>
  </ul>
</template>

<style scoped>

</style>
```



### props配置

路由规则的props配置

在路由下打开props配置，当在News页面中使用到Detail组建的时候，就会在组建标签中附带上对应的占位参数（params）

```
children:[
  {
    path:'detail/:id/:title/:content',
    component:Detail,
    props:true
  },
]
```

Detail中通过defineProps直接用即可，非常方便

```vue
<script setup lang="ts">
defineProps(['id', 'title', 'content'])
</script>

<template>
  <ul class="news-context">
    <li>编号：{{id}}</li>
    <li>标题：{{title}}</li>
    <li>内容：{{content}}</li>
  </ul>
</template>

<style scoped>

</style>
```



### replace属性

浏览器跳转页面有push和replace两种模式

默认的是push模式，这种模式回形成历史记录，方便回顾历史跳回

如果需要修改为replace模式，在对应的路由跳转标签上加入replace即可

```vue
<RouterLink replace to="/home">首页</RouterLink>
<RouterLink replace to="/news">新闻</RouterLink>
<RouterLink replace to="/about">关于</RouterLink>
```



### 编程式导航

脱离RouterLink实现路由跳转，在实际编程中编程式导航使用多余路由

如下是一个挂载三秒主页后完成页面跳转的示例

```vue
<script setup lang="ts">
import {onMounted} from 'vue'
import {useRouter} from 'vue-router'
const router = useRouter()
onMounted(() => {
  setTimeout(() => {
    //编写代码实现跳转
    router.push('/news')
  },3000)
})
</script>

<template>
  <div>
    Home
  </div>
</template>

<style scoped>

</style>
```



### 重定向

访问/路径时重定向至/home路径

```
routes:[
  {
    path:'/',
    redirect:'/home'
  },
```
