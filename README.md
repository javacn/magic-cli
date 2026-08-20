# magic-cli

**MagicAI 团队**推出的免费命令行工具箱（magic 系列 CLI）—— 一个命令家族，各司其职，装完即用。

---

## 🤖 Agent 一键安装提示词

以下提示词可直接整体复制给任意 AI 助手（Trae / Claude / GPT 等），
让它在你的电脑上自动完成 **magic-wiki** 的安装与配置：

```text
你是一名终端助手。请在用户电脑上安装并配置 magic-wiki（MagicAI 团队 magic-cli 系列的命令行工具，用于网页剪藏、B 站 + 公众号搜索与队列管理）。

步骤：
1. 安装：执行 npm install -g @magicstock/magic-wiki
2. 验证：执行 magic-wiki --version，确认输出版本号
3. 查看配置：执行 magic-wiki config path，确认配置文件位置（默认 ~/.magic-wiki/config.json）
4. 配置飞书凭证（用于剪藏到飞书、搜索发送汇总卡片；用户不需要飞书则跳过）：
   - magic-wiki config set feishu.app_id "<用户的飞书应用 APP ID>"
   - magic-wiki config set feishu.app_secret "<用户的飞书应用 APP SECRET>"
5. 可选配置（按用户需要执行）：
   - magic-wiki config set feishu.wiki_node "<知识库节点 token>"        # 保存到飞书知识库
   - magic-wiki config set video_note.notify_target "<用户 open_id>"     # 飞书通知目标
   - magic-wiki config set markdown.vault_path "~/notes"                 # markdown 保存目录
6. 验证配置：执行 magic-wiki config show（敏感字段自动脱敏）
7. 功能自测：执行 magic-wiki search "测试" --limit 3

完成后向用户汇报：安装版本、配置文件路径、已配置的配置项。
```

---

## 简介

**magic-cli** 是 **MagicAI 团队**推出的免费工具箱，由一系列独立的命令行工具组成：
每个 `magic-*` 解决一类具体需求，通过 npm 全局安装、以二进制形式分发，
不依赖用户手动配置 Python 环境，装完即可在终端使用。

## 系列一览

