# DESIGN.md — 铸辞 · PROMPT FOUNDRY（星图版）

> 纯黑的场里，白色圆缓慢漂移；圆里只有一句标题。悬停，世界停一下，圆长大。

**产品**：个人提示词库（SFT / RAG / 低代码 / 绘图）
**形态**：单文件 `index.html`，左侧字盘 + 全屏星图 + 从圆展开的白色方形详情框
**技术**：原生 CSS + JS，rAF 物理，零第三方依赖
**数据**：34 条示例 + localStorage + Supabase 云端

---

## 1. Visual Theme & Atmosphere

**Style**: Monochrome Star Map — 黑白星图
**Keywords**: 黑场、白圆、漂浮、失重、留白、单色、物理、安静
**Tone**: 极简、克制、略带实验性 — NOT 彩色、NOT 霓虹、NOT 玻璃拟态、NOT 卡片网格
**Feel**: 像在暗房里看一群被打了光的圆形铅字，慢慢绕着彼此转；靠近时它们让开，停住时像被按在玻璃上。

**Interaction Tier**: L2（自绘物理动效，无 GSAP）
**Dependencies**: 无；`requestAnimationFrame` + CSS transform

**三条设计原则**
1. **只有黑白**：整页不存在色相，层级只用明度、字重、1px 线。
2. **圆是主角**：界面越薄越好，圆之外不出现装饰性图形。
3. **动要慢**：漂移 4–10px/s，放大 0.55s；任何动效都不能抢阅读。

---

## 2. Color Palette & Roles

```css
:root {
  /* Backgrounds */
  --bg: #000000;               /* 页面 / 星图底色 */
  --surface: #0a0a0a;          /* 面板、浮层 */
  --surface-2: #111111;        /* 输入框、次级面 */
  --surface-3: #1a1a1a;        /* hover 面 */
  --sidebar: #000000;          /* 左侧字盘，与背景同黑，用线分隔 */

  /* Borders */
  --border: #222222;
  --border-strong: #3a3a3a;
  --border-active: rgba(255, 255, 255, .36);

  /* Text */
  --text: #ffffff;
  --text-secondary: #c7c7c7;
  --text-muted: #9a9a9a;
  --text-faint: #737373;
  --text-inverse: #000000;

  /* Accent = 白 */
  --accent: #ffffff;
  --accent-hover: #e8e8e8;
  --accent-press: #d2d2d2;
  --accent-soft: rgba(255, 255, 255, .10);
  --accent-glow: rgba(255, 255, 255, .18);

  /* 圆形 */
  --bubble: #ffffff;
  --bubble-ink: #000000;

  /* 分类标识：仅用灰阶，不用色相 */
  --cat-sft: #ffffff;
  --cat-rag: #d4d4d4;
  --cat-lowcode: #a8a8a8;
  --cat-image: #7a7a7a;

  /* RGB */
  --bg-rgb: 0, 0, 0;
  --accent-rgb: 255, 255, 255;
  --text-rgb: 255, 255, 255;
  --cat-sft-rgb: 255, 255, 255;
  --cat-rag-rgb: 212, 212, 212;
  --cat-lowcode-rgb: 168, 168, 168;
  --cat-image-rgb: 122, 122, 122;
}
```

**Color Rules:**
- **全站零色相**：任何 `rgb()` 的三个通道必须相等（灰阶）；不允许出现品牌色、分类色、语义红绿。错误态用反相（白底黑字）而不是红色。
- 半透明一律基于 `rgba(255,255,255,a)` 或 `rgba(0,0,0,a)`，不叠色。
- 文字对比度：白/黑 21:1；`--text-secondary` ≈ 11:1；`--text-muted` ≈ 7:1；`--text-faint` ≥ 4.5:1。
- 白色圆是唯一大面积亮面；一个视口里除圆与主按钮外不出现第二块纯白。
- 禁止渐变、彩色阴影、彩色描边、彩色图标。

---

## 3. Typography Rules

**Font Stack**
```css
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600&family=IBM+Plex+Sans:wght@400;500;600&family=Noto+Sans+SC:wght@400;500;700&display=swap');

--font-sans: 'IBM Plex Sans', 'Noto Sans SC', -apple-system, 'PingFang SC', 'Microsoft YaHei', sans-serif;
--font-mono: 'IBM Plex Mono', 'Noto Sans SC', 'PingFang SC', ui-monospace, 'SFMono-Regular', monospace;
```

