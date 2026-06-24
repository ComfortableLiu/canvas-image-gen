---
name: canvas-image-gen
description: 用纯文本模型的代码能力，通过 HTML Canvas 代码级作画生成图像。用户描述想要的图，AI 生成 HTML+Canvas+JS 代码，浏览器打开即可查看，点击按钮可保存为 PNG 或 GIF（动画）。就像文生图一样，但用代码画。
version: 3.0.0
author: 说人话的实验室
tags: [canvas, 图像生成, 代码作画, html, 文生图, 可视化, gif, 动画]
---

# Canvas Image Gen

用纯文本模型的代码能力，通过 HTML Canvas 代码级作画生成图像。

就像文生图模型一样——用户描述想要的图，AI 帮你画出来。只不过"画笔"是代码，"画布"是 HTML Canvas。

支持两种输出：
- **静态图**：导出 PNG（默认）
- **动画**：导出 GIF（当内容有动画时自动支持）

## 核心原理

```
用户："帮我画一个红色的太阳在蓝色天空上"
         ↓
AI 生成 HTML+Canvas+JS 代码
         ↓
保存为 HTML 文件，浏览器打开
         ↓
点击"保存图片"按钮 → 导出 PNG
点击"导出 GIF"按钮 → 录制动画 → 导出 GIF（仅动画内容）
```

## 使用方法

用户说"帮我画一个 XXX"或"生成一张 XXX 的图"，AI 按以下流程执行：

### Step 1：理解需求

从用户描述中提取：
- **画什么**：主体元素、场景、构图
- **风格**：扁平/卡通/写实/极简/像素风（默认扁平风）
- **配色**：用户指定或根据主题自选
- **尺寸**：用户指定或根据用途自选（默认 800x600）

### Step 2：生成 HTML 代码

生成完整的 HTML+Canvas+JS 代码。代码必须满足：

**基础要求：**
- 自包含：所有样式和脚本都在 HTML 文件内，不依赖外部资源
- 高分屏适配：使用 `window.devicePixelRatio`
- **必须包含"保存图片"按钮**：点击后将 Canvas 导出为 PNG 并自动下载
- **动画内容必须包含"导出 GIF"按钮**：录制动画并导出为 GIF
- **按钮位置规则**：按钮必须放在画布以外，使用固定定位（position: fixed）悬浮在页面角落，不能遮挡或影响画布内容

**动画导出要求（仅当内容有动画时）：**
- 必须定义 `window.resetAnimation` 函数：将动画重置到第一帧
- 必须定义 `window.getAnimationDuration` 函数：返回动画周期（秒）
- 导出流程：重置动画 → 录制一个完整周期 → 解码视频帧 → NeuQuant 颜色量化 → LZW 编码 GIF
- 使用 MediaRecorder API 录制 canvas stream，不依赖外部库

**画风指导：**
- 优先使用扁平化设计（flat design），简洁干净
- 颜色鲜明，对比度高
- 形状清晰，边缘锐利
- 适当使用渐变增加层次感
- 文字清晰可读（如果有文字的话）

