# bai

[English](README_EN.md) | **简体中文**

一个**单文件、零构建**的 Bash 终端 AI 助手：在终端里与任意 OpenAI 兼容服务对话，支持流式输出、思维链、函数工具调用、MCP、会话保存与续聊、LaTeX 公式渲染，以及可自定义的 logo 与等待动画。
灵感来自 [openai-terminal-assistant](https://github.com/worthable/openai-terminal-assistant)。

![bai 会话预览](preview1.png)

更多预览：[preview2.png](preview2.png) · [preview3.png](preview3.png)

## 特性

- **流式对话**：SSE 增量解析，思维链与正文分别着色，工具调用边生成边展示；
- **函数工具（Function Calling）**：JSON Lines 工具文件，内置 `run_command` 与 `send_image`，支持 MCP（stdio / HTTP）；
- **交互式权限确认**：方向键菜单（本次允许 / 本会话允许 / 拒绝），也支持 `y` / `a` / `n` 快捷键，非交互终端自动拒绝；
- **会话管理**：`--save-session` / `--continue` / `--resume`，自动保存、上下文自动压缩、记录导入导出；
- **输入增强**：`@"文件路径"` 引用文件与图片、`@prefix` 前缀续写、管道输入、`.file` 提交材料；
- **公式渲染**：LaTeX → PNG（`latex` + `dvipng` + `chafa`/`catimg`），失败或非交互终端自动退回 Unicode；
- **外观**：自定义 logo（文件 + `R,G,B` 彩色令牌 + `{{...}}` 渐变标记）、等待动画样式、渐变模式（彩色 / 灰度 / 双色）；
- **接口适配**：`chat/completions` 与 `Responses` 双风格（`--api-style`）；
- **本地模型**：`--llamacpp-launch` 一键拉起 llama.cpp 服务并连接；
- **本地化**：内置简体中文（`zh_CN`），可导入其它语言文件。

## 依赖

必需：

| 命令 | 用途 |
| --- | --- |
| `bash` | 运行本体，建议 **5.3+**（更早版本可运行，但缺少 `read` 补全等新特性） |
| `curl` | 发起 HTTP 请求 |
| `jq` | JSON 解析与请求体构造 |
| `sed` / `cat` / `grep` / `file` | 基础文本处理与文件类型判断 |

可选（按需安装，可用 `--no-extend` 禁用）：

| 命令 | 用途 |
| --- | --- |
| `bat` | 语法高亮（`.code` 与静态输出） |
| `fzf` | 历史记录反向搜索（Ctrl-R）与文件选择 |
| `gum` | 交互增强（部分场景替代内置实现） |
| `yazi` | 内置文件选择增强 |
| `chafa` / `catimg` / `fastfetch` | 终端图片显示（公式、图片预览） |
| `latex` / `dvipng` | LaTeX 公式编译与转图 |

## 安装

方式一：一行安装

```bash
curl https://raw.githubusercontent.com/hornleaf/bai/refs/heads/main/bai | install -vm 755 /dev/stdin /usr/bin/bai
```

方式二：克隆仓库

```bash
git clone https://github.com/hornleaf/bai.git
cd bai
install -m 755 bai /usr/local/bin/bai
```

方式三：手动下载 `bai` 后 `chmod +x bai`。

> 只有一个脚本文件，无编译、无运行时；升级时直接替换 `bai` 即可。

## 快速开始

```bash
export OPENAI_API_KEY="sk-..."                        # 必需
export OPENAI_BASE_URL="https://api.deepseek.com"     # 任意 OpenAI 兼容服务
export OPENAI_API_MODEL="deepseek-flash,deepseek-chat" # 可给候选列表，自动选第一个可用

bai -n                  # 流式连续对话
bai "解释一下这个脚本" < script.sh   # 单次对话 + 管道输入
cat README.md | bai "总结要点"        # 同上
```

把配置写进文件，避免每次重复输入：

```bash
# ~/.config/bai/deepseek.rc
OPENAI_API_KEY="sk-..."
OPENAI_BASE_URL="https://api.deepseek.com"
OPENAI_API_MODEL="deepseek-flash"

bai -f ~/.config/bai/deepseek.rc
```

未在命令行给出提问时进入连续对话模式，此时可用局内指令（见下节）。

需要一份配置模板？运行 `bai --create-config` 生成可选填的配置文件，`bai --edit-rcfile` 编辑启动时执行的 rc 文件。

## 命令行选项

`bai --usage` 查看完整说明，这里是常用部分：

| 选项 | 说明 |
| --- | --- |
| `-b, --baseurl` / `-k, --apikey` / `-m, --model` | 接口地址 / 密钥 / 模型（模型可给逗号分隔列表） |
| `-n, --stream` | 启用流式响应 |
| `-f, --config` | 读取环境变量文件（shell 语法） |
| `-s, --sysfile` | 指定 system prompt 文件 |
| `-c, --callfile` | 指定工具文件（JSON Lines） |
| `-e, --thinking` | 显式启用思维链 |
| `--reasoning-effort` | 思考强度：`default` / `low` / `medium` / `high` / `xhigh` |
| `-g, --search` | 显式启用联网搜索 |
| `-j, --partial` | 启用前缀续写（输入末尾加 @prefix） |
| `--include-usage` | 统计 token 用量 |
| `--api-style` | 接口风格：`chat` / `responses` |
| `--save-session` / `--continue` / `--resume` | 会话保存 / 续聊最近 / 恢复指定会话 |
| `--auto-compact` 等 | 上下文自动压缩（`--compact-model`、`--compact-keep`、`--compact-threshold`） |
| `--permission-mode` | 工具权限模式：`normal` / `full` / `auto` |
| `--tool-timeout` / `--tool-output-limit` | 工具超时秒数 / 单次输出上限字符 |
| `--mcp-config` / `--no-mcp` | MCP 配置路径 / 禁用 MCP |
| `--wait-style` / `--wait-gradient` | 等待动画样式 / 渐变模式 |
| `--logo-file` / `--logo-mode` | 自定义 logo 文件 / logo 色彩模式 |
| `--latex-mode` / `--no-latex` | 公式渲染方式（`auto` / `image` / `unicode` / `off`）/ 禁用 |
| `--llamacpp-launch` | 拉起 llama.cpp 并连接，其后参数全部转发给它 |
| `-ne, --no-extend` / `-nt, --no-tools` | 禁用指定可选依赖 / 禁用指定工具 |
| `-o, --debug` | 调试输出 |

## 会话指令

连续对话模式下以 `.` 开头，`.help <指令>` 查看详细用法：

| 指令 | 说明 |
| --- | --- |
| `.help` | 帮助与按键说明 |
| `.exit [状态码]` | 退出会话 |
| `.clear` | 清空对话并重置轮次 |
| `.cls` | 清屏 |
| `.list` | 列出当前对话 |
| `.code [preview] [索引]` | 复制最近一轮中的代码块 |
| `.file <文件>` | 向当前对话提交（或替换）文件/图片 |
| `.files list / push / del` | 服务商侧文件管理 |
| `.run <命令>` | 在本机执行 Shell 命令 |
| `.eval <提示词>` | 让模型生成命令并执行 |
| `.call <工具> <参数>` | 手动执行已导入的函数工具 |
| `.set <项>? <值>?` | 查看/修改会话配置 |
| `.record <文件> <import / export>` | 导入/导出对话记录 |
| `.compact [show / undo / tools]` | 压缩上下文（摘要旧对话，保留最近若干轮） |
| `.models` | 列出服务商模型与上下文长度 |
| `.sessions` | 列出已保存的会话 |
| `.mcp` | 查看已加载的 MCP server 与工具数 |
| `.perms` | 查看工具权限级别与会话内授权状态 |
| `.api` | 查看接口适配层自检信息 |
| `.term` | 查看终端尺寸来源（排查折行宽度问题） |
| `.logo [preview / off / color / gray / dual / none / <文件>]` | 自定义 logo |

## 函数工具

### 工具文件（`--callfile`，JSON Lines）

```json
{"name": "readfile", "description": "读取文本文件", "function": "cat \"$1\"",
 "parameters": {"type": "object",
   "properties": {"path": {"type": "string", "description": "文件路径"}},
   "required": ["path"]}}
```

可选字段：`explicitly`（是否展示调用过程，`true` / `silence`）、`permission`（`never` / `low` / `medium` / `always`）、`stderr`、`depend`（外部命令依赖）、`arg_alias`（把 JSON 参数映射成命令行选项，支持 `"--search"` / `"--flag!"` / `"--count="` 三种写法）。

### 内置工具

开启 Function Calling（提供 `-c` 工具文件或 MCP）后自动注册：

| 工具 | 权限 | 说明 |
| --- | --- | --- |
| `run_command` | `medium`（首次确认） | 在子 Shell 执行一条非交互命令，返回 stdout + stderr 与退出码 |
| `send_image` | `low` | 把图片作为消息返回给模型（工具结果只能是文本，图片走单独的 user 消息） |

开关：`--no-tools run_command` 单次禁用；`BAI_RUN_COMMAND_TOOL` / `BAI_SELF_IMAGE` 全局开关。

### 权限模型

- **级别**（工具定义里的 `permission`）：`never` 静默放行、`low` 风险检查、`medium`（默认）首次询问、`always` 每次都问；
- **模式**（`--permission-mode`）：`normal` 按级别判断；`full` 全部放行；`auto` 额外调用一次模型判定高危；
- **确认菜单**：行内方向键菜单「本次允许 / 本会话允许 / 拒绝」，默认光标停在「拒绝」，`↑↓` 选择、`Enter` 确认、`Esc` 取消，也可用 `y` / `a` / `n`；`BAI_PERMISSION_ASK_TIMEOUT`（默认 60s）超时按拒绝处理；非交互终端直接拒绝并回写一条 `[权限]` 工具消息；
- 会话内调整：`.perms set <工具> <级别>`、`.perms reset <工具>`、`.perms mode <模式>`；
- `auto` 模式的审查结果按「工具 + 参数」指纹缓存（`BAI_PERMISSION_CACHE_TTL` 控制有效期，`BAI_PERMISSION_CACHE=false` 关闭）。

### MCP

配置文件默认 `${XDG_CONFIG_HOME:-~/.config}/bai/mcp.json`：

```json
{"servers": [
  {"name": "fs", "command": "npx", "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]},
  {"name": "srv", "url": "http://127.0.0.1:8080/mcp"}
]}
```

有 `command` 走 stdio，有 `url` 走 HTTP；工具名统一为 `mcp__<server>__<tool>`。用 `--mcp-config` 指定其它路径，`--no-mcp` 关闭，`.mcp` 查看状态。

## 会话与上下文

- 保存：`--save-session` 或 `BAI_SESSION_AUTOSAVE=true` 自动保存（默认目录 `${XDG_STATE_HOME:-~/.local/state}/bai/sessions`）；
- 续聊：`--continue` 继续最近一次（当前目录优先），`--resume <id|last>` 恢复指定会话，`.sessions` 列出已保存会话；
- 压缩：`--auto-compact` 在上下文占用超过阈值时把旧对话摘要化，保留最近 N 轮原文；`.compact` 手动压缩/查看/撤销；
- 记录：`.record <文件> export/import` 导出或导入对话（不包含指令执行）。

## 外观

### 等待动画

`-w / --wait-style` 选择动画样式（`bar` 默认、`dotmatrix`、`bell`、`block`、`roll`、`invisible` 等），`BAI_WAIT_FRAME=false` 或 `.set wait-frame false` 可隐藏转动的帧前缀，只保留文字与计时。

### 渐变

`--wait-gradient` / `BAI_WAIT_GRADIENT`：

| 模式 | 说明 |
| --- | --- |
| `dual`（默认） | 双色渐变，端点默认 `255,128,255:128,255,255` |
| `color` | 彩色彩虹 |
| `gray` | 灰度（`BAI_WAIT_GRAY_STYLE`：`band` 默认、`sine`、`white`、`breath`、`contrast`、`ramp`） |
| `none` | 不上色（纯文本，最省 CPU） |

双色模式的端点与风格：

```bash
BAI_WAIT_DUAL_COLORS="0,0,255:255,0,0"   # r,g,b:r,g,b，默认 255,128,255:128,255,255
BAI_WAIT_DUAL_STYLE=band                 # band（默认，亮带横扫）/ flow（渐变滑动）/ static（静止）
.set wait-dual-colors 0,0,255:255,0,0    # 会话内同样可改
```

### Logo

```bash
bai --logo-file ./logo.txt          # 文件作为 logo
bai --logo-mode dual                # auto（跟随等待渐变）/ color / gray / dual / none
.logo preview                       # 会话内预览
```

文件里支持 `R,G,B<字符>` 彩色令牌与 `{{...}}` 渐变标记，纯文本原样输出。

### 终端宽度

宽度按 `/dev/tty` → stdin → `tput` → `LINES/COLUMNS` 依次探测；若终端上报宽度不准（Web/移动端终端常见），用 `BAI_TERM_WIDTH=<列数>` 强制指定，`.term` 可查看各来源与最终取值。

## LaTeX 公式

`--latex-mode auto|image|unicode|off`（默认 `auto`）：`auto` 在非交互终端或渲染失败时退回 Unicode；图片模式依赖 `latex` 与 `dvipng`，可用 `chafa`/`catimg` 控制显示尺寸；`BAI_LATEX_DPI` 调整清晰度，`BAI_LATEX_PROBE=false` 跳过启动自检。

## 推理 / 搜索 / 续写 / 用量

- 思维链：`--thinking` 显式启用，`--reasoning-effort low|medium|high|xhigh` 指定强度（`--reasoning-effort-replace` 可替换供应商的取值）；
- 联网搜索：`--search`；
- 前缀续写：`--partial` 后，输入末尾加 `@prefix`；
- 用量统计：`--include-usage`（供应商支持时显示 token 统计）。

## 接口风格

`--api-style chat|responses`：默认 `chat`（`/v1/chat/completions`）；`responses` 走 `/v1/responses`（DeepSeek 等实现），思维链、工具调用、图片输入均已适配。`.api` 可查看当前风格与自检信息。

## 本地模型（llama.cpp）

```bash
bai --llamacpp-launch --model /path/model.gguf -c 8192
```

`--llamacpp-launch` 之后的参数全部转发给 `llama-server`（缺省端口 9931），脚本等待 `/health` 就绪后把 `OPENAI_BASE_URL` 指向它，退出时关闭；`BAI_LLAMACPP_TIMEOUT` 控制等待上限（0 = 一直等）。

## 本地化

内置简体中文 `zh_CN`（不可覆盖），其它语言通过文件导入：

```bash
bai --add-l10n-file=locate/en_US.txt
export BAI_L10N_FILES=locate/en_US.txt,locate/ja_JP.txt
bai --export-l10n-file ./zh_CN.txt      # 导出内置中文作为模板
```

文件格式：首行 `Language: xx_XX`，其后每行 `key value`（空格分隔）。

> **关于 `locate/` 目录**：其中的 `en_US.txt`、`ja_JP.txt`、`ru_RU.txt` 是使用 **Hy-MT2-1.8B-Q8_0** 模型对内置中文做的**简单机器翻译**，**未经过人工校对与修正**，可能存在错译、漏译、术语不一致或格式问题，仅供试用与二次修改参考。欢迎修正后提交 PR。

## 环境兼容

- Linux 为主要的开发与测试环境；
- macOS 与 Windows（MSYS2 / Git Bash）可用，但部分特性（图片显示、OSC 进度、终端尺寸探测、补全）未完全验证；
- 建议使用支持真彩色（24-bit）与 UTF-8 的终端，否则双色/灰度渐变会退化。

## 排错

| 现象 | 处理 |
| --- | --- |
| 权限确认被直接拒绝 | 非交互终端（管道/CI）无法回答确认，需在 TTY 中运行，或用 `--permission-mode full`（仅限可信环境） |
| 折行位置不对 | 执行 `.term` 查看宽度来源，必要时设 `BAI_TERM_WIDTH` |
| 公式显示为 Unicode | 检查 `latex` / `dvipng` 是否安装，或 `--latex-mode image` 强制 |
| 没有颜色 | 检查 `BAI_COLOR`（`never`/auto`/always`）与终端真彩色支持 |
| 工具没有被调用 | 需用 `-c` 指定工具文件或配置 MCP；小模型可能被工具限制，`--no-limit-tools` 可放开 |
| 想看请求细节 | `--debug` / `--inspect` |

## 贡献

欢迎提交 issue / PR：<https://github.com/hornleaf/bai/issues>。
修改本体时请保持单文件形态，并同步更新 `README.md` 与 `README_EN.md`。

## 鸣谢

- **DeepSeek V4.1 Flash 1017** —— 贡献了后期的大部分代码与 README 编写
- **[jq](https://jqlang.github.io/jq/)** —— JSON 解析与请求体构造
- **[bat](https://github.com/sharkdp/bat)** —— 代码高亮渲染
- **[Chafa](https://hpjansson.org/chafa/)** —— 通用终端图像显示
- **[TeX Live](https://www.tug.org/texlive/)** —— LaTeX 公式渲染
- **[Bash](https://www.gnu.org/software/bash/)** —— 让这个荒诞的创作想法成为可能的前提

## 许可证

[MIT License](LICENSE)

## 免责声明

本项目仅供学习与娱乐使用，请勿用于商业用途；使用第三方 API 时请遵守对应服务商的条款，API 密钥与对话内容由你自行保管。

---

完整说明：`bai --usage`；会话内帮助：`.help`。
