# 光头 Obsidian 教程 落地页

## 项目概览

单文件 HTML 落地页，参照 `源码.html`（小米 MiMo 100T Hero 复刻版）改造而来，加入了头像 + 鼠标交互逻辑。整页是一个 hero section + 顶部头像 + 标题 + 底部备案号 footer，背景是 canvas 字符矩阵（glitch effect）。

## 文件清单

- `guangtou.html` —— 唯一会编辑的产出文件，自包含（HTML + CSS + JS）。约 22KB
- `源码.html` —— 参考源码（小米 MiMo 复刻），**不直接修改**，仅作为 cursor-ring / glitch canvas 行为的对照
- `新人手册.html` —— 开发总结报告（给新人看，浏览器打开）
- `README.md` —— 项目首页介绍
- 头像图床（[cklaozhao.ccwu.cc](https://cklaozhao.ccwu.cc) 自家 CDN）：
  - 顶层线条画：`https://cklaozhao.ccwu.cc/b761c6fb3efb55d296b37528570d2e34.jpg`，被 `.logo::after` 引用
  - 底层真人照：`https://cklaozhao.ccwu.cc/c86282f5209ee3162ec92fb3daa6f035.jpg`，被 `.logo` 引用

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
| 擦除 / 露底层环带 | `.logo::after` 的 mask 在 cursor 处 inner-r ~ outer-r 之间是 transparent，露出底层 |
| 顶层小窗 / 线条小窗 | `--inner-r`（mask 中 cursor 0~inner-r 是 #000，露出顶层线条；默认 0 = 无小窗，头像稳定态 40） |
| 底层环带外径 | `--outer-r`（mask 中 outer-r 处突变回 #000，圆外又是顶层；默认 1，hero 空白 100，头像稳定 maxR=对角线长） |
| 进入头像扩散 | `animateOuterR()` —— RAF 驱动 `--outer-r` 从 100 缓动到 maxR（800ms easeOutCubic）；同时 `--inner-r` 瞬切 0→40 |
| 光晕 | 已删除的 `.logo::before`（`mix-blend-mode: screen` 的高光层），不要重新加 |
| 硬边 | mask gradient 同位置两个 stop（如 `transparent var(--inner-r), #000 var(--inner-r)`）= 零渐变带 |
| 背景变化 | canvas 字符矩阵动画（`initGlitchCanvas` 的 `loop()`），始终运行，**不要再加暂停逻辑** |
| 备案区 | `.footer` 元素，`position: fixed; bottom: 0`，在 hero 之外 |

## 交互状态机

`updateInteractiveState`（`initCursor` 内）每次 mousemove 给 `.cursor-ring` 加状态类；`update`（`initLogoTilt` 内）联动写 `.logo` 的 `--reveal-x/y` + `--inner-r` + `--outer-r`，并通过 `animateOuterR` 驱动进入头像扩散过渡。

| 鼠标位置 | cursor-ring 类 / 尺寸 | --inner-r | --outer-r | 视觉 |
|---|---|---|---|---|
| 页面初始 / 鼠标在 hero 外 | 无 / 200px / inline `opacity: 0` 隐藏 | 0（CSS 默认） | 1（CSS 默认） | 顶层（线条头像）完整显示 |
| hero 空白处 | 无 / 200px / 大圆环加自身 mask 在头像位置挖洞 | 0 | 100（cursorRingRadius）| 顶层 + 鼠标处 100px 半径硬边圆形擦除露底层 |
| 进入头像（动画 800ms） | `.is-on-logo` / 40px / `visibility: hidden` | 40（瞬切） | 100 → maxR（RAF 缓动） | cursor 处 40px 顶层线条小窗即时出现 + 底层环带从 40~100 扩散到 40~maxR |
| 头像稳定 | `.is-on-logo` / 40px / `visibility: hidden` | 40 | maxR（= `Math.hypot(rect.w, rect.h)`） | cursor 处 40px 顶层小窗 + 整张其他底层；同时 `.logo` `scale(1.06)` 放大 |
| 鼠标在标题 / footer 链接 | `.is-hidden` / 40px / 无 mask | 0 | 1 | 顶层完整显示，圆环 40px 反色叠加在文字上（200→40 由 cursor-ring 主规则的 width transition 0.2s 平滑过渡） |

cursor-ring 的自身 mask（`--avatar-x/y/r`）在 `move()` 每帧更新，目的：当 200px 大圆环与头像重叠时，挖空头像部分 → 头像不被 difference 反色（仅在 hero 空白大圆环状态生效；`.is-hidden` / `.is-on-logo` 都通过 `mask-image: none` 关闭，且 `is-on-logo` 还加 `visibility: hidden` 整体不可见）。

## 关键 GOTCHA（踩过的坑）

1. **不要让 `--outer-r ≤ --inner-r`**——会导致 mask 中 inner / outer stop 重叠或反序，stop 排序混乱视觉退化。用 `1px` 作为"无擦除"安全值；进入头像扩散动画 outer-r 必须始终 ≥ inner-r（实际从 100 起步、目标 maxR ≈ 250+，inner-r=40，安全）
2. **直径 vs 半径**：`.cursor-ring` 的 CSS `width/height` 是**视觉直径**，但 mask radial-gradient 里 stop 位置是**半径**。让圆环和 mask 露出区视觉重合时 `width = 2 × outer-r`。早期曾因两边都设 50 出现"50px 圆环 + 100px mask 露出区"的双圈 BUG。注意头像上 cursor-ring 已 `visibility: hidden`，这个匹配只对 hero 空白模式生效
3. **不要给 mask-image 加 transition**：`--reveal-x/y` 每帧都变，加 transition 会让圆洞肉眼可见地滞后于鼠标。头像进入扩散用 RAF 驱动 `--outer-r` 数值，绕开了这个限制
4. **不要给 cursor-ring 加 mix-blend-mode 之外的渲染依赖**：source 的 `is-hidden` 原本写 `opacity: 0`，但 `hero.mouseenter` 里 inline `opacity: 1` 会覆盖；当前 `is-hidden` 已删除 `opacity: 0`，依赖 inline 控制；`is-on-logo` 改用 `visibility: hidden` 避开这个冲突
5. **不要恢复"背景暂停"功能**：之前的 `isFrozen` / `setSceneFrozen` / `.is-time-stop` 逻辑全部已清理，glitch loop 始终运行
6. **不要恢复 `.logo::before` 高光**：用户明确不要这个"光晕"
7. **`.logo-link` hit-test 是矩形不是圆**——`border-radius: 50%` 只影响视觉。判定"在头像上"如果用 `under.closest(".logo-link")` 是按矩形；如果想按视觉圆形，用 `Math.hypot(localX-cx, localY-cy) <= rect.width/2`。当前代码两种都用：`onLogo` 用矩形（决定遮罩切换），tilt/scale 用圆形（决定 3D 倾斜）
8. **`.is-inverted` 类与 `--reveal-size` 已淘汰**——历史方案下 `.logo.is-inverted::after` 是反向 mask，现在统一成单一参数化 mask（inner-r / outer-r），不再 toggle `.is-inverted`、不再写 `--reveal-size`；不要在 CSS 钩子或 JS 里再依赖这两个名字

## 关键 JS 函数（位于 `guangtou.html` 内 `<script>` 块）

- `typeTitle()` —— 标题打字机效果（带随机抖动）
- `initGlitchCanvas()` —— 背景字符矩阵；`mutate / draw / fadeColors` 在 `loop()` 里持续运行；不要再添加 `isFrozen` 之类的暂停状态
- `initCursor()` —— cursor-ring 跟随、状态切换、自身 mask 更新（在 `move()` 内每帧调 `link.getBoundingClientRect()` 写 `--avatar-x/y/r`）
- `initLogoTilt()` —— mask 状态机（`--inner-r` / `--outer-r`）+ 3D tilt + scale。`update()` 是状态机入口；`animateOuterR(target, ms, onDone)` 用 RAF 驱动 `--outer-r` 缓动（easeOutCubic），用于进入头像扩散过渡

## 验证

直接浏览器打开 `guangtou.html`：

1. 默认看到完整线条头像（顶层），背景字符滚动
2. 鼠标在 hero 空白处移动 → 200px 反色圆环跟随，碰到头像边缘时硬边露底层
3. 鼠标进头像 → 头像 `scale(1.06)` 放大；cursor 处**即时**出现 80px 直径顶层线条小窗 + 底层环带**同时**从 100r 扩散到对角线长（约 800ms）；扩散完成后整张除小窗外都是底层
4. 鼠标到标题"光头obsidian教程" → 圆环平滑缩到 40px、标题下方 `::after` 下划线 `scaleX(0)→1` 填充展开
5. 鼠标到底部备案号链接 → 系统 pointer 指针、圆环 40px、背景继续滚
6. 鼠标移出窗口 → 圆环消失、小窗收起、头像回顶层
