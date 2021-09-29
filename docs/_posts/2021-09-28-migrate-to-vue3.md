---
layout: post
title: "迁移项目至 Vue3 关键点记录"
date: 2021-09-28
categories: [ "Vue" ]
tags: [ "Vue", "Vue" ]
---

* content
{:toc}

这两天把项目的技术栈从 Vue2 升级到 Vue3.x ，过程主要参考[Vue3迁移指南](https://v3.cn.vuejs.org/guide/migration/introduction.html#%E9%9D%9E%E5%85%BC%E5%AE%B9%E7%9A%84%E5%8F%98%E6%9B%B4)、[Vue-Router迁移指南](https://next.router.vuejs.org/zh/guide/migration/index.html)、[Vuex迁移指南](https://next.vuex.vuejs.org/zh/guide/migrating-to-4-0-from-3-x.html), 我在此记录一些关键点，方便后续查阅参考。
<!-- more -->

## 安装必要的依赖
```bash
npm install -S vue@^3.2.0 vuex@^4.0.0 vue-router@^4.0.0
npm install -D  @vue/compiler-sfc@^3.2.0 @vue/cli-service@4.5.13 vue-loader@16.7.0
```

## Vuex@4.x
Vuex 的变动最小，只需要更改一下初始化 Store 的方式即可：
```js
import { createStore } from 'vuex'

export const store = createStore({
  state () {
    return {
      count: 1
    }
  }
})
```

## VueRouter@4.x
VueRouter 的变动很多，但是大多数都是 API 层面的小变动，大多数情况下不一定能用到，具体的可以参考[文档](https://next.router.vuejs.org/zh/guide/migration/index.html#%E7%A0%B4%E5%9D%8F%E6%80%A7%E5%8F%98%E5%8C%96)。一般情况只需要更改下面几点就能跑起来。
### router 初始化
```diff
+ import { createRouter, createWebHistory, createWebHashHistory } from 'vue-router'

- const router = new Router({
-    mode: 'history',
-    routes: [],
- })
+ const router = createRouter({
+  history: createWebHistory(), // or createWebHashHistory()
+  routes: [],
+ })
```

主要有两个更改点：
- 原来的 `mode` 用 `history` 字段代替，对应的值改成调用新的方法
    - `"history"`: `createWebHistory()`
    - `"hash"`: `createWebHashHistory()`
    - `"abstract"`: `createMemoryHistory()`
- 移除了 `base` 配置，改为 `createWebHistory()` 的第一个参数:
    ```js
    import { createRouter, createWebHistory } from 'vue-router'
    createRouter({
    history: createWebHistory('/base-directory/'),
    routes: [],
    })
    ```


### 删除了 *（星标或通配符）路由
如果代码中有一个用 `*` 定义的兜底路由，需要做如下更改：
```diff
{
-   path: '*',
+   path: '/:pathMatch(.*)*',
    component: () => import('@/views/error-page/404'),
    name: 'page404',
    meta: {
        title: '找不到该页面',
    },
}
```

## Vue3.x
### 升级基础工具

- 安装 `vue@3.2` 和 `@vue/compiler-sfc@3.2`
- 升级 `@vue/cli-service@4.5.13`
    现在的项目是通过 vue-cli 创建的，vue-cli 已经支持创建 vue3 的项目了, 我们需要升级到最新版。
- 增加 `vue-loader@16.7.0`
    新的vue组件需要使用最新版的 loader 进行解析。
    **需要注意的时**，如果在 `vue.config.js` 中直接对 `compilerOptions` 做了更改,需要改变一下写法：
    ```diff
        config.module.rule('vue').use('vue-loader').loader('vue-loader')
            .tap(options => {
    -            options.compilerOptions.preserveWhitespace = true;
    -            return options;
    +            return {
    +                compilerOptions: {
    +                    preserveWhitespace: true
    +                }
    +             }
            }).end();
    ```


### 更改Vue实例的启动
根据 vue3 文档，需要用 `createApp` 来创建新的实例，并将原来使用 `Vue.use()`、`Vue.component()` 的地方，都改成 `app.xxx()`:
```js
import { createApp } from 'vue';
app.use(somePlugin)
const app = createApp({})
app.mount('#app')
```

这里需要注意的是以下几个变动。
- `Vue.config.productionTip` 已经被废弃，不需要再额外设置；
- `Vue.config.ignoredElements` 改成了 [`app.config.compilerOptions.isCustomElement`](https://v3.cn.vuejs.org/guide/migration/global-api.html#config-ignoredelements-%E6%9B%BF%E6%8D%A2%E4%B8%BA-config-iscustomelement)
- 原来通过 `Vue.prototype` 挂载的实例属性，改成了通过 `app.config.globalProperties` 设置：
    ```diff
    -    Vue.prototype.$http = () => {}
    +    app.config.globalProperties.$http = () => {}
    ```
- `Vue.extend()` **被废弃了**。

### 比较常见的重要的语法变更
Vue3.x 涉及到很多不兼容的变更，在[官方文档](https://v3.cn.vuejs.org/guide/migration/introduction.html#%E9%9D%9E%E5%85%BC%E5%AE%B9%E7%9A%84%E5%8F%98%E6%9B%B4)列出了，这里记录几个最常见的语法变动。

#### `v-model` 语法变动
在 Vue2 中，`v-model` 是 `value` 属性和 `input` 事件的语法糖:
```html
<input v-model="name"/>
<!-- 等同于 -->
<input :value="name" @input="val => name = val">
```

在 Vue3 中，`v-model` 是 `modelValue` 属性和 `update:modelValue` 事件的语法糖:
```html
<input v-model="name"/>
<!-- 等同于 -->
<input :modelValue="name" @update:modelValue="val => name = val">
```

所以如果代码中有自定义的 `v-model` ，需要把接收的属性改为 `modelValue`, 把抛出的事件名改为 `update:modelValue`。
```diff
-    props: [ 'value' ],
+    props: [ 'modelValue' ],

-    this.$emit('input')
+    this.$emit('update:modelValue')
```

#### `.sync` 操作符被废弃，由新的 `v-model` 语法替代 
`v-model` 语法支持参数，也就是可以给多个属性使用 `v-model`:
```html
<comp v-model:title="curTitle"></comp>
<!-- 等同于 -->
<comp :title="curTitle" @update:title="val => curTitle = val"></comp>
```

所以代码中的 `.sync` 操作符统一替换成 `v-model`：
```diff
<dialog
- :visible.sync="dlgVisible"
+ v-model:visible="dlgVisible"
></dialog>
```
`v-model:visible` 所代表的属性和事件名并没有变化，所以组件内部的代码并不需要更改。

#### `$listeners` 被废弃，合并进 `$attrs`
所有的事件也都通过 `$attrs` 传入，因此需要把代码中的 `v-on="$listeners"` 都删除掉。

#### `$scopedSlot` 被废弃，使用 `$slots` 代替
在 Vue2 中，`$slots` 表示静态插槽，`$scopedSlots` 表示动态插槽, 现在统一为 `$slots`，并将所有插槽暴露为函数：
```diff
- this.$slots.title
- this.$scopedSlots.content()
+ this.$slots.title()
+ this.$slots.content()
```

#### 插槽的语法变更
所有的插槽都必须包在 `<template></template>` 标签中，并且插槽名使用 `v-slot:default` 或 `#default` 形式指定：
```diff
<el-table-column>
-    <template scope-slot="{ row }"></template>
+    <template #default="{ row }"></template>
</el-table-column>
```


基本上改完上述几个点，程序就能运行跑起来了。

## ElementUI
项目中使用了ElementUI, 这里列出比较常见的、会影响程序运行的更改。
### Dialog 的 visible 属性被废弃
```diff
- <el-dialog :visible.sync="dlgVisible"></el-dialog>
+ <el-dialog v-model="dlgVisible"></el-dialog>
```

### `.sync` 操作符用 `v-model` 代替
Pagination 的 `current-page` 属性等使用了 `.sync` 操作符的，需要替换成 `v-model:xxx`。
```diff
- <el-pagination :current-page.sync="curPage"></el-pagination>
+ <el-pagination v-model:current-page="curPage"></el-pagination>
```

### TimePicker/DatePicker/DateTimePicker 的配置项更改
`picker-options` 中的配置项废弃，所有的配置项都改为直接传给组件。
```diff
<el-date-picker
-  :picker-options="pickerOptions"
-  v-bind="pickerOptions"
></el-date-picker>
```

**`shortcuts` 配置项**的格式也更改为 `[{ text: string, value: Date }]`:
```js
shortcuts: [{
    text: '最近一周',
    value: (() => {
        const end = new Date();
        const start = new Date();
        start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
        return [start, end]
    })()
}]
```

## Sentry
如果项目中使用了 [Sentry](https://www.npmjs.com/package/@sentry/vue)， 需要更改一下初始化的配置，即把参数 `Vue` 改为参数 `app`:
```diff
const app = createApp({});
Sentry.init({
-  Vue,
+  app,
  dsn: '__PUBLIC_DSN__',
});
```

以上就是本文的全部内容了，感谢各位阅读，如果有任何疑问，欢迎电子邮件留言。

