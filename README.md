# haoone-skill

`haoone-skill` 使用此技能可以调用 haoone-cli 处理字幕工作流，基于 qwen3-asr 包括转录音视频、批量转录、查看已安装模型、文稿匹配、格式化文稿、管理热词、读取配置、查看项目、创建项目、删除项目，翻译、双语字幕等。

haoone 是新一代的 AI 专业字幕软件，覆盖剪辑字幕与字幕翻译工作流的所有核心需求，可以永久免费使用基础功能。

haoone 软件官网：[https://www.haoai.pro/haoone](https://www.haoai.pro/haoone)

haoone-skill 是 haoone-cli 命令行工具的封装，你可以根据自己的 AI 工作流需要二次定制，haoone-cli 命令行工具介绍：[https://guide.haoai.pro/guide/haoone/%E4%BD%BF%E7%94%A8haoone-cli%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7](https://guide.haoai.pro/guide/haoone/%E4%BD%BF%E7%94%A8haoone-cli%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7)


添加 skill ：

``` text
npx skills add minghe36/haoone-skill
```

skill 的视频使用教程：[如何使用 haoone-skill 实现 B 站视频内容总结](https://www.bilibili.com/video/BV1z15X6pEyc/)

haoone-skill 的转录准确度与稳定性、词语级对齐吊打 whisper 的套壳转录工具， 默认启用 GPU 加速，自带免费的专业字幕编辑器与快速查错视图，带有热词替换能力。非常适合专业领域视频的内容转录。

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

## 提示词举例

下面这些提示词可以直接作为你在 Codex / ChatGPT / 其他支持 skill 的智能体中的输入参考。

### 转录单个文件

- 帮我用将这个视频转录出字幕：`/path/to/demo.mp4`
- 用 `haoone` 把 `/path/to/interview.wav` 转成字幕
- 转录 `/path/to/lesson.mov`，输出 `srt`

### 批量转录

- 批量转录这个目录下的所有音视频文件：`/path/to/media/`
- 帮我扫描 `/path/to/videos/`，把里面的文件全部转成字幕

### 查看环境和模型

- 帮我检查 `haoone-cli` 是否可用
- 列出当前已安装的模型
- 读取 haoone 的配置给我看看

### 热词管理

- 读取当前haoone的热词配置
- 把这些热词批量加入haoone的热词配置：`cursor=科舍, claude=克劳德`
- 帮我添加2个常用大模型热词，格式：热词=最有可能错误的3个同音词

### 文稿匹配

- 把这份转录文稿做格式化整理
- 用 `format-draft` 处理这段文稿，让标点和分段更自然
- 对这个字幕草稿做文稿匹配
- 用已有文稿和字幕做 `manuscript-matching`

### 项目管理

- 列出当前所有 haoone 项目
- 创建一个项目，名字叫 `播客第12期`
- 删除项目 `测试项目`
- 读取项目 `播客第12期` 下的 `srt` 文件列表

### 组合型提示词

- 先检查 `haoone-cli` 是否可用，再列出已安装模型
- 帮我创建项目 `2026-05-采访稿`，然后批量转录 `/path/to/interview-files/`
- 转录 `/path/to/talk.mp4`，然后把结果做格式化整理

### 使用建议

- 尽量在提示词里明确文件路径、目录路径、项目名
- 如果你已经知道要调用的子命令，可以直接写出，例如：`请执行 haoone-cli transcribe ...`
- 如果你不确定该用哪个命令，也可以直接描述目标，例如：`帮我把这个视频转成字幕`
