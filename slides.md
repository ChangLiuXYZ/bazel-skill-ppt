---
theme: seriph
background: https://cover.sli.dev
title: 用 Skill 让 AI 自动迁移 900+ 个 iOS 组件库
info: |
  ## 用 Skill 让 AI 自动迁移 900+ 个 iOS 组件库
  五次迭代，从跑通到跑稳再到批量提速
class: text-center
transition: slide-left
duration: 15min
---

# 用 Skill 让 AI 自动迁移<br>900+ 个 iOS 组件库

五次迭代，从跑通到跑稳再到批量提速

<div class="abs-br m-6 text-sm opacity-50">
  小组内部分享 · 15min
</div>

<!--
开场：我做了一件事，把一个繁琐的迁移工作交给 AI 来执行。今天不是来讲迁移技术细节的，而是讲这个「让 AI 越来越会干活」的过程。
-->

---
layout: statement
transition: fade-out
---

# 问题背景

我们需要把 **900+** 个 CocoaPods 组件库  
迁移到 Bazel

<div v-click class="mt-8 text-2xl opacity-80">
每个库都要：克隆 → 写配置 → 构建验证 → 推送 → 注册
</div>

<div v-click class="mt-4 text-xl text-orange-400">
重复、繁琐、容易出错
</div>

<!--
背景非常简单：就是有大量重复的迁移工作。每个库的步骤几乎一样，但细节各不相同。
-->

---
layout: center
transition: fade
---

# 我的解法：AI Skill

<div class="flex items-center justify-center gap-12 mt-8">

<div v-click class="text-center">
  <div class="text-5xl mb-3">📝</div>
  <div class="text-lg font-bold">Skill = Markdown 文件</div>
  <div class="text-sm opacity-70 mt-2">写清楚 AI 该怎么做</div>
</div>

<div v-click class="text-4xl opacity-50">→</div>

<div v-click class="text-center">
  <div class="text-5xl mb-3">🤖</div>
  <div class="text-lg font-bold">AI 识别意图</div>
  <div class="text-sm opacity-70 mt-2">自动加载 Skill</div>
</div>

<div v-click class="text-4xl opacity-50">→</div>

<div v-click class="text-center">
  <div class="text-5xl mb-3">✅</div>
  <div class="text-lg font-bold">自动执行</div>
  <div class="text-sm opacity-70 mt-2">粘贴 GitLab 链接就开始</div>
</div>

</div>

<div v-click class="mt-12 text-center text-sm opacity-60 border border-white/20 rounded px-4 py-2 inline-block">
  触发方式：粘贴一个 GitLab 仓库 URL → AI 自动开始迁移
</div>

<!--
Skill 就是一个说明书，AI 读懂之后按步骤执行。用户体验极简：粘贴 URL，剩下的 AI 来。
-->

---
layout: section
transition: slide-up
---

# 第一版

只有迁移，缺乏验证

---
layout: default
transition: slide-up
---

# 第一版能做什么

已经支持完整的迁移流程：

<v-clicks>

- **克隆仓库** — 自动检查是否已迁移
- **分析 .podspec** — 依赖、源文件、编译标志
- **生成配置** — BUILD.bazel / MODULE.bazel / .bazelrc

</v-clicks>

<div v-click class="mt-8 p-4 bg-orange-900/30 rounded">
⚠️ 但迁移完成后，<strong>没有自动验证</strong><br>
<span class="text-sm opacity-80 mt-1 block">需要人工跑 bazel build 检查，才知道配置写得对不对</span>
</div>

---
layout: two-cols
transition: fade
---

# 解决：把验证步骤也写进 Skill

**要求 AI 在迁移完成后必须验证：**

<v-clicks>

- **独立构建** — `bazel build //:LibName`
- **集成验证** — 在 bazel-demo 工程里跑一次完整构建

</v-clicks>

<div v-click class="mt-4">

```markdown
> **硬性卡点：构建必须通过，**
> **方可继续下一步。**
> 禁止在构建未通过的情况下
> 进入提交推送步骤
```

</div>

