# Claude Code CLI 使用手册

## 基本语法

```bash
claude [prompt] [选项]
claude <子命令> [参数] [选项]
```

不带参数直接运行 `claude` 将启动交互式 REPL 会话（基于 Ink 的终端 UI）。在 prompt 位置直接传入文本则以非交互方式运行（等价于 `-p` 模式）。

## 运行模式

| 模式 | 命令 | 说明 |
|------|------|------|
| 交互式 REPL | `claude` | 启动完整的终端 UI 交互会话 |
| 非交互（管道） | `claude -p "提问"` 或 `claude "提问"` | 输出结果后退出，适合脚本/管道 |
| JSON 输出 | `claude -p "提问" --output-format json` | 单次 JSON 结果 |
| 流式 JSON | `claude -p "提问" --output-format stream-json` | 实时流式 JSON 输出 |
| 流式输入+输出（SDK） | `claude -p --input-format stream-json --output-format stream-json` | 全双工流式通信 |

## 常用选项

### 会话管理

| 选项 | 说明 |
|------|------|
| `-c, --continue` | 继续当前目录最近的会话 |
| `-r, --resume [id]` | 按 session ID 恢复会话，或打开交互选择器 |
| `--fork-session` | 恢复时创建新 session ID（与 `--resume` 或 `--continue` 配合） |
| `--from-pr [value]` | 恢复关联到 PR 的会话（按 PR 号/URL） |
| `-n, --name <name>` | 设置会话显示名称（在 `/resume` 列表和终端标题中显示） |
| `--session-id <uuid>` | 指定会话 ID（必须是有效 UUID） |
| `--no-session-persistence` | 禁用会话持久化（仅 `--print` 模式） |
| `--resume-session-at <msg-id>` | 恢复到指定消息位置（与 `--resume` 配合） |
| `--rewind-files <user-msg-id>` | 恢复文件到指定消息时的状态并退出（需要 `--resume`） |

### 模型与性能

| 选项 | 说明 |
|------|------|
| `--model <model>` | 指定模型。支持别名（`sonnet`、`opus`）或完整名称（`claude-sonnet-4-6`） |
| `--effort <level>` | 努力程度：`low`、`medium`、`high`、`max` |
| `--thinking <mode>` | 思考模式：`enabled`（=adaptive）、`adaptive`、`disabled` |
| `--max-thinking-tokens <n>` | [已废弃，请用 `--thinking`] 最大思考 token 数 |
| `--fallback-model <model>` | 主模型过载时的自动回退模型（仅 `--print` 模式） |
| `--max-turns <n>` | 非交互模式最大 agent 回合数（仅 `--print` 模式） |
| `--max-budget-usd <amount>` | API 花费上限（美元，仅 `--print` 模式） |
| `--task-budget <tokens>` | API 端任务预算（`output_config.task_budget`） |
| `--betas <betas...>` | API 请求 Beta 头（仅 API key 用户） |

### 权限控制

| 选项 | 说明 |
|------|------|
| `--permission-mode <mode>` | 权限模式：`default`、`acceptEdits`、`bypassPermissions`、`dontAsk`、`plan` |
| `--dangerously-skip-permissions` | 跳过所有权限检查（仅限无网络访问的沙箱环境） |
| `--allow-dangerously-skip-permissions` | 允许跳过权限但不默认启用 |
| `--allowedTools <tools...>` | 允许的工具，如 `"Bash(git:*) Edit"`（逗号或空格分隔） |
| `--disallowedTools <tools...>` | 禁用的工具，如 `"Bash(git:*) Edit"` |
| `--tools <tools...>` | 可用工具集。`""` 全部禁用，`"default"` 全部可用 |

### 上下文与提示词

| 选项 | 说明 |
|------|------|
| `--system-prompt <prompt>` | 自定义 system prompt |
| `--system-prompt-file <file>` | 从文件读取 system prompt |
| `--append-system-prompt <prompt>` | 在默认 system prompt 后追加内容 |
| `--append-system-prompt-file <file>` | 从文件读取并追加到默认 system prompt |
| `--add-dir <dirs...>` | 添加额外允许工具访问的目录 |
| `--settings <file-or-json>` | 加载额外设置文件或 JSON 字符串 |
| `--setting-sources <sources>` | 设置来源：`user`、`project`、`local`（逗号分隔） |
| `--agents <json>` | 自定义 agent 定义，JSON 格式 |
| `--agent <agent>` | 当前会话使用的 agent（覆盖 settings 中的 `agent`） |

