# Vue Admin Template 二次开发文档

## 一、开发环境要求

| 工具 | 版本要求 | 说明 |
|------|---------|------|
| Node.js | >= 8.9（推荐 12.x - 14.x） | 运行时环境 |
| npm | >= 3.0.0（推荐 6.x+） | 包管理器 |
| Git | 最新稳定版 | 版本控制 |

> **注意：** 该项目基于 Vue CLI 4 + Vue 2，不建议使用 Node.js 16+ 版本，可能会因 OpenSSL 变更导致构建失败。如果使用高版本 Node，需设置环境变量 `NODE_OPTIONS=--openssl-legacy-provider`。

### 推荐开发工具

- **IDE：** VS Code
- **VS Code 插件：** Vetur（Vue 语法支持）、ESLint、EditorConfig

---

## 二、项目结构

```
jianzhi/
├── build/                      # 构建辅助脚本
│   └── index.js                # 预览服务器（构建后本地预览 dist）
│
├── mock/                       # Mock 数据层（开发环境模拟接口）
│   ├── index.js                # Mock 入口，聚合所有模块并导出 mockXHR
│   ├── mock-server.js          # 开发服务器中间件（支持热重载）
│   ├── table.js                # 表格列表接口 Mock
│   ├── user.js                 # 用户登录/信息/登出接口 Mock
│   └── utils.js                # Mock 工具函数
│
├── public/                     # 静态资源（不经过 webpack 处理）
│   ├── favicon.ico
│   └── index.html              # HTML 入口模板
│
├── src/                        # 源代码目录
│   ├── api/                    # API 接口定义层
│   │   ├── table.js            # 表格数据接口
│   │   └── user.js             # 用户相关接口（登录/获取信息/登出）
│   │
│   ├── assets/                 # 静态资源（经过 webpack 处理）
│   │   └── 404_images/         # 404 页面图片
│   │
│   ├── components/             # 全局公共组件
│   │   ├── Breadcrumb/         # 面包屑导航
│   │   ├── Hamburger/          # 侧边栏折叠按钮
│   │   └── SvgIcon/            # SVG 图标组件
│   │
│   ├── icons/                  # SVG 图标管理
│   │   ├── index.js            # 自动注册所有 SVG 图标
│   │   ├── svgo.yml            # SVG 优化配置
│   │   └── svg/                # 存放 SVG 图标文件
│   │
│   ├── layout/                 # 页面布局框架
│   │   ├── index.vue           # 主布局（Sidebar + Navbar + AppMain）
│   │   ├── components/         # 布局子组件
│   │   │   ├── AppMain.vue     # 主内容区（路由视图容器）
│   │   │   ├── Navbar.vue      # 顶部导航栏
│   │   │   └── Sidebar/        # 侧边栏（菜单、Logo）
│   │   └── mixin/
│   │       └── ResizeHandler.js # 响应式处理（移动端检测）
│   │
│   ├── router/                 # 路由配置
│   │   └── index.js            # 路由表定义
│   │
│   ├── store/                  # Vuex 状态管理
│   │   ├── index.js            # Store 入口
│   │   ├── getters.js          # 全局 Getter
│   │   └── modules/
│   │       ├── app.js          # 应用状态（侧边栏、设备类型）
│   │       ├── settings.js     # UI 设置（固定头部、侧边栏 Logo）
│   │       └── user.js         # 用户状态（Token、登录/登出）
│   │
│   ├── styles/                 # 全局样式
│   │   ├── index.scss          # 样式入口文件
│   │   ├── variables.scss      # SCSS 变量（颜色、侧边栏宽度等）
│   │   ├── mixin.scss          # SCSS Mixin
│   │   ├── sidebar.scss        # 侧边栏样式
│   │   ├── element-ui.scss     # Element UI 样式覆盖
│   │   └── transition.scss     # 过渡动画
│   │
│   ├── utils/                  # 工具函数
│   │   ├── auth.js             # Token 管理（Cookie 存取）
│   │   ├── get-page-title.js   # 页面标题拼接
│   │   ├── index.js            # 通用工具（日期格式化等）
│   │   ├── request.js          # Axios 实例（请求/响应拦截器）
│   │   └── validate.js         # 验证函数
│   │
│   ├── views/                  # 页面视图
│   │   ├── login/              # 登录页
│   │   ├── dashboard/          # 首页
│   │   ├── form/               # 表单示例页
│   │   ├── table/              # 表格示例页
│   │   ├── tree/               # 树形控件示例页
│   │   ├── nested/             # 嵌套路由示例页
│   │   └── 404.vue             # 404 页面
│   │
│   ├── App.vue                 # 根组件
│   ├── main.js                 # 应用入口文件
│   ├── permission.js           # 路由权限守卫
│   └── settings.js             # 应用配置（标题、UI 开关）
│
├── tests/                      # 单元测试
│   └── unit/
│       ├── components/         # 组件测试
│       └── utils/              # 工具函数测试
│
├── .editorconfig               # 编辑器统一配置
├── .env.development            # 开发环境变量
├── .env.production             # 生产环境变量
├── .env.staging                # 预发布环境变量
├── .eslintrc.js                # ESLint 配置
├── .eslintignore               # ESLint 忽略文件
├── babel.config.js             # Babel 配置
├── jest.config.js              # Jest 测试配置
├── jsconfig.json               # JS 路径别名配置（IDE 支持）
├── package.json                # 项目依赖和脚本
├── postcss.config.js           # PostCSS 配置
└── vue.config.js               # Vue CLI 项目配置
```