| 工具 | 状态 | 说明 | 安装 |
|------|------|------|------|
| [magic-wiki](#magic-wiki) | ✅ 已发布 v1.0.3 | 网页剪藏 / 搜索 B 站 + 公众号 / 队列管理 | `npm install -g @magicstock/magic-wiki` |
| magic-video | 🚧 开发中 | 视频内容处理 CLI（敬请期待） | 即将推出 |

---

## magic-wiki

网页内容剪藏工具 CLI —— 剪藏网页 / 搜索 B 站 + 公众号 / 队列管理。

以**二进制形式**（`.so` / `.pyd`）发布，用户安装后看不到任何 Python 源码。

### 特色功能

- **网页剪藏**：一键把网页（公众号文章、知乎、新闻等）渲染为 Markdown，保存到本地或飞书云文档
- **视频一键成笔记**：粘贴视频链接即自动完成「下载 → 语音转写 → 场景抽帧 → 画面理解 → AI 生成结构化笔记 → 保存」全流程，支持 YouTube、Bilibili、Twitter/X 等 yt-dlp 支持的所有平台
- **合集 / 播放列表 / 多 P 支持**：支持 B 站空间合集、B 站多 P 视频、YouTube 播放列表，整部合集自动批量处理并归入统一文件夹
- **聚合搜索**：同时搜索 B 站视频与微信公众号文章，结果合并展示，并支持一键入队
- **队列管理**：后台任务队列，限并发、断点续跑，失败自动重试
- **飞书进度卡片**：处理过程实时推送飞书卡片，分阶段展示进度（下载 / 转写 / 抽帧 / 画面 / 生成 / 保存）
- **配置管理**：统一配置 `~/.magic-wiki/config.json`，敏感字段自动脱敏

### 安装

```bash
npm install -g @magicstock/magic-wiki
```

支持平台：macOS（Apple Silicon）、Windows x64（Python 3.10 及以上）。

二进制按 Python Limited API（abi3）编译，兼容 Python 3.10 ~ 3.13，无需与构建机版本一致。
安装包内自动按平台/架构/ABI 选择二进制（精确 ABI 优先，缺失时回退 abi3），
不匹配时给出明确报错。

### 安装后配置

配置统一保存在用户级文件 `~/.magic-wiki/config.json`，**首次运行任一命令时自动创建**
（自动合并默认配置并生成目录，无需手工建文件）。读写推荐使用 `config` 子命令，
敏感字段（`app_secret` 等）展示时自动脱敏。

**第 1 步：查看配置文件位置**

```bash
magic-wiki config path
# 输出示例：
# scripts_dir   .../scripts
# config_file   ~/.magic-wiki/config.json
# db_file       ~/.magic-wiki/tasks.db
# temp_dir      ~/.magic-wiki/temp
```

**第 2 步：配置 LLM 模型（最重要，所有 AI 功能依赖）**

magic-wiki 的笔记生成、视频 / 文章分析、视觉理解等 AI 能力，都通过
**OpenAI 兼容 API** 调用大模型完成，因此必须先配置模型供应商 `llm.providers`。

直接用 `config set` 写入默认模型与备选链：

```bash
magic-wiki config set llm.default_model "minimax/MiniMax-M3"
magic-wiki config set llm.fallback_chain '["minimax/MiniMax-M3","你的备选模型1","你的备选模型2"]'
```

然后在 `~/.magic-wiki/config.json` 的 `llm` 块中填入你的供应商（把 `baseUrl` / `apiKey` 换成你自己的）：

```json
{
  "llm": {
    "default_model": "minimax/MiniMax-M3",
    "fallback_chain": [
      "minimax/MiniMax-M3",
      "你的备选模型1",
      "你的备选模型2"
    ],
    "providers": {
      "minimax": {
        "baseUrl": "https://你的供应商地址/v1",
        "apiKey": "sk-你的密钥",
        "models": [
          { "id": "minimax/MiniMax-M3", "input": ["text", "image"] },
          { "id": "minimax/你的模型", "input": ["text"] }
        ]
      }
    }
  }
}
```

要点：

- 模型 ID 使用 `供应商/模型名` 格式，如 `minimax/MiniMax-M3`
- 视觉分析（视频画面理解、截图理解等）需要 `input` 包含 `"image"` 的模型
- 主模型失败会自动沿 `fallback_chain` 切换备选（429 / 超时自动熔断与冷却）
- 未配置 `llm.providers` 时，会尝试从 `~/.openclaw/openclaw.json` 读取历史配置兜底

**第 3 步：配置飞书凭证（剪藏到飞书 / 搜索发送汇总卡片时需要）**

在[飞书开放平台](https://open.feishu.cn/)创建自建应用，拿到 App ID 与 App Secret 后写入：

```bash
magic-wiki config set feishu.app_id "cli_xxx"        # 你的飞书应用 App ID
magic-wiki config set feishu.app_secret "xxx"        # 你的飞书应用 App Secret
```

**第 4 步：可选配置**

| 配置键 | 作用 | 示例 |
|--------|------|------|
| `video_note.vision_models` | 视觉模型优先级链（生成笔记用，需支持图片） | `magic-wiki config set video_note.vision_models '["MiniMax-M3","Doubao-Seed-2.0-Pro"]'` |
| `video_note.text_models` | 文本模型优先级链（生成笔记用） | `magic-wiki config set video_note.text_models '["Kimi-K2.6","MiniMax-M3"]'` |
| `feishu.wiki_node` | 保存到指定飞书知识库节点 | `magic-wiki config set feishu.wiki_node "CvR..."` |
| `feishu.folder_token` | 保存到指定飞书文件夹 | `magic-wiki config set feishu.folder_token "BU34..."` |
| `feishu.card_icon_key` | 飞书卡片图标 key | `magic-wiki config set feishu.card_icon_key "img_v3..."` |
| `video_note.notify_target` | 飞书通知目标 open_id | `magic-wiki config set video_note.notify_target "ou_xxx"` |
| `video_note.concurrency` | 队列并发数（默认 2） | `magic-wiki config set video_note.concurrency 4` |
| `video_note.asr_engine` | 语音转写引擎 | `magic-wiki config set video_note.asr_engine "faster-whisper"` |
| `markdown.vault_path` | 剪藏到 markdown 的保存目录 | `magic-wiki config set markdown.vault_path "~/notes"` |
| `base_dir` | 工作区根目录（默认自动推导） | `magic-wiki config set base_dir "/path/to/workspace"` |

> `config set` 的值会自动识别 JSON 类型：数字、布尔、数组按原类型写入，其余按字符串处理。

**第 5 步：验证配置**

```bash
magic-wiki config show                # 查看全部配置（敏感字段脱敏）
magic-wiki config get feishu.app_id   # 读取单个配置项（支持点路径）
magic-wiki config env                 # 查看 .env 加载情况
```

配置也可直接编辑 `~/.magic-wiki/config.json` 手动写入，但请自行保证 JSON 格式正确。

### 快速开始

```bash
# 剪藏网页（默认保存到 markdown）
magic-wiki clip https://mp.weixin.qq.com/s/xxx

# 剪藏到飞书
magic-wiki clip https://xxx --target feishu --title "我的笔记"

# 搜索 B 站 + 公众号
magic-wiki search "AI 短剧"

# 队列管理
magic-wiki queue status
magic-wiki queue start
magic-wiki queue stop
magic-wiki queue clean --days 14

# 视频一键成笔记（入队后台处理）
magic-wiki enqueue https://www.bilibili.com/video/BVxxx
magic-wiki enqueue https://www.youtube.com/watch?v=xxx

# 合集 / 播放列表 / 多 P 视频
magic-wiki enqueue https://space.bilibili.com/xxx/lists/xxx
magic-wiki enqueue https://www.youtube.com/playlist?list=xxx

# 配置管理
magic-wiki config show                # 查看配置（敏感字段脱敏）
magic-wiki config path                # 查看路径
magic-wiki config get feishu.app_id   # 读取单个配置项（支持点路径）
magic-wiki config unset markdown.vault_path   # 删除配置项
```

### 子命令一览

| 命令 | 功能 |
|------|------|
| `magic-wiki clip <url>` | 剪藏网页到 markdown / 飞书 |
| `magic-wiki search <query>` | 搜索 B 站 + 公众号 |
| `magic-wiki queue <action>` | 队列管理（start/stop/status/poll/clean） |
| `magic-wiki enqueue <url>` | 视频 / 合集 / 文章快捷入队 |
| `magic-wiki config <action>` | 配置管理（show/path/env/get/set/unset） |

### 相关链接

- npm 包：[@magicstock/magic-wiki](https://www.npmjs.com/package/@magicstock/magic-wiki)

---

## 路线图

- [x] magic-wiki —— 网页剪藏 / 搜索 / 队列管理
- [ ] magic-video —— 视频内容处理 CLI（开发中）

## 费用

magic 系列由 **MagicAI 团队**推出，**永远免费使用**。

## 赞助作者

如果 magic 系列对你有所帮助，欢迎扫码赞助，支持作者持续更新：

<img src="assets/sponsor-qrcode.png" alt="赞助二维码" width="200" />