| Role | Font | Size | Weight | Line Height | Tracking |
|------|------|------|--------|-------------|----------|
| 圆内标题 | Sans | clamp(10.5px, d×0.135, 14px) | 600 | 1.32 | .02em |
| 方框标题 | Sans | 22px（移动 19px） | 600 | 1.35 | .01em |
| 正文（方框） | Sans | 13px | 400 | 1.75 | .02em |
| 提示词正文 | Mono | 13.5px | 400 | 1.8 | .01em |
| UI 控件 | Sans | 13–13.5px | 500 | 1.2 | .02em |
| 元信息 / 计数 | Mono | 10.5–12px | 500 | 1.4 | .04em |
| 圆印「铸」 | Sans | 14px | 600 | 1 | 0 |

**Typography Rules:**
- 圆内标题最多 3 行，超出裁切；标题越长，圆径略大（见 §5），不要靠缩小字号硬塞。
- 中文行高 ≥ 1.7（正文），字距 .02em；中文字族在前、拉丁回退。
- 数字用 tabular-nums 的 Mono，计数变化不抖动。
- **NEVER use**：Inter / Roboto / Arial 作为首选；装饰字体；彩色渐变文字；描边字；全大写英文标签。

**Text Decoration**
- 没有任何渐变、投影、描边；标题只靠黑白对比。
- 圆悬停放大是唯一的“文字特效”。
- 焦点环 2px 纯白，offset 6px（圆）/2px（控件）。

---

## 4. Component Stylings

### 4.1 提示词圆（核心）

```css
.bubble {
  position: absolute; top: 0; left: 0;
  width: var(--d); height: var(--d);
  border: 0; border-radius: 50%;
  background: transparent;
  cursor: pointer;
  z-index: 1;
  user-select: none;
}
.bubble__disc {
  display: grid; place-items: center;
  width: 100%; height: 100%;
  padding: 15%;
  border-radius: 50%;
  background: var(--bubble);
  color: var(--bubble-ink);
  transform: scale(1);
  transition: transform .55s var(--ease-out), box-shadow .55s var(--ease-out);
  box-shadow:
    0 0 0 1px rgba(255, 255, 255, .07),
    0 18px 50px rgba(255, 255, 255, .05);
}
.bubble__title {
  font-size: clamp(10.5px, calc(var(--d) * 0.135), 14px);
  font-weight: 600; line-height: 1.32; letter-spacing: .02em;
  text-align: center; color: var(--bubble-ink);
  display: -webkit-box; -webkit-line-clamp: 3;
  -webkit-box-orient: vertical; overflow: hidden;
  overflow-wrap: anywhere;
}
.bubble:hover, .bubble:focus-visible { z-index: 30; }
.bubble:hover .bubble__disc,
.bubble:focus-visible .bubble__disc {
  transform: scale(1.5);
  box-shadow:
    0 0 0 1px rgba(255, 255, 255, .16),
    0 30px 90px rgba(255, 255, 255, .12);
}
.bubble:focus-visible { outline: 2px solid #fff; outline-offset: 6px; }
```

**状态**：default 静止/漂移；hover 冻结 + 1.5×（0.55s）；focus-visible 同 hover + 白环；active 无额外缩放；disabled 不适用。
**触摸**：`@media (hover: none)` 关闭 hover 放大，点击直接开详情。

### 4.2 顶部条与搜索

```css
.topbar {
  grid-column: 1 / -1; grid-row: 1;
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 560px) minmax(0, 1fr);
  align-items: center; gap: 12px;
  height: 60px; padding: 0 28px;
  background: #000; border-bottom: 1px solid var(--border);
}
.search { width: 100%; height: 36px; border: 1px solid var(--border); border-radius: 4px; background: var(--surface-2); }
.search:focus-within { border-color: var(--border-active); box-shadow: 0 0 0 3px var(--accent-soft); }
.refresh-btn { flex: none; width: 36px; height: 36px; padding: 0; color: #fff; border: 0; background: transparent; }
.topbar__count { font-family: var(--font-mono); font-size: 12px; color: var(--text-muted); }
```

