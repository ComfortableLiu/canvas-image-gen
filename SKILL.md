---
name: canvas-image-gen
description: 用纯文本模型的代码能力，通过 HTML Canvas / SVG 代码级作画生成图像。用户描述想要的图，AI 生成 HTML+Canvas+JS（或 SVG 矢量）代码，浏览器打开即可查看，点击按钮可保存为 PNG、SVG、WebM 或 GIF。就像文生图一样，但用代码画。
version: 3.2.0
author: 说人话的实验室
tags: [canvas, svg, 图像生成, 代码作画, html, 文生图, 可视化, gif, webm, 动画, 高清, 矢量]
---

# Canvas Image Gen

用纯文本模型的代码能力，通过 HTML Canvas / SVG 代码级作画生成图像。

就像文生图模型一样——用户描述想要的图，AI 帮你画出来。只不过"画笔"是代码，"画布"是 HTML Canvas（位图）或内联 SVG（矢量）。

支持四种输出：
- **静态位图**：导出 PNG（默认，HiDPI 高清）
- **静态矢量图**：导出 SVG（无限缩放、可二次编辑、文件极小）
- **动画视频**：导出 WebM（vp9 编码，15Mbps，30fps，无 alpha，最高画质）
- **动画图**：导出 GIF（NeuQuant 颜色量化 + LZW 压缩，20fps）

## 核心原理

```
用户："帮我画一个红色的太阳在蓝色天空上"
         ↓
AI 确认输出格式（没说清就问）：PNG · SVG · WebM · GIF
         ↓
AI 生成 HTML+Canvas+JS（或 SVG 矢量）代码
         ↓
保存为 HTML 文件，浏览器打开
         ↓
点击"保存图片"按钮 → 导出 PNG
点击"导出 SVG"按钮  → 导出 SVG（矢量模式）
点击"导出 GIF"按钮  → 录制动画 → 导出 GIF（仅动画内容）
```

## 使用方法

用户说"帮我画一个 XXX"或"生成一张 XXX 的图"，AI 按以下流程执行：

### Step 1：理解需求

从用户描述中提取：
- **画什么**：主体元素、场景、构图
- **风格**：扁平/卡通/写实/极简/像素风（默认扁平风）
- **配色**：用户指定或根据主题自选
- **尺寸**：用户指定或根据用途自选（默认 800x600）
- **输出格式**：PNG / SVG / WebM / GIF（用户指定，或按下述规则推断，判断不了就问）

#### 输出格式询问选项

用户没指定输出格式时，先按规则推断；**推断不出来才问**（不要每次都打断用户）。

**推断规则：**
- 描述里出现"图标 / Logo / 矢量 / 可缩放 / 放大不糊 / 要改颜色 / 交给设计师 / 印刷" → **SVG**
- "动画 / 视频 / 录屏 / 演示过程 / 算法过程" → **WebM**
- "动图 / 表情包 / 发群里 / 在不支持视频的地方放" → **GIF**
- 其余静态图 → **PNG**（默认）
- 描述与规则冲突（例如既说"要矢量"又要"做动画"）→ **问**

**询问话术**（原样或近似照抄）：

> 想要哪种输出格式？
> 1. **PNG** — 静态位图，光影/粒子/噪点这类效果好
> 2. **SVG** — 静态矢量图，无限放大不糊，能二次编辑、文件小
> 3. **WebM** — 动画视频，最高画质
> 4. **GIF** — 动画图，方便到处贴

**格式对照表：**

| 选项 | 导出 | 画布 | 适合 | 不适合 |
|------|------|------|------|--------|
| PNG | 位图 | Canvas | 光影、粒子、噪点、纹理、复杂特效 | 需要缩放/改色 |
| **SVG** | **矢量** | **内联 `<svg>`** | **图标、Logo、示意图、架构图、图表、扁平插画、UI 稿** | 照片感、粒子、噪点、大量逐帧动画 |
| WebM | 视频 | Canvas | 动画演示、算法过程、视频素材 | 只要静图 |
| GIF | 动图 | Canvas | 动图表情、社媒配图 | 要高画质/长动画 |

> 在支持选项卡片的环境（如 Claude Code 的 `AskUserQuestion`）里用选项卡片列出这 4 项，**SVG 必须作为其中一个选项**。

