# 技术工具箱 — CDN、代码片段与性能陷阱

本文档提供可直接复用的实现代码。原则：**简单效果纯 CSS，复杂编排用 GSAP，平滑滚动用 Lenis**。不要为一个 fade-in 引入整个动画库。

## 目录

- 一、CDN 引入清单
- 二、基础设施片段（Lenis 平滑滚动 / reduced-motion / 缓动体系）
- 三、核心效果片段（逐字浮起 / sticky 叙事 / 横向滚动 / 视差 / 数字滚动 / 磁性按钮 / marquee / 流动渐变 / 噪点 / 自定义光标 / 卡片 3D 倾斜）
- 四、性能与避坑清单

---

## 一、CDN 引入清单

按需引入，放在 `</body>` 前：

```html
<!-- GSAP 核心 + ScrollTrigger：滚动编排的主力 -->
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/ScrollTrigger.min.js"></script>
<!-- Lenis：平滑滚动（惯性滚动是高级感的一半） -->
<script src="https://cdn.jsdelivr.net/npm/lenis@1.1.18/dist/lenis.min.js"></script>
<!-- SplitType：文字拆字/拆行（逐字动画必备） -->
<script src="https://cdn.jsdelivr.net/npm/split-type@0.3.4/umd/index.min.js"></script>
<!-- 可选：Lottie（需要复杂矢量动画时） -->
<script src="https://cdn.jsdelivr.net/npm/lottie-web@5.12.2/build/player/lottie.min.js"></script>
<!-- 可选：canvas-confetti（庆祝粒子，Duolingo 风） -->
<script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.3/dist/confetti.browser.min.js"></script>
```

字体（Google Fonts 国内可能慢，中文项目优先系统字体栈或 fontsource 镜像）：

```html
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;700&family=Inter:wght@400;600&display=swap" rel="stylesheet">
```

图标（**不要用 emoji 当图标**，统一引在线图标库，风格一致且可控大小颜色）：

```html
<!-- 方案一：Remix Icon（class 方式，简单直接） -->
<link href="https://cdn.jsdelivr.net/npm/remixicon@4.5.0/fonts/remixicon.css" rel="stylesheet">
<!-- 用法：<i class="ri-cup-line"></i>，font-size/color 控制大小颜色 -->

<!-- 方案二：Iconify（按需加载任意图标集） -->
<script src="https://cdn.jsdelivr.net/npm/iconify-icon@2.1.0/dist/iconify-icon.min.js"></script>
<!-- 用法：<iconify-icon icon="lucide:coffee" width="32"></iconify-icon> -->
```

## 二、基础设施片段

### Lenis + ScrollTrigger 联动（每个页面的标配开头）

```js
// 平滑滚动初始化，并与 ScrollTrigger 同步
gsap.registerPlugin(ScrollTrigger);
const lenis = new Lenis({ lerp: 0.1 });
lenis.on('scroll', ScrollTrigger.update);
gsap.ticker.add((time) => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0);
```

### prefers-reduced-motion 降级（必须有）

```js
// 用户系统开启"减弱动态效果"时，跳过所有装饰动画
const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
if (reduceMotion) {
  gsap.globalTimeline.timeScale(100); // 动画瞬间完成
  // 同时在 CSS 中用 @media (prefers-reduced-motion: reduce) 关闭 keyframes 动画
}
```

### 统一缓动体系（写在页面 JS 开头，全页引用）

```js
// 全页统一的动画语言：换风格时只改这里
const EASE = 'power3.out';        // 科技/创意系；活泼系换 'back.out(1.7)'
const DUR = { fast: 0.5, base: 0.9, slow: 1.4 };
```

CSS 侧对应：

```css
:root {
  --ease-out: cubic-bezier(0.22, 1, 0.36, 1);     /* 丝滑减速 */
  --ease-bounce: cubic-bezier(0.34, 1.56, 0.64, 1); /* 弹性过冲 */
}
```

## 三、核心效果片段

### 1. 巨字逐字浮起（Hero 签名款）

