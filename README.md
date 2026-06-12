# Canvas Image Gen

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

用纯文本模型的代码能力，通过 HTML Canvas 代码级作画生成图像。

就像文生图模型一样——用户描述想要的图，AI 帮你画出来。只不过"画笔"是代码，"画布"是 HTML Canvas。

## 简介

这个 Skill 让你可以用自然语言描述想要的图，AI 会生成 HTML+Canvas+JS 代码，浏览器打开即可查看，点击按钮可保存为 PNG。

**不需要图像生成 API**，任何能写代码的 AI 模型都能做。

## 使用方法

跟 AI 说：
```
帮我画一个卡通风格的太阳
```

AI 会生成 HTML 代码并保存为文件，浏览器打开后点击"保存图片"按钮即可导出 PNG。

## 适用场景

- ✅ 流程图、架构图、示意图
- ✅ 数据可视化（柱状图、饼图、折线图）
- ✅ 图标、Logo、简单插画
- ✅ 信息图、对比图
- ✅ 卡通形象、扁平插画
- ✅ 公众号配图、社交媒体图片
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
- **一键导出**：内置"保存图片"按钮，点击即导出 PNG

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
4. 点击页面上的"保存图片"按钮导出 PNG

## 工作原理

```
用户描述想要的图
       ↓
AI 生成 HTML+Canvas+JS 代码
       ↓
保存为 HTML 文件，浏览器打开
       ↓
点击"保存图片"按钮 → 导出 PNG
```

## 示例

**用户说**：帮我画一个卡通风格的太阳

**AI 生成**：一个包含太阳图形的 HTML 文件，点击按钮可保存为 PNG。

**用户说**：画一个微服务架构图

**AI 生成**：一个包含前端、后端、数据库等模块的架构图 HTML 文件。

## 贡献

欢迎提交 Issue 和 Pull Request！

## 开源协议

MIT License

## 作者

[说人话的实验室](https://github.com/ComfortableLiu)