- 顶栏横跨整页，**搜索在页面正中**（左右各 1fr，中心列 560px）；刷新按钮紧贴搜索右侧，结果数/标签 chip/新建在右端。
- 刷新按钮：**只有两个白色循环箭头**，无边框/底色；hover 降到 62% 不透明度；点击 `360°` 0.55s。
- 顶栏只放：菜单（移动）、搜索、刷新、`20 / 34 条`、标签 chip、登录/账号 chip、描边「+ 新建」。
- 当前视图名不在顶栏重复；由侧栏选中态承担。

### 4.3 左侧字盘 / 收起态

```css
.sidebar { grid-column: 1; grid-row: 2; width: 200px; height: 100%; background: #000; border-right: 1px solid var(--border); }
.nav-item { height: 36px; border-radius: 3px; color: var(--text-secondary); }
.nav-item[aria-current="true"] { background: var(--surface-2); color: #fff; }
.nav-item[aria-current="true"]::before { content: ""; position: absolute; left: -1px; top: 8px; bottom: 8px; width: 2px; background: #fff; }
```

- 品牌：30px 描边方印「铸」（纯品牌，不可点）+ 右侧「‹」收起；下方一条全宽分隔线。
- 导航只放文字：全部提示词 / SFT / RAG / 低代码 / 绘图 /（分隔线）收藏；**没有分类点、数量、新建按钮、存量条、页脚**。
- 选中项只保留 2px 白色左条 + 白字，不加背景块。
- 「+ 新建」在顶栏右端，描边按钮。
- 收起：字盘右上角「‹」按钮；整栏 `translateX(-100%)` 滑出，主区铺满，**不留窄轨**。
- 展开：左边缘垂直居中的白色椭圆「铸」（52×82，黑字），点击恢复侧栏；收起/展开状态存 `zhuci.rail.v1`。

```css
.rail-fab {
  position: fixed; left: 10px; top: 50%;
  width: 52px; height: 82px; border-radius: 999px;
  background: #fff; color: #000;
  opacity: 0; visibility: hidden;
  transform: translate(-14px, -50%);
  transition: opacity .25s ease, transform .3s var(--ease-out), visibility .25s;
}
body.rail-collapsed .rail-fab { opacity: 1; visibility: visible; transform: translate(0, -50%); }
@media (min-width: 1024px) {
  .shell.is-rail { grid-template-columns: 0 minmax(0, 1fr); }
  .shell.is-rail .sidebar { transform: translateX(-100%); opacity: 0; pointer-events: none; }
}
```
- 分类点用灰阶（白 → 深灰），不使用色相。

### 4.4 按钮 / Chip

```css
.btn { height: 34px; border: 1px solid var(--border); border-radius: 4px; background: var(--surface-2); color: var(--text); }
.btn--primary { background: #fff; border-color: #fff; color: #000; font-weight: 600; }
.btn--primary:hover { background: #e8e8e8; }
.btn--danger { background: #fff; border-color: #fff; color: #000; }
.filter-chip { border: 1px solid var(--border-active); background: var(--accent-soft); color: #fff; }
```

### 4.5 白色详情框（点击圆）

点击圆 → **不从右侧弹抽屉**，而是从这枚圆的视觉矩形出发，在 **1s** 内扩成居中正方形；圆角 `50% → 4px`，白底黑字。

```css
.prompt-box {
  position: fixed; z-index: 75;
  display: flex; flex-direction: column;
  background: #fff; color: #000;
  border-radius: 4px; overflow: hidden;
  box-shadow: 0 40px 140px rgba(0, 0, 0, .65);
  transition: top 1s var(--ease-out), left 1s var(--ease-out),
              width 1s var(--ease-out), height 1s var(--ease-out),
              border-radius 1s var(--ease-out);
}
.prompt-box__inner { opacity: 0; transition: opacity .3s ease .45s; }
.prompt-box.is-expanded .prompt-box__inner { opacity: 1; transition-delay: .55s; }
.pb-btn { background: #000; color: #fff; border: 1px solid #000; }
.pb-icon { background: #fff; color: #000; border: 1px solid #cfcfcf; }
```

- 尺寸见 §5 的 `side` 公式：桌面约 58vw × 72vh，移动端约 88vw，上限 720px，四边至少留 12px；居中。
- 内容：分类、标题、更新/字符/变量、摘要、正文（Mono 12.5px）、标签、来源；正文在框内滚动。
- 底部：复制全文（黑底白字）、收藏、编辑、删除（白底黑线）。删除二次确认。
- 关闭：0.5s 缩回原圆，恢复星图漂移；期间星图暂停。
- 遮罩：`rgba(0,0,0,.58)`，无模糊；白色方框是唯一亮面。

