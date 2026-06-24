---
name: canvas-image-gen
description: 用纯文本模型的代码能力，通过 HTML Canvas 代码级作画生成图像。用户描述想要的图，AI 生成 HTML+Canvas+JS 代码，浏览器打开即可查看，点击按钮可保存为 PNG、WebM 或 GIF（动画）。就像文生图一样，但用代码画。
version: 3.1.0
author: 说人话的实验室
tags: [canvas, 图像生成, 代码作画, html, 文生图, 可视化, gif, webm, 动画, 高清]
---

# Canvas Image Gen

用纯文本模型的代码能力，通过 HTML Canvas 代码级作画生成图像。

就像文生图模型一样——用户描述想要的图，AI 帮你画出来。只不过"画笔"是代码，"画布"是 HTML Canvas。

支持三种输出：
- **静态图**：导出 PNG（默认，HiDPI 高清）
- **动画视频**：导出 WebM（vp9 编码，15Mbps，30fps，无 alpha，最高画质）
- **动画图**：导出 GIF（NeuQuant 颜色量化 + LZW 压缩，20fps）

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
- **必须 HiDPI 适配**：canvas 物理尺寸 = CSS 尺寸 × devicePixelRatio，ctx.scale(dpr, dpr)
- **必须包含"保存图片"按钮**：点击后将 Canvas 导出为 PNG 并自动下载
- **动画内容必须包含"导出 WebM"和"导出 GIF"按钮**
- **按钮位置规则**：按钮必须放在画布以外，使用固定定位（position: fixed）悬浮在页面角落，不能遮挡或影响画布内容

**HiDPI 适配（必须）：**
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

**动画导出要求（仅当内容有动画时）：**
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
  - 静态图：页面内置"保存图片"按钮，点击即导出 PNG（HiDPI 高清）
  - 动画 WebM：页面内置"导出 WebM"按钮，vp9.0 编码，15Mbps，30fps
  - 动画 GIF：页面内置"导出 GIF"按钮，NeuQuant + LZW 编码，20fps
- 预览方式：自动打开浏览器

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
