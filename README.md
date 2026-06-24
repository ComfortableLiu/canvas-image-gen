# Canvas Image Gen

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

用纯文本模型的代码能力，通过 HTML Canvas 代码级作画生成图像。

就像文生图模型一样——用户描述想要的图，AI 帮你画出来。只不过"画笔"是代码，"画布"是 HTML Canvas。

## 简介

这个 Skill 让你可以用自然语言描述想要的图，AI 会生成 HTML+Canvas+JS 代码，浏览器打开即可查看，点击按钮可保存为 PNG、WebM 或 GIF（动画）。

**不需要图像生成 API**，任何能写代码的 AI 模型都能做。

## 使用方法

跟 AI 说：
```
帮我画一个卡通风格的太阳
```

AI 会生成 HTML 代码并保存为文件，浏览器打开后点击按钮导出：
- **静态图**：点击「📸 保存图片」→ 导出 PNG（HiDPI 高清）
- **动画视频**：点击「🎥 导出 WebM」→ 导出 WebM（vp9.0，15Mbps，30fps）
- **动画图**：点击「🎬 导出 GIF」→ 导出 GIF（NeuQuant + LZW，20fps）

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

## 特性

- **零依赖**：不依赖任何图像生成 API
- **精确控制**：每一个像素都在代码控制之下
- **可修改**：生成的代码可以随时调整
- **自包含**：HTML 文件，任何浏览器都能打开
- **HiDPI 高清**：自动适配 Retina 屏，canvas 物理分辨率 = CSS 尺寸 × devicePixelRatio
- **一键导出 PNG**：内置"保存图片"按钮，点击即导出
- **高质量 WebM**：vp9.0 编码，15Mbps 码率，30fps，无 alpha 通道
- **动画导出 GIF**：直接从 canvas 抓帧，NeuQuant 颜色量化 + LZW 压缩，20fps
- **动画时间自适应**：录制时长自动跟随动画周期，从第一帧录到最后一帧

## 安装

### 方式 1：直接下载

下载 `SKILL.md` 文件，放到你的 AI 助手的 Skills 目录中。

### 方式 2：Git 克隆

```bash
git clone https://github.com/ComfortableLiu/canvas-image-gen.git
```

## 如何使用

1. 将 `SKILL.md` 放到你的 AI 助手的 Skills 目录
2. 跟 AI 说"帮我画一个 XXX"
3. AI 会生成 HTML 文件并自动打开浏览器
4. 点击页面上的按钮导出：
   - 「📸 保存图片」→ 导出 PNG
   - 「🎥 导出 WebM」→ 导出高质量 WebM 视频（动画内容）
   - 「🎬 导出 GIF」→ 导出 GIF 动画（动画内容）

## 工作原理

```
用户描述想要的图
       ↓
AI 生成 HTML+Canvas+JS 代码（HiDPI 适配）
       ↓
保存为 HTML 文件，浏览器打开
       ↓
静态图 → 点击"保存图片" → 导出 PNG（高清）
动画  → 点击"导出 WebM" → MediaRecorder 录制 → 导出 WebM（vp9.0, 15Mbps, 30fps）
动画  → 点击"导出 GIF"  → canvas 抓帧 → NeuQuant 量化 → LZW 编码 → 导出 GIF（20fps）
```

## 动画导出技术细节

### HiDPI 适配
Canvas 物理像素 = CSS 尺寸 × devicePixelRatio。在 Retina 屏（dpr=2）上，900×350 的 canvas 实际渲染 1800×700 像素，通过 ctx.scale(dpr, dpr) 让绘图代码继续使用逻辑坐标。

### WebM 导出（高质量视频）
- 编码：vp9.0（Profile 0，无 alpha 通道）
- 码率：15Mbps
- 帧率：30fps
- 分辨率：跟随 canvas 物理像素（Retina 下 1800×700）

### GIF 导出（动画图）
- 采样：直接从 canvas 逐帧抓取（requestAnimationFrame），不经 WebM 中转
- 帧率：20fps
- 颜色量化：NeuQuant 神经网络算法，每帧独立 256 色调色板
- 压缩：LZW 编码
- RGBA→RGB：抓帧后剥离 alpha 通道

动画文件需要实现两个接口：
- `window.resetAnimation()`：重置动画到第一帧
- `window.getAnimationDuration()`：返回动画周期（秒）

## 示例

**静态图**：帮我画一个卡通风格的太阳

→ AI 生成一个包含太阳图形的 HTML 文件，点击按钮保存为 PNG。

**动画**：画一个展示二分查找过程的动画

→ AI 生成一个带动画的 HTML 文件，展示查找范围逐步缩小的过程，点击按钮可导出为 WebM 或 GIF。

**架构图**：画一个微服务架构图

→ AI 生成一个包含前端、后端、数据库等模块的架构图 HTML 文件。

## 贡献

欢迎提交 Issue 和 Pull Request！

## 开源协议

MIT License

## 作者

[说人话的实验室](https://github.com/ComfortableLiu)