---

## 三、依赖包说明

### 3.1 生产依赖 (dependencies)

| 包名 | 版本 | 用途说明 |
|------|------|---------|
| `vue` | 2.6.10 | 核心框架，MVVM 响应式视图层 |
| `vue-router` | 3.0.6 | 官方路由管理器，实现页面导航和路由守卫 |
| `vuex` | 3.1.0 | 官方状态管理库，集中管理应用状态（用户信息、侧边栏等） |
| `element-ui` | 2.13.2 | 饿了么 UI 组件库，提供表单、表格、弹窗等现成组件 |
| `axios` | 0.18.1 | HTTP 请求库，封装在 `src/utils/request.js` 中统一管理 |
| `js-cookie` | 2.2.0 | Cookie 操作库，用于存储登录 Token |
| `normalize.css` | 7.0.0 | CSS Reset，统一不同浏览器的默认样式 |
| `nprogress` | 0.2.0 | 页面顶部加载进度条（路由切换时显示） |
| `path-to-regexp` | 2.4.0 | 路径匹配工具，面包屑组件中用于路由路径解析 |
| `core-js` | 3.6.5 | JavaScript Polyfill 库，为旧浏览器提供新 API 支持 |

### 3.2 开发依赖 (devDependencies)

| 包名 | 版本 | 用途说明 |
|------|------|---------|
| `@vue/cli-service` | 4.4.4 | Vue CLI 核心服务，提供 dev/build/lint 命令 |
| `@vue/cli-plugin-babel` | 4.4.4 | Babel 集成插件，ES6+ 代码转译 |
| `@vue/cli-plugin-eslint` | 4.4.4 | ESLint 集成插件，代码规范检查 |
| `@vue/cli-plugin-unit-jest` | 4.4.4 | Jest 单元测试集成 |
| `sass` | 1.26.8 | SCSS 编译器（Dart Sass） |
| `sass-loader` | 8.0.2 | Webpack 的 SCSS 加载器 |
| `svg-sprite-loader` | 4.1.3 | SVG 雪碧图加载器，实现 SVG 图标系统 |
| `svgo` | 1.2.0 | SVG 文件优化工具（去除多余属性） |
| `mockjs` | 1.0.1-beta3 | 数据模拟库，生成随机 Mock 数据 |
| `babel-plugin-dynamic-import-node` | 2.3.3 | 开发环境将动态 import 转为 require，加速热更新 |
| `eslint` | 6.7.2 | 代码静态检查工具 |
| `eslint-plugin-vue` | 6.2.2 | Vue 文件专用 ESLint 规则 |
| `babel-eslint` | 10.1.0 | ESLint 的 Babel 解析器 |
| `autoprefixer` | 9.5.1 | CSS 自动添加浏览器前缀 |
| `connect` | 3.6.6 | HTTP 服务器框架（预览构建产物） |
| `serve-static` | 1.13.2 | 静态文件服务中间件（预览构建产物） |
| `chokidar` | 2.1.5 | 文件监听（Mock 热重载） |
| `body-parser` | 1.19.0 | 请求体解析中间件（Mock 服务） |
| `script-ext-html-webpack-plugin` | 2.1.3 | 将 runtime chunk 内联到 HTML |
| `vue-template-compiler` | 2.6.10 | Vue 模板编译器（必须与 Vue 版本一致） |
| `@vue/test-utils` | 1.0.0-beta.29 | Vue 组件测试工具 |
| `babel-jest` | 23.6.0 | Jest 的 Babel 转译支持 |
| `jest-transform-stub` | 2.0.0 | 在测试中处理非 JS 资源的 stub |

