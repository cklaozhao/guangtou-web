# 光头 Obsidian 教程落地页

> 单文件 HTML 落地页，作为「光头 Obsidian 教程」（飞书 wiki）的入口页。
> 在线：<https://guangtou.me>

## 来龙去脉

灵感来自 [小米 MiMo 100T Hero 页](https://100t.xiaomimimo.com/)，被它的 hero 视觉打动。我用 [Claude Code](https://claude.com/claude-code) 把对方的 HTML 源码扒下来作为起点（仓库里的 `源码.html`），再结合自己的需求一轮又一轮魔改，最终成了现在这个样子：保留了 cyber/glitch 字符矩阵背景 + 鼠标光圈跟随的核心氛围，加上了头像双层切换、扩散过渡、标题打字机等定制交互。

整个项目的 6 次迭代历程、踩过的坑、以及"当你不知道一个东西叫什么时该怎么和 AI 描述"的方法论，都整理在 `新人手册.html` 里 —— 浏览器打开看。

## 视觉特色

- 背景：canvas 字符矩阵滚动（"glitch" 风格）
- 头像：双层结构 —— 顶层线条画，底层真人照，鼠标进出时切换
- cursor-ring：自定义鼠标光圈，跟随 + `mix-blend-mode: difference`（颜色反色）
- 标题：打字机动画 + hover 下划线展开
- 鼠标进头像时：底层从鼠标位置向外扩散（约 800ms），同时 cursor 处出现一个 80px 直径的线条头像小窗

## 文件结构

| 文件 | 说明 |
|---|---|
| `guangtou.html` | 唯一会编辑的产出文件，自包含（HTML + CSS + JS） |
| `源码.html` | 参考源码（小米 MiMo 100T Hero 复刻），不直接修改 |
| `CLAUDE.md` | 项目协作约定 + 术语对齐表 + 状态机 + GOTCHA |
| `新人手册.html` | 给新人看的开发总结报告（浏览器打开） |
| `README.md` | 你正在看的这份 |

两层头像图片都走 `cklaozhao.ccwu.cc` 图床，仓库里不留图。

## 本地预览

浏览器双击 `guangtou.html` 即可（无构建、无依赖）。

## 部署

整个项目就一个 HTML 入口 + 一份新人手册 HTML，扔到任意静态托管（GitHub Pages / Vercel / Cloudflare Pages 等）即可。当前部署在 <https://guangtou.me>。

## 协作

继续改之前先读 `CLAUDE.md` 和 `新人手册.html`：

- 大改动：先方案后实施
- 落定后：commit + push
- 分支策略：每个迭代一个分支（`logo-1` / `logo-2` / ...），完成后 fast-forward merge 回 main

## 致谢

- 视觉灵感 / 源码起点：[小米 MiMo 100T Hero](https://100t.xiaomimimo.com/)
- 协作工具：[Claude Code](https://claude.com/claude-code)
