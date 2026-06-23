# 技术工具箱 — CDN、代码片段与性能陷阱

本文档提供可直接复用的实现代码。原则：**简单效果纯 CSS，复杂编排用 GSAP，平滑滚动用 Lenis**。不要为一个 fade-in 引入整个动画库。

## 目录

- 一、CDN 引入清单
- 二、基础设施片段（Lenis 平滑滚动 / reduced-motion / 缓动体系）
- 三、核心效果片段（逐字浮起 / sticky 叙事 / 横向滚动 / 视差 / 数字滚动 / 磁性按钮 / marquee / 流动渐变 / 噪点 / 自定义光标 / 卡片 3D 倾斜 / OS-HUD / 封面遮挡舞台 / Three.js 签名物）
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

Three.js 自身用模块方式按需引入：

```html
<script type="module">
  import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.165.0/build/three.module.js';
</script>
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

### 13. OS-HUD 状态栏与坐标反馈

适合个人作品集、设计师主页、实验工作室。HUD 只放状态和导航，长文案留给舞台内容。

```html
<header class="hud" aria-label="站点状态栏">
  <a class="hud-brand" href="#home">YOUR.STUDIO</a>
  <nav class="hud-nav" aria-label="主导航">
    <button data-state-target="work">WORK</button>
    <button data-state-target="contact">CONTACT</button>
    <button data-theme-toggle>THEME[A]</button>
  </nav>
  <div class="hud-meta">
    <span data-clock>GMT+8 00:00</span>
    <span data-coord>0000 X 0000 Y</span>
  </div>
</header>
```

```css
.hud {
  position: fixed; inset: 0; z-index: 50; pointer-events: none;
  display: grid; grid-template-columns: 1fr auto; grid-template-rows: auto 1fr auto;
  padding: clamp(20px, 4vw, 56px); font-family: var(--mono, ui-monospace, monospace);
  font-size: clamp(.72rem, 1vw, .95rem); letter-spacing: 0;
}
.hud a, .hud button { pointer-events: auto; }
.hud-nav { display: flex; gap: clamp(16px, 3vw, 44px); }
.hud button, .hud a {
  color: inherit; background: none; border: 0; padding: .45rem .55rem; font: inherit; text-decoration: none;
}
.hud button:hover, .hud button:focus-visible, .hud a:hover, .hud a:focus-visible {
  outline: 2px dotted currentColor; outline-offset: 2px;
}
.hud-meta { grid-column: 1 / -1; align-self: end; display: flex; justify-content: space-between; gap: 1rem; }
@media (max-width: 700px) {
  .hud { position: absolute; min-height: 100dvh; }
  .hud-nav { justify-self: end; gap: .5rem; }
  .hud-meta { font-size: .8rem; }
}
```

```js
// 更新坐标和时间，制造"可操作系统"的反馈感
const coord = document.querySelector('[data-coord]');
const clock = document.querySelector('[data-clock]');
const pad4 = (value) => String(Math.round(value)).padStart(4, '0');

if (matchMedia('(pointer: fine)').matches && coord) {
  addEventListener('pointermove', (event) => {
    coord.textContent = `${pad4(event.clientX)} X ${pad4(event.clientY)} Y`;
  }, { passive: true });
}

