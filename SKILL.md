---
name: haoone-skill
description: 使用此技能可以调用 haoone-cli 处理字幕工作流，基于 qwen3-asr 包括转录音视频、批量转录、查看已安装模型、文稿匹配、格式化文稿、管理热词、读取配置、查看项目、创建项目、删除项目，翻译、双语字幕等。
---

# Haoone Skill

## 使用 skill 的前置条件

1 安装了 [haoone](https://www.haoai.pro/haoone/download) 软件，并完成登录
2 在软件中下载本地模型
3 点击软件的设置按钮，启动命令行工具

先确认命令可用：

```bash
haoone-cli --help
```

如果失败，再检查：

```bash
which haoone-cli
```

2. 先确认本地模型已安装：

```bash
haoone-cli installed-models
```

3. 实际执行时，直接使用对应子命令：

```bash
haoone-cli transcribe --audio-file-path /path/to/media.mp4
```

4. 如果用户只说“帮我转字幕”或“转录这个视频”，优先使用：

- 单文件：`transcribe`
- 多文件：`batch-transcribe`

5. 如果用户需要知道有哪些命令，或要求封装全部能力，覆盖以下全部子命令：

- `transcribe`：转录单个音频或视频
- `installed-models`：列出所有已安装模型
- `manuscript-matching`：文稿匹配
- `format-draft`：格式化文稿
- `batch-transcribe`：批量转录媒体文件
- `batch-add-hotwords`：批量添加热词
- `get-hotwords`：读取热词配置
- `get-config`：读取软件配置
- `get-project-list`：读取项目列表
- `get-project-srt-list`：读取项目下的 SRT 文件
- `create-project`：创建项目并切换为当前项目
- `delete-project`：删除项目

6. 除了 `haoone-cli` 原生命令，还要支持一条这个 skill 自定义的工作指令：

- 读取 `.srt` 文件内容
- 基于字幕内容生成结构化文稿
- 文稿输出优先使用中文
- 不改写原意，不凭空补充事实

## 命令概览

查看某个命令的帮助：

```bash
haoone-cli transcribe --help
```

当用户没有明确说用哪个子命令时，按下面方式判断：

- 用户要把音频或视频转成字幕：`transcribe`
- 用户一次处理多个媒体文件：`batch-transcribe`
- 用户想确认模型是否装好：`installed-models`
- 用户想把已有文稿整理得更适合阅读：`format-draft`
- 用户要把文稿和已有转录结果对齐：`manuscript-matching`
- 用户要维护热词：`batch-add-hotwords` 或 `get-hotwords`
- 用户要查看 Haoone 当前状态、项目、配置：`get-config`、`get-project-list`、`get-project-srt-list`
- 用户要管理项目：`create-project`、`delete-project`
- 用户给了一个 `.srt` 文件，希望整理成可阅读文稿：先读取 `.srt`，再执行“结构化文稿生成”

## 常用场景

### 1. 查看本机已安装模型

```bash
haoone-cli installed-models
```

适合先确认本地模型是否已经准备好。

### 2. 转录单个音频或视频

```bash
haoone-cli \
  transcribe \
  --timeline-name demo \
  --audio-file-path /path/to/audio.mp3
```

关键参数：

- `--timeline-name` / `-n`：时间线名称，可不传
- `--audio-file-path` / `-f`：输入音频或视频路径，必传
- `--output` / `-o`：输出目录，可不传
- `--model` / `-m`：模型名，默认 `中英-v2-2026`
- `--language` / `-l`：语言代码，默认 `zh`
- `--enable-ai-correction` / `-a`：是否启用 AI 智能拆行与热词替换
- `--max-subtitle-length` / `-s`：单行字幕最大长度，默认 `25`

示例，本地转录：

```bash
haoone-cli \
  transcribe \
  -n interview01 \
  -f /Users/you/Desktop/interview01.mp3 \
  -m 中英-v2-2026 \
  -l zh
```

示例，启用 AI 智能拆行热词替换：

```bash
haoone-cli \
  transcribe \
  -n interview01 \
  -m 中英-v2-2026 \
  -f /Users/you/Desktop/interview01.mp4 \
  -a true \
  -l zh \
  -s 22
```

输出结果通常包括：

- `*.srt`
- `*.json`

如果没有显式指定 `--output`，默认优先写入当前项目的 `transcriptions` 目录；如果当前没有项目，则写到输入文件同级的 `transcriptions` 目录。

脚本集成或自动化时，重点关注这些标准输出：

```text
media_path=...
segmentLength=...
srt_file_path=...
json_file_path=...
```

### 3. 批量转录多个媒体文件

```bash
haoone-cli \
  batch-transcribe \
  --audio-file-path /path/to/a.mp3,/path/to/b.mp4 \
  --output /path/to/out
```

关键参数：

- `--audio-file-path` / `-f`：输入媒体文件，必传
- `--output` / `-o`：输出目录，可不传
- `--model` / `-m`：模型名，默认 `中英-v2-2026`
- `--language` / `-l`：语言代码，默认 `zh`
- `--enable-online-transcript` / `-w`：是否启用在线转录
- `--enable-ai-correction` / `-a`：是否启用 AI 智能拆行与热词替换，默认通常为 `true`
- `--max-subtitle-length` / `-s`：单行字幕最大长度，默认 `25`

### 4. 文稿格式化

```bash
haoone-cli \
  format-draft \
  -p /path/to/manuscript.txt
```

这个命令会把格式化结果直接打印到终端。如果需要保存文件，可以重定向：

```bash
haoone-cli \
  format-draft \
  -p /path/to/manuscript.txt > /path/to/manuscript.formatted.txt
```

### 5. 文稿匹配

```bash
haoone-cli \
  manuscript-matching \
  --manuscript-path /path/to/manuscript.txt \
  -n transcript_name \
  -p project_name
```

关键参数：

- `--manuscript-path`：文稿文件路径，必传
- `--name` / `-n`：转录名称，必传
- `--project` / `-p`：项目名称，默认当前项目
- `--enable-splite-follows-script` / `-f`：是否按文稿拆行

注意：文稿匹配仅限激活用户使用。

### 6. 获取软件配置信息

```bash
haoone-cli get-config
```

适合读取应用数据目录、当前项目、激活状态、自定义 API 配置等信息。

### 7. 获取项目列表

```bash
haoone-cli get-project-list
```

适合查看项目名、项目路径、更新时间、最近打开时间、是否为当前项目。

### 8. 获取项目下的 SRT 文件列表

```bash
haoone-cli \
  get-project-srt-list \
  -p project_name
```

适合递归查找指定项目目录下的所有 `.srt` 文件路径。

### 9. 创建项目

```bash
haoone-cli \
  create-project \
  -n demo-project
```

### 10. 删除项目

```bash
haoone-cli \
  delete-project \
  -n demo-project
```

如果不传 `--name`，程序会尝试删除当前项目。

### 11. 批量添加热词

直接传字符串：

```bash
haoone-cli \
  batch-add-hotwords \
  -l zh \
  --input "OpenAI=欧喷AI,Open A I"
```

从文件读取：

```bash
haoone-cli \
  batch-add-hotwords \
  -l zh \
  -f /path/to/hotwords.txt
```

规则：

- `--language` / `-l` 必传
- `--file` 和 `--input` 至少传一个
- 两者同时传时，优先读取 `--file`

支持两种输入格式：

```text
OpenAI=欧喷AI,Open A I
ChatGPT=chat g p t,Chat GP T
```

或：

```text
OpenAI
欧喷AI
Open A I

ChatGPT
chat g p t
Chat GP T
```

### 12. 获取热词配置

```bash
haoone-cli get-hotwords
```

适合查看语言、热词、别名、拼音、首字母等信息。

### 13. 读取 SRT 并生成结构化文稿

这不是 `haoone-cli` 原生命令，而是这个 skill 额外提供的工作流。

适用场景：

- 用户提供了 `.srt` 文件
- 用户只提供了字幕文件名
- 用户说“帮我整理成文稿”
- 用户说“把字幕转成结构化笔记”
- 用户说“按章节输出内容摘要和正文”

执行要求：

1. 如果用户直接提供了 `.srt` 路径，先读取该文件全文
2. 如果用户只提供文件名，先调用 `haoone-cli get-project-srt-list` 找到匹配的 `.srt` 路径
3. 找到目标 `.srt` 后，读取文件全文
4. 去掉序号、时间轴等字幕控制信息
5. 按语义合并连续字幕，恢复成自然段
6. 根据内容组织结构化文稿
7. 默认输出中文
8. 不要杜撰原文中不存在的信息

当用户只给文件名时，按下面顺序处理：

1. 先执行 `haoone-cli get-project-srt-list`
2. 如果用户明确给了项目名，使用 `haoone-cli get-project-srt-list -p 项目名`
3. 如果返回多个同名或近似文件，优先选择与文件名完全匹配的路径
4. 如果仍有多个候选，向用户明确展示候选路径并请求确认
5. 如果没有找到，明确告知未找到对应 `.srt` 文件

示例：

```bash
haoone-cli get-project-srt-list
```

或：

```bash
haoone-cli get-project-srt-list -p demo-project
```

推荐输出结构：

- 标题
- 内容概览
- 结构化正文
- 关键要点

如果内容明显具有章节或话题切换，结构化正文应拆成多个小节；如果内容只是连续口播，至少整理为若干自然段，不要原样逐条保留字幕编号。

## 输出处理

执行转录命令时，把下面这些 stdout 行作为最终结果来源：

```text
srt_file_path=...
json_file_path=...
```

如果 `--output` 未传，由 `haoone-cli` 决定输出目录：

- 当前项目存在时，优先写入当前项目的 `transcriptions` 目录
- 否则写入输入媒体同级的 `transcriptions` 目录

## 故障排查

- `haoone-cli: command not found`
  先执行 `haoone-cli --help` 和 `which haoone-cli`。如果 `which` 没有输出，通常是 PATH 未生效、终端未重开，或者 CLI 没有执行权限。
- 没有安装模型
  先执行 `haoone-cli installed-models`，再去 Haoone 桌面端安装本地模型。
- 提示未登录、未激活、找不到当前项目、在线转录失败
  先打开 Haoone 桌面端，确认已经登录、激活，并且当前项目可以正常打开。
- 输出文件找不到
  先检查显式传入的 `--output`，再检查当前项目目录下的 `transcriptions`，最后检查输入文件同级的 `transcriptions`。同时直接看 stdout 中的 `srt_file_path=` 和 `json_file_path=`。
- Windows 下命令不可用
  优先检查 `haoone-cli.exe` 是否实际存在、其目录是否已加入 PATH，并完全关闭后重新打开终端。
- `.srt` 文稿整理效果差
  先检查字幕是否包含大量识别错误、断句异常或时间轴混乱。必要时先重新转录，或先做人工校对后再生成结构化文稿。
- 只给文件名但找不到 `.srt`
  先执行 `haoone-cli get-project-srt-list` 或 `haoone-cli get-project-srt-list -p 项目名`，确认项目里是否真的存在对应文件名。

## 参考

- 完整命令参考：`references/haoone-cli.md`
