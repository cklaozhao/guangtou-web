# 光头 Obsidian 教程落地页 · 主题库

> 单文件 HTML 门户主题库，作为「光头 Obsidian 教程」（飞书 wiki）的入口页。
> 在线：<https://guangtou.me>

## 项目定位

一个**单文件 HTML 静态门户主题库**——主项目骨架已成型（hero + canvas glitch 背景 + 头像交互 + 跳转链接 + 备案区），后续以"多主题"形式持续扩展。

每见到一个想复刻的爆款 UI（来自小红书 / YouTube / X 等），就用 [Claude Code](https://claude.com/claude-code) 把它复刻成本项目的一个新主题，作为独立 HTML 文件沉淀进项目。使用者按喜好挑选，丢进自己的博客即可部署。

适合人群：知识博主 / 自由职业者 / 个人品牌开发者——不想为 landing page 单独购买 SaaS、也不想折腾域名的人。

## 文件结构

每一个主题独立放在自己的文件夹里，文件夹内包含**复刻源码 + 成品主题 + 配置速查**三件套：

```
.
├── README.md            # 你正在看的这份
├── CLAUDE.md            # 项目协作约定 + 术语对齐表 + GOTCHA
├── LICENSE
├── 新人手册.html         # 给新人看的开发总结报告（浏览器打开）
└── xiaomi-MIMO/         # 主题 1：小米 MiMo 风
    ├── xiaomi-MIMO.html              # 复刻源码（参考起点，不直接修改）
    ├── guangtou-xiaomi-MIMO.html     # 成品主题（实际部署用）
    └── 配置速查.md                    # 该主题的关键参数 / 调试速查
```

**未来新增主题的命名约定：**

```
<主题名>/
├── <主题名>.html              # 源码起点（爬下来的参考站）
├── guangtou-<主题名>.html     # 集成进光头门户后的成品
└── 配置速查.md                # 该主题的参数 / 交互 / GOTCHA
```

## 视觉特色（以小米 MiMo 主题为例）

- **背景**：canvas 字符矩阵滚动（"glitch" 风格）
- **头像**：双层结构——顶层线条画，底层真人照，鼠标进出时切换
- **cursor-ring**：自定义鼠标光圈，跟随 + `mix-blend-mode: difference`（颜色反色）
- **标题**：打字机动画 + hover 下划线展开
- **进入头像时**：底层从鼠标位置向外扩散（约 800ms），中央保留一个反色线条画小窗

不同主题会有完全不同的视觉语言——以上仅适用于 `xiaomi-MIMO` 主题。

## 本地预览

浏览器双击对应主题文件夹内的成品文件即可（无构建、无依赖）。例如：

```
xiaomi-MIMO/guangtou-xiaomi-MIMO.html
```

## 部署

整个主题库的每一个成品 HTML 都是自包含的，扔到任意静态托管（GitHub Pages / Vercel / Cloudflare Pages 等）即可。当前 <https://guangtou.me> 部署的是 `xiaomi-MIMO` 主题。

要换主题，把博客上的入口文件替换成另一个主题的成品 HTML 即可，零迁移成本。

## 协作

继续改之前先读 `CLAUDE.md` 和 `新人手册.html`：

- 大改动：先方案后实施
- 落定后：commit + push
- 分支策略：每个迭代一个分支（`logo-1` / `logo-2` / ...），完成后 fast-forward merge 回 main

## 致谢

- 首个主题视觉灵感 / 源码起点：[小米 MiMo 100T Hero](https://100t.xiaomimimo.com/)
- 协作工具：[Claude Code](https://claude.com/claude-code)
