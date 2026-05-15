# Obsidian 笔记库

这是一个双链笔记系统，用于个人知识管理和博客文章创作。

## 目录结构

```
obsidian-vault/
├── 00 - Index/          # 索引和总览
├── 01 - Blog/           # 博客文章
└── 02 - Notes/          # 笔记
    ├── 00 - Inbox/      # 收集箱
    ├── 01 - Learning/   # 学习笔记
    ├── 02 - Projects/    # 项目笔记
    ├── 03 - Areas/      # 领域知识
    └── 04 - Resources/   # 资源收藏
```

## 使用说明

1. 使用 Obsidian 打开此目录
2. 博客文章放在 `01 - Blog/` 文件夹
3. 笔记放在 `02 - Notes/` 对应分类
4. 系统会自动同步到博客数据库

## 同步机制

博客系统会监控 `01 - Blog/` 文件夹的变化，自动将 Markdown 文件导入到 MongoDB 数据库中。