<div v-click class="mt-4 p-3 bg-blue-900/20 rounded text-sm">
效果：<strong>自动验收</strong>，从迁移到验证一气呵成
</div>

::right::

<div v-click class="mt-2 ml-4 transition-all duration-500">

<div class="text-xs opacity-60 mb-2">此时 Skill 骨架</div>

```markdown {all|11-15}
# CocoaPods → Bazel 迁移 Skill

## 第一步：克隆仓库
- 前置检查是否已迁移
- 分配 bazel-demo 工程

## 第二步：执行迁移
- 分析 .podspec
- 生成 BUILD.bazel / MODULE.bazel

## 第三步：验证构建  ← 新增
### 3.1 独立构建验证
  bazel build //:LibName
### 3.2 集成验证（bazel-demo）
  硬性卡点：必须通过才能继续
```

</div>

<!--
关键发现：对 AI 说"建议"没用，要说"禁止"。AI 把规则当命令，措辞很重要。
-->

---
layout: section
transition: slide-up
---

# 第二版

验证通了，但注册还要手动

---
transition: slide-up
---

# 问题：迁移和注册是两件事

<div class="mt-4">

Bazel 的库需要注册到 `bazel-registry` Git 仓库，  
其他工程才能通过 Bazel Module 引用它：

<div v-click class="mt-4 p-4 bg-slate-800/50 rounded text-sm font-mono">

```
bazel-registry/
  modules/
    ne_action_sheet/
      1.0.0/
        source.json
```

</div>

<div v-click class="mt-4 text-orange-400">
每次迁移完，还要手动执行注册命令，把组件信息写入 registry
</div>

<div v-click class="mt-2 text-sm opacity-70">
命令出错、版本号不对... 既繁琐又容易出错
</div>

</div>

---
layout: two-cols
transition: fade
---

# 解决：把注册提交也写进 Skill

**要求 AI 在验证通过后自动完成注册：**

<v-clicks>

1. 提交并推送组件库的 feature/bazel 分支
2. 执行注册命令，写入 bazel-registry
3. 推送到 registry Git 仓库
4. <span class="text-yellow-400 font-bold">再用 registry 方式做一次构建验证，确认注册完全正确</span>

</v-clicks>

<div v-click class="mt-4 p-3 bg-blue-900/20 rounded text-sm">
效果：完完整整完成一个组件的迁移到注册，<strong>形成完整闭环</strong>
</div>

::right::

<div v-click class="mt-2 ml-4 transition-all duration-500">

<div class="text-xs opacity-60 mb-2">此时 Skill 骨架</div>

```markdown {all|11-17}
# CocoaPods → Bazel 迁移 Skill

## 第一步：克隆仓库

## 第二步：执行迁移
- 分析 .podspec
- 生成 BUILD.bazel / MODULE.bazel

## 第三步：验证构建
- 独立构建 + 集成验证（硬性卡点）

## 第四步：提交推送 + 注册  ← 新增
### 4.1 推送 feature/bazel 分支
### 4.2 注册到 bazel-registry
  创建 source.json，提交推送
### 4.3 注册表验证
  用 registry 方式再构建一次
```

</div>

---
layout: section
transition: slide-up
---

# 第三版

AI 的思考用完就丢，经验无法复用

---
layout: two-cols
transition: slide-up
---

# 问题：思考完成就消失了

<div class="mt-4">

迁移一个组件，AI 会遇到各种编译问题：

<v-clicks>

- xcframework 头文件找不到
- Swift module 冲突
- C++ 标准不兼容
- ...

</v-clicks>

<div v-click class="mt-4">
AI 经过思考、尝试、终于解决了
</div>

<div v-click class="mt-3 p-3 bg-red-900/20 rounded text-sm">
迁移完成 → 对话结束 → <span class="text-red-400">这些思考全部消失</span><br>
下次遇到同样的问题，从头再来
</div>

</div>

::right::

<div v-click class="mt-8">

```
每次迁移
  ├── AI 分析错误
  ├── AI 思考方案
  ├── AI 修复验证
  ├── 迁移完成
  └── 💨 所有思考就此消失
```

