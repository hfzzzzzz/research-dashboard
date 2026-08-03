# 科研记录平台

单文件 HTML 仪表盘,通过 GitHub API 汇总展示科研项目仓库的研究进展——无需构建、无外部依赖,双击即可打开,也可部署为 GitHub Pages。

| 页面 | 内容 | 数据来源 |
|------|------|----------|
| 概览 | 各项目卡片:提交活跃度、实验状态计数、最新结果 | 聚合下列各项 |
| 更新记录 | 按日分组的提交时间线,类型徽章、搜索与筛选 | GitHub commits API |
| 实验结果 | 指标块 + 折线/柱状/散点图(悬浮读数、数据表视图)+ 图片 | 仓库 `research/results.json` |
| 研究方法 | CLAUDE.md 渲染为带目录的阅读版 | 仓库 `CLAUDE.md` |
| 实验计划 | 状态统计、甘特时间线、里程碑、假设/预期/实际对照 | 仓库 `research/plan.json` |

## 快速开始

1. **打开** `index.html`(直接双击即可;数据全部通过 GitHub API 获取,本地打开也能正常工作)。首次打开显示内置演示项目,可先浏览全部功能。
2. **关联你的仓库**:点击右上角 ⚙ 设置 → 填入 `owner/repo`(每行一个,可 `owner/repo#分支` 指定分支),或输入 GitHub 用户名一键加载仓库列表勾选。

   也可以用 URL 参数一次带入,适合在多台设备间同步同一组项目:
   `…/research-dashboard/?repos=owner/repo1,owner/repo2#results`
   打开后项目会写入该设备的 localStorage,之后不带参数访问也在。**私有项目建议用这种方式**——清单只存在于你的链接和浏览器里,不会进入公开的 Pages 仓库。
3. **私有仓库 / 提高限额**(可选):在 GitHub → Settings → Developer settings → Fine-grained tokens 创建只读令牌(仓库权限勾选 **Contents: Read-only**),填入设置。不填令牌时公开仓库可用,限每小时 60 次请求(每个项目一次完整加载约 5 次请求)。令牌仅保存在本机浏览器 localStorage。

## 让一个科研项目接入平台

在科研项目仓库中添加三样东西:

1. **`CLAUDE.md`**(仓库根目录)——复制 [templates/CLAUDE-template.md](templates/CLAUDE-template.md),替换占位符。它同时是 Claude Code 的项目工作说明与平台「研究方法」页的渲染源,一份文件两用。
2. **`research/plan.json`** —— 实验安排、假设、预期结果、时间线。参考 [templates/plan.example.json](templates/plan.example.json)。
3. **`research/results.json`** —— 实验结果(图表数据、指标、结论)。参考 [templates/results.example.json](templates/results.example.json)。图片放 `figures/` 目录,在结果的 `images` 字段引用。

之后的日常流程(模板中的「实验与记录规范」已把这些约定写给 Claude Code,由它在每次实验后自动维护):

```
实验开始 → plan.json 登记(planned→running)→ 提交 "exp: ..."
实验完成 → plan.json 更新状态与 actual;results.json 头部追加结果 → 提交 "result: 结论 + 关键数字"
```

提交信息前缀(`exp:` `result:` `data:` `analysis:` `fig:` `paper:` `feat:` `fix:` `docs:` `chore:`)会在「更新记录」页渲染为类型徽章并支持筛选。也支持 `type(scope):` 格式与项目自定义前缀(如 `stage2:` `sync:` `downstream:`),未知前缀会渲染为中性徽章并同样可筛选,不必为平台改动既有提交习惯。

## 发布到 GitHub Pages(可选)

本地双击打开已完全可用;想要一个固定网址时:

```bash
cd research-dashboard
git init -b main   # 若尚未初始化
git add -A && git commit -m "feat: 科研记录平台"
# 在 github.com 上新建仓库(建议私有:research-dashboard),然后:
git remote add origin https://github.com/<你的用户名>/research-dashboard.git
git push -u origin main
```

仓库 Settings → Pages → Source 选 **GitHub Actions**(本仓库带 [.github/workflows/pages.yml](.github/workflows/pages.yml),首次需在设置里选一次,之后每次 `git push` 自动部署),保存后访问 `https://<用户名>.github.io/research-dashboard/`。

> 注意:GitHub Pages 页面本身是公开的(免费账户下,即使仓库私有,Pages 站点也公开)。令牌保存在浏览器本地、不会进入仓库,但若担心他人访问到你的面板页面,建议仅本地使用,或使用私有仓库 + 不启用 Pages。
>
> **不要把未发表项目的仓库名写进部署目录下的 `projects.json`**——那个文件会随 Pages 一起公开,等于在投稿前公开了选题。私有项目请用上面的 `?repos=` 链接或 ⚙ 设置,两者都只写入本机浏览器。`projects.json` 只适合公开项目。

## results.json / plan.json 字段规范

完整规范见 [templates/CLAUDE-template.md](templates/CLAUDE-template.md) 的「research 数据文件说明」一节。要点:

- `chart.type`:`line` / `bar` / `scatter`;`series[].data` 为 `[x, y]` 数组,x 可为数字或类目字符串
- 散点图最多 3 个系列(色觉安全约束);`"diag": true` 加 y=x 参考线
- `metrics[].delta` + `good`(`down`/`up`)控制增减着色;新结果放数组**头部**
- 日期一律 `YYYY-MM-DD`

## 常见问题

- **打开后一直「加载中」/ 报请求失败**:检查网络;`file://` 打开时 GitHub API 依然可用(接口允许跨域),无需本地服务器。
- **提示请求次数用完(403)**:未填令牌时限每小时 60 次,填入令牌后为 5000 次/小时。
- **私有仓库 404**:需要填入有该仓库 Contents 读取权限的 fine-grained token。
- **CLAUDE.md 里的 HTML 不显示**:渲染器出于安全只支持 Markdown 语法,原始 HTML 会被转义为文本。
- **图表不显示**:检查 `research/results.json` 是否为合法 JSON(页面会显示解析错误位置);`chart.series` 不能为空。
- **修改了仓库数据但页面没变**:点右上角 ⟳ 刷新(页面会缓存本次会话的数据)。
