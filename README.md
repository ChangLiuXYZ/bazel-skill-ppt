# 用 Skill 让 AI 自动迁移 900+ 个 iOS 组件库

> 一个内部分享演示文稿，讲述如何通过编写 AI Skill，让 AI 自动完成将 900+ 个 iOS CocoaPods 组件库迁移到 Bazel 的全流程——记录从跑通到跑稳再到批量提速的五次迭代过程。

## 在线预览

**GitHub Pages：** <https://changliuxyz.github.io/bazel-skill-ppt/>

## 本地开发

```bash
# 安装依赖
npm install

# 启动开发服务器（自动打开浏览器，支持热重载）
npm run dev
```

访问 <http://localhost:3030> 查看演示文稿。

编辑 [`slides.md`](./slides.md) 即可实时看到变更。

## 构建 & 部署

```bash
# 静态构建，输出到 dist/
npm run build

# 导出为 PDF / PPTX
npm run export
```

部署由 GitHub Actions 自动触发，推送到 `main` 分支后自动发布到 GitHub Pages。