<div class="mt-4 text-sm opacity-60">
900+ 个库要迁移<br>
同样的坑可能踩无数次
</div>

</div>

---
layout: two-cols
transition: fade
---

# 解决：完成后把经验写进 cases 目录

**Skill 新增：迁移完成后，把遇到的问题和解决方案总结成文档，放到 `cases/` 目录**

<v-clicks>

- 下次遇到构建问题，AI 优先去 cases 查
- 找到类似问题 → 直接复用方案

</v-clicks>

<div v-click class="mt-3 p-3 bg-slate-800/50 rounded text-xs font-mono">

```
cases/
  could_not_build_module.md
  undefined_symbols_for_architecture.md
  framework_not_found_bundle_name_mismatch.md
  ...
```

</div>

<div v-click class="mt-3 p-3 bg-blue-900/20 rounded text-sm">
效果：<strong>构建数据飞轮</strong> — 迁移越多，积累越多，下次越快
</div>

::right::

<div v-click class="mt-2 ml-4 transition-all duration-500">

<div class="text-xs opacity-60 mb-2">此时 Skill 骨架</div>

```markdown {all|10-11,14-18}
# CocoaPods → Bazel 迁移 Skill

## 第一步：克隆仓库

## 第二步：执行迁移
- 分析 .podspec
- 生成 BUILD.bazel / MODULE.bazel

## 第三步：验证构建
- 遇到构建失败：
  优先查 cases/ 目录  ← 新增
- 独立构建 + 集成验证（硬性卡点）

## 第四步：提交推送 + 注册

## 第五步：经验沉淀  ← 新增
- 回顾本次遇到的构建错误
- 总结「错误 + 根因 + 解法」
- 写入 cases/ 目录
```

<div class="mt-3 text-xs opacity-60 mb-1">此时 Skill 目录结构</div>

```
podlib-migration-to-bazel-guide/
├── SKILL.md
└── cases/
    └── could_not_build_module.md
```

</div>

<!--
这是整个方案里我最满意的设计：AI 不只是执行者，它还在给系统喂数据。
-->

---
layout: section
transition: slide-up
---

# 第四版

cases 越多，上下文越大，效果越差

---
transition: slide-up
---

# 问题：知识越积越多，上下文越撑越大

<div class="mt-6 flex items-center justify-center gap-8">

<div v-click class="text-center p-4 bg-slate-800/50 rounded w-40">
  <div class="text-3xl mb-2">📚</div>
  <div class="text-sm">cases 越来越多</div>
</div>

<div v-click class="text-2xl opacity-50">→</div>

<div v-click class="text-center p-4 bg-orange-900/30 rounded w-52">
  <div class="text-3xl mb-2">📊</div>
  <div class="text-sm">每次都全部读一遍<br>上下文占用爆炸</div>
</div>

<div v-click class="text-2xl opacity-50">→</div>

<div v-click class="text-center p-4 bg-red-900/30 rounded w-48">
  <div class="text-3xl mb-2">🐢</div>
  <div class="text-sm">AI 变慢<br>变不准</div>
</div>

</div>

<div v-click class="mt-8 text-center text-sm p-3 bg-slate-700/30 rounded">
cases 积累 → 上下文占用增加 → 效果越来越差 → 知识反成负担
</div>

---
layout: two-cols
transition: fade
---

# 解决：文件名归类 + XML Header 速读

**用文件名归类问题，每个文件的头部做简短描述：**

```yaml
---
title: "could not build module 'XXX'"
error_pattern: "could not build module"
keywords:
  - enable_modules
  - xcframework header import
---
```

<div v-click class="mt-3 text-sm opacity-80">
AI 读 cases 时，<strong>先只读 header</strong> 做匹配<br>
命中了再读全文，未命中直接跳过
</div>

<div v-click class="mt-3 p-3 bg-blue-900/20 rounded text-sm">
效果：<strong>优化上下文占用</strong> — 无论 cases 多少，查阅消耗可控
</div>

::right::

