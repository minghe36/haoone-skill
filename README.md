# haoone-skill

`haoone-skill` 使用此技能可以调用 haoone-cli 处理字幕工作流，基于 qwen3-asr 包括转录音视频、批量转录、查看已安装模型、文稿匹配、格式化文稿、管理热词、读取配置、查看项目、创建项目、删除项目，翻译、双语字幕等。

haoone 是新一代的 AI 专业字幕软件，此 skill 为软件配套的技能，可以免费使用。
haoone 软件介绍：[https://www.haoai.pro/haoone](https://www.haoai.pro/haoone)
haoone-cli 命令行工具介绍：[https://guide.haoai.pro/guide/haoone/%E4%BD%BF%E7%94%A8haoone-cli%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7](https://guide.haoai.pro/guide/haoone/%E4%BD%BF%E7%94%A8haoone-cli%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7)


添加 skill ：

``` text
npx skills add minghe36/haoone-skill
```

## 功能范围

当前支持以下 `haoone-cli` 子命令：

- `transcribe`：转录单个音频或视频
- `installed-models`：列出所有已安装模型
- `manuscript-matching`：文稿匹配
- `format-draft`：格式化文稿
- `batch-transcribe`：批量转录媒体文件
- `batch-add-hotwords`：批量添加热词
- `get-hotwords`：读取热词配置
- `get-config`：读取软件关键配置信息
- `get-project-list`：读取项目列表
- `get-project-srt-list`：读取指定项目下的 `.srt` 文件路径
- `create-project`：创建项目并切换为当前项目
- `delete-project`：删除项目

额外支持一个自定义工作流：

- 读取 `.srt` 文件内容
- 清理序号、时间轴等控制信息
- 合并连续字幕
- 生成中文结构化文稿

## 目录结构

```text
haoone-skill/
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    └── haoone-cli.md
```

## 使用方式

### 1. 直接调用 haoone-cli

示例，转录单个文件：

```bash
haoone-cli transcribe --audio-file-path /path/to/media.mp4
```

示例，批量转录：

```bash
haoone-cli batch-transcribe -f /path/to/a.mp3,/path/to/b.mp4 -l zh
```

示例，查看已安装模型：

```bash
haoone-cli installed-models
```

### 2. 读取 SRT 并生成结构化文稿

这个工作流不是 `haoone-cli` 原生命令，而是 skill 的扩展约定。

适用场景：

- 用户直接提供 `.srt` 文件路径
- 用户只提供字幕文件名
- 用户要求“把字幕整理成文稿”
- 用户要求“把 srt 转成结构化笔记”

处理规则：

1. 如果用户提供 `.srt` 完整路径，直接读取文件内容
2. 如果用户只提供文件名，先调用 `haoone-cli get-project-srt-list`
3. 如果用户同时提供项目名，优先调用 `haoone-cli get-project-srt-list -p 项目名`
4. 匹配出对应的 `.srt` 路径后，读取文件内容
5. 删除字幕编号和时间轴
6. 合并连续字幕并恢复自然段
7. 生成结构化文稿

输出要求：

- 默认输出中文
- 不杜撰原文没有的信息
- 内容有明显主题切换时，按小节组织
- 输出中不要保留原始 `.srt` 编号和时间轴

推荐输出结构：

- 标题
- 内容概览
- 结构化正文
- 关键要点

## 依赖前提

使用本 skill 前，请先确认：

- 已安装 Haoone 桌面端
- 已登录 Haoone 账号
- 已启用 `haoone-cli`
- 当前终端里可以直接执行 `haoone-cli --help`
- 本地模型已安装完成

建议先检查：

```bash
haoone-cli --help
haoone-cli installed-models
```

## 常见排查

### 1. 命令找不到

先执行：

```bash
which haoone-cli
```

如果没有输出，通常是：

- PATH 未生效
- 终端未重开
- CLI 没有执行权限

### 2. 模型不可用

先执行：

```bash
haoone-cli installed-models
```

如果没有模型，先去 Haoone 桌面端安装本地模型。

### 3. 找不到输出文件

优先检查：

1. 显式传入的 `--output` 目录
2. 当前项目目录下的 `transcriptions`
3. 输入文件同级目录下的 `transcriptions`

转录完成后，重点看 stdout：

```text
srt_file_path=...
json_file_path=...
```

### 4. 只给文件名但找不到 srt

先执行：

```bash
haoone-cli get-project-srt-list
```

如果用户知道项目名，则执行：

```bash
haoone-cli get-project-srt-list -p project_name
```

如果匹配到多个候选路径，不应擅自猜测，应让用户确认目标文件。

## 说明

仓库中的核心文件：

- [SKILL.md](./SKILL.md)：skill 主说明
- [agents/openai.yaml](./agents/openai.yaml)：skill UI 元数据
- [references/haoone-cli.md](./references/haoone-cli.md)：完整命令参考