**代码结构（静态图）：**
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<style>
  body { margin: 0; display: flex; align-items: center; justify-content: center; min-height: 100vh; background: #f0f0f0; font-family: -apple-system, sans-serif; }
  canvas { border-radius: 8px; box-shadow: 0 4px 20px rgba(0,0,0,0.1); }
  .save-btn { position: fixed; bottom: 24px; right: 24px; padding: 10px 24px; background: #378ADD; color: white; border: none; border-radius: 6px; font-size: 14px; cursor: pointer; box-shadow: 0 2px 12px rgba(0,0,0,0.15); z-index: 100; }
  .save-btn:hover { background: #2a6db8; }
</style>
</head>
<body>
<canvas id="canvas"></canvas>
<button class="save-btn" onclick="saveImage()">保存图片</button>
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

**代码结构（带动画）：**
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
<canvas id="canvas"></canvas>
<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const W = 800, H = 600;

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

<!-- 导出按钮 -->
<div id="saveBtn" onclick="saveImage()" style="position:fixed;bottom:16px;right:16px;z-index:9999;background:#378ADD;color:#fff;border:none;border-radius:8px;padding:10px 20px;font-size:14px;cursor:pointer;">📸 保存 PNG</div>
<div id="gifExportBtn" onclick="exportGif()" style="position:fixed;bottom:16px;right:130px;z-index:9999;background:linear-gradient(135deg,#f093fb,#f5576c);color:#fff;border:none;border-radius:8px;padding:10px 20px;font-size:14px;cursor:pointer;">🎬 导出 GIF</div>
<div id="gifProgress" style="position:fixed;bottom:56px;right:130px;z-index:9999;background:rgba(0,0,0,0.8);color:#fff;padding:8px 16px;border-radius:8px;font-size:12px;display:none;">准备中...</div>
<script>
function saveImage() {
  const link = document.createElement('a');
  link.download = 'canvas-image.png';
  link.href = canvas.toDataURL('image/png');
  link.click();
}

async function exportGif() {
  const btn = document.getElementById('gifExportBtn');
  const prog = document.getElementById('gifProgress');
  btn.style.opacity = '0.5'; btn.style.pointerEvents = 'none';
  prog.style.display = 'block';
  if (typeof window.resetAnimation === 'function') window.resetAnimation();
  const animDur = typeof window.getAnimationDuration === 'function' ? window.getAnimationDuration() : 3;
  prog.textContent = '录制中（' + animDur + '秒）...';
  try {
    const stream = canvas.captureStream(10);
    const recorder = new MediaRecorder(stream, {mimeType: 'video/webm;codecs=vp8'});
    const chunks = [];
    recorder.ondataavailable = e => { if (e.data.size > 0) chunks.push(e.data); };
    const webmBlob = await new Promise((resolve, reject) => {
      recorder.onstop = () => resolve(new Blob(chunks, {type: 'video/webm'}));
      recorder.onerror = reject;
      recorder.start();
      setTimeout(() => recorder.stop(), animDur * 1000);
    });
    prog.textContent = '解码视频帧...';
    const video = document.createElement('video');
    video.muted = true;
    video.src = URL.createObjectURL(webmBlob);
    await new Promise(r => { video.onloadedmetadata = r; });
    await new Promise(r => { video.oncanplay = r; });
    const fps = 10, totalFrames = Math.min(Math.ceil(video.duration * fps), 50);
    const tmpCanvas = document.createElement('canvas');
    tmpCanvas.width = W; tmpCanvas.height = H;
    const tmpCtx = tmpCanvas.getContext('2d');
    const frames = [];
    for (let i = 0; i < totalFrames; i++) {
      video.currentTime = i / fps;
      await new Promise(r => { video.onseeked = r; });
      tmpCtx.drawImage(video, 0, 0, W, H);
      frames.push(new Uint8Array(tmpCtx.getImageData(0, 0, W, H).data));
      prog.textContent = '解码帧 ' + (i+1) + '/' + totalFrames + '...';
    }
    URL.revokeObjectURL(video.src);
    prog.textContent = '编码GIF...';
    // NeuQuant + LZW 编码（完整代码见 GitHub）
    // https://github.com/ComfortableLiu/canvas-image-gen
    const gif = encodeGIF(W, H, frames, 100);
    const blob = new Blob([gif], {type: 'image/gif'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.download = 'animation.gif';
    a.href = url;
    a.click();
    URL.revokeObjectURL(url);
    prog.textContent = '✅ 导出完成！';
  } catch(e) { prog.textContent = '❌ ' + e.message; }
  setTimeout(() => { prog.style.display = 'none'; }, 2000);
  btn.style.opacity = '1'; btn.style.pointerEvents = 'auto';
}

// encodeGIF + NeuQuant + lzwEncode 完整代码：
// https://github.com/ComfortableLiu/canvas-image-gen
</script>
</body>
</html>
```

### Step 3：保存并预览

1. 保存为 HTML 文件：`{workspace}/generated-images/{主题}/canvas-{简短描述}.html`
2. 自动打开浏览器预览
3. 告知用户：点击页面上的"保存图片"按钮即可导出 PNG

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

## 适用场景

- ✅ 流程图、架构图、示意图
- ✅ 数据可视化（柱状图、饼图、折线图）
- ✅ 图标、Logo、简单插画
- ✅ 信息图、对比图
- ✅ 卡通形象、扁平插画
- ✅ 公众号配图、社交媒体图片
- ✅ **动画可视化**（算法演示、数据流动、交互过程）
- ✅ 任何你能用文字描述的图

## 不适用场景

- ❌ 照片级真实图像（用文生图模型）
- ❌ 复杂的人物肖像（用文生图模型）
- ❌ 需要大量细节的场景图（用文生图模型）

## 输出规范

- 文件格式：HTML（自包含，无外部依赖）
- 文件名：`canvas-{简短描述}.html`
- 保存位置：`{workspace}/generated-images/{主题}/`
- 导出方式：
  - 静态图：页面内置"保存图片"按钮，点击即导出 PNG
  - 动画：页面内置"导出 GIF"按钮，录制一个完整动画周期后导出 GIF
- 预览方式：自动打开浏览器

## 动画导出技术细节

GIF 导出流程：
1. **录制**：使用 `canvas.captureStream(10)` + `MediaRecorder` 录制 WebM
2. **解码**：用 `<video>` 元素回放 WebM，逐帧抓取到临时 canvas
3. **量化**：使用 NeuQuant 神经网络算法将颜色量化到 256 色
4. **编码**：使用 LZW 压缩算法编码 GIF

关键函数：
- `window.resetAnimation()`：重置动画到第一帧（录制前调用）
- `window.getAnimationDuration()`：返回动画周期秒数（录制时长）
- `encodeGIF(w, h, frames, delay)`：GIF 编码器
- `NeuQuant(pixels, samplefac)`：NeuQuant 颜色量化
- `lzwEncode(out, pixels, minCodeSize)`：LZW 压缩

## 开源信息

- 仓库：https://github.com/ComfortableLiu/canvas-image-gen
- License：MIT
