# Claude Desktop 多账号切换器

在多个 Claude Desktop 账号之间一键切换，不用反复登录登出。

## 问题背景

Claude Desktop 不支持多账号。如果你同时有个人号和工作号，每次切换都要登出再登录，Cowork 虚拟机状态也会丢失。

## 工作原理

Claude Desktop 是 Electron 应用，认证信息分散存储在多个浏览器存储中（不只是一个配置文件）。本工具切换时会交换**所有会话文件**（约 5MB），同时保持大体积文件共享：

**按账号切换的文件**（认证和会话状态）：
- `config.json` — OAuth 令牌
- `Local Storage/` — 本地存储（认证状态）
- `Network/` — Cookies、HSTS
- `Session Storage/`、`IndexedDB/`、`WebStorage/`
- `Preferences`、`DIPS`、`SharedStorage`、`ant-did`

**所有账号共享的文件**（不会被动）：
- `vm_bundles/` — 12GB+ Cowork 虚拟机（Hyper-V）
- `claude_desktop_config.json` — MCP 服务器配置
- `Cache/`、`Code Cache/`、`GPUCache/`

## 环境要求

- Windows 10/11
- Claude Desktop（Microsoft Store 版或独立安装版）
- PowerShell 5.1+

## 快速开始

```powershell
# 1. 在 Claude Desktop 中登录第一个账号

# 2. 保存为配置
.\claude-switcher.ps1 create personal

# 3. 在 Claude Desktop 中登出，登录第二个账号

# 4. 同样保存
.\claude-switcher.ps1 create work

# 5. 以后随时切换！
.\claude-switcher.ps1 switch personal
.\claude-switcher.ps1 switch work
```

## 命令一览

| 命令 | 说明 |
|------|------|
| `create <名称>` | 将当前登录状态保存为一个配置 |
| `switch <名称>` | 切换到指定配置 |
| `list` | 列出所有配置，显示当前激活的 |
| `current` | 显示当前激活的配置名 |
| `repair` | 修复切换后 Cowork 虚拟机无法启动的问题 |

## 设置快捷命令（可选）

在 PowerShell 配置文件（`$PROFILE`）中添加别名，实现快速切换：

```powershell
function ppp { & "C:\你的路径\claude-switcher.ps1" switch personal }
function www { & "C:\你的路径\claude-switcher.ps1" switch work }
function ccc { & "C:\你的路径\claude-switcher.ps1" list }
```

## 常见问题

### Cowork 报错 "Failed to start Claude's workspace"

切换后 `sessiondata.vhdx`（Cowork 虚拟机会话磁盘）可能丢失。运行：

```powershell
.\claude-switcher.ps1 repair
```

会用 `diskpart` 重建 VHDX 文件，弹出管理员权限确认框时点"是"。

**千万不要删 `vm_bundles/` 重新下载** —— 那是 12GB+ 的东西，而且重装也不会重建 `sessiondata.vhdx`。

### 切换后账号没变

确保 Claude Desktop **完全关闭**（不是最小化）。脚本会提示你右键系统托盘图标 → Exit。

### 提示"无法加载脚本，因为禁止运行脚本"

用管理员身份打开 PowerShell，执行一次：

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

## 配置文件存储位置

```
~\.claude-instances\
├── personal\          # 账号 A 的会话文件
│   ├── config.json
│   ├── Local Storage\
│   ├── Network\
│   └── ...
├── work\              # 账号 B 的会话文件
│   ├── config.json
│   ├── Local Storage\
│   ├── Network\
│   └── ...
└── _current_profile   # 记录当前激活的配置名
```

## 为什么不能只换 config.json？

最初的版本就是只换 config.json，结果切换后账号不变。因为 Claude Desktop（和所有 Electron 应用一样）把认证状态缓存在多个浏览器存储里。只换 config.json 的话，旧的会话还留在 Local Storage 和 Cookies 中，Claude 会忽略新的令牌继续用旧账号。必须**所有会话文件一起换**。

## 许可证

MIT
