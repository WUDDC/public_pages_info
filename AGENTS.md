# AGENTS.md
- 尽量使用简体中文回复
- 尽量使用codebuddy的hy3作为模型（免费）
- 尽量同时开多个subagent执行任务，加快开发速度

## CodeBuddy CLI

本机装有 **CodeBuddy Code CLI**(腾讯,别名 `codebuddy` / `cbc`),通常**不在 PATH 上**,需用完整路径调用。

**定位可执行文件**(不同机器/系统路径不同):
- Windows: `%LOCALAPPDATA%\codebuddy\bin\codebuddy.exe`
  (本机实测:`C:\Users\<user>\AppData\Local\codebuddy\bin\codebuddy.exe`)
- macOS/Linux: 一般在 `~/.codebuddy` 或包管理器安装目录,`which codebuddy` 查
- 配置目录统一在 `~/.codebuddy/`

**调用方式**
```bash
# 非交互(脚本/管道用)
codebuddy -p --model <model-id> "your prompt"
# 结构化输出
codebuddy -p --output-format json --model <model-id> "..."
# 交互
codebuddy
```
- 需先登录:交互模式里 `/login`。
- `-p` 模式若卡权限提示,加 `-y` 或 `--permission-mode auto` / `dontAsk`。
- 限制 agent 轮数:`--max-turns N`。

### 可用模型:怎么查、怎么知道更新了

**别信 `codebuddy --help` 里 `--model` 那串**——那是硬编码提示(含 gemini/gpt-5.x 等),和账号真实授权对不上(实测 `gpt-5.5` 报 "only available for authorized users")。

权威来源(可靠度从高到低):
1. **交互 `/model` 面板** —— 最准,带授权状态 + credits 倍率。TUI,需人工查看。
2. **本地缓存文件**(可被 agent 直接读、可 diff 检测更新):
   `~/.codebuddy/local_storage/entry_*.info`(JSON)。字段 `data.agents[].models` 数组 = 账号授权的完整模型 ID 清单。
   ⚠️ **只在 CLI 重启/联网同步后刷新**,文件 mtime ≈ 上次启动时间;服务端新增模型但未重启则缓存仍是旧的。
3. `codebuddy -p --model <id> "ping"` —— 实测单个模型是否真能用。

**查/设默认模型**
```bash
codebuddy config get model          # 查当前默认
codebuddy config set model <id>     # 设默认(也可改 ~/.codebuddy/settings.json 的 "model")
```
没有 `list models` 子命令;`config list` 列全部配置,`ps` 列会话。

### 模型清单快照(2026-07-10,本账号实测)

全部为国产模型。`hy3` 为默认(混元,internal Usage Billing,x0.00 credits)。

| 显示名 | `--model` ID | credits |
|---|---|---|
| Hy3(默认) | `hy3` | x0.00 |
| GLM-5.2 | `glm-5.2` | x0.79 |
| GLM-5.1 | `glm-5.1` | x0.79 |
| GLM-5.0 | `glm-5.0` | x0.80 |
| GLM-5.0-Turbo | `glm-5.0-turbo` | x0.95 |
| GLM-5v-Turbo | `glm-5v-turbo` | x0.95 |
| GLM-4.7 | `glm-4.7` | x0.23 |
| MiniMax-M3 | `minimax-m3` | x0.25 |
| MiniMax-M2.7 | `minimax-m2.7` | x0.19 |
| Kimi-K2.7-Code | `kimi-k2.7` | x0.57 |
| Kimi-K2.6 | `kimi-k2.6` | x0.52 |
| Kimi-K2.5 | `kimi-k2.5` | x0.45 |
| Deepseek-V4-Pro | `deepseek-v4-pro` | x0.16 |
| Deepseek-V4-Flash | `deepseek-v4-flash` | x0.06(最便宜) |
| DeepSeek-V3.2 | `deepseek-v3-2-volc` | x0.29 |

> 这是快照,可能已过期。判断是否更新:读上述缓存文件的 `data.agents[].models` 与本表对比。