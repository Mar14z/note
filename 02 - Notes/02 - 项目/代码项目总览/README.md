---
title: 代码项目总览
type: 项目
created: 2026-05-26
updated: 2026-10-10
tags:
  - 项目
  - 代码
  - 索引
status: 进行中
---

# 代码项目总览

> 汇总 `D:\documents\Code` 下的全部代码项目进度
> 原 `项目计划.md` 与 `项目进度汇总.md` 合并于本页

## 📊 项目一览表

| # | 项目名称 | 类型 | 优先级 | 完成度 | 状态 | 技术栈 |
|:--:|---------|------|:------:|:------:|------|--------|
| 1 | 静墨博客 (blog) | Web应用 | P1 | 90% | 🚀 成熟期 | Node.js + Express + MongoDB |
| 2 | SysMonitor - Linux系统监控 | 系统工具 | P0 | 5% | 📋 规划中 | C + Make |
| 3 | spiral-list - DNA螺旋列表 | 实验项目 | P2 | 100% | ✅ 已完成 | HTML + CSS |
| 4 | VSCode tmux 可视化插件 | 工具插件 | P1 | 0% | 🎯 待启动 | TypeScript |
| 5 | WordLand - 单词异世界 | 游戏化学习 | P1 | 0% | 🎯 规划中 | HTML + CSS + JS |

---

## 1. 静墨博客 (blog)

- **项目路径**：`D:\documents\Code\blog`
- **完成度**：90% ｜ **最后更新**：2026-05-10 ｜ **状态**：🚀 成熟期

极简优雅的博客系统，支持 Obsidian 双向同步。

| 功能 | 状态 |
|------|:----:|
| 文章管理 CRUD | ✅ |
| Obsidian 双向同步 | ✅ |
| RSS 订阅 (/feed.xml) | ✅ |
| SEO 优化 (OG标签、JSON-LD、sitemap) | ✅ |
| 相册页面 (瀑布流 + 懒加载) | ✅ |
| 管理后台 | ✅ |
| 安全中间件 (helmet/compression/rate-limit) | ✅ |
| XSS 防护 (sanitize-html) | ✅ |

**待办**：
- [ ] index.html 内联 CSS 提取到外部文件
- [ ] articles.html 内联 CSS/JS 提取
- [ ] about.html 语义化标签优化

**相关文档**：`spec/README.md`、`spec/changelog.md`、`README.md`（均在项目目录内）

> 项目笔记见 [[../博客系统/README|博客系统]]

---

## 2. SysMonitor - Linux 系统监控工具

- **项目路径**：`D:\documents\Code\linux-system-monitor`
- **完成度**：5% ｜ **创建时间**：2026-05-16 ｜ **状态**：📋 规划中

轻量级 Linux 系统监控工具，用于学习系统编程、多线程、进程通信等核心技能。

| 阶段 | 内容 | 状态 |
|:----:|------|:----:|
| 1 | 项目初始化和构建系统 | ✅ 基础完成 |
| 2 | 基础工具函数库 | ⏳ 待开始 |
| 3 | CPU 监控模块 | ⏳ 待开始 |
| 4 | 内存监控模块 | ⏳ 待开始 |
| 5 | 磁盘监控模块 | ⏳ 待开始 |
| 6 | 网络监控模块 | ⏳ 待开始 |
| 7 | 进程监控模块 | ⏳ 待开始 |
| 8 | 数据缓存层 | ⏳ 待开始 |
| 9 | 命令行界面 | ⏳ 待开始 |
| 10 | 配置管理系统 | ⏳ 待开始 |
| 11 | 告警系统 | ⏳ 待开始 |
| 12 | 文本界面 (TUI) | ⏳ 待开始 |
| 13 | Web服务接口 | ⏳ 待开始 |

**相关文档**：`SPEC.md`、`LEARNING_PLAN.md`、`README.md`（均在项目目录内）

> 项目笔记见 [[../系统监控/README|系统监控]]

---

## 3. spiral-list - DNA 螺旋列表

- **项目路径**：`D:\documents\Code\TraeCode\experiments\spiral-list`
- **完成度**：100% ｜ **状态**：✅ 已完成

具有 DNA 双螺旋视觉效果的博客文章列表，纯 CSS 动画实现。

- DNA 旋转动画（每行左右交替摆动）
- 黑白像素风格
- 纯 CSS 实现（高性能、GPU 加速）
- 响应式设计
- 无障碍支持（减少动画偏好）

---

## 4. VSCode tmux 可视化插件

- **项目路径**：待定 ｜ **状态**：🎯 待启动

在 VSCode 侧边栏可视化显示 tmux 窗口和窗格结构，点击可跳转到对应窗格。

| 功能 | 优先级 | 状态 |
|------|:------:|:----:|
| 树形结构展示（会话 > 窗口 > 窗格） | P0 | 📋 待设计 |
| 实时刷新 tmux 状态 | P0 | 📋 待设计 |
| 点击跳转到对应窗格 | P0 | 📋 待设计 |
| 当前窗格高亮显示 | P1 | 📋 待设计 |

**技术方案**：TypeScript + VSCode Extension API + child_process 调用 tmux + TreeView/Webview

---

## 5. WordLand - 单词异世界

- **项目路径**：`D:\documents\Code\WordLand` ｜ **状态**：🎯 规划中

游戏化英语学习工具，像素风 + 养成 + 沙盒世界。完成任务获得金币，用金币装饰像素世界。

| 模块 | 规划 |
|------|------|
| **学习系统** | 背单词 + 听力 + 阅读，RPG任务形式 |
| **金币系统** | 完成学习任务获得金币 |
| **沙盒世界** | 像素风，类似 WorldBox / 矮人要塞 |
| **养成元素** | 建筑、自然、居民（纯装饰性起步） |
| **未来扩展** | Agent 自动拆解任务、Android 应用 |

**金币奖励（初期）**：背单词 5-10 💰 ｜ 听力 8-15 💰 ｜ 阅读 10-20 💰 ｜ 连续 7 天 +50 💰

---

## 📈 优先级与规划

| 优先级 | 项目 | 建议 |
|:------:|------|------|
| P0 | SysMonitor | 按学习计划推进，每天一个模块 |
| P1 | blog | 完成后可作为长期博客平台 |
| P2 | TraeCode experiments | 技术探索，碎片化时间进行 |

**近期计划**
- 短期（1-2周）：WordLand 完成设计并启动；VSCode tmux 插件完成设计并启动
- 中期（1个月）：WordLand MVP；SysMonitor 阶段 2-3；blog 完成剩余优化
- 长期：WordLand Agent 拆解 + Android；SysMonitor 全部 13 阶段；blog 生产部署

## 🛠 技术栈分布

| 技术栈 | 项目 |
|--------|------|
| Node.js / Express | blog |
| MongoDB | blog |
| C / Make | linux-system-monitor |
| CSS Animation | spiral-list |

## 📝 备注

- blog 项目已具备生产环境部署能力（Docker 支持）
- SysMonitor 基于具身智能学习路径的阶段性项目
- TraeCode 是实验性项目的集合平台
- WordLand 网页版先行，后续扩展 Android

---

## 🔗 相关笔记

- [[../博客系统/README|博客系统项目笔记]]
- [[../系统监控/README|系统监控项目笔记]]
- [[../README|项目笔记索引]]
- [[../../01 - 学习计划/机器人/README|机器人学习计划]]