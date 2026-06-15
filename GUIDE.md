# Vue Admin Template 开发指南

> 本文档面向二次开发者，帮助你快速理解项目依赖、结构、运行和打包发布流程。

---

## 目录

- [1. 开发环境要求](#1-开发环境要求)
- [2. 安装与启动](#2-安装与启动)
- [3. 项目目录结构总览](#3-项目目录结构总览)
- [4. 核心依赖说明](#4-核心依赖说明)
- [5. src 源码目录详解](#5-src-源码目录详解)
- [6. 环境配置与变量](#6-环境配置与变量)
- [7. 构建配置说明](#7-构建配置说明)
- [8. Mock 数据机制](#8-mock-数据机制)
- [9. 权限控制流程](#9-权限控制流程)
- [10. 常用脚本命令](#10-常用脚本命令)
- [11. 打包与发布](#11-打包与发布)
- [12. 二次开发建议](#12-二次开发建议)

---

## 1. 开发环境要求

| 环境       | 版本要求       | 说明                                      |
| ---------- | -------------- | ----------------------------------------- |
| Node.js    | >= 8.9（推荐 12.x ~ 14.x） | `package.json` 中 `engines` 指定 >= 8.9，但实际依赖兼容性推荐 12~14 |
| npm        | >= 3.0.0       | 推荐使用 npm，也可使用 yarn               |
| 操作系统   | Windows / macOS / Linux | 无特殊限制                        |

> **注意**：该项目基于 Vue CLI 4.4.4，**不建议使用 Node.js 16+**，否则部分依赖（如 `node-sass`/`sass`）可能存在兼容问题。推荐使用 **nvm** 管理 Node 版本：
>
> ```bash
> nvm install 14
> nvm use 14
> ```

---

## 2. 安装与启动

### 2.1 安装依赖

```bash
npm install
```

如果安装速度慢，可先配置国内镜像：

```bash
npm install --registry=https://registry.npmmirror.com
```

### 2.2 启动开发服务器

```bash
npm run dev
```

启动后浏览器自动打开 `http://localhost:9528`。如需自定义端口：

```bash
npm run dev --port=9529
# 或
port=9529 npm run dev
```

### 2.3 默认登录账号

项目内置 Mock 数据，默认账号：

| 用户名  | 密码   |
| ------- | ------ |
| admin   | 任意密码（Mock 不校验） |
| editor  | 任意密码（Mock 不校验） |

---

## 3. 项目目录结构总览

```
jianzhi/
├── build/                  # 构建辅助脚本（预览服务）
│   └── index.js            # preview 命令的入口，启动本地静态服务
├── mock/                   # Mock 数据（模拟后端 API）
│   ├── index.js            # Mock 路由汇总 & mockXHR 方法
│   ├── mock-server.js      # 开发环境 Mock Server（基于 devServer.before）
│   ├── user.js             # 用户相关 Mock 数据（登录/信息/登出）
│   ├── table.js            # 表格相关 Mock 数据
│   └── utils.js            # Mock 工具函数
├── public/                 # 静态公共资源（不经过 Webpack 处理）
│   ├── favicon.ico         # 网站图标
│   └── index.html          # HTML 模板入口
├── src/                    # 源代码目录（核心）
│   ├── api/                # 后端 API 接口封装
│   ├── assets/             # 静态资源（图片等，经 Webpack 处理）
│   ├── components/         # 全局公共组件
│   ├── icons/              # SVG 图标（svg-sprite-loader）
│   ├── layout/             # 页面布局框架（侧边栏 + 顶部导航 + 内容区）
│   ├── router/             # 路由配置
│   ├── store/              # Vuex 状态管理
│   ├── styles/             # 全局样式（SCSS）
│   ├── utils/              # 工具函数库
│   ├── views/              # 页面视图组件
│   ├── App.vue             # 根组件
│   ├── main.js             # 应用入口文件
│   ├── permission.js       # 路由权限控制（导航守卫）
│   └── settings.js         # 全局应用配置（标题、侧边栏等）
├── .env.development        # 开发环境变量
├── .env.production         # 生产环境变量
├── .env.staging            # 预发布（staging）环境变量
├── .eslintrc.js            # ESLint 代码规范配置
├── .eslintignore           # ESLint 忽略规则
├── .gitignore              # Git 忽略规则
├── babel.config.js         # Babel 配置
├── jest.config.js          # Jest 单元测试配置
├── jsconfig.json           # JS 项目配置（路径别名等）
├── postcss.config.js       # PostCSS 配置
├── vue.config.js           # Vue CLI 项目配置（Webpack 配置）
├── package.json            # 项目依赖 & 脚本
└── GUIDE.md                # 本文档
```

---

## 4. 核心依赖说明

### 4.1 运行时依赖（dependencies）

| 包名             | 版本    | 用途说明                                                  |
| ---------------- | ------- | --------------------------------------------------------- |
| `vue`            | 2.6.10  | Vue 2 框架核心                                            |
| `vue-router`     | 3.0.6   | Vue 2 官方路由管理器，实现 SPA 页面跳转                   |
| `vuex`           | 3.1.0   | Vue 2 官方状态管理，集中管理全局数据（用户信息、设置等）  |
| `element-ui`     | 2.13.2  | Element UI 组件库（表格、表单、弹窗、菜单等后台常用组件） |
| `axios`          | 0.18.1  | HTTP 请求库，封装在 `src/utils/request.js` 中            |
| `js-cookie`      | 2.2.0   | Cookie 操作库，用于存取 Token                             |
| `normalize.css`  | 7.0.0   | CSS Reset 库，统一浏览器默认样式                          |
| `nprogress`      | 0.2.0   | 页面加载进度条（路由切换时顶部蓝色进度条）                |
| `path-to-regexp` | 2.4.0   | 路径匹配正则工具，用于权限路由匹配                        |
| `core-js`        | 3.6.5   | ES 新特性的 Polyfill，兼容旧浏览器                        |

### 4.2 开发依赖（devDependencies）

| 包名                              | 版本      | 用途说明                                                |
| --------------------------------- | --------- | ------------------------------------------------------- |
| `@vue/cli-service`                | 4.4.4     | Vue CLI 核心服务，提供 `serve`/`build` 等命令           |
| `@vue/cli-plugin-babel`           | 4.4.4     | Babel 集成插件，将 ES6+ 代码转译为兼容代码              |
| `@vue/cli-plugin-eslint`          | 4.4.4     | ESLint 集成插件，代码规范检查                           |
| `@vue/cli-plugin-unit-jest`       | 4.4.4     | Jest 单元测试集成插件                                   |
| `vue-template-compiler`           | 2.6.10    | Vue 模板编译器（版本须与 `vue` 一致）                   |
| `sass` / `sass-loader`            | 1.26.8 / 8.0.2 | SCSS 样式预处理器及其 Webpack Loader              |
| `svg-sprite-loader`               | 4.1.3     | SVG 雪碧图 Loader，将 SVG 图标合并为一个 Sprite         |
| `svgo`                            | 1.2.2     | SVG 文件压缩优化工具                                    |
| `eslint` / `eslint-plugin-vue`    | 6.7.2 / 6.2.2 | 代码静态检查工具及 Vue 规则插件                    |
| `babel-eslint`                    | 10.1.0    | ESLint 的 Babel 解析器                                  |
| `babel-plugin-dynamic-import-node`| 2.3.3     | 开发环境将 `import()` 转为 `require()`，加速热更新      |
| `mockjs`                          | 1.0.1-beta3 | 生成随机 Mock 数据的库                                 |
| `script-ext-html-webpack-plugin`  | 2.1.3     | 扩展 HtmlWebpackPlugin，支持 inline 脚本               |
| `html-webpack-plugin`             | 3.2.0     | 生成 HTML 入口文件                                      |
| `autoprefixer`                    | 9.5.1     | 自动添加 CSS 浏览器前缀                                 |
| `@vue/test-utils`                 | 1.0.0-beta.29 | Vue 组件测试工具                                   |
| `babel-jest`                      | 23.6.0    | Jest 的 Babel 转换器                                    |
| `chalk`                           | 2.4.2     | 终端彩色输出                                            |
| `connect` / `serve-static`        | 3.6.6 / 1.13.2 | 轻量级静态文件服务器（用于 `preview` 命令）         |
| `runjs`                           | 4.3.2     | 简易任务运行器                                          |

---

## 5. src 源码目录详解

### 5.1 `api/` — 后端接口封装

按业务模块封装 HTTP 请求函数。每个文件对应一个业务模块。

| 文件       | 说明                                      |
| ---------- | ----------------------------------------- |
| `user.js`  | 用户相关接口：`login`、`getInfo`、`logout` |
| `table.js` | 表格数据接口：`getList`                    |

**二次开发时**：新增页面如需调用后端 API，在 `api/` 下新建对应文件，导入 `@/utils/request` 发起请求。

### 5.2 `assets/` — 静态资源

存放图片等静态资源，会经过 Webpack 处理（如 hash 命名、压缩等）。

```
assets/
└── 404_images/
    ├── 404.png           # 404 页面主图
    └── 404_cloud.png     # 404 页面云朵装饰
```

### 5.3 `components/` — 全局公共组件

| 组件目录       | 说明                                              |
| -------------- | ------------------------------------------------- |
| `Breadcrumb/`  | 面包屑导航组件，根据当前路由自动生成面包屑路径    |
| `Hamburger/`   | 侧边栏折叠按钮（汉堡菜单图标）                    |
| `SvgIcon/`     | SVG 图标组件，配合 `icons/` 目录使用 SVG 雪碧图    |

### 5.4 `icons/` — SVG 图标管理

```
icons/
├── index.js     # 自动导入所有 SVG 文件并注册 svg-sprite-loader
├── svg/         # 存放所有 SVG 图标文件
│   ├── dashboard.svg
│   ├── example.svg
│   ├── eye-open.svg
│   ├── eye.svg
│   ├── form.svg
│   ├── link.svg
│   ├── nested.svg
│   ├── password.svg
│   ├── table.svg
│   ├── tree.svg
│   └── user.svg
└── svgo.yml     # SVGO 优化配置
```

**二次开发时**：将新的 SVG 文件放入 `svg/` 目录即可自动注册，在组件中使用 `<svg-icon icon-class="文件名" />` 引用。

### 5.5 `layout/` — 页面布局框架

整体布局采用经典的后台管理系统结构：

```
layout/
├── index.vue                        # 布局主组件（包含侧边栏 + 顶部导航 + 内容区）
├── components/
│   ├── index.js                     # 组件导出汇总
│   ├── AppMain.vue                  # 主内容区（<router-view> 渲染子页面）
│   ├── Navbar.vue                   # 顶部导航栏（面包屑 + 全屏 + 用户头像菜单）
│   └── Sidebar/
│       ├── index.vue                # 侧边栏容器
│       ├── SidebarItem.vue          # 菜单项（递归渲染多级菜单）
│       ├── Item.vue                 # 单个菜单项的图标和标题
│       ├── Link.vue                 # 菜单项的链接（内部路由 / 外部链接）
│       ├── Logo.vue                 # 侧边栏 Logo
│       └── FixiOSBug.js             # 修复 iOS 下的滚动 Bug
└── mixin/
    └── ResizeHandler.js             # 响应式处理（检测屏幕宽度，自动折叠侧边栏）
```

### 5.6 `router/` — 路由配置

| 文件       | 说明                                                                    |
| ---------- | ----------------------------------------------------------------------- |
| `index.js` | 定义所有前端路由，分为 `constantRoutes`（无需权限）和动态路由两部分     |

路由配置中关键字段说明：

| 字段            | 说明                                                   |
| --------------- | ------------------------------------------------------ |
| `hidden: true`  | 不在侧边栏菜单中显示                                   |
| `alwaysShow`    | 设为 true 时始终显示根菜单（即使只有一个子路由）       |
| `redirect`      | 设为 `noRedirect` 时面包屑中不可点击                   |
| `meta.roles`    | 控制页面访问角色（如 `['admin','editor']`）            |
| `meta.title`    | 侧边栏和面包屑中显示的标题                             |
| `meta.icon`     | 侧边栏图标（SVG 文件名或 Element UI 图标名）           |
| `meta.breadcrumb` | 设为 false 时不在面包屑中显示                        |
| `meta.activeMenu` | 高亮指定的侧边栏菜单项                               |

### 5.7 `store/` — Vuex 状态管理

```
store/
├── index.js           # Store 入口，注册所有模块
├── getters.js         # 全局 Getters（sidebar、device、token、avatar、name）
└── modules/
    ├── app.js         # 应用状态：侧边栏开关、设备类型（mobile/desktop）
    ├── settings.js    # 设置状态：fixedHeader、sidebarLogo 等
    └── user.js        # 用户状态：token、name、avatar，及 login/logout/getInfo 等 Action
```

**Getters 一览**：

| Getter    | 来源模块 | 说明                  |
| --------- | -------- | --------------------- |
| `sidebar` | app      | 侧边栏状态对象        |
| `device`  | app      | 当前设备类型           |
| `token`   | user     | 用户登录 Token         |
| `avatar`  | user     | 用户头像 URL           |
| `name`    | user     | 用户名                 |

### 5.8 `styles/` — 全局样式

```
styles/
├── index.scss         # 样式入口文件，导入所有其他样式
├── variables.scss     # SCSS 变量定义（侧边栏宽度、颜色等）
├── mixin.scss         # SCSS Mixin 集合（clearfix 等）
├── sidebar.scss       # 侧边栏专用样式
├── element-ui.scss    # Element UI 组件样式覆盖/自定义
└── transition.scss    # 过渡动画样式（侧边栏折叠、淡入淡出等）
```

### 5.9 `utils/` — 工具函数库

| 文件              | 说明                                                              |
| ----------------- | ----------------------------------------------------------------- |
| `request.js`      | Axios 实例封装（请求/响应拦截器、Token 注入、错误处理）            |
| `auth.js`         | Token 相关操作：`getToken`、`setToken`、`removeToken`（基于 Cookie）|
| `validate.js`     | 校验工具：`isExternal`（外链判断）、`validUsername`（用户名校验）  |
| `index.js`        | 通用工具：`parseTime`（时间格式化）、`formatTime`、`param2Obj`    |
| `get-page-title.js` | 生成页面标题（结合路由 meta.title 和 settings.title）            |

### 5.10 `views/` — 页面视图组件

```
views/
├── login/
│   └── index.vue          # 登录页面
├── dashboard/
│   └── index.vue          # 首页仪表盘
├── form/
│   └── index.vue          # 表单示例页
├── table/
│   └── index.vue          # 表格示例页
├── tree/
│   └── index.vue          # 树形控件示例页
├── nested/                # 多级嵌套菜单示例
│   ├── menu1/
│   │   ├── index.vue
│   │   ├── menu1-1/index.vue
│   │   ├── menu1-2/
│   │   │   ├── index.vue
│   │   │   ├── menu1-2-1/index.vue
│   │   │   └── menu1-2-2/index.vue
│   │   └── menu1-3/index.vue
│   └── menu2/
│       └── index.vue
└── 404.vue                # 404 页面
```

### 5.11 入口及核心文件

| 文件             | 说明                                                                          |
| ---------------- | ----------------------------------------------------------------------------- |
| `main.js`        | 应用入口：导入全局样式、注册 Element UI、挂载 Store/Router、生产环境启用 Mock |
| `App.vue`        | 根组件，仅包含 `<router-view />`                                              |
| `permission.js`  | 全局路由守卫：检查 Token → 获取用户信息 → 权限控制跳转                        |
| `settings.js`    | 全局应用配置：`title`（页面标题）、`fixedHeader`、`sidebarLogo`               |

---

## 6. 环境配置与变量

项目使用 Vue CLI 的环境变量机制，通过 `.env.*` 文件配置不同环境的参数。

### 6.1 环境文件对照

| 文件                | 环境       | 触发命令               |
| ------------------- | ---------- | ---------------------- |
| `.env.development`  | 开发环境   | `npm run dev`          |
| `.env.production`   | 生产环境   | `npm run build:prod`   |
| `.env.staging`      | 预发布环境 | `npm run build:stage`  |

### 6.2 变量说明

| 变量名              | 开发环境值     | 生产环境值     | 预发布环境值     | 说明                     |
| ------------------- | -------------- | -------------- | ---------------- | ------------------------ |
| `VUE_APP_BASE_API`  | `/dev-api`     | `/prod-api`    | `/stage-api`     | API 请求的基础路径前缀   |
| `ENV`               | `development`  | `production`   | `staging`        | 环境标识                 |

> **二次开发时**：如需修改 API 地址前缀，直接修改对应 `.env.*` 文件中的 `VUE_APP_BASE_API`。

### 6.3 在代码中使用

```javascript
// 通过 process.env 访问
process.env.VUE_APP_BASE_API  // => '/dev-api' (开发环境)
process.env.NODE_ENV           // => 'development' | 'production'
```

---

## 7. 构建配置说明

### 7.1 vue.config.js 关键配置

| 配置项                  | 值                              | 说明                                            |
| ----------------------- | ------------------------------- | ----------------------------------------------- |
| `publicPath`            | `/`                             | 部署路径，子路径部署时需修改                    |
| `outputDir`             | `dist`                          | 打包输出目录                                    |
| `assetsDir`             | `static`                        | 静态资源子目录                                  |
| `lintOnSave`            | 仅开发环境                      | 保存时 ESLint 检查                              |
| `productionSourceMap`   | `false`                         | 生产环境不生成 Source Map（保护源码）           |
| `devServer.port`        | `9528`                          | 开发服务器端口                                  |
| `devServer.open`        | `true`                          | 启动后自动打开浏览器                            |
| `devServer.before`      | `require('./mock/mock-server')` | 注入 Mock Server 中间件                          |

### 7.2 Webpack 路径别名

```javascript
// vue.config.js 中配置：
resolve: {
  alias: {
    '@': resolve('src')   // @ 指向 src/ 目录
  }
}
```

使用示例：`import request from '@/utils/request'` 等价于 `import request from 'src/utils/request'`

### 7.3 生产环境代码分割策略

生产打包时通过 `splitChunks` 将代码拆分为多个 chunk，优化加载性能：

| Chunk 名称        | 内容                                        | 说明                         |
| ----------------- | ------------------------------------------- | ---------------------------- |
| `chunk-libs`      | `node_modules` 中的第三方库（初始依赖）     | 基础依赖包                   |
| `chunk-elementUI` | Element UI 组件库                           | 单独拆分（体积大，利于缓存） |
| `chunk-commons`   | `src/components` 下被引用 >= 3 次的公共组件 | 高频公共组件                 |
| `runtime`         | Webpack 运行时代码                          | 内联到 HTML 中               |

### 7.4 babel.config.js

```javascript
module.exports = {
  presets: ['@vue/cli-plugin-babel/preset'],
  env: {
    development: {
      plugins: ['dynamic-import-node']  // 开发环境将 import() 转为 require()，加速热更新
    }
  }
}
```

### 7.5 SVG 图标处理

通过 `svg-sprite-loader` 将 `src/icons/svg/` 下的所有 SVG 文件打包为雪碧图：

```javascript
// webpack chain 配置
config.module.rule('icons')
  .test(/\.svg$/)
  .include.add(resolve('src/icons'))    // 仅处理 icons 目录下的 SVG
  .end()
  .use('svg-sprite-loader')
  .loader('svg-sprite-loader')
  .options({ symbolId: 'icon-[name]' }) // 生成 id 格式：icon-文件名
```

---

## 8. Mock 数据机制

项目提供两种 Mock 方式：

### 8.1 开发环境 Mock Server（推荐）

- **原理**：利用 Webpack DevServer 的 `before` 钩子注入 Express 中间件
- **入口**：`mock/mock-server.js`
- **特点**：支持热重载，修改 `mock/` 下的文件后自动重新注册路由
- **请求路径**：`/dev-api/vue-admin-template/user/login` → Mock 匹配 `/vue-admin-template/user/login`

### 8.2 生产环境 MockJS（仅用于演示）

- **原理**：通过 `Mock.mock()` 重写 `XMLHttpRequest`，在前端直接拦截请求
- **入口**：`mock/index.js` 中的 `mockXHR()` 方法
- **触发条件**：`process.env.NODE_ENV === 'production'` 时在 `main.js` 中自动调用
- **注意**：上线前务必移除此段代码，否则真实 API 请求也会被拦截

### 8.3 Mock 数据结构示例

```javascript
// mock/user.js
module.exports = [
  {
    url: '/vue-admin-template/user/login',
    type: 'post',
    response: config => {
      const { username } = config.body
      return {
        code: 20000,
        data: { token: username + '-token' },
        message: 'Success'
      }
    }
  },
  // ...
]
```

> **约定**：API 返回 `code: 20000` 表示成功，其他值视为错误（见 `request.js` 响应拦截器）。

---

## 9. 权限控制流程

权限控制的核心逻辑在 `src/permission.js`（全局路由守卫）：

```
用户访问页面
    │
    ├── 有 Token？
    │     ├── 是 → 访问 /login？
    │     │         ├── 是 → 重定向到首页 /
    │     │         └── 否 → 已获取用户信息？
    │     │                   ├── 是 → 放行
    │     │                   └── 否 → 调用 store.dispatch('user/getInfo')
    │     │                             ├── 成功 → 放行
    │     │                             └── 失败 → 清除 Token，跳转登录页
    │     │
    │     └── 否 → 路径在白名单中（/login）？
    │               ├── 是 → 放行
    │               └── 否 → 重定向到 /login?redirect=目标路径
```

**白名单**：`const whiteList = ['/login']`，无需登录即可访问。

---

## 10. 常用脚本命令

```bash
# 开发环境启动（热更新）
npm run dev

# 生产环境打包
npm run build:prod

# 预发布环境打包
npm run build:stage

# 打包并启动本地预览服务（端口 9526）
npm run preview

# 打包并生成分析报告
npm run preview -- --report

# 代码检查
npm run lint

# SVG 图标压缩优化
npm run svgo

# 单元测试（先清除缓存）
npm run test:unit

# CI 测试（lint + 单元测试）
npm run test:ci
```

---

## 11. 打包与发布

### 11.1 打包命令

```bash
# 生产环境打包
npm run build:prod
```

### 11.2 打包输出

打包完成后在项目根目录生成 `dist/` 目录：

```
dist/
├── static/
│   ├── css/          # 样式文件（含 hash）
│   ├── js/           # JS 文件（含 hash，已代码分割）
│   ├── img/          # 图片资源
│   └── fonts/        # 字体文件（Element UI 图标字体）
├── favicon.ico
└── index.html        # 入口 HTML
```

### 11.3 本地预览

打包后在本地启动预览服务：

```bash
npm run preview
```

访问 `http://localhost:9526` 查看打包结果。

### 11.4 部署到服务器

将 `dist/` 目录下的所有文件上传到 Web 服务器（Nginx / Apache）的静态资源目录。

**Nginx 配置参考**：

```nginx
server {
    listen       80;
    server_name  your-domain.com;

    location / {
        root   /path/to/dist;
        index  index.html;
        try_files $uri $uri/ /index.html;   # SPA 路由必须配置
    }

    # API 代理（将 /prod-api 转发到真实后端）
    location /prod-api/ {
        proxy_pass http://your-backend-server/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

> **关键**：由于项目使用 Hash 路由（非 History 模式），`try_files` 配置可选。但如果后续切换为 History 模式，必须配置 `try_files` 将所有请求指向 `index.html`。

### 11.5 子路径部署

如需部署到子路径（如 `https://example.com/admin/`），需修改：

```javascript
// vue.config.js
module.exports = {
  publicPath: '/admin/'
}
```

然后重新打包。

---

## 12. 二次开发建议

### 12.1 新增页面步骤

1. **创建视图组件**：在 `src/views/` 下新建页面 `.vue` 文件
2. **注册路由**：在 `src/router/index.js` 的 `constantRoutes` 中添加路由配置
3. **创建 API 文件**（可选）：在 `src/api/` 下新建接口封装文件
4. **添加菜单**：路由配置中的 `meta.title` 和 `meta.icon` 会自动生成侧边栏菜单

### 12.2 对接真实后端

1. 移除 `src/main.js` 中的 Mock 代码：
   ```javascript
   // 删除或注释以下代码
   if (process.env.NODE_ENV === 'production') {
     const { mockXHR } = require('../mock')
     mockXHR()
   }
   ```

2. 修改 `.env.production` 中的 `VUE_APP_BASE_API` 为真实后端地址：
   ```
   VUE_APP_BASE_API = 'https://api.your-domain.com'
   ```

3. 修改 `src/utils/request.js` 中的响应判断逻辑，适配后端的返回格式（当前约定 `code: 20000` 为成功）

4. 如需跨域，在 `vue.config.js` 中配置 `devServer.proxy`：
   ```javascript
   devServer: {
     proxy: {
       '/dev-api': {
         target: 'http://your-backend-server',
         changeOrigin: true,
         pathRewrite: { '^/dev-api': '' }
       }
     }
   }
   ```

### 12.3 Element UI 切换为中文

在 `src/main.js` 中修改：

```javascript
// 当前为英文
import locale from 'element-ui/lib/locale/lang/en'
Vue.use(ElementUI, { locale })

// 改为中文：
import locale from 'element-ui/lib/locale/lang/zh-CN'
Vue.use(ElementUI, { locale })

// 或者直接使用（默认中文）：
Vue.use(ElementUI)
```

### 12.4 添加新 SVG 图标

1. 将 `.svg` 文件放入 `src/icons/svg/` 目录
2. 运行 `npm run svgo` 压缩优化
3. 在组件中使用：`<svg-icon icon-class="你的文件名" />`
