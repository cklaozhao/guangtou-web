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
| 顶层小窗 / 线条小窗 | `--inner-r`（mask 中 cursor 0~inner-r 是 #000，露出顶层线条；默认 0 = 无小窗，头像稳定态 50 = 直径 100px，与字上 cursor-ring 一致） |
| 底层环带外径 | `--outer-r`（mask 中 outer-r 处突变回 #000，圆外又是顶层；默认 1，hero 空白 100，头像稳定 maxR=对角线长） |
| 头像进入 / 离开过渡 | `animateMask(targetInner, targetOuter, 800ms)` —— RAF 同时驱动 `--inner-r` 和 `--outer-r` 从当前值缓动到目标值（easeOutCubic）。**v2.0 起 `hero_blank` 也带小圆**，所以进入/离开头像时 inner-r 已经是 50，实际只 outer-r 在缓动。进入：(50,100) → (50, maxR)；离开（落 hero 空白）：(50, maxR) → (50, 100)；离开（落字上/出 hero）：(50, maxR) → (0, 1)。中途变向不停止，直接从当前位置反向追新目标 |
| 光晕 | 已删除的 `.logo::before`（`mix-blend-mode: screen` 的高光层），不要重新加 |
| 硬边 | mask gradient 同位置两个 stop（如 `transparent var(--inner-r), #000 var(--inner-r)`）= 零渐变带 |
| 背景变化 | canvas 字符矩阵动画（`initGlitchCanvas` 的 `loop()`），始终运行，**不要再加暂停逻辑** |
| 备案区 | `.footer` 元素，`position: fixed; bottom: 0`，在 hero 之外 |

## 交互状态机

`updateInteractiveState`（`initCursor` 内）每次 mousemove 给 `.cursor-ring` 加状态类；`update`（`initLogoTilt` 内）按当前鼠标命中区域算出 `targetState`（`on_logo` / `hero_blank` / `outside`），仅当状态变化时执行：进入或离开 `on_logo` 走 `animateMask` 800ms 缓动；`hero_blank` ↔ `outside` 之间瞬切。`--reveal-x/y` 每帧跟手。

| 鼠标位置 | cursor-ring 类 / 尺寸 | --inner-r | --outer-r | 视觉 |
|---|---|---|---|---|
| 页面初始 / 鼠标在 hero 外 | 无 / 200px / inline `opacity: 0` 隐藏 | 0（CSS 默认） | 1（CSS 默认） | 顶层（线条头像）完整显示 |
| hero 空白处（v2.0 起带嵌套小圆）| 无 / 200px / 大圆环加自身 mask 在头像位置挖洞 | **50** | 100（cursorRingRadius）| cursor 处一个直径 100px 顶层小圆 + 50~100r 底层环带 + 100+ 顶层。远离头像时 mask 在视野外，看不到嵌套结构；接近头像边缘时同时露顶层小圆与底层环带 |
| 进入头像（动画 800ms） | `.is-on-logo` / 100px / 显示（difference 反色） | 50（保持） | 100 → maxR（缓动） | 小圆已经在 hero 空白时存在，进入头像时 inner-r 不变，只 outer-r 扩散到对角线长 |
| 头像稳定 | `.is-on-logo` / 100px / 显示（difference 反色） | 50 | maxR（= `Math.hypot(rect.w, rect.h)`） | cursor 处看到 100px 反色小圆叠加在线条小窗上 = "反色的线条头像小圆"；外圈整张其他底层；`.logo` `scale(1.06)` 放大 |
| 离开头像（动画 800ms） | 取决于落点 | 50（保持，落字上/出 hero 时 → 0 缓动） | maxR → 100（落 hero 空白）或 → 1（落字上/出 hero）| 落 hero 空白：小圆保持，底层环带从 maxR 收回到 100r 跟手圆。落字上/出 hero：小圆和环带都缩到 0/1 |
| 鼠标在标题 / footer 链接 | `.is-hidden` / 100px / 无 mask | 0 | 1 | 顶层完整显示，圆环 100px 反色叠加在文字上（200→100 由 cursor-ring 主规则的 width transition 0.2s 平滑过渡） |

cursor-ring 的自身 mask（`--avatar-x/y/r`）在 `move()` 每帧更新，目的：当 200px 大圆环与头像重叠时，挖空头像部分 → 头像不被 difference 反色（仅在 hero 空白大圆环状态生效；`.is-hidden` / `.is-on-logo` 都通过 `mask-image: none` 关闭，让 100px 圆完整显示在头像 / 文字上参与 difference 反色叠加）。

## 关键 GOTCHA（踩过的坑）