### MCP 集成

| 选项 | 说明 |
|------|------|
| `--mcp-config <configs...>` | 加载 MCP 服务器配置（JSON 文件或字符串） |
| `--strict-mcp-config` | 仅使用 `--mcp-config` 指定的 MCP 服务器，忽略所有其他 MCP 配置 |
| `--mcp-debug` | [已废弃，请用 `--debug`] 显示 MCP 服务器错误 |

### 输出格式

| 选项 | 说明 |
|------|------|
| `--output-format <fmt>` | 输出格式（仅 `--print` 模式）：`text`（默认）、`json`、`stream-json` |
| `--json-schema <schema>` | JSON Schema 用于结构化输出验证 |
| `--input-format <fmt>` | 输入格式（仅 `--print` 模式）：`text`（默认）、`stream-json` |
| `--include-hook-events` | 输出流中包含 hook 生命周期事件（仅 `--output-format=stream-json`） |
| `--include-partial-messages` | 包含部分消息块（仅 `--print` + `--output-format=stream-json`） |
| `--replay-user-messages` | 将用户消息回显到 stdout（仅 `--input-format=stream-json` + `--output-format=stream-json`） |

### 调试与诊断

| 选项 | 说明 |
|------|------|
| `-d, --debug [filter]` | 启用调试模式，可选过滤器如 `"api,hooks"` 或 `"!1p,!file"` |
| `-d2e, --debug-to-stderr` | 调试日志输出到 stderr |
| `--debug-file <path>` | 调试日志写入文件（隐式启用 debug） |
| `--verbose` | 覆盖配置中的 verbose 模式 |
| `-v, --version` | 显示版本号 |

### 其他

| 选项 | 说明 |
|------|------|
| `--bare` | 最小模式，跳过 hooks、LSP、插件同步、归属、自动记忆、后台预取、keychain 读取和 CLAUDE.md 自动发现。认证严格使用 `ANTHROPIC_API_KEY` 或 apiKeyHelper |
| `--init` | 运行 Setup hooks（init 触发器）后继续 |
| `--init-only` | 运行 Setup 和 SessionStart:startup hooks 后退出 |
| `--maintenance` | 运行 Setup hooks（maintenance 触发器）后继续 |
| `--ide` | 启动时自动连接 IDE（仅当恰好有一个有效 IDE 可用时） |
| `--chrome` / `--no-chrome` | 启用/禁用 Claude in Chrome 集成 |
| `--disable-slash-commands` | 禁用所有 skill |
| `--plugin-dir <path>` | 加载额外插件目录（可重复使用） |
| `--file <specs...>` | 下载文件资源，格式：`file_id:relative_path` |
| `--prefill <text>` | 预填充输入文本（不自动提交） |
| `--enable-auto-mode` | 启用自动模式 |
| `--proactive` | 启动为主动自主模式（ANT/Kairos） |

## 子命令

### MCP 管理 (`claude mcp`)

```bash
claude mcp serve                  # 启动 Claude Code MCP 服务器
claude mcp list                   # 列出已配置的 MCP 服务器
claude mcp get <name>             # 查看 MCP 服务器详情
claude mcp add-json <name> <json> # 通过 JSON 添加 MCP 服务器
claude mcp remove <name>          # 移除 MCP 服务器
claude mcp add-from-claude-desktop # 从 Claude Desktop 导入 MCP 服务器
claude mcp reset-project-choices  # 重置项目级 .mcp.json 的审批决定
```

### 认证管理 (`claude auth`)

```bash
claude auth login     # 登录 Anthropic 账户
  --email <email>     # 预填邮箱地址
  --sso               # 强制 SSO 登录
  --console           # 使用 Anthropic Console 计费
  --claudeai          # 使用 Claude 订阅计费（默认）
claude auth logout    # 登出
claude auth status    # 查看认证状态
  --json              # JSON 格式输出（默认）
  --text              # 人类可读文本输出

claude setup-token    # 生成长效认证 token（需要 Claude 订阅）
```

### 插件管理 (`claude plugin`)

