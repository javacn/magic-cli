# magic-cli

免费命令行工具箱（magic 系列 CLI）—— 一个命令家族，各司其职，装完即用。

> 🌐 在线介绍页：https://javacn.github.io/magic-cli/

---

## 🤖 Agent 一键安装提示词

以下提示词可直接整体复制给任意 AI 助手（Trae / Claude / GPT 等），
让它在你的电脑上自动完成 **magic-wiki** 的安装与配置：

```text
你是一名终端助手。请在用户电脑上安装并配置 magic-wiki（magic-cli 系列的命令行工具，用于网页剪藏、B 站 + 公众号搜索与队列管理）。

步骤：
1. 安装：执行 npm install -g @magicstock/magic-wiki（国内网络慢时追加 --registry=https://registry.npmmirror.com）
2. 验证：执行 magic-wiki --version，确认输出版本号
3. 查看配置：执行 magic-wiki config path，确认配置文件位置（默认 ~/.magic-wiki/config.json）
4. 配置模型与飞书凭证（模型必填；飞书用于剪藏到飞书、搜索发送汇总卡片，用户不需要飞书则跳过）：
   - 推荐：运行 magic-wiki init，先按向导填写默认模型 / API 地址 / API Key（必填），再填飞书 App ID / App Secret 等（可选）
   - 或手动：magic-wiki config set feishu.app_id "<用户的飞书应用 APP ID>"，magic-wiki config set feishu.app_secret "<用户的飞书应用 APP SECRET>"
5. 可选配置（按用户需要执行）：
   - magic-wiki config set feishu.wiki_node "<知识库节点 token>"        # 保存到飞书知识库
   - magic-wiki config set video_note.notify_target "<用户 open_id>"     # 飞书通知目标
   - magic-wiki config set raw_dir "~/notes"                                 # 笔记保存根目录（默认 ~/.magic-wiki/wiki）
6. 验证配置：执行 magic-wiki config show（敏感字段自动脱敏）
7. 功能自测：执行 magic-wiki search "测试" --limit 3

完成后向用户汇报：安装版本、配置文件路径、已配置的配置项。
```

---

## 简介

**magic-cli** 是免费的命令行工具箱，由一系列独立的命令行工具组成：
每个 `magic-*` 解决一类具体需求，通过 npm 全局安装、装完即用，
不依赖用户手动配置 Python 环境。

## 系列一览

