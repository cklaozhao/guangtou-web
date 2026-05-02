# 光头 Obsidian 教程 落地页

## 项目概览

单文件 HTML 落地页，参照 `源码.html`（小米 MiMo 100T Hero 复刻版）改造而来，加入了头像 + 鼠标交互逻辑。整页是一个 hero section + 顶部头像 + 标题 + 底部备案号 footer，背景是 canvas 字符矩阵（glitch effect）。

## 文件清单

- `guangtou.html` —— 唯一会编辑的产出文件，自包含（HTML + CSS + JS + 内嵌 base64 头像）。约 125KB，其中 ~109KB 是底层头像的 base64
- `头像.jpg` —— 原始底层头像（1528×1526 雪山真人照），已经压缩到 480px / quality 82 后内嵌进 `guangtou.html`，保留原图作备份
- `源码.html` —— 参考源码（小米 MiMo 复刻），**不直接修改**，仅作为 cursor-ring / glitch canvas 行为的对照
- 远端 URL `https://photograph-1307041810.cos.ap-nanjing.myqcloud.com/202509041415597.jpg` —— 顶层线条头像，被 `.logo::after` 引用

## 用户协作偏好

- 用中文沟通
- 喜欢简洁回复 + 表格归纳多状态
- 迭代式微调，常给"保持其他功能不变 + 改 X"的局部需求；预期快速一改一验证
- 对视觉精度敏感（硬边 vs 渐变、像素级对齐）
- 多次主动要求"对齐技术术语"——下面的术语表很重要
- 会反复改主意（同一行为前后要求过相反方向，比如"hover 整张露底层" → 改成"只露一小块"），不要假设之前的设计意图永远成立
- 重要改动前应先讲方案再动手；细微微调（改个数值/颜色）直接改

## 中英术语对齐

| 用户说 | 代码里对应 |
|---|---|
| 顶层头像 / 线条头像 | `.logo::after` 的 `background-image`（远端线条 URL） |
| 底层头像 / 真人照 | `.logo` 的 `background-image`（内嵌 base64 的 `头像.jpg`） |
| 圆环 / 光标圆环 / 鼠标光圈 | `.cursor-ring` 元素 |
| 颜色反转 / 反色 | `mix-blend-mode: difference` |
| 遮罩 | CSS `mask-image` / `-webkit-mask-image` |
| 擦除 / 橡皮擦效果 | `.logo::after` 的 radial-gradient mask 在鼠标处的透明洞 |
| 反向遮罩 / 反转的遮罩 | `.logo.is-inverted::after` —— 中心 `#000`（露顶层）、外部 `transparent`（露底层），方向与默认相反 |
| 光晕 | 已删除的 `.logo::before`（`mix-blend-mode: screen` 的高光层），不要重新加 |
| 硬边 | mask gradient 的 stop 设成 `transparent 100%, #000 100%`（同位置两个 stop = 零渐变带） |
| 背景变化 | canvas 字符矩阵动画（`initGlitchCanvas` 的 `loop()`），始终运行，**不要再加暂停逻辑** |
| 备案区 | `.footer` 元素，`position: fixed; bottom: 0`，在 hero 之外 |

## 交互状态机

`updateInteractiveState`（`initCursor` 内）每次 mousemove 给 `.cursor-ring` 加状态类；`update`（`initLogoTilt` 内）联动给 `.logo` 切 `.is-inverted` 并写 `--reveal-x/y/size`。

| 鼠标位置 | cursor-ring 类 / 尺寸 | logo 类 | `--reveal-size` | 视觉 |
|---|---|---|---|---|
| 页面初始 / 鼠标在 hero 外 | 无 / 200px / inline `opacity: 0` 隐藏 | 无 | 1px (CSS 默认) | 顶层（线条头像）完整显示 |
| hero 空白处 | 无 / 200px / 大圆环加自身 mask 在头像位置挖洞 | 无 | 100 (cursorRingRadius) | 顶层 + 鼠标处 100px 半径硬边圆形擦除露底层 |
| 鼠标进头像 | `.is-on-logo` / 40px / 无 mask | `.is-inverted` | 40 | 头像几乎全是底层，鼠标处 80px 直径反向露顶层；同时 `.logo` `scale(1.06)` 放大 |
| 鼠标在标题 / footer 链接 | `.is-hidden` / 16px / 无 mask | 无 | 1px | 顶层完整显示，圆环 16px 反色叠加在文字上 |