function updateClock() {
  if (!clock) return;
  const now = new Date();
  clock.textContent = `GMT+8 ${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`;
}
updateClock();
setInterval(updateClock, 30000);
```

### 14. OS-HUD 单屏状态切换

用在没有传统长滚动叙事的作品集页。按钮改变页面状态，同时把对应舞台滚到视口。

```html
<section id="home" data-panel="home" class="stage-panel"></section>
<section id="work" data-panel="work" class="stage-panel"></section>
<section id="contact" data-panel="contact" class="stage-panel"></section>
```

```css
.stage-panel { min-height: 100dvh; position: relative; display: grid; place-items: center; }
.stage-panel + .stage-panel { margin-top: 12vh; }
body[data-state="work"] .hud [data-state-target="work"],
body[data-state="contact"] .hud [data-state-target="contact"] {
  outline: 2px dotted currentColor; outline-offset: 2px;
}
```

```js
// 让 HUD 成为状态控制器，而不是普通跳转菜单
document.querySelectorAll('[data-state-target]').forEach((button) => {
  button.addEventListener('click', () => {
    const state = button.dataset.stateTarget;
    const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;
    document.body.dataset.state = state;
    document.querySelector(`[data-panel="${state}"]`)?.scrollIntoView({
      behavior: reduce ? 'auto' : 'smooth',
      block: 'start'
    });
  });
});
```

### 15. 封面遮挡舞台层

滚过封面或进入 `WORK` 状态时，用一个不透明档案面板盖住底层 3D canvas。颜色跟随品牌语气：可以是纸色、浅灰、近黑或品牌中性色；核心不是再堆一个复杂 3D，而是让页面从“沉浸封面”切到“可阅读作品档案”。

```html
<canvas id="signature-scene" class="signature-scene" aria-hidden="true"></canvas>

<section id="work" data-panel="work" class="stage-panel archive-panel">
  <canvas class="archive-paper" aria-hidden="true"></canvas>
  <div class="archive-content">
    <!-- 作品档案网格放这里 -->
  </div>
</section>
```

```css
:root {
  /* 示例值。按品牌替换为纸色、浅灰、近黑或其他不透明档案表面色。 */
  --archive-surface: #f4efe5;
}

.signature-scene {
  position: fixed; inset: 0; z-index: -1; width: 100%; height: 100%;
}

.archive-panel {
  position: relative; z-index: 2; overflow: hidden;
  min-height: 100dvh; background: var(--archive-surface); color: #111;
  box-shadow: 0 -1px 0 rgba(0,0,0,.08);
}

.archive-paper {
  position: absolute; inset: 0; z-index: 0; width: 100%; height: 100%;
  pointer-events: none;
}

.archive-content {
  position: relative; z-index: 1;
  padding: clamp(96px, 14vw, 190px) clamp(20px, 4vw, 56px);
}
```

```js
// 可选：给作品区画一层很轻的纸纹，避免大色块像默认背景。
const paperCanvas = document.querySelector('.archive-paper');
function paintPaperTexture() {
  if (!paperCanvas) return;
  const ratio = Math.min(devicePixelRatio || 1, 2);
  const rect = paperCanvas.getBoundingClientRect();
  paperCanvas.width = Math.max(1, Math.floor(rect.width * ratio));
  paperCanvas.height = Math.max(1, Math.floor(rect.height * ratio));

  const ctx = paperCanvas.getContext('2d');
  ctx.setTransform(ratio, 0, 0, ratio, 0, 0);
  ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--archive-surface').trim() || '#f4efe5';
  ctx.fillRect(0, 0, rect.width, rect.height);

  ctx.fillStyle = 'rgba(20, 16, 10, 0.035)';
  for (let i = 0; i < rect.width * rect.height / 1800; i++) {
    ctx.fillRect(Math.random() * rect.width, Math.random() * rect.height, 1, 1);
  }
}
paintPaperTexture();
addEventListener('resize', paintPaperTexture);
```

如果需要更戏剧性的“盖住”动作，可在进入作品区时给 `.archive-panel` 加 `clip-path` 动画：

```js
gsap.fromTo('.archive-panel',
  { clipPath: 'inset(0 0 100% 0)' },
  {
    clipPath: 'inset(0 0 0% 0)',
    duration: 0.9,
    ease: 'power3.out',
    scrollTrigger: { trigger: '.archive-panel', start: 'top 85%', once: true }
  }
);
```

### 16. Three.js 全屏签名物

用于 OS-HUD 系或科技旗舰系。先用几何体占位，后续可替换为品牌 3D 字标、GLTF 模型或 SVG 挤出模型。

```html
<canvas id="signature-scene" class="signature-scene" aria-hidden="true"></canvas>
```

```css
.signature-scene {
  position: fixed; inset: 0; z-index: -1; width: 100%; height: 100%;
  background: radial-gradient(circle at 50% 55%, rgba(255,255,255,.62), transparent 42%);
}
```

```js
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.165.0/build/three.module.js';