| 工具 | 状态 | 说明 | 安装 |
|------|------|------|------|
| [magic-wiki](#magic-wiki) | ✅ 已发布 v1.0.4 | 网页剪藏 / 搜索 B 站 + 公众号 / 队列管理 | `npm install -g @magicstock/magic-wiki` |
| [magic-video](#magic-video) | ✅ 已发布 v0.1.1 | 短剧 + 小说 + 视频分镜 一体化内容生产 | `npm install -g @magicstock/magic-cli` |
| magic-ppt | 🚧 规划中 | 演示文稿生成 CLI | 敬请期待 |
| magic-test | 🚧 规划中 | 自动化测试 CLI | 敬请期待 |
| magic-canvas | 🚧 规划中 | 无限画布 CLI | 敬请期待 |
| magic-game | 🚧 规划中 | 小游戏生成 CLI | 敬请期待 |
| magic-matrix | 🚧 规划中 | 短视频矩阵分发工具 | 敬请期待 |

---

## magic-wiki

网页内容剪藏工具 CLI —— 剪藏网页 / 搜索 B 站 + 公众号 / 队列管理。

`npm install -g @magicstock/magic-wiki` 一键安装、装完即用，无需手动配置 Python 环境，开箱即可上手。

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

国内用户下载较慢时，建议使用 npmmirror（原淘宝镜像）加速：

```bash
# 方式一：临时指定镜像源（仅本次安装生效）
npm install -g @magicstock/magic-wiki --registry=https://registry.npmmirror.com

# 方式二：永久切换 npm 镜像源（全局生效，推荐）
npm config set registry https://registry.npmmirror.com
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

**第 1 步：运行初始化向导（推荐）**

`magic-wiki init` 是 v1.0.4 新增的交互式配置向导，先引导你填写**模型配置（必填，所有 AI 功能依赖）**，再依次是飞书凭证、笔记保存根目录与通知目标（可选）。必填项不能留空，可选项直接回车表示保留当前值 / 使用默认 / 跳过：

```bash
magic-wiki init
```

**第 2 步：查看配置文件位置**

```bash
magic-wiki config path
# 输出示例：
# scripts_dir   .../scripts
# config_file   ~/.magic-wiki/config.json
# db_file       ~/.magic-wiki/tasks.db
# temp_dir      ~/.magic-wiki/temp
```

**第 3 步：配置 LLM 模型（最重要，所有 AI 功能依赖）**

magic-wiki 的笔记生成、视频 / 文章分析、视觉理解等 AI 能力，都通过
**OpenAI 兼容 API** 调用大模型完成，因此必须先配置模型供应商 `llm.providers`。
运行 `magic-wiki init` 时模型即为必填项；也可以直接用 `config set` 写入默认模型与备选链：

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

**第 4 步：配置飞书凭证（剪藏到飞书 / 搜索发送汇总卡片时需要）**

在[飞书开放平台](https://open.feishu.cn/)创建自建应用，拿到 App ID 与 App Secret 后写入：

```bash
magic-wiki config set feishu.app_id "cli_xxx"        # 你的飞书应用 App ID
magic-wiki config set feishu.app_secret "xxx"        # 你的飞书应用 App Secret
```

**第 5 步：可选配置**

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
| `raw_dir` | 笔记保存根目录（markdown 落盘位置） | `magic-wiki config set raw_dir "~/notes"` |

> `config set` 的值会自动识别 JSON 类型：数字、布尔、数组按原类型写入，其余按字符串处理。

**第 6 步：验证配置**

```bash
magic-wiki config show                # 查看全部配置（敏感字段脱敏）
magic-wiki config get feishu.app_id   # 读取单个配置项（支持点路径）
magic-wiki config env                 # 查看 .env 加载情况
```

配置也可直接编辑 `~/.magic-wiki/config.json` 手动写入，但请自行保证 JSON 格式正确。

### 快速开始

```bash
# 首次使用：交互式初始化配置（飞书凭证 / 保存路径 / 通知目标）
magic-wiki init

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
magic-wiki config unset raw_dir   # 删除配置项
```

### 使用效果

飞书端实时收到处理进度卡片，分阶段展示下载 / 转写 / 抽帧 / 画面 / 生成 / 保存进度：

<img src="assets/feishu-usage.png" alt="飞书使用截图" width="600" />

### 子命令一览

| 命令 | 功能 |
|------|------|
| `magic-wiki clip <url>` | 剪藏网页到 markdown / 飞书 |
| `magic-wiki search <query>` | 搜索 B 站 + 公众号 |
| `magic-wiki queue <action>` | 队列管理（start/stop/status/poll/clean） |
| `magic-wiki enqueue <url>` | 视频 / 合集 / 文章快捷入队 |
| `magic-wiki init` | 交互式初始化配置向导 |
| `magic-wiki config <action>` | 配置管理（show/path/env/get/set/unset） |

### 相关链接

- npm 包：[@magicstock/magic-wiki](https://www.npmjs.com/package/@magicstock/magic-wiki)

---

## magic-video

中文网文 / 短剧剧本 / 视频分镜一体化内容生产 CLI —— 从大纲到分镜的完整内容生产流水线。

`npm install` 装完即用，无需安装 Go / Node 等任何运行环境。

### 特色功能

- **小说创作（novel）**：一键开大纲（智能默认 50 章 / 3 卷），逐章 Beat Sheet → 写作（2500-4000 字）→ 独立质检 → 修订，支持 `batch generate` 批量生成整本
- **三重合议质检**：anti-AI / 文本退化本地校验 + Save the Cat 章级节拍 + 6 维 LLM 评分（setting / character / pacing / hook / prose / density），总分低于 72 自动 BLOCK
- **自动精修循环**：`chapter autorevise` 让写手模型与评审模型多轮对抗（默认 5 轮），直到分数达标（默认 80）或达到轮数上限
- **跨章节校审**：`cross-review` 检查全本连续性，`global-refine` 按校审报告做最小化精修
- **短篇与适配**：1200-2000 字短篇单稿；`to_script` 把小说章节直接压缩成 60-90 秒短剧剧本
- **作家风格蒸馏（style）**：web 搜索语料 → LLM 蒸馏 8 维风格指纹（节奏 / 词汇 / 视角 / 对话 / 描写 / 分段 / 情绪 / 癖好）+ 反 AI 词表，可绑定到项目注入写作
- **短剧剧本（script）**：单集 60-90 秒，`episode generate` 生成 / `episode improve` 多轮改良（默认 3 轮、阈值 90），支持角色资产与跨集连续性
- **视频分镜（video）**：`storyboard generate` 把剧本全文拆成导演可用分镜台本（中文台本 + 英文图像 prompt），注入癫爽硬约束与 12 铁律镜头设计，支持首尾帧 / 全拆 / 关键镜头模式
- **风格与模型注入**：内置 11 个美术风格预设（国风 / 3D 国潮 / 真人都市等）、题材叙事卡与 Seedance 2.0 / 镜头设计技能包（含多语言词表）、5 个视频模型 prompt（Seedance 2 多参数 / 短剧短提示 / 首尾帧 / Wan 2.6 单图首帧等），按项目 `art_style` 自动装配
- **飞书实时进度卡**：批量生成时推送飞书卡片，折叠面板实时展示大纲 / 章节生成 / 全局校审 / 全局精修各阶段进度
- **用量成本透明**：`magic-video cost` 按 stage 聚合 LLM 调用次数、token 与人民币成本（内置模型定价表），日志存 `~/.magic-video/usage_log.json`
- **15 项健康检查**：`magic-video doctor` 一键体检 Go 环境、配置、LLM 凭据、prompts、技能包与内置资源，`--json` 输出对 CI 友好

### 安装

```bash
npm install -g @magicstock/magic-cli
```

国内用户下载较慢时，建议使用 npmmirror（原淘宝镜像）加速：

```bash
# 方式一：临时指定镜像源（仅本次安装生效）
npm install -g @magicstock/magic-cli --registry=https://registry.npmmirror.com

# 方式二：永久切换 npm 镜像源（全局生效，推荐）
npm config set registry https://registry.npmmirror.com
npm install -g @magicstock/magic-cli
```

支持平台：macOS（Apple Silicon）、Windows x64。

npm 安装即用，**无需 GitHub token、无需额外下载**；GitHub Releases 下载仅作为安装包缺失时的兜底（私有仓库需 `GITHUB_TOKEN`，含 sha256 校验）。

> 若安装后 `magic-video` 提示 binary not found，说明 postinstall 被 npm `allow-scripts` 拦截，执行：
> ```bash
> npm approve-scripts @magicstock/magic-cli
> node "$(npm root -g)/@magicstock/magic-cli/scripts/install.js"
> ```

### 配置

配置统一保存在用户级文件 `~/.magic-video/config.json`（首次运行任一命令时自动创建，旧版 `~/.magic-cli` 会自动迁移数据）。

```bash
magic-video config init            # 创建最小可用 config.json
magic-video config bind openclaw   # 从 ~/.openclaw/openclaw.json 同步模型供应商
magic-video config status          # 查看当前配置模式与同步状态
magic-video config show            # 显示当前配置
```

LLM 通过 OpenAI 兼容 API 调用（minimax / papergames 等多供应商），主模型失败自动沿 fallback 链切换备选（429 / 超时自动熔断与冷却）；`config bind openclaw` 后可直接复用本机 OpenClaw 的模型配置，无需重复填写 API Key。

### 快速开始

```bash
# 环境体检
magic-video doctor

# ① 小说：一键开大纲 → 完整生成单章 → 批量生成整本
magic-video novel +new --project 凡人末日传 --genre 科幻末世 --premise "重生回末日前7天"
magic-video novel chapter generate --project 凡人末日传 --chapter ch-001
magic-video novel batch generate --project 凡人末日传

# 短篇单稿 + 作家风格蒸馏
magic-video novel short --project 凡人末日传 --genre 沙雕脑洞 --prompt "末日囤泡面"
magic-video novel style distill --author 猫腻 --work 择天记

# ② 短剧剧本：开项目 → 单集生成 → 多轮改良
magic-video script +new --project 凡人末日传 --title "凡躯觉醒(第1集)" --roles "主持人,IT君"
magic-video script episode generate --project 凡人末日传 --episode episode_001
magic-video script episode improve --project 凡人末日传 --episode episode_001 --max-rounds 3

# ③ 视频分镜：把剧本拆成导演台本
magic-video video storyboard generate --project 凡人末日传 --episode episode_001 \
  --mode first_last_frame --style 2D_chinese_guofeng

# 项目管理与成本
magic-video projects list
magic-video cost
```

### 子命令一览

| 命令 | 功能 |
|------|------|
| `magic-video novel +new` | 一键开新大纲（智能默认 50 章 / 3 卷） |
| `magic-video novel outline generate` | 生成顶层大纲 |
| `magic-video novel chapter plan` | 生成 Beat Sheet |
| `magic-video novel chapter write` | 写单章（2500-4000 字） |
| `magic-video novel chapter review` | 独立质检（本地校验 + 节拍 + 6 维 LLM 评分） |
| `magic-video novel chapter revise` | 按 review.json 问题清单修订章节 |
| `magic-video novel chapter generate` | 完整生成单章（plan+write+review+revise） |
| `magic-video novel chapter autorevise` | 自动精修循环（写手 vs 评审多轮对抗） |
| `magic-video novel chapter cross-review` | 跨章节连续性检查（全本） |
| `magic-video novel chapter global-refine` | 基于校审报告做最小化精修 |
| `magic-video novel batch generate` | 批量生成所有章节 |
| `magic-video novel short` | 短篇单稿（1200-2000 字一次性） |
| `magic-video novel to_script convert` | 小说章节 → 短剧剧本适配 |
| `magic-video novel style <action>` | 作家风格指纹蒸馏与管理（distill/list/show/set） |
| `magic-video script +new` | 开新短剧项目（创建 project_spec.json） |
| `magic-video script episode generate` | 单集剧本生成（60-90 秒/集） |
| `magic-video script episode improve` | 多轮改良剧本 |
| `magic-video video storyboard generate` | 完整分镜生成（中文导演台本 + 英文图像 prompt） |
| `magic-video doctor` | 15 项环境健康检查（`--json` 输出 CI 友好） |
| `magic-video config <action>` | 配置管理（init/show/bind/unbind/status） |
| `magic-video projects <action>` | 项目管理（list/info/rm） |
| `magic-video cost` | LLM 用量与成本聚合（按 stage 分组） |
| `magic-video skills list` | 查看内置 AI Agent Skills 注册表 |
| `magic-video feishu <action>` | 飞书应用绑定 / OAuth 登录 / 进度卡片 |

### 相关链接

- npm 包：[@magicstock/magic-cli](https://www.npmjs.com/package/@magicstock/magic-cli)
- 源码仓库：github.com/javacn/magic-video

---

## 路线图

- [x] magic-wiki —— 网页剪藏 / 搜索 / 队列管理
- [x] magic-video —— 短剧 + 小说 + 视频分镜一体化内容生产 CLI
- [ ] magic-ppt —— 演示文稿生成 CLI（规划中）
- [ ] magic-test —— 自动化测试 CLI（规划中）
- [ ] magic-canvas —— 无限画布 CLI（规划中）
- [ ] magic-game —— 小游戏生成 CLI（规划中）
- [ ] magic-matrix —— 短视频矩阵分发工具（规划中）

## 费用

magic 系列**永远免费使用**。

## 赞助作者

如果 magic 系列对你有所帮助，欢迎扫码赞助，支持作者持续更新：

<img src="assets/sponsor-qrcode.png" alt="赞助二维码" width="200" />

## 联系方式

如有问题、建议或合作意向，欢迎邮件联系：

<j2ee_cn@126.com>
