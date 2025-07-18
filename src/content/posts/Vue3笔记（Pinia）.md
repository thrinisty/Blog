---
title: Vue3笔记（Pinia，组件通信）
published: 2025-07-09
updated: 2025-07-09
description: 'Vue Pinia，组件通信'
image: ''
tags: [Vue]
category: 'Frame'
draft: false 
---

# Vue3笔记

今天应该可以把Vue结束

## Pinia

### 环境准备

集中式状态（数据）管理

把所有需要集中管理的数据放在同一个位置

通过以下指令安装pinia

```
npm i pinia
```



在main.js中引入Pinia

```js
import './assets/main.css'

import { createApp } from 'vue'
import App from './App.vue'
//引入Pinia
import {createPinia} from "pinia";
//创建Pinia
const app = createApp(App);
const pinia = createPinia()
app.use(createPinia())
app.mount('#app')
```



我们先准备两个组件

其中sum的值会在其他组件中使用到，我们从count.ts中获取数据

```vue
<script setup lang="ts">
import {reactive, ref} from "vue";
  import axios from "axios"
  const count = ref(4)
  const talkList = reactive([
    {id:1, title:':我没什么事儿，就想听听你的声音'},
    {id:2, title:'您好，您有新的恋爱订单，请及时收取我'},
    {id:3, title:'我们，是我遇见的最美好的字眼'},
  ])
  async function addTalk(){
    const response = await axios.post('https://api.uomg.com/api/rand.qinghua?format=json');
    const obj = {id:count.value, title:response.data.content};
    count.value++
    talkList.push(obj);
  }
</script>

<template>
  <div>
    <button @click="addTalk">获取语句</button>
    <ul>
      <li v-for="talk in talkList" :key="talk.id">{{talk.id}}:{{talk.title}}</li>
    </ul>
  </div>
</template>

<style scoped>

</style>
```

```vue
<script setup lang="ts">
import {ref} from "vue";
const sum = ref(10)
const n = ref(1)
function addSum(){
  sum.value += n.value
}
function subSum(){
  sum.value -= n.value
}
</script>

<template>
  <div>当前sum为：{{sum}}</div><br/>
  <select v-model.number="n">
    <option value="1">1</option>
    <option value="2">2</option>
    <option value="3">3</option>
  </select><br/>
  <button @click="addSum">加n</button>
  <button @click="subSum">减n</button>
</template>

<style scoped>

</style>
```



### 更改数据

第一种更改数据的方式

更改为从全局的/store/count中获取sum值

```js
import {defineStore} from "pinia";
export const useSumStore = defineStore('sum',{
    state(){
        return {
            sum:60
        }
    }
})
```

注意在解构的时候保持响应式，通过toRefs实现（其实storeToRefs会更好一点）

```vue
<script setup lang="ts">
import {ref, toRefs} from "vue";
import {useSumStore} from '@/store/count'

const sumStore = useSumStore();
const {sum} = toRefs(sumStore);
const n = ref(1)
function addSum(){
  sum.value += n.value
}
function subSum(){
  sum.value -= n.value
}
</script>

<template>
  <div>当前sum为：{{sumStore.sum}}</div><br/>
  <select v-model.number="n">
    <option value="1">1</option>
    <option value="2">2</option>
    <option value="3">3</option>
  </select><br/>
  <button @click="addSum">加n</button>
  <button @click="subSum">减n</button>
</template>

<style scoped>

</style>
```

talk

```js
import {defineStore} from "pinia";
import {reactive} from "vue";
export const useTalkStore = defineStore('talks',{
    state(){
        return {
            talkList:[
                {id:1, title:':我没什么事儿，就想听听你的声音'},
                {id:2, title:'您好，您有新的恋爱订单，请及时收取我'},
                {id:3, title:'我们，是我遇见的最美好的字眼'},
            ]
        }
    }
})
```

```
import {useTalkStore} from "@/store/loveTalk";
const talkStore = useTalkStore();
const {talkList} = toRefs(talkStore);
```



