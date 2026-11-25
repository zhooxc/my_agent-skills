---
name: win-omnicom
description: "Use when operating OmniCOM, the Windows serial workbench, from Linux through WinLinuxBridge (wl): locate the app and its CLI, discover COM ports and current serial settings, send/read/monitor, run scripted sessions, transfer files, toggle DTR/RTS, drive full-screen consoles, or operate its GUI. Discover ports, baud rates, and device commands per task; never hardcode them."
---

# 操作 OmniCOM（Windows 串口工作台）

## 这个技能是什么

- 对象是 **OmniCOM 软件本身**，不是某块板子、某个 COM 口。
- 端口号、波特率、设备指令、应答标记、安装路径、窗口数量都会变：**先发现，再操作**。
- 优先用 OmniCOM 的 CLI（机器接口，单 JSON）；GUI 只用于 CLI 没有的软件操作。
- Linux 侧通过 WinLinuxBridge 的 `wl` 到 Windows；不要从 Linux 反向连 Windows 端口。

## 0. 定位 OmniCOM 与 CLI

```bash
WL=~/WinLinuxBridge/linux/wl
"$WL" status
"$WL" ps | grep -i omnicom
```

如果窗口在运行，取进程路径：

```bash
"$WL" run --json --exe powershell -- -NoProfile -Command 'Get-Process | Where-Object { $_.MainWindowTitle -match "OmniCOM" } | Select-Object -First 1 Id,Path,MainWindowTitle | ConvertTo-Json -Compress'
```

判定：

- 路径以 `\runtime\pythonw.exe` 结尾 → 便携包。安装目录 `OM` 是 `runtime` 的父目录的父目录；CLI 是
  `"$OM\runtime\python.exe" "$OM\OmniCOM.pyz"`。
- 路径是普通 `python.exe`，且命令行含 `run.py` / `-m omnicom.cli` → 源码运行。`OM` 为仓库根；CLI 用 `python -m omnicom.cli`。
- 没有窗口 → 向用户要安装/源码目录；不要全局搜盘。

验证：

```bash
"$WL" run --json --exe "$OM\runtime\python.exe" --cwd "$OM" -- "$OM\OmniCOM.pyz" cli info
```

便携包的 CLI **必须**用 `runtime\python.exe`；`OmniCOM.cmd` 是 `start` 出去的无控制台进程，stdout 会丢。
要启动 GUI，用 `"$WL" start "$OM\OmniCOM.cmd"`。

首次发现后，把 `OM` 的绝对路径记在任务笔记里；后续 bash 调用直接写该路径。

## 1. 先发现，不假设

每次任务按顺序确认：

1. **端口**：`cli ports`，按 `description` / `VID` / `PID` / `serial` 匹配；有多个候选就问用户。
2. **窗口与占用**：`cli info`（窗口列表、PID、串口、connected）；需要时 `cli state --port <PORT>`。
3. **串口参数**：窗口已打开该口时，`cli lines --port <PORT>` 返回的 `opts` 是实际参数；否则参数由用户/项目提供，不要猜。
4. **设备指令与应答**：由用户/项目文档提供；没有就先 `read` / `monitor` 观察，不要假设 AT/OK。
5. **数据目录**：`cli info` 的 `data_dir`；CLI 与窗口必须一致，否则 `via` 会退回 `direct`，界面上看不到。

## 2. CLI 能力地图

| 想要 | 子命令 |
| --- | --- |
| 环境、版本、窗口、数据目录 | `info` |
| 列串口 | `ports` |
| 发一条并等应答 | `send` |
| 读一段（可记录） | `read` |
| 按行流式输出 | `monitor` |
| 多步脚本 | `session` |
| 原样发送文件 | `send-file` |
| X / Y / ZMODEM | `transfer` |
| 读/改 DTR、RTS | `lines` |
| 复位脉冲 | `reset` |
| 读全屏终端字符网格 | `screen` |
| 发裸按键 | `keys` |
| 窗口日志 / 统计 / 状态 / 关串口 | `log` / `stats` / `state` / `close` |
| 无硬件自检 | `selftest` |