<div v-click class="mt-2 ml-4 transition-all duration-500">

<div class="text-xs opacity-60 mb-2">此时 Skill 骨架</div>

```markdown {all|10-14,22-23}
# CocoaPods → Bazel 迁移 Skill
## 第一步：克隆仓库
## 第二步：执行迁移
- 分析 .podspec
- 生成 BUILD.bazel / MODULE.bazel
## 第三步：验证构建
- 遇到构建失败，查 cases：
  1. 先扫所有文件的 header  ← 优化
     匹配 error_pattern
  2. 只读命中文件的全文
  3. 未命中直接跳过
- 独立构建 + 集成验证（硬性卡点）
## 第四步：提交推送 + 注册
## 第五步：经验沉淀
- 三道过滤判断是否值得写  ← 优化
- 写入 cases/（带 header）
```

<div class="mt-3 text-xs opacity-60 mb-1">此时 Skill 目录结构</div>

```
podlib-migration-to-bazel-guide/
├── SKILL.md
└── cases/
    ├── could_not_build_module.md          ← 带 header
    └── undefined_symbols_for_architecture.md
```

</div>

---
layout: section
transition: slide-up
---

# 第五版

一个一个迁移太累了

---
transition: slide-up
---

# 问题：串行迁移，效率太低

<div class="mt-4">

900+ 个库，一个一个迁移：

<div v-click class="mt-4 p-4 bg-slate-800/50 rounded text-sm font-mono">

```
迁移 NEActionSheet    → 等待完成 → 迁移 NENetworking
→ 等待完成 → 迁移 NELogger → 等待完成 → ...
```

</div>

<div v-click class="mt-6 text-orange-400 text-xl">
每个库平均需要 10~20 分钟<br>
串行跑完 900 个，需要数百小时
</div>

<div v-click class="mt-4 text-sm opacity-70">
一个一个迁移
</div>

</div>

---
layout: two-cols
transition: fade
---

# 解决：新建批量迁移 Skill

**再写一个 Skill，专门负责并行调度：**

<v-clicks>

- 接收一批 GitLab 链接
- 启动 N 个 Subagent 并行处理
- 每个 Subagent 独立调用迁移 Skill
- 全部完成后汇总结果

</v-clicks>

<div v-click class="mt-4 p-3 bg-blue-900/20 rounded text-sm">
效果：<strong>N 倍速度</strong>，人只需粘贴链接列表，剩下的全交给 AI
</div>

::right::

<div v-click class="mt-2 ml-4 transition-all duration-500">

<div class="text-xs opacity-60 mb-2">批量迁移 Skill 架构</div>

```
batch-pod-migration Skill
  ↓ 解析输入，分配任务
  ├── Subagent 1
  │     └── 调用迁移 Skill → NEActionSheet
  ├── Subagent 2
  │     └── 调用迁移 Skill → NENetworking
  └── Subagent 3
        └── 调用迁移 Skill → NELogger
  ↓ 等待全部完成
  汇总结果表格
```

<div class="mt-3 text-xs opacity-60 mb-1">Skill 目录结构</div>

```
.claude/skills/
├── podlib-migration-to-bazel-guide/
│   ├── SKILL.md
│   └── cases/
└── batch-pod-migration/
    └── SKILL.md
```

</div>

---
transition: slide-up
---

# 批量迁移带来的新问题：Demo 工程冲突

每个组件验证都需要一个 Demo 工程跑 `bazel build`

<div v-click class="mt-6 p-4 bg-slate-800/50 rounded text-sm font-mono">

```
Subagent 1 迁移 NEActionSheet  ─┐
Subagent 2 迁移 NENetworking   ─┼─ 同时抢占同一个 bazel-demo ──→ 💥 互相覆盖
Subagent 3 迁移 NELogger       ─┘
```

</div>

<div v-click class="mt-6 grid grid-cols-2 gap-4">

<div class="p-3 bg-red-900/20 rounded text-sm">
  <div class="font-bold text-red-400 mb-2">❌ 如果共用同一个 Demo</div>
  <div>MODULE.bazel 被互相覆盖<br>构建结果互相污染<br>验证结论完全不可信</div>