---

## 四、环境变量配置

项目通过 `.env.*` 文件区分不同环境：

| 文件 | 环境 | NODE_ENV | VUE_APP_BASE_API | 用途 |
|------|------|----------|-----------------|------|
| `.env.development` | 开发 | development | `/dev-api` | 本地开发，接口通过 Mock 拦截 |
| `.env.production` | 生产 | production | `/prod-api` | 正式打包，需替换为真实接口地址 |
| `.env.staging` | 预发布 | production | `/stage-api` | 预发布打包，连接预发布环境接口 |

**二次开发时**，将 `VUE_APP_BASE_API` 修改为实际后端接口地址：

```bash
# .env.development
VUE_APP_BASE_API = 'http://localhost:8080/api'

# .env.production
VUE_APP_BASE_API = 'https://your-domain.com/api'
```

---

## 五、运行与构建

### 5.1 安装依赖

```bash
# 克隆项目后，进入项目目录
npm install
```

> 如遇到 node-sass 安装失败（国内网络问题），可使用淘宝镜像：
> ```bash
> npm install --registry=https://registry.npmmirror.com
> ```

### 5.2 本地开发

```bash
npm run dev
```

- 启动开发服务器，默认端口 **9528**
- 自动打开浏览器访问 `http://localhost:9528`
- 支持热更新（HMR）
- Mock 接口自动启用，无需后端服务
- 默认登录账号：`admin` / `111111`

### 5.3 代码检查

```bash
npm run lint
```

- 对 `src/` 下所有 `.js` 和 `.vue` 文件执行 ESLint 检查
- 开发环境保存文件时会自动触发 lint

### 5.4 运行测试

```bash
npm run test:unit
```

- 执行 `tests/unit/` 下所有 `*.spec.js` 测试文件
- 覆盖率统计范围：`src/utils/`（不含 auth.js 和 request.js）和 `src/components/`

### 5.5 生产构建

```bash
# 生产环境打包
npm run build:prod

# 预发布环境打包
npm run build:stage
```

- 输出目录：`dist/`
- 关闭 Source Map（生产环境）
- 自动进行代码分割优化：
  - `chunk-libs`：第三方库（vue、vue-router 等）
  - `chunk-elementUI`：Element UI 单独分包（体积较大）
  - `chunk-commons`：公共组件复用

### 5.6 本地预览构建产物

```bash
npm run preview
```

- 在端口 **9526** 启动一个静态服务器来预览打包后的 `dist/` 目录
- 可附加 `--report` 参数生成 Webpack 构建分析报告