精确参数以 `cli <子命令> --help` 为准；完整参考在 `$OM\CLI.md`（可用 `wl run` 读）。

## 3. 调用模板

```bash
"$WL" run --json --exe "$OM\runtime\python.exe" --cwd "$OM" -- "$OM\OmniCOM.pyz" cli <子命令> ...
```

- `--json` 包一层任务元数据；取内层 JSON：`... --json ... | jq -r '.stdout.text' | jq .`。
- 文本模式直接打印 OmniCOM 的 JSON；自动化优先 `--json`。
- 所有子命令共用串口参数：`--port`、`--baud`、`--bytesize`、`--stopbits`、`--parity`、`--flow`、`--dtr/--rts`、`--open-timeout`、`--lock-timeout`、`--direct`、`--window`。

## 4. 窗口模式、寻址与锁

- 只要 OmniCOM 窗口在跑，CLI 就把请求交给窗口：优先已打开该口的窗口，其次空闲窗口，最后才自己直连。
- `--window <PID>` 钉死某个窗口；`--port` 按串口选。
- `--direct` 强制 CLI 自己开串口：不碰窗口、界面看不到、`screen/keys/log` 不可用。
- 窗口关掉串口时，正在跑的 CLI 会话会立即报错结束；重新发命令即可重新绑定。
- 两个 CLI 同时操作同一串口会在 `<data>/locks/<PORT>.lock` 上排队（`--lock-timeout`，默认 20 秒）。
- 多窗口用 `cli info` 看 PID/slot/serial；CLI 与窗口必须共用同一个 `omnicom-data`（`--cwd` 或 `OMNICOM_DATA_DIR`）。

## 5. 串口工作流（全部用占位符）

```bash
# 1) 发现
"$WL" run --json --exe "$OM\runtime\python.exe" --cwd "$OM" -- "$OM\OmniCOM.pyz" cli ports
"$WL" run --json --exe "$OM\runtime\python.exe" --cwd "$OM" -- "$OM\OmniCOM.pyz" cli info

# 2) 只观察，不发送
"$WL" run --json --exe "$OM\runtime\python.exe" --cwd "$OM" -- "$OM\OmniCOM.pyz" cli monitor --port <PORT> --duration 5

# 3) 发一条命令并等设备自己的标记
"$WL" run --json --exe "$OM\runtime\python.exe" --cwd "$OM" -- "$OM\OmniCOM.pyz" cli send --port <PORT> --baud <BAUD> --data "<CMD>" --line-end crlf --expect "<MARKER>" --wait 3 --drain-ms 100
```

- 端口已由窗口打开时，实际参数以窗口为准；要强制波特率/校验位，用 `--direct`（界面看不到）或 `session --opts`（见 `--help`）。
- 设备本来就不回话时，用 `--settle` 等静默；只有「没回话也算成功」才用 `--ok-if-silent`。
- 二进制命令用 `--hex "01 03 ..."`；不要拼 shell 引号，优先 `--exe` 的 argv。
- 多端口并行：`--ports A,B`（NDJSON，每口一行）。
- 高频采样：`--repeat N --interval S`（NDJSON，每行带 `sample`）。

## 6. 脚本化 session

`session` 一次打开串口，按顺序执行，适合「复位 → 等启动 → 发命令 → 抓输出」。

```bash
"$WL" put ./steps.json shared/to_win/steps.json
"$WL" run --json --exe "$OM\runtime\python.exe" --cwd "$OM" -- "$OM\OmniCOM.pyz" cli session --port <PORT> --script 'C:\Users\<you>\WinLinuxBridge\shared\to_win\steps.json'
```

步骤字段：`note`、`delay`、`set`、`line`、`lines`、`pulse`、`send` / `send_hex`、`expect` / `expect_regex`、`drain`、`record`、`send_file`、`transfer`、`reset_input`。
默认遇错即停；要验证多个失败步骤就拆成多个脚本。脚本文件必须在 Windows 可读的路径（共享目录最稳）。

## 7. 长任务与输出预算