```html
<h1 class="hero-title">晨光，注入每一杯</h1>
<style>.hero-title .line { overflow: hidden; }</style>
```

```js
// 拆字后逐字从行内底部翻上来
const split = new SplitType('.hero-title', { types: 'lines, chars' });
gsap.from(split.chars, {
  yPercent: 110, rotate: 6, opacity: 0,
  duration: DUR.base, ease: EASE, stagger: 0.035, delay: 0.2
});
```

### 2. scroll reveal 通用进场（全页基础款）

```js
// 给所有 [data-reveal] 元素挂进场动画，stagger 由 data-stagger 控制
document.querySelectorAll('[data-reveal]').forEach((el) => {
  gsap.from(el.dataset.stagger ? el.children : el, {
    y: 60, opacity: 0, duration: DUR.base, ease: EASE,
    stagger: el.dataset.stagger ? +el.dataset.stagger : 0,
    scrollTrigger: { trigger: el, start: 'top 80%' }
  });
});
```

### 3. sticky 定格叙事（Apple 式签名款）

```html
<section class="pin-section"><div class="pin-stage"><!-- 多个 .step 内容 --></div></section>
```

```js
// 固定一屏，滚动 3 屏的距离内依次切换 step
const steps = gsap.utils.toArray('.pin-stage .step');
const tl = gsap.timeline({
  scrollTrigger: { trigger: '.pin-section', start: 'top top', end: '+=300%', pin: true, scrub: 0.6 }
});
steps.forEach((step, i) => {
  if (i) tl.fromTo(step, { opacity: 0, y: 80 }, { opacity: 1, y: 0 })
          .to(steps[i - 1], { opacity: 0, y: -80 }, '<');
});
```

### 4. 横向滚动章节

```js
// 垂直滚动驱动横向画布平移，track 内为并排的卡片
const track = document.querySelector('.h-track');
gsap.to(track, {
  x: () => -(track.scrollWidth - innerWidth),
  ease: 'none',
  scrollTrigger: {
    trigger: '.h-section', start: 'top top',
    end: () => '+=' + (track.scrollWidth - innerWidth),
    pin: true, scrub: 1, invalidateOnRefresh: true
  }
});
```

### 5. 视差分层

```js
// data-speed 为深度系数：0.5 慢速远景，1.5 快速近景
gsap.utils.toArray('[data-speed]').forEach((el) => {
  gsap.to(el, {
    y: () => (1 - +el.dataset.speed) * 200, ease: 'none',
    scrollTrigger: { trigger: el.closest('section'), scrub: true }
  });
});
```

### 6. 数字滚动累加

```js
document.querySelectorAll('[data-count]').forEach((el) => {
  const target = +el.dataset.count;
  gsap.fromTo(el, { textContent: 0 }, {
    textContent: target, duration: 1.6, ease: 'power1.out',
    snap: { textContent: target > 100 ? 1 : 0.1 },  // 大数取整，小数保留一位
    scrollTrigger: { trigger: el, start: 'top 85%' }
  });
});
```

### 7. 磁性按钮

```js
// 鼠标靠近时按钮被吸向光标
document.querySelectorAll('.magnetic').forEach((btn) => {
  btn.addEventListener('mousemove', (e) => {
    const r = btn.getBoundingClientRect();
    gsap.to(btn, { x: (e.clientX - r.left - r.width / 2) * 0.35,
                   y: (e.clientY - r.top - r.height / 2) * 0.35, duration: 0.3 });
  });
  btn.addEventListener('mouseleave', () => gsap.to(btn, { x: 0, y: 0, duration: 0.5, ease: 'elastic.out(1, 0.4)' }));
});
```

### 8. marquee 无限滚动（纯 CSS）

```html
<div class="marquee"><div class="marquee-track"><!-- 内容重复两份 --></div></div>
```

```css
.marquee { overflow: hidden; }
.marquee-track { display: flex; gap: 3rem; width: max-content; animation: scroll 24s linear infinite; }
.marquee:hover .marquee-track { animation-play-state: paused; }
@keyframes scroll { to { transform: translateX(-50%); } } /* 内容必须恰好重复两份 */
```

