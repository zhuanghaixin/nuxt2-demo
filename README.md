# Nuxt2 Demo 项目

基于 Nuxt2 + Composition API + Ant Design Vue 构建的演示项目。
## 访问
http://localhost:3000/highlight-demo

## 📋 技术栈

- **Nuxt.js**: ^2.17.1 - Vue.js 的通用应用框架
- **Vue.js**: ^2.7.14 - 渐进式 JavaScript 框架
- **@nuxtjs/composition-api**: ^0.34.0 - Vue 3 Composition API 在 Vue 2 中的实现
- **Ant Design Vue**: ^1.7.8 - 企业级 UI 设计语言和组件库

## 🚀 快速开始

### 安装依赖

```bash
# 使用 npm
npm install

# 或使用 yarn
yarn install
```

### 开发模式

```bash
# 启动开发服务器 (localhost:3000)
npm run dev

# 或使用 yarn
yarn dev
```

### 生产构建

```bash
# 构建生产版本
npm run build
npm run start

# 或使用 yarn
yarn build
yarn start
```

### 静态生成

```bash
# 生成静态站点
npm run generate

# 或使用 yarn
yarn generate
```

## 📁 项目结构

```
nuxt2-demo/
├── pages/                # 页面目录 (路由自动生成)
│   └── index.vue         # 首页
├── components/           # 组件目录
├── plugins/              # 插件目录
│   └── antd-ui.js       # Ant Design Vue 插件配置
├── static/              # 静态文件目录
├── nuxt.config.js       # Nuxt 配置文件
├── package.json         # 项目依赖配置
└── README.md           # 项目说明文档
```

## 🎯 主要功能展示

### 1. Composition API 使用
项目展示了如何在 Nuxt2 中使用 Vue 3 的 Composition API：

```vue
<script>
import { defineComponent, ref, reactive } from '@nuxtjs/composition-api'

export default defineComponent({
  setup() {
    const count = ref(0)
    const form = reactive({
      username: '',
      email: ''
    })

    return {
      count,
      form
    }
  }
})
</script>
```

### 2. Ant Design Vue 组件
集成了 Ant Design Vue 组件库，包括：
- 布局组件 (Row, Col, Card)
- 表单组件 (Form, Input, Button)
- 反馈组件 (Message, Tag)
- 数据展示组件 (Typography, Divider)

### 3. 响应式设计
使用 Ant Design Vue 的栅格系统实现响应式布局。

## 🔧 配置说明

### Nuxt 配置 (nuxt.config.js)
- 配置了 `@nuxtjs/composition-api` 模块
- 引入了 Ant Design Vue 样式
- 设置了 Ant Design Vue 插件

### 插件配置 (plugins/antd-ui.js)
- 全局注册 Ant Design Vue 组件
- 自动引入所有 Ant Design Vue 组件

## 📝 开发注意事项

1. **版本兼容性**: 确保所有依赖版本与 package.json 中指定的版本一致
2. **Composition API**: 使用 `@nuxtjs/composition-api` 包提供的 API
3. **样式导入**: Ant Design Vue 样式已在 nuxt.config.js 中全局引入
4. **组件导入**: 无需手动导入 Ant Design Vue 组件，已通过插件全局注册

## 🛠️ 可用脚本

```bash
npm run dev      # 启动开发服务器
npm run build    # 构建生产版本
npm run start    # 启动生产服务器
npm run generate # 生成静态站点
npm run lint     # 代码格式检查
```

## 📚 相关文档

- [Nuxt.js 官方文档](https://nuxtjs.org/)
- [Vue Composition API 文档](https://composition-api.vuejs.org/)
- [Ant Design Vue 文档](https://www.antdv.com/docs/vue/introduce-cn/)

## �� 许可证

MIT License 
