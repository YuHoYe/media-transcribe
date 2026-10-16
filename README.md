# media-transcribe

为 Codex、Claude Code 等支持 Agent Skills 的助手提供本地音视频转录工作流。

输入播客/视频链接或本地文件，输出两份文档：

- `transcript-original.md`：保留原语言的完整转录，附时间戳与转录限制。
- `summary-zh.md`：中文总结，保留关键论据、例子、说话人归因和时间戳。

默认使用本地 Whisper large-v3；Apple Silicon 优先使用 MLX Whisper。检查重复循环、时间轴和内容覆盖后，再基于完整转录总结。视频中的画面证据按需检查；仅处理音轨时会明确标注。

## 安装

```bash
npx skills add YuHoYe/media-transcribe
```

只安装到 Codex，并跳过交互确认：

```bash
npx skills add YuHoYe/media-transcribe --skill media-transcribe --agent codex -y
```

默认安装到当前项目；加 `-g` 安装到用户目录。安装方式见 [Skills CLI 官方说明](https://github.com/vercel-labs/skills)。

## 使用

在助手中输入：

```text
用 media-transcribe 转录这个播客，并生成中文总结：https://example.com/episode
```

```text
用 $media-transcribe 处理本地 interview.mp4，把结果放到 ./transcripts/interview/
```

这是给助手使用的技能，不是通过 npx 直接执行的转录程序。`npx skills add` 安装技能文件；助手随后检查并配置本地 Whisper、FFmpeg，以及远程下载按需使用的 yt-dlp。依赖与命令示例见 [本地 Whisper 指南](references/local-whisper.md)。

首次使用可能需要下载模型。语音识别在本机执行，中文总结由当前助手生成；“本地转录”不代表整个总结过程离线。

技能只交付转录和中文总结；不会写入 Bear、整理 Wiki 或上传媒体。需要字幕时可另外要求输出 SRT/VTT。