### 4.5b 编辑抽屉（新建 / 编辑 / 登录 / 账号 / 审核）

- 宽 `min(560px, 100vw)`，背景 `--surface`，左侧 1px 线。
- 查看详情不走这里；新建、编辑、登录、账号、待审核列表共用这个抽屉。
- 新建/编辑底部：取消 / 保存；保存时选「只存本机 / 提交云端」。
- 登录/注册底部：注册 / 登录；注册后需点邮件确认链接，回跳地址 = `location.origin + pathname`。

### 4.5c 站内账号与云端（Supabase）

- 顶栏右侧 `登录` chip：未登录显示「登录」，已登录显示账号名；点击打开抽屉。
- 注册/登录只用**账号 + 密码**（账号 2–20 个字符，可用中文），不出现邮箱输入；旧邮箱账号从抽屉底部「已有邮箱账号？」进入。
- 账号经 SHA-256 映射为内部标识 `u_<hash>@zhuci.local` 写入 Supabase Auth；密码由 Supabase 哈希保存，用户名存 `user_metadata.username`。用户不接触邮箱，也不发确认邮件。
- 注册与登录共用抽屉；`mailer_autoconfirm` 必须为 `true`，否则注册前直接提示管理员关闭 Confirm email，**不创建未确认账号**。注册成功后 `signUp` 返回 session，直接进入登录态。
- 浏览、搜索、复制、收藏不要求登录；**打开新建抽屉前要求登录**，未登录先进入登录界面。
- 云端条目状态：`pending → approved`；`profiles.role = 'admin'` 的账号看到「审核」，通过后进入公共库；本机条目不上传。
- 前端只使用 anon key（可公开），读写权限交给 Supabase RLS；页面通过 jsDelivr 加载 `supabase-js`，首次需要联网。管理员角色必须写在 `profiles.role`，前端只做显示，最终由 RLS 判定。
- 数据双轨：`localStorage` 保存本机库/收藏/侧栏状态；Supabase 保存账号、云端提示词和审核状态。

### 4.6 空状态 / 启动兜底

- 空状态居中在星图上方，白字 + 两个按钮（清除筛选 / 新建）。
- 脚本被拦截时显示虚线框说明；正常初始化后移除。

---

## 5. Layout Principles

**Shell**
```css
.shell {
  display: grid;
  grid-template-columns: 200px minmax(0, 1fr);
  grid-template-rows: 60px minmax(0, 1fr);
  height: 100dvh; overflow: hidden;
}
.topbar  { grid-column: 1 / -1; grid-row: 1; }
.sidebar { grid-column: 1; grid-row: 2; height: 100%; }
.main    { grid-column: 2; grid-row: 2; height: 100%; }
.stage   { position: relative; flex: 1; overflow: hidden; background: #000; }
```

- 顶栏横跨整页，占第一行 60px；侧栏与星图在第二行。
- 侧栏收起：`grid-template-columns: 0 1fr`，侧栏 `translateX(-100%)`，过渡 300ms；顶栏与搜索位置不变，星图随宽度重排。

**分批（20 枚）**
- 一次只渲染 20 枚圆；顶部显示 `20 / 34 条`，少于 20 时显示 `9 条`。
- 刷新：`batchOffset = (batchOffset + 20) % total`，清空旧圆、按新批重新 best-candidate 排布；新圆 0.5s 淡入。
- 搜索 / 标签 / 分类变化时 `batchOffset` 归零，回到该筛选下的第一批。
- 刷新是轮换而不是随机：34 条时每批 20 枚，相邻两批重叠 6 枚，其余 14 枚是新出现的。

**圆径与密度**
```js
const area = stageW * stageH * 0.28;          // 目标覆盖率 28%
let d = 2 * Math.sqrt(area / (n * Math.PI)); // n = 总条数
d = clamp(d, stageW < 600 ? 70 : 82, 128);
d *= 0.92 + Math.min(16, title.length) * 0.01; // 标题越长略大
```

**详情框尺寸**
```js
const w = innerWidth, h = innerHeight, mobile = w < 768;
const side = Math.min(
  Math.max(300, w * (mobile ? 0.88 : 0.58)),
  h * (mobile ? 0.9 : 0.72),
  720, w - 24, h - 24
);
```

