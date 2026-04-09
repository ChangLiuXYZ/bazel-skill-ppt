# AGENTS.md — AI 编码代理指南

本文件面向在此仓库中工作的 AI 编码代理（如 CodeMaker、Cursor、Copilot 等）。

---

## 项目概述

本项目是一个基于 **Slidev** 的演示文稿项目，主题为「用 AI Skill 自动迁移 900+ iOS CocoaPods 组件库到 Bazel」。核心内容为 Markdown 幻灯片（`slides.md`），辅以少量 Vue 组件和 TypeScript 代码片段。

**技术栈：**
- Slidev `^52.14.2`（基于 Vite + Vue 3 的 Markdown 演示框架）
- Vue 3 `^3.5.29`（Composition API + `<script setup>`）
- TypeScript（代码片段示例）
- UnoCSS / Tailwind 原子化 CSS（内联样式）
- 模块系统：ES Modules（`"type": "module"`）

---

## 构建 / 开发命令

```bash
# 安装依赖（推荐 pnpm，CI 中使用 npm）
pnpm install
# 或
npm install

# 开发模式（热重载，自动打开浏览器）
npm run dev
# 等价于：slidev --open

# 静态构建（输出到 dist/）
npm run build
# 等价于：slidev build

# 构建时指定 base 路径（GitHub Pages 子路径部署时使用）
npm run build -- --base /bazel-skill-ppt/

# 导出为 PDF 或 PPTX
npm run export
# 等价于：slidev export
```

> **注意：** 本项目**没有测试框架**，无 `test` 相关脚本，也没有 Lint 配置文件。
> 项目性质为演示文稿，不需要单元测试。

---

## 部署配置

本项目同时支持三种部署方式：

| 平台 | 配置文件 | 构建命令 | 输出目录 |
|------|----------|----------|----------|
| GitHub Pages | `.github/workflows/deploy.yml` | `npm run build -- --base /bazel-skill-ppt/` | `dist/` |
| Netlify | `netlify.toml` | `npm run build` | `dist/` |
| Vercel | `vercel.json` | `npm run build` | `dist/` |

---

## 项目结构

```
bazel-skill-ppt/
├── .github/workflows/deploy.yml  # GitHub Actions 自动部署
├── components/
│   └── Counter.vue               # 自定义 Vue 组件（示例）
├── pages/
│   └── imported-slides.md        # 子页面（Slidev 多文件拆分）
├── snippets/
│   └── external.ts               # 外部代码片段（在幻灯片中引用）
├── slides.md                     # 主幻灯片文件（核心内容）
├── package.json
├── netlify.toml
└── vercel.json
```

---

## Slidev 内容规范

### 幻灯片文件格式（slides.md / pages/*.md）

```markdown
---
# 每页幻灯片用 `---` 分隔
# Frontmatter 使用 YAML 格式
layout: cover
transition: slide-left
---

# 标题

内容...

<!-- 演讲者注释放在 HTML 注释块中 -->
```

**常用布局（layout）：**
- `cover` — 封面页
- `center` — 内容居中
- `two-cols` — 两栏布局
- `image-right` — 右侧图片
- `default`（默认）

**常用动画指令：**
- `<v-clicks>` / `</v-clicks>` — 列表逐条点击出现
- `v-click` — 单个元素点击出现
- `v-after` — 跟随前一个元素出现

### 引用外部代码片段

在幻灯片中引用 `snippets/` 目录下的代码：

```markdown
<<< @/snippets/external.ts#snippet
```

在 TypeScript 文件中使用区块标记：

```ts
// #region snippet
export function myFunc() {
  // 这部分会被引用到幻灯片中
}
// #endregion snippet
```

---

## 代码风格规范

### Vue 组件（components/*.vue）

```vue
<script setup lang="ts">
// 使用 Composition API + TypeScript
import { ref } from 'vue'

// defineProps 使用对象配置语法
const props = defineProps({
  count: {
    default: 0,
  },
})

const counter = ref(props.count)
</script>

<template>
  <!-- 使用 UnoCSS Attributify 原子化 CSS 风格 -->
  <div flex="~" w="min" border="~ main">
    <button @click="counter--">-</button>
    <span>{{ counter }}</span>
    <button @click="counter++">+</button>
  </div>
</template>
```

**Vue 组件规范：**
- 必须使用 `<script setup lang="ts">` — Composition API + TypeScript
- 使用 `ref()` 管理组件内部状态
- 模板中事件处理使用 `@` 简写（如 `@click`）
- 优先使用 UnoCSS Attributify 内联原子化 CSS
- 不使用 Options API

### TypeScript（snippets/*.ts）

```ts
/* eslint-disable no-console */

// 使用具名导出，不使用 export default
export function emptyArray<T>(length: number): T[] {
  return Array.from<T>({ length })
}

export function sayHello() {
  console.log('Hello World!')
}
```

**TypeScript 规范：**
- 2 空格缩进
- 单引号字符串（`'vue'` 而非 `"vue"`）
- 使用具名导出（`export function`），不使用 `export default`
- 泛型参数使用单字母大写（`<T>`）
- 使用现代 JS 语法（`Array.from<T>({length})`）
- 抑制不必要的 ESLint 警告时使用 `/* eslint-disable rule-name */`

### 导入约定

```ts
// 命名导入，单引号
import { ref, computed } from 'vue'
```

- 始终使用具名导入
- 使用单引号
- 路径别名：暂无自定义别名，Slidev 内置 `@/` 指向项目根目录

---

## 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| Vue 组件文件 | PascalCase | `Counter.vue` |
| TS/MD 文件 | kebab-case | `external.ts`、`imported-slides.md` |
| 函数名 | camelCase | `emptyArray`, `sayHello` |
| 变量名 | camelCase | `counter`, `props` |
| 泛型参数 | 单字母大写 | `<T>` |
| npm 包名 / 项目名 | kebab-case | `bazel-skill-ppt` |

---

## 注意事项

1. **不要引入测试框架**：本项目是演示文稿，不需要单元测试。
2. **不要添加 ESLint/Prettier 配置**：除非明确被要求，否则不添加 Lint 工具配置。
3. **主内容在 slides.md**：修改演示文稿内容时，编辑 `slides.md` 或 `pages/` 下的子文件。
4. **组件放在 components/**：Slidev 会自动注册该目录下的 Vue 组件，无需手动导入。
5. **代码片段放在 snippets/**：演示用的代码示例文件放在此目录，通过 `<<< @/snippets/` 语法引用。
6. **包管理器**：本地开发推荐使用 pnpm（已有 `.npmrc` 配置），CI 中使用 npm。
7. **Node.js 版本**：使用 Node.js 20（与 CI/CD 配置一致）。
