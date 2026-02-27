# Claude Profile Switcher

Switch between multiple Claude Desktop accounts without logging in/out each time.

## The Problem

Claude Desktop doesn't support multiple profiles. If you have both a personal and work account, you need to logout and re-login every time you switch — losing your Cowork VM state, session data, and sanity.

## How It Works

Claude Desktop is an Electron app that stores authentication in multiple browser-like storages (not just a single config file). This tool swaps **all session files** (~5MB) between profiles while keeping the heavy stuff shared:

**Swapped per profile** (auth & session state):
- `config.json` — OAuth token
- `Local Storage/` — localStorage (auth state)
- `Network/` — Cookies, HSTS
- `Session Storage/`, `IndexedDB/`, `WebStorage/`
- `Preferences`, `DIPS`, `SharedStorage`, `ant-did`

**Shared across profiles** (never touched):
- `vm_bundles/` — 12GB+ Cowork VM (Hyper-V)
- `claude_desktop_config.json` — MCP server config
- `Cache/`, `Code Cache/`, `GPUCache/`

## Requirements

- Windows 10/11
- Claude Desktop (Microsoft Store or standalone)
- PowerShell 5.1+

## Quick Start

```powershell
# 1. Login to your first account in Claude Desktop

# 2. Save it as a profile
.\claude-switcher.ps1 create personal

# 3. Logout in Claude Desktop, login to second account

# 4. Save that too
.\claude-switcher.ps1 create work

# 5. Switch between them anytime!
.\claude-switcher.ps1 switch personal
.\claude-switcher.ps1 switch work
```

## Commands

| Command | Description |
|---------|-------------|
| `create <name>` | Save current Claude login as a named profile |
| `switch <name>` | Switch to a saved profile |
| `list` | Show all profiles and which is active |
| `current` | Show the active profile name |
| `repair` | Fix Cowork VM if it breaks after switching |

## PowerShell Aliases (Optional)

Add to your PowerShell profile (`$PROFILE`) for quick switching:

```powershell
function ppp { & "C:\path\to\claude-switcher.ps1" switch personal }
function www { & "C:\path\to\claude-switcher.ps1" switch work }
function ccc { & "C:\path\to\claude-switcher.ps1" list }
```

## Troubleshooting

### Cowork shows "Failed to start Claude's workspace"

This happens when `sessiondata.vhdx` (Cowork's VM session disk) is missing or corrupted. Run:

```powershell
.\claude-switcher.ps1 repair
```

This uses `diskpart` to recreate the VHDX file. A UAC admin prompt will appear — click Yes.

**Do NOT delete `vm_bundles/` and reinstall** — that's 12GB+ of re-downloading and the reinstall won't recreate `sessiondata.vhdx` anyway.

### Profile switch didn't change account

Make sure Claude Desktop is **fully closed** before switching (not just minimized). The script will prompt you to right-click the tray icon → Exit.

### "cannot be loaded because running scripts is disabled"

Run this once in an admin PowerShell:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

## How Profiles Are Stored

```
~\.claude-instances\
├── personal\          # Profile A session files
│   ├── config.json
│   ├── Local Storage\
│   ├── Network\
│   └── ...
├── work\              # Profile B session files
│   ├── config.json
│   ├── Local Storage\
│   ├── Network\
│   └── ...
└── _current_profile   # Text file with active profile name
```

## Why Not Just Swap config.json?

That was v1. It didn't work because Claude Desktop (like all Electron apps) caches auth state in multiple browser storages. Swapping only `config.json` leaves the old session in Local Storage and Cookies, so Claude ignores the new token and stays on the old account. You need to swap **all** session files.

## License

MIT