第二种更改数据的方式

在ts中添加acions，并在页面组件中调用

```js
import {defineStore} from "pinia";
export const useSumStore = defineStore('sum',{
    actions: {
      increment(value) {
          this.sum += value;
      }
    },
    state(){
        return {
            sum:60
        }
    }
})
```

```js
const sumStore = useSumStore();
const n = ref(1)
//通过action进行修改
function addSum(){
  sumStore.increment(n.value)
}
```



第三种方式

通过sumStore修改数据

```js
function subSum(){
  sumStore.sum -= n.value
}
```



**土味情话改造**

使得将请求的方法数据在全局中存储，页面更加聚焦于前端展示的实现

```js
import {defineStore} from "pinia";
import {reactive} from "vue";
import axios from "axios";
export const useTalkStore = defineStore('talks',{
    actions: {
        async loadTalks() {
            const response = await axios.get('https://api.uomg.com/api/rand.qinghua?format=json');
            const obj = {id:this.count, title:response.data.content};
            this.count++
            console.log(obj)
            this.talkList.push(obj);
        }
    },
    state(){
        return {
            talkList:[
                {id:1, title:':我没什么事儿，就想听听你的声音'},
                {id:2, title:'您好，您有新的恋爱订单，请及时收取我'},
                {id:3, title:'我们，是我遇见的最美好的字眼'},
            ],
            count: 4
        }
    }
})
```

```js
<script setup lang="ts">
  import {useTalkStore} from "@/store/loveTalk";
  const talkStore = useTalkStore();

  function addTalk(){
    talkStore.loadTalks()
  }
</script>

<template>
  <div>
    <button @click="addTalk">获取语句</button>
    <ul>
      <li v-for="talk in talkStore.talkList" :key="talk.id">{{talk.id}}:{{talk.title}}</li>
    </ul>
  </div>
</template>

<style scoped>

</style>
```



### 组合式写法

```js
import { defineStore } from "pinia";
import { ref } from "vue";

export const useSumStore = defineStore('sum', () => {
    // 使用 ref/reactive 定义状态
    const sum = ref(60);

    // 定义 actions（普通函数）
    function increment(value: number) {
        sum.value += value;
    }

    // 返回 state 和 actions
    return {
        sum,
        increment,
    };
});
```

和Hooks有些相似，但是使用场景不同

| 特性           | **Pinia Setup Store**                       | **Vue Composables (Hooks)**             |
| -------------- | ------------------------------------------- | --------------------------------------- |
| **设计目的**   | 管理全局状态（State Management）            | 逻辑复用（Logic Reusability）           |
| **作用域**     | 全局共享状态（所有组件共用同一个Store实例） | 可局部使用（每个组件实例有自己的状态）  |
| **响应式机制** | 自动与 Pinia 响应式系统集成                 | 依赖 Vue 的 `ref`/`reactive`/`computed` |
| **生命周期**   | 长期存在（除非手动清理）                    | 随组件销毁自动清理                      |
| **典型用途**   | 存储登录状态、主题配置、全局业务数据        | 封装可复用的逻辑（如表单校验、API请求） |



## 组件通信

有很多的方式，这里用mitt实现

在emitter.ts创建mitt

```js
import mitt from 'mitt'
const emitter = mitt()
export default emitter
```

在main.ts中引用

```js
import {createApp} from 'vue'
import App from './App.vue'
import router from './router'
import emitter from '@/utils/emitter.ts'
createApp(App).use(router).mount('#app')
```



发送消息方

```js
import emitter from '@/utils/emitter.ts'
const message = ref("来自父亲的消息")
```

```vue
<button @click="emitter.emit('Message', message)">发送消息</button>
```



接收消息方

```js
import emitter from '@/utils/emitter.ts'
const message = ref("")
emitter.on('Message', (value:string)=>{
  message.value = value
})
```

```vue
<h2>父组件消息:{{message}}</h2>
```