cursor-ring 的自身 mask（`--avatar-x/y/r`）在 `move()` 每帧更新，目的：当 200px 大圆环与头像重叠时，挖空头像部分 → 头像不被 difference 反色（仅在大圆环状态生效；`.is-hidden` / `.is-on-logo` 都通过 `mask-image: none` 关闭）。

## 关键 GOTCHA（踩过的坑）

1. **不要用 `--reveal-size: 0px`**——`radial-gradient(circle 0 …)` 在 Chromium / WebKit 都是退化值，会回退到 `farthest-corner` 默认尺寸 → mask 整张透明 → 默认就漏底层。永远用 `1px` 作为"无擦除"的安全值
2. **直径 vs 半径**：`.cursor-ring` 的 CSS `width/height` 是**视觉直径**，但 `mask-image: radial-gradient(circle <size> …)` 里的 `<size>` 是**半径**。让圆环和 mask 露出区视觉重合时记得 `width = 2 × reveal-size`。曾经因为两边都设成 50 出现"50px 圆环 + 100px mask 露出区"的双圈 BUG
3. **不要给 mask-image 加 transition**：`--reveal-x/y` 每帧都变，加 transition 会让圆洞肉眼可见地滞后于鼠标
4. **不要给 cursor-ring 加 mix-blend-mode 之外的渲染依赖**：source 的 `is-hidden` 原本写 `opacity: 0`，但 `hero.mouseenter` 里 inline `opacity: 1` 会覆盖；当前 `is-hidden` 已删除 `opacity: 0`，依赖 inline 控制
5. **不要恢复"背景暂停"功能**：之前的 `isFrozen` / `setSceneFrozen` / `.is-time-stop` 逻辑全部已清理，glitch loop 始终运行
6. **不要恢复 `.logo::before` 高光**：用户明确不要这个"光晕"
7. **`.logo-link` hit-test 是矩形不是圆**——`border-radius: 50%` 只影响视觉。判定"在头像上"如果用 `under.closest(".logo-link")` 是按矩形；如果想按视觉圆形，用 `Math.hypot(localX-cx, localY-cy) <= rect.width/2`。当前代码两种都用：`onLogo` 用矩形（决定遮罩切换），tilt/scale 用圆形（决定 3D 倾斜）

## 关键 JS 函数（位于 `guangtou.html` 内 `<script>` 块）

- `typeTitle()` —— 标题打字机效果（带随机抖动）
- `initGlitchCanvas()` —— 背景字符矩阵；`mutate / draw / fadeColors` 在 `loop()` 里持续运行；不要再添加 `isFrozen` 之类的暂停状态
- `initCursor()` —— cursor-ring 跟随、状态切换、自身 mask 更新（在 `move()` 内每帧调 `link.getBoundingClientRect()` 写 `--avatar-x/y/r`）
- `initLogoTilt()` —— 名字其实管两件事：mask reveal 状态切换 + 3D tilt + scale。`update()` 是核心

## 验证

直接浏览器打开 `guangtou.html`：

1. 默认看到完整线条头像（顶层），背景字符滚动
2. 鼠标在 hero 空白处移动 → 200px 反色圆环跟随，碰到头像边缘时硬边露底层
3. 鼠标进头像 → 圆环缩 40px、头像 scale 放大、头像变底层、鼠标处反向露顶层
4. 鼠标到标题"光头obsidian教程" → 圆环 16px、标题下方 `::after` 下划线 `scaleX(0)→1` 填充展开
5. 鼠标到底部备案号链接 → 系统 pointer 指针、圆环 16px、背景继续滚
6. 鼠标移出窗口 → 圆环消失、头像回顶层