### 9. 流动渐变背景（Stripe 风轻量版，纯 CSS）

```css
.aurora { position: absolute; inset: 0; overflow: hidden; filter: blur(90px); }
.aurora i { position: absolute; border-radius: 50%; width: 45vw; height: 45vw; opacity: .55; }
.aurora i:nth-child(1) { background: #6366f1; top: -10%; left: -5%; animation: drift1 16s ease-in-out infinite alternate; }
.aurora i:nth-child(2) { background: #ec4899; bottom: -15%; right: -5%; animation: drift2 20s ease-in-out infinite alternate; }
@keyframes drift1 { to { transform: translate(30vw, 20vh) scale(1.2); } }
@keyframes drift2 { to { transform: translate(-25vw, -15vh) scale(0.9); } }
```

### 10. 噪点 overlay（消除塑料感，纯 CSS + 内联 SVG）

```css
body::after {
  content: ''; position: fixed; inset: 0; pointer-events: none; z-index: 9999; opacity: .04;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='200' height='200'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}
```

### 11. 自定义光标

```js
// 圆点即时跟随 + 圆环 lerp 延迟跟随；触屏设备直接跳过
if (matchMedia('(pointer: fine)').matches) {
  const ring = document.querySelector('.cursor-ring');
  let mx = 0, my = 0, rx = 0, ry = 0;
  addEventListener('mousemove', (e) => { mx = e.clientX; my = e.clientY; });
  gsap.ticker.add(() => {
    rx += (mx - rx) * 0.15; ry += (my - ry) * 0.15;
    ring.style.transform = `translate(${rx}px, ${ry}px) translate(-50%, -50%)`;
  });
  // hover 可交互元素时给 ring 加 .is-active 放大
}
```

### 12. 卡片 3D 倾斜

```js
document.querySelectorAll('.tilt').forEach((card) => {
  card.addEventListener('mousemove', (e) => {
    const r = card.getBoundingClientRect();
    gsap.to(card, { rotateY: ((e.clientX - r.left) / r.width - 0.5) * 14,
                    rotateX: -((e.clientY - r.top) / r.height - 0.5) * 14,
                    transformPerspective: 600, duration: 0.4 });
  });
  card.addEventListener('mouseleave', () => gsap.to(card, { rotateX: 0, rotateY: 0, duration: 0.6 }));
});
```

## 四、性能与避坑清单

- **只动画 `transform` 和 `opacity`**。动画 `top/left/width/height/margin` 会触发重排，滚动时必卡。需要模糊渐变时 `filter` 可用但控制范围。
- **scrub 动画加缓冲**：`scrub: 0.6~1` 比 `scrub: true` 跟手且不生硬。
- **`will-change` 只给正在动的大元素**，且数量克制（全页超过十几个反而更卡）。
- **pin 容器内不要用百分比高度的子元素**，ScrollTrigger pin 会改 DOM 结构，易错位；pin 元素的父级避免 `overflow: hidden`。
- **图片占位**：渐变 + SVG 图形 + 图标库大图标组合造视觉图，或 `https://picsum.photos/800/600?random=1`。绝不引用不存在的本地路径（页面会出现破图）。全页不出现 emoji。
- **移动端**：`mousemove` 类效果（视差、磁性、自定义光标、3D 倾斜）必须用 `matchMedia('(pointer: fine)')` 包裹；横向滚动章节在窄屏可改为原生横滑或纵向排列（用 `ScrollTrigger.matchMedia` / `gsap.matchMedia()`）。
- **字体闪动**：标题动画前先等 `document.fonts.ready`，否则 SplitType 拆字后字体加载完成会错位。
- **资源加载顺序**：所有动画初始化包在 `window.addEventListener('load', ...)` 或 DOMContentLoaded + fonts.ready 之后。
- **首屏不空白**：进场动画用 `gsap.from()`（初始态由 JS 设置）而非 CSS 先 `opacity: 0`——这样 JS 加载失败时页面仍可读（渐进增强）。