```bash
claude plugin list                      # 列出已安装插件
  --json                                # JSON 格式输出
  --available                           # 包含市场中可用插件

claude plugin install <plugin>          # 安装插件
  -s, --scope <scope>                   # 安装范围：user/project/local

claude plugin uninstall <plugin>        # 卸载插件
  --keep-data                           # 保留插件数据目录

claude plugin enable <plugin>           # 启用已禁用的插件
claude plugin disable [plugin]          # 禁用插件
  -a, --all                             # 禁用所有已启用插件

claude plugin update <plugin>           # 更新插件
claude plugin validate <path>           # 验证插件或市场清单

# 市场管理
claude plugin marketplace add <source>     # 添加市场（URL/路径/GitHub 仓库）
  --sparse <paths...>                      # git sparse-checkout（monorepo）
claude plugin marketplace list             # 列出所有市场
claude plugin marketplace remove <name>    # 移除市场
claude plugin marketplace update [name]    # 更新市场
```

### 后台会话管理 (`claude ps/logs/attach/kill`)

```bash
claude ps               # 列出所有后台会话
claude logs <id>        # 查看会话日志
claude attach <id>      # 附加到后台会话
claude kill <id>        # 终止后台会话
claude --bg             # 启动后台会话
```

### 远程控制 (`claude remote-control`)

```bash
claude remote-control [name]   # 启动远程控制会话（别名：rc、remote、sync、bridge）
claude ssh <host> [dir]        # 通过 SSH 在远程主机运行 Claude Code
  --permission-mode <mode>     # 远程会话的权限模式
  --dangerously-skip-permissions # 远程跳过所有权限检查
  --local                      # e2e 测试模式（本地模拟远程）
claude server                  # 启动 Claude Code 会话服务器
  --port <number>              # HTTP 端口（默认 0=随机）
  --host <string>              # 绑定地址（默认 0.0.0.0）
  --auth-token <token>         # Bearer token 认证
  --unix <path>                # Unix domain socket 监听
  --workspace <dir>            # 默认工作目录
  --idle-timeout <ms>          # 空闲超时（默认 600000ms）
  --max-sessions <n>           # 最大并发会话数（默认 32）
```

### 系统维护

```bash
claude doctor           # 系统健康检查（跳过信任对话框）
claude update           # 检查并安装更新
claude install [target] # 安装原生构建版本
  --force               # 强制安装（即使已安装）
```

## 会话内斜杠命令（REPL）

在交互式 REPL 中，输入 `/` 触发命令面板：

### Git & 代码管理

| 命令 | 功能 |
|------|------|
| `/commit` | 智能生成 Git commit |
| `/commit-push-pr` | Commit + Push + 创建 PR 一体化 |
| `/review` | 代码审查 |
| `/diff` | 差异查看 |
| `/pr_comments` | PR 评论分析 |
| `/branch` | 分支管理 |
| `/issue` | Issue 管理 |

### 会话管理

| 命令 | 功能 |
|------|------|
| `/resume` | 恢复历史会话 |
| `/session` | 会话管理 |
| `/clear` | 清除对话内容 |
| `/compact` | 压缩上下文 |
| `/export` | 导出会话 |
| `/copy` | 复制内容 |
| `/rename` | 重命名会话 |

### 模式切换

| 命令 | 功能 |
|------|------|
| `/plan` | 计划模式（先规划后执行） |
| `/fast` | 快速模式 |
| `/vim` | Vim 模式 |
| `/permissions` | 权限管理 |
| `/model` | 切换模型 |
| `/effort` | 设置努力程度 |
| `/output-style` | 输出风格设置 |

### 配置 & 设置

| 命令 | 功能 |
|------|------|
| `/config` | 配置管理 |
| `/init` | 初始化项目（生成 CLAUDE.md） |
| `/theme` | 主题设置 |
| `/color` | 颜色配置 |
| `/keybindings` | 快捷键绑定 |
| `/hooks` | 钩子配置 |
| `/env` | 环境变量管理 |
| `/memory` | 记忆系统管理 |

### 认证

| 命令 | 功能 |
|------|------|
| `/login` | 登录账户 |
| `/logout` | 登出账户 |
| `/status` | 查看状态 |

### MCP & 插件

| 命令 | 功能 |
|------|------|
| `/mcp` | MCP 服务器管理 |
| `/plugin` | 插件管理 |
| `/skills` | 技能系统 |