```bash
ID=$("$WL" run --no-wait --json --exe "$OM\runtime\python.exe" --cwd "$OM" -- "$OM\OmniCOM.pyz" cli monitor --port <PORT> --duration 300 | jq -r .job_id)
"$WL" job status "$ID" --json
"$WL" job logs   "$ID" --since 0 --json
"$WL" job result "$ID" --output summary
"$WL" job cancel "$ID"
```

- `wl run` 默认执行超时 120 秒；DSH 的 bash 默认 120 秒、单次上限 600 秒。
- 前台命令加 `--timeout 60`；更长一律 `--no-wait` + `job status/logs`。
- 单流采集上限 32 MB，超出原地截断（`capture_truncated:true`）；OmniCOM 用 `--limit`，wl 用 `--output summary` 或游标。
- `monitor`、`send --repeat`、`send --ports` 是 NDJSON：逐行解析，不要把整段塞进上下文。

## 8. 文件、数据目录与 D:\

- `wl put/get/ls/rm` 只允许 `%USERPROFILE%` 以内，越界 403；单文件 64 MB。
- OmniCOM 的 `data_dir` 可能在 `D:\` 等用户目录之外，`wl get` 读不到。要取记录/接收文件：先用 PowerShell 复制到 `%USERPROFILE%\WinLinuxBridge\shared\from_win`，再 `wl get` 或等 30 秒同步。
- 传固件/脚本：`wl put` 到 `shared/to_win`，CLI 的 `--file` / `--script` 用共享目录的绝对路径。
- 接收文件用 `--save-dir`；注意目标目录可写。
- `OMNICOM_DATA_DIR` 可以把数据目录放到别处；CLI 与 GUI 必须用同一个值，否则互相找不到。

## 9. GUI（CLI 没有的软件操作）

GUI 适合：首选项、主题/壁纸、命令库编辑、插件商店、软件更新、重置布局、多窗口/快捷启动。
串口收发不要用 GUI，CLI 更稳定、可判定。

```bash
"$WL" ps | grep -i omnicom
"$WL" focus "OmniCOM"
"$WL" shot /tmp/omnicom.png
# 用 read_image 查看截图后再决定坐标
"$WL" click <X> <Y>
"$WL" type "<TEXT>"
"$WL" key CTRL+S
```

要求 Windows 已登录且未锁屏；锁屏时截图黑屏、点击无效。坐标易碎，每次操作前后截图确认。

## 10. 常见失败

- `ok:false` 但桥接正常：业务失败，读 `error`；不要只看 `exit_code`。
- `silent:true`：一个字节都没收到，不是成功。
- `matched:false`：`--expect` 没等到；先 `monitor` 看原始字节，再调 `--drain-ms`、`--settle`、`--wait`。
- `被其他程序占用`：窗口或别的串口工具占着口；`cli info` 看窗口端口，先关占用方。
- `via:"direct"` 但预期是窗口：CLI 与窗口的数据目录不一致；检查 `--cwd` / `OMNICOM_DATA_DIR`。
- 全屏程序（U-Boot、menuconfig、vim）的日志是乱码：用 `screen` 读网格、`keys` 发按键。
- 桥接 `125` / `124`：`wl status` 看隧道；`124` 是等待超时、状态未知，先 `job status`，不要盲目重跑。
- 重试有副作用的操作先加 `--request-id`，避免重复复位/烧录。

## 11. 安全红线

- 不读取、不打印 `~/WinLinuxBridge/config/bridge.env` 的令牌；不写进脚本或共享目录。
- 烧录、擦除、重启、DTR 复位、关闭串口等操作，先向用户确认。
- `wl job cancel` 只取消自己发起的任务，不清理不认识的进程。
- 大日志、固件不进上下文，用文件路径 + 摘要字段。

## 12. 验证清单

- `wl status` → `tunnel: OK`
- `cli info` → `ok:true`，版本、`data_dir`、窗口状态符合预期
- `cli ports` → 目标口存在（按描述/序列号确认）
- 一次只读观察（`monitor` / `read`）成功
- 一次带设备标记的 `send` 成功，或明确记录 `ok:false` 的业务原因
- `session` / `transfer` / `reset` 等有副作用的操作，执行前已确认
- 用 GUI 时，已截图确认窗口和坐标
