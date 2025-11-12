## Vue 工程化

### node.js

#### npm

NodeJS 的软件包管理器, 前端项目中, 可直接通过 **npm install xxx** 从远程仓库下载依赖

#### 项目创建

**npm creat vue(@版本号)**, 不指定版本号默认最新版

- Project name: 项目名
- Add TypeScript?: 是否加入 **TypeScript 组件**
- Add JSX Support?: 是否加入 **JSX 支持**
- Add Vue Router...: 是否为单页应用程序开发添加 **Vue Router 路由管理组件**
- Add Pinia..: 是否添加 **Pinia 组件** 来进行 **状态管理**
- Add Vitest..: 是否添加 **Vitest** 来进行 **单元测试**
- Add an End-to-End...: 是否添加 **端到端测试**
- Add ESLint for code quality?: 是否添加 **ESLint** 来进行 **代码质量检查**

**npm install** 安装当前项目中的依赖

**npm run dev** 启动项目

#### 项目结构

- node_modules: 下载的第三方包存放目录
- src: 源代码存放目录
  - assets: 静态资源目录
  - component: 组件目录
  - App.vue: 根组件
  - main.js: 入口文件
- package.json: 项目配置文件
- vite.config.js: Vue 项目中的配置信息

#### vue 单文件组件

<script> js 代码, 控制模板的数据及行为 </script>

<template> html 代码, 模板部分 </template>

<style> css 代码, 当前组件的 css 样式 </style>

#### API 风格

##### 组合式 API

基于函数的组件编写方式:

```vue
<script setup> // setup是一个标识
import { ref, onMounted } from 'vue';
    // 在vue文件中script标签内可以直接导入外部包,通常不需要设置type属性
const count = ref(0); // 声明响应式变量
    // ref函数,用于定义响应式数据,即数据更新,页面也随之更新
    // 接收一个内部值,返回一个响应式的ref对象,此对象只有一个指向内部值的属性value

function increment(){ // 声明函数
   count.value++;
}

onMounted(() => { // 声明钩子函数,该钩子方法,注册一个回调函数,在组件挂载完成后执行
  console.log('Vue Mounted....'); 
})
</script>

<template>
<!--
create(App).mount('#app')
创建一个Vue应用实例,以App组件为根组件
将根组件的 <template> 内容渲染到DOM中的#app元素内
替换#app的原有内容
-->
   <input type="button" @click="increment"> Api Demo1 Count : {{ count }}
<!-- 声明响应式数据后,可通过模板部分将响应式数据直接展示在页面上 -->
</template>

<style scoped>
   
</style>
```

##### 选项式 API

用包含多个选项的对象来描述组件的逻辑, 选项定义的属性都会暴露在函数内部的 this 上, 其会指向当前组件的实例

组合式API没有this

```vue
<script>
export default{
   data() { // 返回值即所声明的数据模型,也就是响应式数据
      return {
         count: 0
      }
   },
   methods: { // 描述组件中所涉及的方法,所有方法必须定义在methods中,可以通过组件实例访问
      increment: function(){
         this.count++
      }
   },
   mounted() { // 声明钩子函数
      console.log('vue mounted.....');
   }
}
</script>

<template>
  <input type="button" @click="increment">Api Demo1 Count :  {{ count }}
</template>

<style scoped> /* 当前的css只会影响当前的文件的样式 */

</style>
```

## ElementPlus

一套基于Vue3的网站组件库,用于快速构建网页,其提供了很多组件供给使用

安装组件库:

```
npm install element-plus@2.4.4 --save
```

### container容器布局

<el-container>:外部容器,当子元素包含<el-header>或<el-footer>全部子元素垂直上下排列,否则左右排列

<el-header>:顶栏容器

<el-aside>:侧边栏容器

<el-main>:主要区域容器

<el-footer>:底栏容器

## VueRouter

路由实现的逻辑分析:

当点击左侧员工管理的菜单栏的时候，就会路由到员工管理的 /emp的地址，此时地址栏就会发生变化，就会去访问/emp的路径。

向路由实例发起请求，首先匹配/emp的前缀“/”，对应的组件是layout，然后在App.vue(默认在根组件)中的routerView渲染展示layout组件到页面。

接下来匹配匹配的是第二层路径emp，在layout的子路由中匹配是否有emp，如果存在就在上层组件layout.vue中的routerView组件在页面渲染展示emp组件

- VueRouter:路由器类,根据路由请求在路由视图中动态渲染选中的组件
- <router-link>:请求连接组件,被浏览器解析为<a>
- <router-view>:动态视图组件,用于渲染展示与路由路径对应的组件