### Step 2：生成 HTML 代码

生成完整的 HTML 代码。**先定输出模式**：SVG 走矢量画布（内联 `<svg>`），PNG / WebM / GIF 走位图画布（`<canvas>`）。代码必须满足：

**基础要求（两种模式通用）：**
- 自包含：所有样式和脚本都在 HTML 文件内，不依赖外部资源
- **按钮位置规则**：按钮必须放在画布以外，使用固定定位（position: fixed）悬浮在页面角落，不能遮挡或影响画布内容。**严禁把按钮放在 canvas / svg 的 wrapper 内或使用 position: absolute 相对于画布定位。** 推荐：按钮放在 `<body>` 下，用 `position: fixed; top: 16px; right: 16px;` 悬浮在页面右上角。

**位图模式要求（PNG / WebM / GIF）：**
- **必须 HiDPI 适配**：canvas 物理尺寸 = CSS 尺寸 × devicePixelRatio，ctx.scale(dpr, dpr)
- **必须包含"保存图片"按钮**：点击后将 Canvas 导出为 PNG 并自动下载
- **动画内容必须包含"导出 WebM"和"导出 GIF"按钮**

**矢量模式要求（SVG）：**
- 画布用**内联 `<svg>`**，不要用 `<canvas>`
- `<svg>` 必须带 `xmlns="http://www.w3.org/2000/svg"`、`viewBox`、`width`、`height`（缺 viewBox 导出后无法正确缩放）
- 绘制代码用矢量元素：`<rect> <circle> <ellipse> <line> <polyline> <polygon> <path> <text> <g>`，渐变/图案放 `<defs>` 里（`<linearGradient>` / `<radialGradient>` / `<pattern>`），用 `<use>` 复用重复图形
- **必须包含"导出 SVG"按钮**：序列化 SVG DOM → 下载 .svg
- **必须包含"保存图片"按钮**：把 SVG 光栅化到 canvas（HiDPI）→ 导出 PNG
- **严禁外链资源**（外部图片、web 字体、外链 filter / CSS）：SVG 必须完全自包含，否则导出的 .svg 会丢内容，PNG 光栅化也会丢内容甚至污染 canvas
- **想要不透明底就画一个满幅背景 `<rect>`**：CSS `background` 只影响页面显示，不会进导出的 .svg 和 PNG（导出 PNG 会是透明底）
- 文字用系统字体族（`-apple-system, "PingFang SC", sans-serif`）；要跨机完全一致就把文字转成 `<path>`
- 尽量用 SVG 原生属性 / `<filter>`（feGaussianBlur、feDropShadow 等），别用只在浏览器才认的 CSS filter，否则在 Figma / PPT / Illustrator 里会变形
- **动画 SVG（可选）**：动画用 SMIL（`<animate>` / `<animateTransform>` / `<animateMotion>`），导出的 .svg 自带动画。注意 PowerPoint 和部分设计工具不支持 SMIL —— 需要兼容就改回 Canvas 走 WebM/GIF

**HiDPI 适配（位图模式，必须）：**
```html
<canvas id="c" style="width:900px;height:350px;"></canvas>
<script>
(function(){
  const c = document.getElementById('c');
  const dpr = window.devicePixelRatio || 1;
  c.width = 900 * dpr;
  c.height = 350 * dpr;
  c.getContext('2d').scale(dpr, dpr);
})();
</script>
```
注意：canvas.width/height 是物理像素（放大后），后续绘图代码用 CSS 尺寸（如 const W = 900）。

**动画导出要求（位图模式，仅当内容有动画时）：**
- 必须定义 `window.resetAnimation` 函数：将动画重置到第一帧
- 必须定义 `window.getAnimationDuration` 函数：返回动画周期（秒）
- **WebM 导出**：MediaRecorder + canvas.captureStream(30)，vp9.0 编码（无 alpha），15Mbps 码率
- **GIF 导出**：直接从 canvas 逐帧抓取（requestAnimationFrame），RGBA→RGB 转换，NeuQuant 颜色量化 + LZW 编码，20fps