</div>

<div v-click class="p-3 bg-slate-700/30 rounded text-sm">
  <div class="font-bold text-yellow-400 mb-2">需要解决</div>
  <div>N 个并行 Subagent<br>需要 N 个独立的 Demo 工程<br>且要安全地「分配」与「归还」</div>
</div>

</div>

---
layout: two-cols
transition: fade
---

# 解决：Demo 工程池

**每个迁移任务独立申请一个 Demo 工程：**

<v-clicks>

- 迁移开始时，从工程池申请一个**空闲** Demo 工程
- 没有空闲工程时，自动 clone 一个新的加入池子
- 在该工程上完成专项验证构建
- 迁移完成后，将工程**归还**到池子，恢复空闲状态
- 下一个迁移任务申请时，直接复用已有工程

</v-clicks>

<div v-click class="mt-4 p-3 bg-blue-900/20 rounded text-sm">
工程池保留 <code>bazel-output/</code> 编译缓存，复用时无需重新下载依赖，<strong>大幅加速构建</strong>
</div>

::right::

<div v-click class="mt-4 ml-4 transition-all duration-500">

<div class="text-xs opacity-60 mb-2">工程池运转示意</div>

```
开始批量迁移
  ├── Subagent 1 → 申请 → bazel-demo-1（专项适配）
  ├── Subagent 2 → 申请 → bazel-demo-2（专项适配）
  └── Subagent 3 → 申请 → bazel-demo-3（自动新建）

Subagent 1 完成 → 归还 bazel-demo-1
  └── Subagent 4 → 申请 → bazel-demo-1（复用）

Subagent 2 完成 → 归还 bazel-demo-2
  └── Subagent 5 → 申请 → bazel-demo-2（复用）
```

</div>

---
layout: section
transition: slide-up
---

# 批量迁移带来新的问题

电脑越跑越慢

---
transition: fade
---

# 问题：fseventsd 吃掉整台 Mac

<div v-click class="mt-4 grid grid-cols-2 gap-4">

<div class="p-3 bg-red-900/20 rounded text-sm">
  <div class="text-red-400 font-bold mb-2">😱 问题现象</div>
  <div>批量迁移跑了一晚上，早上来看执行进度不理想</div>
  <div class="mt-2 opacity-80">CPU 和内存全部爆满，系统整体卡顿</div>
</div>

<div v-click class="p-3 bg-slate-700/30 rounded text-sm">
  <div class="text-yellow-400 font-bold mb-2">🔍 排查后发现</div>
  <div>罪魁祸首不是 AI，而是 Bazel 编译产物触发了系统级的文件监听风暴</div>
</div>

</div>

<div class="mt-6 flex items-center justify-center gap-6">

<div v-click class="text-center p-4 bg-slate-800/50 rounded w-44">
  <div class="text-3xl mb-2">⚡️</div>
  <div class="text-sm">Bazel 并发编译<br>海量文件写入</div>
</div>

<div v-click class="text-2xl opacity-50">→</div>

<div v-click class="text-center p-4 bg-orange-900/30 rounded w-52">
  <div class="text-3xl mb-2">🔍</div>
  <div class="text-sm">Spotlight 疯狂建索引<br>30+ mdworker 进程</div>
</div>

<div v-click class="text-2xl opacity-50">→</div>

<div v-click class="text-center p-4 bg-red-900/30 rounded w-52">
  <div class="text-3xl mb-2">💀</div>
  <div class="text-sm">fseventsd ~100% CPU<br>内存积累至 14GB</div>
</div>

</div>

<div v-click class="mt-4 text-center text-sm p-3 bg-slate-700/30 rounded">
VS Code 的 Code Helper 也在监听这些目录的文件变化，进一步加剧 CPU 占用
</div>

<!--
根本原因：Bazel 编译产物（bazel-bin、bazel-out 等）的大量文件写入，触发了 fseventsd → Spotlight → mdworker 这条事件链。VS Code 的文件监听雪上加霜。
-->