**位置**
- 初值：best-candidate 采样 40 次，取“离最近圆最远”的点；间隔 ≥ 24px。
- 边界：圆心距边 ≥ `1.5r + 4`，保证悬停放大 1.5× 不被裁切。
- 碰撞：圆-圆最小间距 `r1 + r2 + 8`；位置分离 + 沿法线交换速度。

**间距**：4 / 8 / 12 / 16 / 24 / 32 / 48；侧栏内 14px，顶栏左右 28px。

---

## 6. Depth & Elevation

| Level | Treatment | Use |
|-------|-----------|-----|
| Flat | 无阴影，1px 灰线 | 侧栏、顶栏、圆默认 |
| Halo | 白色 1px 内描边 + 极轻白色外发光 | 圆悬停 |
| Overlay | 黑遮罩 + blur 6px | 编辑抽屉 |
| Scrim | `rgba(0,0,0,.58)`，无模糊 | 白色详情框 |
| Float | 黑色大投影 | 详情框、编辑抽屉、Toast |
| Invert | 白底黑字 | 详情框、主按钮、错误 Toast |

- 禁止渐变、彩色光晕、玻璃卡片。
- 白色圆的“发光”只能是 `rgba(255,255,255,.05–.12)`，不能变成彩色。
- 移动元素禁止 `filter: blur()`。

---

## 7. Animation & Interaction

**Motion Philosophy**：慢、连续、可打断；悬停是唯一“停”的时刻。
**Tier**：L2（自绘 rAF 物理）

### 漂移
- 速度 4–10 px/s，方向随机；`dt` 上限 0.05s，切后台暂停。
- 边界反弹；圆-圆碰撞分离并交换法向速度；位置每帧写 `translate3d`。

### 悬停
- `pointerenter` / `focus`：`hovered = true`，该圆不再积分，也不被邻居推动。
- CSS `.bubble:hover .bubble__disc { transform: scale(1.5) }`，0.55s `cubic-bezier(.22,.61,.36,1)`。
- `pointerleave` / `blur`：恢复漂移。
- 打开详情框时清空所有 hover 状态，避免“冻结残留”。

### 入场
- 新圆 0.5s `opacity 0 → 1`；不做位移入场（位置由物理决定）。
- 筛选后保留已有圆的位置，只增删集合内的圆。

### 点击 → 白色方框
1. 记录圆的视觉矩形（`.bubble__disc`，含 hover 1.5×）与最终方形（居中，`side`）。
2. 固定定位的 `.prompt-box` 先放在圆的矩形上（`border-radius: 50%`），强制 reflow。
3. 下一帧设为最终 top/left/width/height 与 `border-radius: 4px`；CSS transition **1s**。
4. 内容 0.55s 后淡入；1s 后焦点移到关闭按钮。
5. 关闭：0.5s 缩回原圆（圆角回到 50%），恢复星图；星图在打开期间 `stopLoop()`。

### Reduced Motion
```css
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: .01ms !important; transition-duration: .01ms !important; }
}
```
- JS 检测 `prefers-reduced-motion` 时**不启动 rAF**：圆静止，悬停仍放大（瞬时）。

### 性能
- 34 个绝对定位元素，单页无 WebGL、无 canvas。
- `ResizeObserver` 防抖 160ms，视口/侧栏变化时重算圆径并夹紧位置。
- 页面隐藏时 `cancelAnimationFrame`。

---

## 8. Do's and Don'ts

### Do
- 保持全站零色相：只用 #000 / #fff / 中性灰。
- 圆里只放标题，最长 3 行；需要摘要/标签就进详情。
- 悬停必须冻结 + 居中放大 1.5×；放大后不得被星图边缘裁切。
- 圆与圆之间至少留 8px 空隙；初始分布用 best-candidate，不要排成网格。
- 移动端关闭悬停放大，点击直接开白色方形框；页面可滚动看当前批次的圆。
- 一次只显示 20 枚圆；刷新按钮负责轮换到下一批，不要一次铺满全部。
- 点击圆后必须从圆本身长成方框，时长 1s，不能改为从右侧滑出。
- 侧栏收起必须整栏滑出，左缘只留 52×82 白色椭圆「铸」；点击它展开。
- 尊重 `prefers-reduced-motion`：完全静止，别只减速。
- 方框 / 编辑抽屉、Toast 的层级与焦点管理必须完整（焦点陷阱 + 归还）。
- 任何筛选后保留已有圆的位置，避免整场重排。