1. **不要让 `--outer-r ≤ --inner-r`**——会导致 mask 中 inner / outer stop 重叠或反序，stop 排序混乱视觉退化。用 `1px` 作为"无擦除"安全值；进入头像扩散动画 outer-r 必须始终 ≥ inner-r（实际从 100 起步、目标 maxR ≈ 250+，inner-r=50，安全）
2. **直径 vs 半径**：`.cursor-ring` 的 CSS `width/height` 是**视觉直径**，但 mask radial-gradient 里 stop 位置是**半径**。让圆环和 mask 露出区视觉重合时 `width = 2 × outer-r`。早期曾因两边都设 50 出现"50px 圆环 + 100px mask 露出区"的双圈 BUG。v2.1 起头像内 `.is-on-logo` 也是 100px（与 inner-r=50 直径重合，刻意完美叠加做反色效果）
3. **不要给 mask-image 加 transition**：`--reveal-x/y` 每帧都变，加 transition 会让圆洞肉眼可见地滞后于鼠标。头像进入扩散用 RAF 驱动 `--outer-r` 数值，绕开了这个限制
4. **不要给 cursor-ring 加 mix-blend-mode 之外的渲染依赖**：source 的 `is-hidden` 原本写 `opacity: 0`，但 `hero.mouseenter` 里 inline `opacity: 1` 会覆盖；当前 `is-hidden` 已删除 `opacity: 0`，依赖 inline 控制（v2.1 后 `.is-on-logo` 也不再用 `visibility: hidden`，让 difference 反色叠加在线条小窗上）
5. **不要恢复"背景暂停"功能**：之前的 `isFrozen` / `setSceneFrozen` / `.is-time-stop` 逻辑全部已清理，glitch loop 始终运行
6. **不要恢复 `.logo::before` 高光**：用户明确不要这个"光晕"
7. **`.logo-link` hit-test 是矩形不是圆**——`border-radius: 50%` 只影响视觉。判定"在头像上"如果用 `under.closest(".logo-link")` 是按矩形；如果想按视觉圆形，用 `Math.hypot(localX-cx, localY-cy) <= rect.width/2`。当前代码两种都用：`onLogo` 用矩形（决定遮罩切换），tilt/scale 用圆形（决定 3D 倾斜）
8. **`.is-inverted` 类与 `--reveal-size` 已淘汰**——历史方案下 `.logo.is-inverted::after` 是反向 mask，现在统一成单一参数化 mask（inner-r / outer-r），不再 toggle `.is-inverted`、不再写 `--reveal-size`；不要在 CSS 钩子或 JS 里再依赖这两个名字

## 关键 JS 函数（位于 `guangtou.html` 内 `<script>` 块）

- `typeTitle()` —— 标题打字机效果（带随机抖动）
- `initGlitchCanvas()` —— 背景字符矩阵；`mutate / draw / fadeColors` 在 `loop()` 里持续运行；不要再添加 `isFrozen` 之类的暂停状态
- `initCursor()` —— cursor-ring 跟随、状态切换、自身 mask 更新（在 `move()` 内每帧调 `link.getBoundingClientRect()` 写 `--avatar-x/y/r`）
- `initLogoTilt()` —— mask 状态机（`--inner-r` / `--outer-r`）+ 3D tilt + scale。`update()` 是状态机入口（计算 `targetState`，仅状态变化时切换）；`animateMask(targetInner, targetOuter, ms)` 用 RAF 同时驱动两个变量缓动（easeOutCubic），覆盖进入和离开头像两种过渡；中途变向时取消旧动画从当前值反向追新目标

## 验证

直接浏览器打开 `guangtou.html`：

1. 默认看到完整线条头像（顶层），背景字符滚动
2. 鼠标在 hero 空白处移动 → 200px 反色圆环跟随；接近头像边缘时同时呈现"直径 100px 顶层小圆 + 50~100r 底层环带"（v2.0 嵌套小圆视觉）
3. 鼠标进头像 → 头像 `scale(1.06)` 放大；小圆**已经在那不再扩散**，只有底层环带从 100r 缓动扩散到对角线长（约 800ms easeOutCubic）；最终整张除小圆外都是底层
4. 鼠标移出头像（落 hero 空白）→ 小圆保持直径 100px、底层环带从对角线长缩回到 100r（约 800ms 反向缓动）。落字上 / 出 hero 时小圆和环带都缩到 0/1（瞬切或缓动）
5. 中途快进快出 → 当前动画立刻被取消，从当前值反向追新目标，整体丝滑无闪
6. 鼠标到标题"知识管理系统" → 圆环平滑缩到 100px、标题下方 `::after` 下划线 `scaleX(0)→1` 填充展开
7. 鼠标到底部备案号链接 → 系统 pointer 指针、圆环 100px、背景继续滚
8. 鼠标移出窗口 → 圆环消失、小窗瞬切、头像回顶层（这种情况鼠标已经离开 viewport，过渡观察不到，所以做瞬切）
