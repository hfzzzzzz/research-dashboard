# 科研记录平台(research-dashboard)

> 单文件 HTML 仪表盘:通过 GitHub API 展示科研项目仓库的提交时间线、实验结果图表、CLAUDE.md 阅读版与实验计划。

## 项目约束

- **单文件、零依赖**:全部功能在 `index.html` 内(内联 CSS/JS),不引入任何构建步骤、npm 包或 CDN 资源。必须保证 `file://` 双击打开可用(因此数据一律走 GitHub API/contents 接口,不做相对路径 fetch)。
- **兼容性**:面向现代浏览器(Chrome/Edge/Firefox/Safari 最近版本),允许使用 `??`、`color-mix()`、`<dialog>`。
- **安全**:所有动态文本经 `textContent` 或 `esc()` 插入;Markdown 渲染不放行原始 HTML;URL 经 `safeUrl()` 协议白名单过滤。修改渲染逻辑时保持这三条不变。

## 目录结构

```
index.html                     仪表盘(全部代码)
templates/CLAUDE-template.md   科研项目 CLAUDE.md 标准模板
templates/plan.example.json    plan.json 示例
templates/results.example.json results.json 示例
README.md                      使用与部署文档
```

## 数据契约

平台读取科研项目仓库的三个文件:`CLAUDE.md`、`research/plan.json`、`research/results.json`。字段规范以 `templates/CLAUDE-template.md` 的「research 数据文件说明」为准——**修改解析逻辑时必须同步更新该模板与 README**,反之亦然。

## 设计规范(图表)

- 配色使用文件顶部 CSS 变量(`--s1`~`--s8`、状态色),浅/深双模式各自校验过色觉安全;不要新增或改动色值顺序。
- 图表规格:线宽 2px、柱宽 ≤24px 顶部 4px 圆角、数据点带 2px 表面色环、网格线为实线细线;≥2 系列必有图例;散点图最多 3 个系列;每张图保留「数据表」切换。
- 文本永远使用文本色令牌(`--ink`/`--ink2`/`--muted`),不用系列色写字。

## 提交规范

`<type>: <说明>`,type ∈ feat / fix / docs / refactor / chore。改动 UI 后在本地打开 `index.html` 验证演示数据的五个页签均正常渲染(含深色模式)再提交。
