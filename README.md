# vue-ssr
vue-ssr：nuxt-服务端渲染；
# 可参考博客：http://blog.zhanghaoran.ren/detail/2.html
> *本篇笔记来自2020年10月全栈-Vue篇章 1-7-4 Vue07课后补充：nuxt实践、ssr自动构建*


# Nuxt.js实战
Nuxt.js是一个基于 Vue.js的通用应用框架
通过对客户端/服务端基础架构的抽象组织，Nuxt.js 主要关注的市应用的 UI 渲染。

# 资源
[Nuxt.js官方文档](https://zh.nuxtjs.org/)

# nuxt.js特性
* 代码分层
* 服务端渲染
* 强大的路由功能
* 静态文件服务
* ……
	
# nuxt.js 渲染流程
一个完整的服务器请求到渲染的流程
如图（渲染流程.png）-


# nuxt安装
运行create-nuxt-app
`npx create-nuxt-app <项目名>`
选项：如图（配置项.png）-
    * package manager：包管理器
    * UI  framework：UI 框架
    * custom server framework：服务端 *框架/架构* 选型
    * Nuxt.js modules：异步请求的模块
    * linting tools：代码检测
    * test framework：测试
    * rendering mode：渲染方式
    * development tools：开发工具包
 

# 案例
实现如下功能点
	* 服务端渲染
	* 权限控制
	* 全局状态管理
	* 数据接口调用


# 路由
**路由生成**
pages目录中所有 `*.vue` 文件自动生成应用的路由配置，新建：
	* pages/admin.vue 商品管理页
	* pages/login.vue 登录页
> *访问[http://localhost:3000/](http://localhost:3000/)试试，并查看.nuxt/router.js验证生成路由*

**导航**
添加路由导航，layouts/default.vue
`<nav>`
`      <nuxt-link to="/">首页</nuxt-link>`
`      <nuxt-link to="/admin" no-prefetch>管理</nuxt-link>`
`      <nuxt-link to="/cart">购物车</nuxt-link>`
`</nav>`
功能和router-link等效
 禁用预加载行为：`<nuxt-link no-prefetch>page not pre-fetched<nuxt-link>`

商品列表，index.vue
`<template>`
`  <div>`
`    <h2>商品列表</h2>`
`    <ul>`
`      <li v-for="good in goods" :key="good.id">`
`        <nuxt-link :to="/detail/${good.id}">`
`          <span>{{ good.text }}</span>`
`          <span>￥{{ good.price }}</span>`
`        </nuxt-link>`
`      </li>`
`    </ul>`
`  </div>`
`</template>`
`<script>`
`export default {`
`  name:"Index",`
`  data(){`
`    return {`
`      goods:[`
`        {id:1,text:"商品1",price:8999},`
`        {id:2,text:"商品2222",price:666}`
`      ]`
`    }`
`  }`
`};`
`</script>`

**动态路由**
~以下划线作为前缀~的 .vue文件 或 目录会被定义为动态路由，如下面文件结构
`pages/`
`----|	detail/`
`--------|	_id.vue`
会生成如下路由配置：
`{`
`    path: "/detail/:id?",`
`    component: _8ddcd494,`
`    name: "detail-id",`
`},`
> *如果 detail里面不存在index.vue，:id 将被作为参数*

**嵌套路由**
创建内嵌子路由，你需要添加一个 .vue 文件，同时添加一个 ~与该文件同名~的目录用来存放子视图组件。构建文件结构如下：
`pages/`
`----|	detail/`
`--------|	_id.vue`
`----|	detail.vue`
生成的路由配置如下：
`{`
`    path: "/detail",`
`    component: 'pages/detail.vue',`
`    children:[`
`		{path:':id?',name:"detail-id"}`
`	 ]`
`},`
测试代码，detail.vue
`<template>`
`  <div>`
`    <h1>Deatil page</h1>`
`    <!-- nuxt-child 表示嵌套关系  ----  类似于router-view -->`
`    <nuxt-child></nuxt-child>`
`  </div>`
`</template>`
> *nuxt-child 等效于 router-view*
  
**配置路由**
要扩展 Nuxt.js 创建的路由，可以通过router.extendRoutes选项配置。例如添加自定义路由：
`// nuxt.config.js`
`const pkg = require("./package")`
`export default {`
`  router: {`
`    extendRoutes(routes, resolve) {`
`      routes.push({`
`        path: "/foo",`
`        component: resolve(__dirname, 'pages/othername.vue')`
`      })`
`    }`
`  },`
`}`


# 视图
如图（视图.png）展示了 Nuxt.js 如何为指定的路由配置数据和视图-

**默认布局**
`<template>`
`  <nuxt/>`
`</template>`

**自定义布局**
创建空白布局页面`layout/blank.vue`，用于 `login.vue`
`<template>`
`  <div>`
`      <nuxt></nuxt>`
`  </div>`
`</template>`
页面 `pages/login.vue`使用自定义布局：
`<script>`
`export default {`
`  name: "Login",`
`  layout: "blank",`
`};`
`</script>`

**自定义错误页面**
创建layouts/error.vue
`<template>`
`  <div class="container">`
`    <h1 v-if="error.statusCode === 404">页面不存在</h1>`
`    <h1 v-else>应用发生错误异常</h1>`
`    <p>{{error}}</p>`
`    <nuxt-link to="/">首 页</nuxt-link>`
`  </div>`
`</template>`
`<script>`
`export default {`
`  props: ['error'],`
`  layout:'blank'`
`}`
`</script>`
*测试：访问一个不存在的页面*

**页面**
页面组件就是 Vue 组件，只不过 Nuxt.js 为这些组件添加了一些特殊的配置项
给首页添加标题和meta等，index.vue
`export default {`
`  name: "Index",`
`  head() {`
`    return {`
`      title: "课程列表",`
`      // vue-meta 利用hid确定要更新meta`
`      meta: [{ name: "description", hid: "description", content: "set page meta" }],`
`      link: [{ rel: "favicon", href: "favcion.icon" }],`
`    };`
`  },`
`}`
> *更多 ~[页面配置项](https://zh.nuxtjs.org/docs/2.x/configuration-glossary/configuration-head/)~*


# 异步数据获取
`asyncData` 方法使得我们可以在~设置组件数据之前异步获取或处理数据~

范例：获取商品数据
**接口准备**
	* 安装依赖：`npm i koa-router koa-bodyparser -S`
	* 接口文件：`server/api.js`

整合axios
安装 @nuxt/axios 模块：`npm install @nuxtjs/axios -S`
配置：nuxt.config.js
`modules:[`
`	'@nuxtjs/axios'`
`],`
`axios:{`
`	proxy:true`
`},`
`proxy:{`
`	'/api':"http://localhost:8080"`
`}`
> *注意：配置重启生效*
测试代码：获取商品列表，index.vue
`<script>`
`export default {`
`  name: "Index",`
`  async asyncData({ error,  $axios }) {`
`    const { ok, goods } = await $axios.$get("/api/goods");`
`    if (ok) {`
`      // 此处返回的数据会和data中进行合并`
`      return {`
`        goods,`
`      };`
`    }`
`    // 错误处理`
`    error({ statusCode: 400, message: "数据查询失败请重试~" });`
`  },`
`};`
`</script>`
测试代码：获取商品详情，/index/_id.vue
`<template>`
`  <div>`
`    <pre v-if="goodInfo">{{ goodInfo }}</pre>`
`  </div>`
`</template>`
`<script>`
`export default {`
`  async asyncData({ params, error, $axios }) {`
`    if (params.id) {`
`      // asyncData 中不能使用 this 获取组件实例`
`      // 但是可以通过 上下文获取相关数据`
`      const { data: goodInfo } = await $axios.$get("/api/detail", { params });`
`      if (goodInfo) {`
`        return { goodInfo };`
`      }`
`      error({ statusCode: 400, message: "商品详情查询失败" });`
`    } else {`
`      return { goodInfo: null };`
`    }`
`  },`
`};`
`</script>`


# 中间件
中间件会在一个页面或一组页面渲染之前运行我们定义的函数，常用于权限控制、校验等任务。

范例代码：管理员页面保护，创建 middleware/auth.js
`export default function ({ route, redirect, store }) {`
`    // 上下文中通过store访问vuex中的全局状态`
`    // 通过vuex中令牌存在与否判断是否登录`
`    if (!store.state.user.token) {`
`        redirect("/login?redirect=" + route.path)`
`    }`
`}`
注册中间件，admin.vue
`<script>`
`export default {`
`  name: "Admin",`
`  middleware: ["auth"],`
`};`
`</script>`
> *全局注册：将会对所有页面起作用，nuxt.config.js*
> *`router: {`*
> *`   middleware: ['auth']`*
> *`},`*
> *运行报错，因为不存在user模块*


# 状态管理 vuex
应用根目录下如果存在 store 目录，Nuxt.js 将启用 vuex 状态数。定义各状态树时具名导出state，mutations，getters，actions 即可。

## 范例：
### 01- 用户登录及登录状态保存，创建store/user.js
`// 工厂函数`
`export const state = () => ({`
`    token: ''`
`})`
`// 具名函数`
`export const mutations = {`
`    init(state, token) {`
`        state.token = token`
`    }`
`}`
`export const getters = {`
`    isLogin(state) {`
`        return !!state.token`
`    }`
`}`
`export const actions = {`
`    login({ commit, getters }, u) {`
`        return this.$axios.$post("/api/login", u).then(({ token }) => {`
`            if (token) {`
`                commit("init", token)`
`            }`
`            return getters.isLogin`
`        })`
`    }`
`}`

### 02- 登录页面逻辑，login.vue
`<template>`
`  <div>`
`    <h2>用户登录</h2>`
`    <el-input v-model="user.username"></el-input>`
`    <el-input type="password" v-model="user.password"></el-input>`
`    <el-button @cllick="onLogin">登录</el-button>`
`  </div>`
`</template>`
`<script>`
`export default {`
`  name: "Login",`
`  layout: "blank",`
`  data() {`
`    return {`
`      user: {`
`        username: "",`
`        password: "",`
`      },`
`    };`
`  },`
`  methods: {`
`    onLogin() {`
`      this.$store.dispatch("user/login", this.user).then((ok) => {`
`        if (ok) {`
`          // 登录成功重定向`
`          const redirect = this.$route.query.redirect || "/";`
`          this.$router.push(redirect);`
`        }`
`      });`
`    },`
`  },`
`};`
`</script>`


# 插件
Nuxt.js 会在运行应用之前执行插件函数，需要引入或设置 Vue 插件、自定义模块和第三方模块时特别有用。

范例代码：接口注入，利用插件机制将服务接口注入组件实例、store实例中，创建 plugins/api-inject.js
`export default ({ $axios }, inject) => {`
`    inject("login", user => {`
`        return $axios.$post("/api/login", user)`
`    })`
`}`
注册插件，nuxt.config.js
`plugins: [  // 插件`
`    "@/plugins/api-inject"`
`],`

范例：添加请求拦截器附加token，创建plugins/interceptor.js
`export default function ({ $axios, store }) {`
`    $axios.onRequest(config => {`
`        if (store.state.user.token) {`
`            config.headers.Authorization = 'Bearer ' + store.state.user.token`
`        }`
`        return config`
`    })`
`}`
注册插件，nuxt.config.js
`plugins: [  // 插件`
`    "@/plugins/interceptor"`
`],`


# nuxtServerInit
通过在 store 的根模块中定义 nuxtServerInit 方法，Nuxt.js 调用它的时候会将页面的上下文对象作为第2个参数传给它。当我们想将服务端的一些数据传到客户端时，这个方法非常好用。

范例：登录状态初始化，store/index.js
`export const actions = {`
`    nuxtServerInit({ commit }, { app }) {`
`        const token = app.$cookies.get("token")`
`        if (token) {`
`            console.log("nuxtServerInit:token" + token)`
`            commit("user/init", token)`
`        }`
`    }`
`}`
> *安装依赖模块：cookie-universal-nuxt*
> 	`npm  i  -S  cookie-universal-nuxt`
> 	注册，nuxt.config.js
> 	`modules :  [ "cookie-universal-nuxt" ]`
> nuxtServerInit只能写在 store/index.js
> nuxtServerInit仅在服务端执行


# 发布部署
## 服务端渲染应用部署
先进行编译构建，然后再启动 Nuxt 服务
`npm run build `
`npm start`
生成内容在 .nuxt/dist 中

## 静态应用部署
Nuxt.js 可依据路由配置将应用静态化，使得我们可以将应用部署至任何一个静态站点主机服务商。
`npm  run  generate`
> *注意渲染和接口服务器都需要处于启动状态*
> *生成内容在dist中*