### 上下文管理

| 命令 | 功能 |
|------|------|
| `/context` | 查看当前上下文 |
| `/add-dir` | 添加额外工作目录 |
| `/files` | 管理文件权限 |
| `/ctx_viz` | 上下文可视化 |

### 工具 & 诊断

| 命令 | 功能 |
|------|------|
| `/doctor` | 系统诊断 |
| `/cost` | 费用统计 |
| `/usage` | 使用量统计 |
| `/help` | 帮助信息 |
| `/feedback` | 发送反馈 |
| `/upgrade` | 升级版本 |
| `/release-notes` | 发布说明 |

### 进阶功能（ANT 内部）

| 命令 | 功能 |
|------|------|
| `/agents` | 多 agent 管理 |
| `/assistant` | 助手模式 |
| `/tasks` | 任务列表 |
| `/security-review` | 安全检查 |
| `/bug-hunter` | Bug 猎手 |
| `/ant-trace` | 调用链追踪 |
| `/insights` | 洞察分析 |
| `/onboarding` | 用户引导 |
| `/teleport` | 远程传送会话 |

## 环境变量

### 认证

| 变量 | 用途 |
|------|------|
| `ANTHROPIC_API_KEY` | API 密钥（`--bare` 模式下唯一认证方式） |
| `ANTHROPIC_BASE_URL` | 自定义 API 基础 URL |

### 模型

| 变量 | 用途 |
|------|------|
| `ANTHROPIC_MODEL` | 设置默认模型（可被 `--model` 覆盖） |
| `ANTHROPIC_SMALL_FAST_MODEL` | 设置小型快速模型 |

### 功能开关

| 变量 | 用途 |
|------|------|
| `CLAUDE_CODE_SIMPLE` | 启用最小模式（等价于 `--bare`） |
| `CLAUDE_CODE_DISABLE_THINKING` | 禁用扩展思考 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | 禁用自动记忆 |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | 禁用后台任务 |
| `DISABLE_COMPACT` | 禁用上下文压缩 |
| `DISABLE_AUTO_COMPACT` | 禁用自动上下文压缩 |
| `DISABLE_INTERLEAVED_THINKING` | 禁用交错思考模式 |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | 禁用终端标题更新 |

### 远程与调试

| 变量 | 用途 |
|------|------|
| `CLAUDE_CODE_REMOTE` | 标记远程容器环境（自动调大为 8GB 堆内存） |
| `CLAUDE_CODE_ENTRYPOINT` | 入口点标识（用于遥测归属） |
| `CLAUDE_CODE_ATTRIBUTION_HEADER` | 控制归属头行为 |
| `ENABLE_GROWTHBOOK_DEV` | 使用 GrowthBook 开发环境（ANT 内部） |

## 配置文件

配置使用 JSON 格式，支持三个层级（后加载的合并覆盖先加载的）：

1. **全局** — `~/.claude/settings.json`
2. **项目** — `<project>/.claude/settings.json`
3. **本地** — `<project>/.claude/settings.local.json`

可通过 `--settings` 选项加载额外配置，通过 `--setting-sources` 限定加载来源。

其他数据存储路径（均在 `~/.claude/` 下）：

| 目录/文件 | 内容 |
|-----------|------|
| `~/.claude/sessions/` | 会话历史记录 |
| `~/.claude/cache/` | 缓存（changelog 等） |
| `~/.claude/backups/` | 配置文件备份 |
| `~/.claude/plugins/` | 插件和数据目录 |

## 典型用法示例

```bash
# 启动交互会话
claude

# 非交互提问
claude -p "解释这个项目的架构"

# 指定模型和权限模式
claude --model opus --permission-mode plan

# 管道模式输出 JSON
claude -p "列出所有 .ts 文件" --output-format json

# 继续上次会话
claude -c

# 恢复特定会话
claude -r <session-id>

# 在后台运行
claude -p "运行测试并修复失败的用例" --bg

# 使用自定义 system prompt
claude --system-prompt "你是一个 Rust 专家" -p "审查这个函数"

# 最小模式（适合 CI/沙箱）
claude --bare -p "检查代码风格问题"

# 通过 SSH 在远程服务器运行
claude ssh myserver /path/to/project

# 启动 MCP 服务器
claude mcp serve

# 诊断系统状态
claude doctor
```