---
layout: two-cols
transition: fade
---

# 解决：三步切断事件链

<v-clicks>

**1. 排除 Spotlight 索引**

系统设置 → 聚焦 → 搜索隐私，将 `/Users/changliu/workspace` 加入排除列表

Bazel 编译产物（`bazel-bin`、`bazel-out` 等临时文件）不再进入 Spotlight 索引，mds_stores、mdworker 进程大幅减少

**2. 排除 VS Code 文件监听**

在 `.vscode/settings.json` 中配置：

```json
{
  "files.watcherExclude": {
    "**/bazel-bin/**": true,
    "**/bazel-out/**": true,
    "**/bazel-testlogs/**": true
  },
  "search.exclude": {
    "**/bazel-bin/**": true,
    "**/bazel-out/**": true
  }
}
```

VS Code 不再监听 Bazel 编译产物目录，Code Helper 进程的 CPU 占用恢复正常

**3. 立即释放内存（可选）**

```bash
sudo pkill -HUP fseventsd
```

</v-clicks>

::right::

<div v-click class="ml-6 mt-2">

<div class="text-sm opacity-60 mb-3">事件链：切断前 vs 切断后</div>

<div class="p-3 bg-red-900/20 rounded mb-4 text-sm">
  <div class="text-red-400 font-bold mb-1">切断前</div>
  Bazel 写入 → fseventsd 积累 → Spotlight 建索引 → 系统卡顿
</div>

<div class="p-3 bg-green-900/20 rounded text-sm">
  <div class="text-green-400 font-bold mb-1">切断后</div>
  Bazel 写入 → fseventsd 正常 → Spotlight 不介入 → 系统流畅
</div>

<div class="mt-6 p-3 bg-blue-900/20 rounded text-sm">
  💡 批量自动化任务跑起来之后，<strong>系统级的资源竞争</strong>也要纳入考虑
</div>

</div>

<!--
这个问题在单次迁移时几乎感知不到，批量并发之后才暴露出来。解决思路就是切断"文件写入 → 系统监听"这条事件链的每一个节点。
-->

---
transition: slide-up
---

# 五个阶段的演进

<div class="mt-4">

| 版本 | 问题 | 解决 | 效果 |
|------|------|------|------|
| v1 | 只有迁移，缺乏验证 | 把验证步骤写进 Skill | 自动验收，从迁移到验证 |
| v2 | 注册到 Git 仓库还要手动 | 把注册提交也写进 Skill | 完整闭环，迁移到注册 |
| v3 | AI 思考用完就消失 | cases 目录沉淀问题与解法 | 构建数据飞轮 |
| v4 | cases 越多上下文越大 | 文件名归类 + Header 速读 | 优化上下文占用 |
| v5 | 串行迁移效率太低 | 批量 Skill + 并行 Subagent | N 倍速度提升 |

</div>

<div v-click class="mt-6 text-center text-sm opacity-70">
每一版都是在实际迁移中发现问题，然后改进 Skill
</div>

---
layout: statement
transition: fade
---

# 把 AI 变成流水线工人

# 把自己变成流水线设计师

<div v-click class="mt-8 text-xl opacity-70">
Skill 就是那条流水线的说明书
</div>

<div v-click class="mt-4 text-lg opacity-50">
你不断优化说明书 → AI 越来越高效 → 积累的经验反哺说明书
</div>

<!--
这是整个分享最想传达的一句话。人的价值不在于重复执行，而在于设计和优化系统。
-->

---
layout: center
class: text-center
transition: fade
---

# 这不只是一个迁移工具

<div v-click class="text-2xl mt-6 text-blue-400">
这是一个可以自我成长的系统
</div>

<div v-click class="mt-8 text-sm opacity-60">
cases 目录会越来越丰富 · 评分会越来越准 · 速查表会自动更新
</div>

---
layout: end
---

# 谢谢

<div class="mt-4 opacity-60 text-sm">

欢迎交流 · 如果你的工作也有大量重复任务，欢迎探讨怎么用 Skill 解决

</div>