### Don't
- ❌ 不要引入任何彩色（包括品牌色、分类色、红色错误态）。
- ❌ 不要用渐变、玻璃拟态、彩色阴影。
- ❌ 不要把圆做成等距网格或统一大小——那会退回“卡片墙”。
- ❌ 不要在圆里塞摘要、标签、按钮、封面图。
- ❌ 不要用 `filter: blur()` 处理运动中的圆；不要常驻 `will-change`。
- ❌ 不要用滚动劫持、pin、视差背景。
- ❌ 不要让悬停放大以左上角为原点（必须居中）。
- ❌ 不要在移动端依赖 hover；触摸必须可点开。
- ❌ 不要在星图中间放常驻说明文字/图例。
- ❌ 不要让“查看详情”从右侧抽屉弹出；详情只能是白底黑字的方形框。
- ❌ 不要在展开过程中显示未排版好的内容——内容必须在框成形后淡入。
- ❌ 不要把侧栏收成 64px 窄轨或半截；收起后不应在页面里留下任何侧栏文字。
- ❌ 不要在顶栏写「字盘 / 存量」这类分组小标题；也不要一次渲染全部 34 枚圆。

---

## 9. Responsive Behavior

| Name | Width | Key Changes |
|------|-------|-------------|
| Desktop | ≥ 1200px | 侧栏 200px（可整栏收起）；星图填满视口；圆径 82–128 |
| Laptop | 1024–1199px | 侧栏 200px（可整栏收起）；圆径按面积重算 |
| Tablet | 768–1023px | 侧栏 off-canvas；圆径 ≥ 70 |
| Mobile | < 768px | 顶栏 56px；星图固定 1100px 高、页面滚动；圆径 ≥ 70；详情框 88vw；触摸目标 ≥44px |

**Collapsing Strategy**
- 桌面端点字盘右上角「‹」收起，整栏滑出；左边缘白色椭圆「铸」点击展开；状态存 `zhuci.rail.v1`，<1024px 不生效。
- <1024px 侧栏为 280px 抽屉，菜单按钮打开，侧栏内「‹」关闭。
- 移动端星图 1100px 高，34 枚圆分布其中，滚动查看。

```css
@media (max-width: 767px) {
  .topbar { height: 56px; padding: 0 14px; }
  .stage { flex: none; height: 1100px; }
  .drawer { width: 100vw; }
  .bubble:hover .bubble__disc { transform: scale(1); } /* 触摸端 */
}
```

---

## 附录 A：数据

```js
// Prompt
{
  id: 'sft-cot',
  category: 'sft',            // sft | rag | lowcode | image
  title: '中文思维链数据构造',
  summary: '……',
  tags: ['思维链', '数据构造'],
  body: '……{{变量}}……',
  source: { name: '本站整理', url: '' },
  createdAt: 1762000000000,
  updatedAt: 1762000000000
}
```

**localStorage**
- `zhuci.library.v1` — 提示词数组
- `zhuci.favorites.v1` — 收藏 id
- `zhuci.seed.v1` — 示例库版本
- `zhuci.rail.v1` — 侧栏是否收起

**Supabase**
- Auth：邮箱 + 密码；注册确认邮件；session 由 supabase-js 持久化。
- `prompts`：云端提示词，含 `status`（pending / approved）、作者、时间。
- `profiles`：账号资料与管理员标记；RLS 控制读写。

## 附录 B：页面结构

```
.shell
├─ header.topbar（横跨整页：菜单 / 居中搜索 / 刷新 / 20 / 34 条 / 标签 chip / 登录 chip / + 新建）
├─ aside.sidebar（第二行，200px）
│   ├─ 方印「铸」 + 收起「‹」
│   ├─ 分隔线
│   └─ nav（全部 / SFT / RAG / 低代码 / 绘图 / 收藏，无点无数）
└─ .main（第二行）
    └─ main.stage
        ├─ .bubbles（当前批次 20 个 .bubble）
        ├─ .field-empty
        └─ .boot-note

.rail-fab                        （收起后的白色椭圆「铸」，固定左中）
.prompt-backdrop + .prompt-box   （点击圆展开的白色方形详情框）
.drawer                          （新建 / 编辑 / 登录 / 账号 / 审核）
.toast
```
