# vinceAI

一个用于中文口播短视频的 Codex Skill，覆盖授权音色配音、粗剪、字幕、真实 B-roll、动效和视频封面。可以从一段口播开始做完整成片，也可以单独返修或做封面。

## 安装

在 Codex 中发送这条指令。

```text
请用 $skill-installer 安装 https://github.com/Vince9933/vinceAI/tree/main/skills/vinceai
```

安装后可在新会话里调用。

```text
用 $vinceai 剪这条口播。字幕每屏一行，配真实产品画面。
成片只要 9:16，另做 3:4 和 4:3 两张纯图文封面。
```

## 工作标准

- 需要不出镜配音时，可用授权参考录音和本地 Qwen3-TTS 生成旁白，再进入剪辑。见 [授权音色配音](skills/vinceai/references/voice-clone.md)。
- 按实听剪掉影响节奏的气口，检查音画接缝和字幕时序。中文字幕每屏一行，最多 12 个可见字符。
- 选和叙述直接相关的实拍、录屏和原文摘图。完整视频按内容挑 4–7 种动效，给真实画面和重要文字留出观看时间。
- 横竖封面分别排版，常用规格为 3:4 和 4:3。默认用深蓝底、白色大字和亮黄色重点，从文案选择一个主题图形，封面不抽帧。做封面时见 [封面设计](skills/vinceai/references/cover-design.md)。

Skill 提供制作和验收流程。运行时使用你当前可访问的剪辑工具、素材和渲染环境。[动效与封面参考](skills/vinceai/references/visual-standards.md)

本仓库原创说明使用 [MIT 许可证](LICENSE)。引用的第三方项目、商标和外部素材遵循各自的使用条件。