### 5.7 SVG 图标优化

```bash
npm run svgo
```

- 对 `src/icons/svg/` 目录下的 SVG 文件进行优化压缩
- 新增 SVG 图标后建议运行一次

---

## 六、部署发布

### 6.1 基本流程

```
npm run build:prod  →  dist/  →  部署到 Web 服务器
```

### 6.2 Nginx 部署示例

```nginx
server {
    listen       80;
    server_name  your-domain.com;

    location / {
        root   /path/to/dist;
        index  index.html;
        try_files $uri $uri/ /index.html;  # 支持前端路由
    }

    # 接口代理（如需反向代理后端）
    location /prod-api/ {
        proxy_pass http://backend-server:port/;
    }
}
```

### 6.3 路由模式说明

当前项目使用 **Hash 模式**（`#/path`），部署时无需额外配置。如切换为 History 模式，需在服务器配置 fallback 到 `index.html`（如上方 Nginx 的 `try_files` 配置）。

---

## 七、关键配置文件说明

### vue.config.js 核心配置

| 配置项 | 值 | 说明 |
|--------|---|------|
| `publicPath` | `/` | 部署路径，如部署在子路径需修改（如 `/admin/`） |
| `outputDir` | `dist` | 构建输出目录 |
| `assetsDir` | `static` | 静态资源子目录 |
| `lintOnSave` | `development` | 仅开发环境启用保存时 lint |
| `productionSourceMap` | `false` | 生产环境不生成 Source Map |
| `devServer.port` | `9528` | 开发服务器端口 |
| `devServer.open` | `true` | 启动后自动打开浏览器 |

### 路径别名

在代码中可使用 `@` 代替 `src/` 路径：

```javascript
import request from '@/utils/request'
import Layout from '@/layout'
```

---

## 八、二次开发指引

### 8.1 新增页面

1. 在 `src/views/` 下创建页面组件
2. 在 `src/router/index.js` 中添加路由配置
3. 侧边栏菜单会根据路由自动生成

### 8.2 新增 API 接口

1. 在 `src/api/` 下创建接口文件
2. 使用 `@/utils/request` 发送请求
3. 开发阶段在 `mock/` 下创建对应 Mock 数据

### 8.3 新增 SVG 图标

1. 将 SVG 文件放入 `src/icons/svg/`
2. 运行 `npm run svgo` 优化
3. 在组件中使用 `<svg-icon icon-class="文件名" />`

### 8.4 修改主题/样式

- 全局颜色变量：`src/styles/variables.scss`
- Element UI 样式覆盖：`src/styles/element-ui.scss`
- 侧边栏宽度：修改 `variables.scss` 中的 `$sideBarWidth`

### 8.5 对接真实后端

1. 修改 `.env.development` 中的 `VUE_APP_BASE_API` 为后端地址
2. 删除 `src/main.js` 中生产环境的 `mockXHR()` 调用
3. 删除 `vue.config.js` 中 `devServer.before` 的 Mock 中间件（或保留作为后端未就绪时的 fallback）
4. 根据后端实际返回格式修改 `src/utils/request.js` 中的响应拦截器

---

## 九、常见问题

| 问题 | 解决方案 |
|------|---------|
| `npm install` 失败 | 尝试删除 `node_modules` 和 `package-lock.json`，使用淘宝镜像重新安装 |
| Node 16+ 构建报错 `ERR_OSSL_EVP_UNSUPPORTED` | 设置 `NODE_OPTIONS=--openssl-legacy-provider` |
| 修改文件后页面未更新 | 检查是否保存文件，尝试重启 `npm run dev` |
| 打包后页面空白 | 检查 `vue.config.js` 中的 `publicPath` 是否与部署路径匹配 |
| ESLint 报错太多 | 执行 `npm run lint -- --fix` 自动修复部分问题 |