**画风指导：**
- 优先使用扁平化设计（flat design），简洁干净
- 颜色鲜明，对比度高
- 形状清晰，边缘锐利
- 适当使用渐变增加层次感
- 文字清晰可读（如果有文字的话）
- 矢量模式额外注意：造型用基本几何 + `<path>` 组合，控制锚点数量；避免上万节点的描边草图（导出文件会爆炸）

**代码结构（位图 · 静态图）：**
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<style>
  body { margin: 0; display: flex; align-items: center; justify-content: center; min-height: 100vh; background: #f0f0f0; font-family: -apple-system, sans-serif; }
  canvas { border-radius: 8px; box-shadow: 0 4px 20px rgba(0,0,0,0.1); }
</style>
</head>
<body>
<canvas id="canvas"></canvas>
<div onclick="saveImage()" style="position:fixed;top:16px;right:16px;z-index:9999;background:#378ADD;color:#fff;border:none;border-radius:6px;padding:10px 20px;font-size:14px;cursor:pointer;">保存图片</div>
<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const dpr = window.devicePixelRatio || 1;
const W = 800, H = 600; // 根据需求调整

canvas.width = W * dpr;
canvas.height = H * dpr;
canvas.style.width = W + 'px';
canvas.style.height = H + 'px';
ctx.scale(dpr, dpr);

// ============ 绘制代码开始 ============
// ... AI 根据用户描述生成绘制代码 ...
// ============ 绘制代码结束 ============

// 保存图片
function saveImage() {
  const link = document.createElement('a');
  link.download = 'canvas-image.png';
  link.href = canvas.toDataURL('image/png');
  link.click();
}
</script>
</body>
</html>
```

**代码结构（位图 · 带动画）：**
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<style>
  body { margin: 0; display: flex; align-items: center; justify-content: center; min-height: 100vh; background: #1a1a2e; font-family: -apple-system, sans-serif; }
  canvas { border-radius: 8px; box-shadow: 0 4px 20px rgba(0,0,0,0.3); }
</style>
</head>
<body>
<canvas id="canvas" style="width:900px;height:350px;"></canvas>
<script>
// HiDPI 适配
(function(){
  const c = document.getElementById('canvas');
  const dpr = window.devicePixelRatio || 1;
  c.width = 900 * dpr;
  c.height = 350 * dpr;
  c.getContext('2d').scale(dpr, dpr);
})();

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const W = 900, H = 350; // CSS 尺寸（逻辑尺寸）

// 动画状态
let t0 = performance.now();
window.resetAnimation = function() { t0 = performance.now(); };
window.getAnimationDuration = function() { return 6; }; // 动画周期（秒）

function frame(now) {
  const t = (now - t0) / 1000; // 相对时间（秒）
  ctx.clearRect(0, 0, W, H);

  // ============ 绘制代码开始 ============
  // ... AI 根据用户描述生成动画绘制代码，使用 t 控制动画 ...
  // ============ 绘制代码结束 ============

  requestAnimationFrame(frame);
}
requestAnimationFrame(frame);
</script>

<!-- 导出按钮（三个） -->
<div id="saveBtn" onclick="saveImage()" style="position:fixed;top:16px;right:16px;z-index:9999;background:linear-gradient(135deg,#667eea,#764ba2);color:#fff;border:none;border-radius:8px;padding:10px 20px;font-size:14px;font-family:sans-serif;cursor:pointer;box-shadow:0 4px 15px rgba(0,0,0,0.3);transition:transform 0.2s;" onmouseover="this.style.transform='scale(1.05)'" onmouseout="this.style.transform='scale(1)'">📸 保存 PNG</div>
<div id="webmBtn" onclick="exportWebm()" style="position:fixed;top:16px;right:130px;z-index:9999;background:linear-gradient(135deg,#4facfe,#00f2fe);color:#fff;border:none;border-radius:8px;padding:10px 20px;font-size:14px;font-family:sans-serif;cursor:pointer;box-shadow:0 4px 15px rgba(0,0,0,0.3);transition:transform 0.2s;" onmouseover="this.style.transform='scale(1.05)'" onmouseout="this.style.transform='scale(1)'">🎥 导出 WebM</div>
<div id="gifExportBtn" onclick="exportGif()" style="position:fixed;top:16px;right:260px;z-index:9999;background:linear-gradient(135deg,#f093fb,#f5576c);color:#fff;border:none;border-radius:8px;padding:10px 20px;font-size:14px;font-family:sans-serif;cursor:pointer;box-shadow:0 4px 15px rgba(0,0,0,0.3);transition:transform 0.2s;" onmouseover="this.style.transform='scale(1.05)'" onmouseout="this.style.transform='scale(1)'">🎬 导出 GIF</div>
<div id="exportProgress" style="position:fixed;top:56px;right:16px;z-index:9999;background:rgba(0,0,0,0.8);color:#fff;padding:8px 16px;border-radius:8px;font-size:12px;font-family:sans-serif;display:none;">准备中...</div>
<script>
function saveImage() {
  const link = document.createElement('a');
  link.download = 'canvas-image.png';
  link.href = canvas.toDataURL('image/png');
  link.click();
}

async function exportWebm() {
  const prog = document.getElementById('exportProgress');
  const btn = document.getElementById('webmBtn');
  btn.style.opacity = '0.5'; btn.style.pointerEvents = 'none';
  prog.style.display = 'block';
  if (typeof window.resetAnimation === 'function') window.resetAnimation();
  const dur = typeof window.getAnimationDuration === 'function' ? window.getAnimationDuration() : 3;
  prog.textContent = '录制WebM（' + dur + '秒）...';
  try {
    const stream = canvas.captureStream(30);
    const recorder = new MediaRecorder(stream, {mimeType: 'video/webm;codecs=vp9.0', videoBitsPerSecond: 15000000});
    const chunks = [];
    recorder.ondataavailable = e => { if (e.data.size > 0) chunks.push(e.data); };
    const blob = await new Promise((resolve, reject) => {
      recorder.onstop = () => resolve(new Blob(chunks, {type: 'video/webm'}));
      recorder.onerror = reject;
      recorder.start();
      setTimeout(() => recorder.stop(), dur * 1000);
    });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.download = (document.title || 'animation') + '.webm';
    a.href = url; a.click();
    URL.revokeObjectURL(url);
    prog.textContent = '✅ WebM 导出完成！';
  } catch(e) { prog.textContent = '❌ ' + e.message; }
  setTimeout(() => { prog.style.display = 'none'; }, 2000);
  btn.style.opacity = '1'; btn.style.pointerEvents = 'auto';
}

// exportGif + encodeGIF + NeuQuant + lzwEncode 完整代码见下方
</script>

<!-- GIF 导出脚本（NeuQuant + LZW，直接从 canvas 抓帧） -->
<script>
function exportGif() { /* ... */ }
function encodeGIF(w, h, frames, delay) { /* ... */ }
function lzwEncode(out, pixels, minCodeSize) { /* ... */ }
function NeuQuant(pixels, samplefac) { /* ... */ }
</script>
</body>
</html>
```

**代码结构（矢量 · SVG）：**
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<style>
  body { margin: 0; display: flex; align-items: center; justify-content: center; min-height: 100vh; background: #f0f0f0; font-family: -apple-system, "PingFang SC", sans-serif; }
  svg { border-radius: 8px; box-shadow: 0 4px 20px rgba(0,0,0,0.1); background: #fff; }
</style>
</head>
<body>
<svg id="art" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 600" width="800" height="600"></svg>

<div onclick="saveImage()" style="position:fixed;top:16px;right:16px;z-index:9999;background:#378ADD;color:#fff;border:none;border-radius:6px;padding:10px 20px;font-size:14px;font-family:sans-serif;cursor:pointer;">📸 保存 PNG</div>
<div onclick="exportSvg()" style="position:fixed;top:16px;right:130px;z-index:9999;background:linear-gradient(135deg,#11998e,#38ef7d);color:#fff;border:none;border-radius:6px;padding:10px 20px;font-size:14px;font-family:sans-serif;cursor:pointer;">🧬 导出 SVG</div>

<script>
const svg = document.getElementById('art');
const NS = 'http://www.w3.org/2000/svg';
const W = 800, H = 600; // viewBox 尺寸，根据需求调整

// 矢量元素工厂：el('circle', {cx:400, cy:300, r:100, fill:'#FFB300'}, parent)
function el(tag, attrs, parent) {
  const n = document.createElementNS(NS, tag);
  for (const k in attrs) n.setAttribute(k, attrs[k]);
  (parent || svg).appendChild(n);
  return n;
}

// ============ 绘制代码开始 ============
// ... AI 根据用户描述生成矢量绘制代码 ...
// 例：
// const defs = el('defs', {});
// const g = el('linearGradient', {id:'sky', x1:0, y1:0, x2:0, y2:1}, defs);
// el('stop', {offset:'0%',  'stop-color':'#8ED6FF'}, g);
// el('stop', {offset:'100%','stop-color':'#E8F7FF'}, g);
// el('rect', {x:0, y:0, width:W, height:H, fill:'url(#sky)'});
// el('circle', {cx:400, cy:300, r:100, fill:'url(#sun)'});
// ============ 绘制代码结束 ============

// 序列化 SVG（导出用，去掉页面展示用的样式）
function serializeSvg() {
  const clone = svg.cloneNode(true);
  clone.setAttribute('xmlns', NS);
  clone.setAttribute('xmlns:xlink', 'http://www.w3.org/1999/xlink');
  clone.setAttribute('width', W);
  clone.setAttribute('height', H);
  clone.removeAttribute('style');
  return '<?xml version="1.0" encoding="UTF-8"?>\n' + new XMLSerializer().serializeToString(clone);
}

// 导出 SVG（矢量，可二次编辑）
function exportSvg() {
  const blob = new Blob([serializeSvg()], {type: 'image/svg+xml;charset=utf-8'});
  const link = document.createElement('a');
  link.download = 'canvas-image.svg';
  link.href = URL.createObjectURL(blob);
  link.click();
  setTimeout(function(){ URL.revokeObjectURL(link.href); }, 1000);
}

// 保存 PNG（把 SVG 光栅化到 canvas，HiDPI 高清）
function saveImage() {
  const dpr = window.devicePixelRatio || 1;
  const url = URL.createObjectURL(new Blob([serializeSvg()], {type: 'image/svg+xml;charset=utf-8'}));
  const img = new Image();
  img.onload = function() {
    const c = document.createElement('canvas');
    c.width = W * dpr;
    c.height = H * dpr;
    const ctx = c.getContext('2d');
    ctx.scale(dpr, dpr);
    ctx.drawImage(img, 0, 0, W, H);
    URL.revokeObjectURL(url);
    try {
      const link = document.createElement('a');
      link.download = 'canvas-image.png';
      link.href = c.toDataURL('image/png');
      link.click();
    } catch (e) {
      alert('PNG 导出失败（canvas 被污染，通常是 SVG 里引用了外部资源）。\n请改用「导出 SVG」，矢量图更好用。');
    }
  };
  img.onerror = function() {
    URL.revokeObjectURL(url);
    alert('SVG 光栅化失败，请直接用「导出 SVG」。');
  };
  img.src = url;
}
</script>
</body>
</html>
```

**动画 SVG（矢量模式带动画时）**：把动画写成 SMIL，导出的 .svg 自带播放。

```html
<g>
  <animateTransform attributeName="transform" type="rotate"
                    from="0 400 300" to="360 400 300" dur="6s" repeatCount="indefinite"/>
  <circle cx="400" cy="200" r="24" fill="#38ef7d"/>
</g>
```
导出按钮与上面完全一致（序列化 SVG DOM 时 SMIL 会一起带走）。注意 PowerPoint / Figma / Illustrator 对 SMIL 支持不一，需要跨软件兼容就改用 Canvas 走 WebM / GIF。

### Step 3：保存并预览

1. 保存为 HTML 文件：`{workspace}/generated-images/{主题}/canvas-{简短描述}.html`
2. 自动打开浏览器预览
3. 告知用户可用的导出按钮：
   - 位图模式：「📸 保存图片」→ PNG
   - 矢量模式：「🧬 导出 SVG」→ SVG（首选），「📸 保存 PNG」→ PNG
   - 动画内容：另加「🎥 导出 WebM」「🎬 导出 GIF」

### Step 4：迭代优化

如果用户对结果不满意，根据反馈调整：
- "颜色太亮了" → 调整配色
- "加一个 XXX" → 修改绘制代码
- "放大一点" → 调整尺寸或缩放

## 生成示例

**用户说**：帮我画一个卡通风格的太阳

**AI 生成的代码要点**：
```javascript
// 太阳主体 - 渐变圆形
const gradient = ctx.createRadialGradient(400, 300, 50, 400, 300, 100);
gradient.addColorStop(0, '#FFD700');
gradient.addColorStop(1, '#FF6B00');
ctx.fillStyle = gradient;
ctx.beginPath();
ctx.arc(400, 300, 100, 0, Math.PI * 2);
ctx.fill();

// 光芒
for (let i = 0; i < 12; i++) {
  const angle = (i / 12) * Math.PI * 2;
  ctx.beginPath();
  ctx.moveTo(400 + 110 * Math.cos(angle), 300 + 110 * Math.sin(angle));
  ctx.lineTo(400 + 150 * Math.cos(angle), 300 + 150 * Math.sin(angle));
  ctx.strokeStyle = '#FFD700';
  ctx.lineWidth = 8;
  ctx.lineCap = 'round';
  ctx.stroke();
}
```

**用户说**：画一个极简风格的架构图，有前端、后端、数据库

**AI 生成的代码要点**：
```javascript
// 三个方块 + 箭头连接
drawBox(300, 50, 200, 60, '#4CAF50', '前端');
drawBox(300, 200, 200, 60, '#2196F3', '后端');
drawBox(300, 350, 200, 60, '#FF9800', '数据库');
drawArrow(400, 110, 400, 200);
drawArrow(400, 260, 400, 350);
```

**用户说**：给我一个放大镜图标，要矢量的

**AI 动作**：推断输出格式 = SVG（用户明说"矢量"），不打扰用户，直接生成矢量模式代码：
```javascript
const defs = el('defs', {});
const grad = el('linearGradient', {id:'g', x1:0, y1:0, x2:1, y2:1}, defs);
el('stop', {offset:'0%', 'stop-color':'#4facfe'}, grad);
el('stop', {offset:'100%','stop-color':'#00f2fe'}, grad);

const g = el('g', {stroke:'url(#g)', fill:'none', 'stroke-linecap':'round'});
el('circle', {cx:340, cy:340, r:190, 'stroke-width':56}, g);   // 镜片
el('line',   {x1:478, y1:478, x2:640, y2:640, 'stroke-width':72}, g); // 手柄
el('circle', {cx:340, cy:340, r:150, fill:'#EAF6FF', stroke:'none'}, g); // 高光
```
→ 页面上「🧬 导出 SVG」导出矢量图，「📸 保存 PNG」出高清位图。

**用户说**：需求没说清时（比如只说"做个东西"）

**AI 动作**：先问输出格式（Step 1 的四个选项：PNG / SVG / WebM / GIF），拿到答案再画。

## 适用场景

- ✅ 流程图、架构图、示意图（**推荐 SVG**）
- ✅ 数据可视化（柱状图、饼图、折线图）（**推荐 SVG**）
- ✅ 图标、Logo、简单插画（**推荐 SVG**）
- ✅ 信息图、对比图（**推荐 SVG**）
- ✅ 卡通形象、扁平插画（**推荐 SVG**）
- ✅ 公众号配图、社交媒体图片
- ✅ **动画可视化**（算法演示、数据流动、交互过程）
- ✅ **矢量素材**：UI 图标、App 图标、字体图标、需要放大或改色的交付物
- ✅ 任何你能用文字描述的图

## 不适用场景

- ❌ 照片级真实图像（用文生图模型）
- ❌ 复杂的人物肖像（用文生图模型）
- ❌ 需要大量细节的场景图（用文生图模型）
- ❌ 大量粒子 / 噪点 / 纹理细节（别走 SVG，用 Canvas 出 PNG）

## 输出规范

- 文件格式：HTML（自包含，无外部依赖）
- 文件名：`canvas-{简短描述}.html`
- 保存位置：`{workspace}/generated-images/{主题}/`
- 导出方式：
  - 静态位图：页面内置"保存图片"按钮，点击即导出 PNG（HiDPI 高清）
  - 静态矢量图：页面内置"导出 SVG"按钮，点击即导出 .svg（自带"保存 PNG"兜底）
  - 动画 WebM：页面内置"导出 WebM"按钮，vp9.0 编码，15Mbps，30fps
  - 动画 GIF：页面内置"导出 GIF"按钮，NeuQuant + LZW 编码，20fps
- 预览方式：自动打开浏览器

## SVG 导出技术细节

### 为什么 SVG 是独立模式
Canvas 画完就是位图，没法凭空变回矢量。所以要矢量就直接画 SVG（矢量元素本身就是"代码作画"），而不是在 Canvas 上画完再想办法转。

### 画布与坐标
- 画布是内联 `<svg>`，`viewBox="0 0 W H"` 定义逻辑坐标系，后续绘制代码全部用这套逻辑坐标
- 不需要 HiDPI 处理：矢量天生无限清晰，`width/height` 只决定页面上显示多大

### 导出 SVG
- `XMLSerializer` 序列化 SVG DOM，前面拼上 XML 声明，`Blob` + `createObjectURL` 下载
- 克隆节点时补齐 `xmlns` / `xmlns:xlink` / `width` / `height`，去掉页面展示用的 `style`，保证在别的软件里打开尺寸正确

### 导出 PNG（SVG 光栅化）
- 序列化后的 SVG 字符串 → `Blob` → `Image` → `drawImage` 到离屏 canvas
- 离屏 canvas 尺寸 = `W*dpr × H*dpr`，`ctx.scale(dpr, dpr)` 后按 `W × H` 绘制，和位图模式一样拿到 HiDPI 高清
- **canvas 污染陷阱**：SVG 里只要引用了外部资源（外链图片、web 字体、外链 filter），`toDataURL` 就会抛错。所以 SVG 必须完全自包含；导出时用 try/catch 兜底提示用户改用 SVG 文件

### 自包含红线
| 不要 | 改成 |
|------|------|
| `<image href="https://...">` | 用矢量图形画，或 base64 data URI 内嵌 |
| `@import` web 字体 | 系统字体族，或把文字转 `<path>` |
| CSS `filter: blur()` | SVG `<filter>` + `feGaussianBlur` / `feDropShadow` |
| 外链 `<use xlink:href="...">` | 同文档 `<defs>` + `<use href="#id">` |

### 动画 SVG
- 用 SMIL：`<animate>` / `<animateTransform>` / `<animateMotion>`，`repeatCount="indefinite"` 或固定次数
- SMIL 会随 SVG DOM 一起序列化导出，浏览器打开 .svg 仍在播
- 兼容性：Chrome / Edge / Firefox / Safari 都支持 SMIL；PowerPoint、Illustrator、部分设计工具不支持 —— 这些场景改用 Canvas 走 WebM / GIF

关键函数（矢量模式）：
- `el(tag, attrs, parent)`：矢量元素工厂
- `serializeSvg()`：序列化 SVG DOM
- `exportSvg()`：导出 .svg
- `saveImage()`：SVG 光栅化导出 PNG


## 动画导出技术细节

### WebM 导出（高质量视频）
- 编码：vp9.0（Profile 0，无 alpha 通道）
- 码率：15Mbps（videoBitsPerSecond: 15000000）
- 帧率：30fps（canvas.captureStream(30)）
- 分辨率：跟随 canvas 物理像素（HiDPI 下是 CSS 尺寸的 2 倍）

### GIF 导出（动画图）
- 采样：直接从 canvas 逐帧抓取（requestAnimationFrame），不经 WebM 中转
- 帧率：20fps
- 颜色量化：NeuQuant 神经网络算法，每帧独立 256 色调色板
- 压缩：LZW 编码，数字字符串键（避免 String.fromCharCode 的 null 字符 bug）
- RGBA→RGB：抓帧后立即剥离 alpha 通道再传给 NeuQuant

关键函数：
- `window.resetAnimation()`：重置动画到第一帧（录制前调用）
- `window.getAnimationDuration()`：返回动画周期秒数（录制时长）
- `exportWebm()`：WebM 录制导出
- `exportGif()`：GIF 录制导出
- `encodeGIF(w, h, frames, delay)`：GIF 编码器
- `NeuQuant(pixels, samplefac)`：NeuQuant 颜色量化
- `lzwEncode(out, pixels, minCodeSize)`：LZW 压缩

## 开源信息

- 仓库：https://github.com/ComfortableLiu/canvas-image-gen
- License：MIT