const canvas = document.querySelector('#signature-scene');
const reduceSceneMotion = matchMedia('(prefers-reduced-motion: reduce)').matches;
if (canvas && !reduceSceneMotion) {
  const renderer = new THREE.WebGLRenderer({ canvas, alpha: true, antialias: true });
  renderer.setPixelRatio(Math.min(devicePixelRatio, 1.8));

  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(35, innerWidth / innerHeight, 0.1, 100);
  camera.position.set(0, 0, 7);

  const mesh = new THREE.Mesh(
    new THREE.TorusKnotGeometry(1.35, 0.36, 180, 24),
    new THREE.MeshPhysicalMaterial({
      color: 0x7db7ff, roughness: 0.18, metalness: 0.08,
      transmission: 0.35, thickness: 0.9, transparent: true, opacity: 0.86
    })
  );
  scene.add(mesh);
  scene.add(new THREE.HemisphereLight(0xffffff, 0x8ab7ff, 2.2));

  let targetX = 0;
  let targetY = 0;
  let currentX = 0;
  let currentY = 0;

  addEventListener('pointermove', (event) => {
    targetX = (event.clientX / innerWidth - 0.5) * 2;
    targetY = (event.clientY / innerHeight - 0.5) * 2;
  }, { passive: true });

  function resizeScene() {
    const width = innerWidth;
    const height = innerHeight;
    renderer.setSize(width, height, false);
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
  }
  addEventListener('resize', resizeScene);
  resizeScene();

  function render(time) {
    currentX += (targetX - currentX) * 0.08;
    currentY += (targetY - currentY) * 0.08;
    mesh.rotation.y = time * 0.00018 + currentX * 0.35;
    mesh.rotation.x = -0.15 + currentY * 0.18;
    renderer.render(scene, camera);
    requestAnimationFrame(render);
  }
  requestAnimationFrame(render);
}
```

## 四、性能与避坑清单

- **只动画 `transform` 和 `opacity`**。动画 `top/left/width/height/margin` 会触发重排，滚动时必卡。需要模糊渐变时 `filter` 可用但控制范围。
- **WebGL 要有降级**：Three.js 签名物必须配静态文字/SVG/CSS 背景 fallback；`prefers-reduced-motion` 下停止渲染或改为静态图。
- **scrub 动画加缓冲**：`scrub: 0.6~1` 比 `scrub: true` 跟手且不生硬。
- **`will-change` 只给正在动的大元素**，且数量克制（全页超过十几个反而更卡）。
- **pin 容器内不要用百分比高度的子元素**，ScrollTrigger pin 会改 DOM 结构，易错位；pin 元素的父级避免 `overflow: hidden`。
- **图片占位**：渐变 + SVG 图形 + 图标库大图标组合造视觉图，或 `https://picsum.photos/800/600?random=1`。绝不引用不存在的本地路径（页面会出现破图）。全页不出现 emoji。
- **移动端**：`mousemove` 类效果（视差、磁性、自定义光标、3D 倾斜）必须用 `matchMedia('(pointer: fine)')` 包裹；横向滚动章节在窄屏可改为原生横滑或纵向排列（用 `ScrollTrigger.matchMedia` / `gsap.matchMedia()`）。
- **字体闪动**：标题动画前先等 `document.fonts.ready`，否则 SplitType 拆字后字体加载完成会错位。
- **资源加载顺序**：所有动画初始化包在 `window.addEventListener('load', ...)` 或 DOMContentLoaded + fonts.ready 之后。
- **首屏不空白**：进场动画用 `gsap.from()`（初始态由 JS 设置）而非 CSS 先 `opacity: 0`——这样 JS 加载失败时页面仍可读（渐进增强）。
