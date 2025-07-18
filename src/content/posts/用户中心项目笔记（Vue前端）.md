---
title: 用户中心项目笔记（Vue）
published: 2025-07-04
updated: 2025-07-04
description: '用户中心Vue前端'
image: ''
tags: [Project]
category: 'Project'
draft: false 
---

# 用户中心笔记

## 相关软件安装

node JS下载

```
PS C:\Users\71460\Desktop> node -v
v22.11.0
```



Vue CLI

使用以下任一指令安装脚手架

```
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```



验证安装成功

```
PS C:\Users\71460\Desktop> vue --version
@vue/cli 5.0.8
```



在对应文件cmd窗口

使用如下指令创建对应的vue工程

```
vue create user-center-frontend-vue
```



## Ant Design Vue

使用npm安装组件

```
npm i --save ant-design-vue@4.x
```



全局引入

```tsx
import { createApp } from 'vue';
import Antd from 'ant-design-vue';
import App from './App';
import 'ant-design-vue/dist/reset.css';

const app = createApp(App);

app.use(Antd).mount('#app');
```



在main.ts中融合一下

```tsx
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";
import Antd from "ant-design-vue";
import "ant-design-vue/dist/reset.css";

createApp(App).use(Antd).use(router).mount("#app");
```



## 代码风格

组合式API

```vue
<script setup>
import { ref, onMounted } from 'vue'

// 响应式状态
const count = ref(0)

// 用来修改状态、触发更新的函数
function increment() {
  count.value++
}

// 生命周期钩子
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```



## 页面布局

通过一个BasicLayout页面，使其他的页面引用其实现各个网页模板不发送改变

```vue
<template>
  <div id="BasicLayout">
    <a-layout>
      <a-layout-header :style="headerStyle">Header</a-layout-header>
      <a-layout-content class="content" :style="contentStyle">
        <router-view />
      </a-layout-content>
      <a-layout-footer class="footer" :style="footerStyle">
        <a href="https://thrinisty.github.io/Blog/" target="_blank">
          thrinisty博客网址
        </a>
      </a-layout-footer>
    </a-layout>
  </div>
</template>

<script setup lang="ts"></script>

<style scoped>
#BasicLayout .footer {
  background-color: #96ebf4;
  text-align: center;
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 16px;
}

#BasicLayout .content {
  padding: 20px;
  margin-bottom: 20px;
  background: linear-gradient(to right, #cae8ec 0%, #70d9ea 80%);
}
</style>

```



在其他的页面中引入这个BasicLayout基础布局

```vue
<template>
  <div id="app">
    <BasicLayout></BasicLayout>
  </div>
</template>

<style></style>
<script setup lang="ts">
import BasicLayout from "@/loayouts/BasicLayout.vue";
</script>
```



全局导航菜单

```vue
<template>
  <div id="GlobalHeader">
    <a-row :wrap="false">
      <a-col flex="200px">
        <div class="header-container">
          <div class="title">用户论坛</div>
        </div></a-col
      >
      <a-col flex="auto"
        ><a-menu
          v-model:selectedKeys="current"
          mode="horizontal"
          :items="items"
      /></a-col>
      <a-col flex="80px"
        ><div class="user-login-status">
          <a-button type="primary" href="/user/login">Login</a-button>
        </div></a-col
      >
    </a-row>
  </div>
</template>
<script lang="ts" setup>
import { h, ref } from "vue";
import {
  MailOutlined,
  AppstoreOutlined,
  SettingOutlined,
} from "@ant-design/icons-vue";
import { MenuProps } from "ant-design-vue";
const current = ref<string[]>(["mail"]);
const items = ref<MenuProps["items"]>([
  {
    key: "mail",
    icon: () => h(MailOutlined),
    label: "论坛",
    title: "论坛",
  },
  {
    key: "app",
    icon: () => h(AppstoreOutlined),
    label: "推文",
    title: "推文",
  },
  {
    key: "sub1",
    icon: () => h(SettingOutlined),
    label: "关注博主",
    title: "关注博主",
    children: [
      {
        type: "group",
        label: "Item 1",
        children: [
          {
            label: "Option 1",
            key: "setting:1",
          },
          {
            label: "Option 2",
            key: "setting:2",
          },
        ],
      },
      {
        type: "group",
        label: "Item 2",
        children: [
          {
            label: "Option 3",
            key: "setting:3",
          },
          {
            label: "Option 4",
            key: "setting:4",
          },
        ],
      },
    ],
  },
  {
    key: "alipay",
    icon: () => h(SettingOutlined),
    label: h(
      "a",
      { href: "https://thrinisty.github.io/Blog/", target: "_blank" },
      "主页链接"
    ),
    title: "返回主页",
  },
]);
</script>
```

在BasicLayout的header里引入全局导航的设置

```vue
<a-layout-header class="header" :style="headerStyle">
  <GlobalHeader />
</a-layout-header>
```



在a-col标签中通过@click="doMenuClick"

```vue
<a-col flex="auto"
  ><a-menu
    v-model:selectedKeys="current"
    mode="horizontal"
    :items="items"
    @click="doMenuClick"
/></a-col>
```

```javascript
const doMenuClick = ({ key }: { key: string }) => {
  router.push({
    path: "/" + key,
  });
};
```

这样我们选取水平菜单的时候就会请求对应key的路径，在全局路由进行配置即可轻松实现页面的切换



## 与后端交互数据

### 全局自定义请求

编写请求配置文件request.ts  包括全局接口请求地址，超时时间，自定义请求响应拦截器等

运用场景：需要对接口的通用响应进行统一处理，例如从response中取出data；或者根据code去集中处理错误，这样不用在每个接口请求中都去写相同的逻辑

也可以在全局响应拦截器中读取结果中的data，并校验code是否合法，如果是未登录状态，则自动登录

```rst
const myAxios = axios.create({
  baseURL: "https://localhost:8080",
  timeout: 10000,
  withCredentials: true,
});

// 添加请求拦截器
myAxios.interceptors.request.use(
  function (config) {
    // 在发送请求之前做些什么
    console.log(config);
    const { data } = config;
    console.log(data);
    return config;
  },
  function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  }
);

// 添加响应拦截器
myAxios.interceptors.response.use(
  function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response;
  },
  function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    return Promise.reject(error);
  }
);

export default myAxios;
```
