---
name: obsidian-vault-save
description: "Summarize the current answer into markdown and save it to the Windows Obsidian vault D:\\Ian.ZHAO\\Documents\\Quectel_MCU per vault routing, naming, tags and index rules via WinLinuxBridge."
---

# 存档当前回答到 Obsidian Quectel_MCU 仓库

把**本会话最近一次有效回答**提炼成一篇笔记，按库规约放到 Windows 侧 `D:\Ian.ZHAO\Documents\Quectel_MCU`。
Linux 侧不可直写 D 盘，必须经 WinLinuxBridge（`wl`）中转。

## 0. 前置检查（每次必做）

```bash
WL=~/WinLinuxBridge/linux/wl
"$WL" status
# 要求 tunnel: OK
"$WL" run --json --exe powershell -- -NoProfile -Command '[Console]::OutputEncoding=[Text.Encoding]::UTF8; Test-Path -LiteralPath "D:\Ian.ZHAO\Documents\Quectel_MCU"'
```

- `tunnel` 非 OK → 停下，先修桥，不猜。
- 只写 `.md` 笔记**不用**退出 Obsidian，改完提醒用户 `Ctrl+R`；绝不碰 `.obsidian/`。
- 只新增、不删除 / 不改正文 / 不重命名已归档笔记。

## 1. 提取源回答

- 源 = 本会话最近一次助手有效回答全文；若有多轮，只取最后一次与用户问题相关的完整回答。
- 提炼要素：结论先行、现象、硬件/模组、SDK/固件分支版本、复现步骤、关键日志（节选）、已尝试项、结论与验证状态；区分**已验证事实 / 推测 / 待办**。
- 脱敏：账号、密码、令牌、密钥、内部日志敏感段只留结论，不复述原文秘密，不外发。

## 2. 判定归属（按库规约）

目录地图：

- `00-收件箱/`：暂时判不准时放这里，并注明待分类
- `10-日常/`：日报、待办
- `20-模组/<模组>/`：FCM360W · FCM362K · FCM962K · FCMH65D · FGM840R · RTL8721F
- `30-客户项目/<客户>/`：海尔 · 迪奇普 · 光鉴 · 鼎匠 · EDV · 芯云 · 图力换 · 宏天微 · 顾家
- `40-流程与工艺/<Git|编译与环境|烧录与产测>/`：Git、编译环境、烧录产测
- `50-参考资料/<主题索引|芯片与手册|通用>/`：跨模组主题、手册
- `99-attachments/`：仅放图片/日志/代码附件，正文用 `![[名]]` 或相对路径引用

判定顺序：

1. 文中出现 JIRA/FAE 号（如 `FAE-254544`、`220412`）→ 仍按模组/客户归目录，JIRA 号进文件名（见第 3 章）。
2. 出现模组关键词 → `20-模组/<模组>/`；同时出现客户 → 以客户项目优先还是模组优先拿不准时，选更具体的一边，另一边用 `客户/` 标签 + 索引链接补上，并在报告里说明。
3. 纯流程（编译/烧录/Git）→ `40-流程与工艺/` 对应子目录；纯主题干货 → `50-参考资料/`。
4. 日报/待办 → `10-日常/`。
5. 都不命中 → `00-收件箱/`。

## 3. 命名与 frontmatter

命名（中文短语，一眼看出问题）：

- 通用：` <中文问题简述>.md`，模组通用文档自带模组名防同名，如 `AT指令（FCM962K）.md`、`连接32字符热点名称失败.md`、`R03分支SPI问题.md`
- **JIRA 问题必须带号**：`<JIRA号>（<中文简述>）.md`，如 `FAE-254544（FCM962K IPC USB固件主控DHCP获取IP失败）.md`；JIRA 号从回答或用户问题里提取，提取不到就问用户，不要编。
- 日报固定 `YYYY年MM月.md`；待办 `To-Do List YYYY-MM-DD.md`（历史命名，保持不动）。
- **全库禁止同名**（Obsidian 按 basename 解析，重名静默指错）。落盘前必查：

```bash
"$WL" run --json --exe powershell -- -NoProfile -Command '[Console]::OutputEncoding=[Text.Encoding]::UTF8; Get-ChildItem -LiteralPath "D:\Ian.ZHAO\Documents\Quectel_MCU" -Recurse -File -Filter "<basename>.md" | ForEach-Object { $_.FullName }'
```

有同名 → 文件名加模组或日期后缀，如 `xxx（FCM962K）.md`、`xxx-2026-11-25.md`，并在报告里说明。

frontmatter（保留既有格式，不加 `onenote-id`）：

```yaml
---
tags: [模组/FCM962K, 客户/海尔, 主题/休眠与低功耗]
created: 2026-11-25
source: DSH问答总结
jira: FAE-254544
---
```

- `模组/` 由所在目录决定，必加；`客户/` 客户项目笔记必加；`主题/` 从正文挑、最多 3 个（AT指令 · OTA与升级 · SPI与I2C外设 · WiFi连接与配网 · git与版本 · 休眠与低功耗 · 内存与资源 · 加密与安全 · 编译与构建）。
- 非 JIRA 笔记省略 `jira` 行；不要编造标签取值。

正文模板：

```markdown
---
tags: [...]
created: YYYY-MM-DD
source: DSH问答总结
---

# <与文件名一致的标题>

> 来源：本会话问答提炼；<已验证/部分待验证>；日期 YYYY-MM-DD

## 结论
...

## 环境
- 模组/硬件：
- SDK/固件分支与版本：
- 客户/项目（如有）：

## 现象 / 复现
...

## 关键日志（节选）
```text
...
```

## 已尝试 / 分析
...

## 待办
- [ ] ...

## 关联
- [[相关笔记名]]（有才写，没有不写）
```

- 代码围栏必须配对（偶数个 ``` ）；表格头下一行必须是 `| --- |` 且行间无空行；`AT+XXX=<mode>` 这类含 `<>` 的命令包在反引号里；附件用 `![[文件名.扩展名]]`。

## 4. 生成与中转落盘（推荐链路）

大文本不要直接拼进 powershell 命令行，用 `wl put` 中转：

```bash
# 1) Linux 侧先写临时文件（文件名与最终一致，UTF-8）
mkdir -p /tmp/obsidian-save
cat > /tmp/obsidian-save/"<文件名>.md" <<'EOF'
<全文>
EOF

# 2) 推到 Windows 共享（成功回显里会打出 Windows 侧绝对路径，记下它；本机实测为 D:\Fatmear\WinLinuxBridge\shared\to_win\）
"$WL" put /tmp/obsidian-save/"<文件名>.md" shared/to_win/"<文件名>.md"

# 3) Windows 侧移到 vault（示例目标为 FCM962K；按第 2 章替换目录）
#     不要硬编码共享目录：先按候选逐个 Test-Path，找到真实 $src
"$WL" run --json --exe powershell -- -NoProfile -Command '[Console]::OutputEncoding=[Text.Encoding]::UTF8; $cands=@("D:\Fatmear\WinLinuxBridge\shared\to_win\<文件名>.md", "$env:USERPROFILE\WinLinuxBridge\shared\to_win\<文件名>.md"); $src=$cands | Where-Object { Test-Path -LiteralPath $_ } | Select-Object -First 1; $dst="D:\Ian.ZHAO\Documents\Quectel_MCU\20-模组\FCM962K\<文件名>.md"; if(-not $src){ Write-Output "SHARED_NOT_FOUND"; exit 1 }; Move-Item -LiteralPath $src -Destination $dst -Force; Test-Path -LiteralPath $dst'
```

- `shared/to_win` 是受限共享目录（本机 Windows 侧实为 `D:\Fatmear\WinLinuxBridge\shared\to_win\`，以 `wl put` 成功回显为准）；不要试图 `wl put` 直写 D 盘 vault（403 越界）。
- 附件（如 `.png/.log`）同理 `wl put` 后移到 `99-attachments/`，正文保留 `![[...]]` 引用。
- 落盘后必验：`Test-Path` 为 True；读回前 20 行确认 frontmatter 与标题正常。

## 5. 自动更新索引（本技能默认做）

落盘成功后自动给**目标目录的 `* 索引.md`** 加一行，不改其他正文：

1. 先读索引原文（只读、不猜格式）。
2. 在表格末尾追加 `| [[<无扩展名basename>]] | <一句话摘要> |`，摘要来自本次结论（≤40 字）。
3. 主题页（`50-参考资料/主题索引/<主题>.md`）**只建议、不自动改**，把建议链接放在最终报告里。
4. 若目标是 `00-收件箱/`，不碰任何索引页，只报告建议路径与标签。

索引写入同样经 `wl put` + PowerShell 追加，追加前先读原文、保持无关内容不变；索引不存在或格式对不上时停下，只报告、不强写。

## 6. 最终报告（固定格式）

- 新文件：`D:\...\<文件名>.md` + Obsidian 链接 `[[<basename>]]`
- 归属依据：命中关键词 → 选择目录的一句话解释
- 标签：本次 `tags`
- 索引：已更新 `xx 索引.md（+1 行）` / 主题页建议链接 x 条
- 同名检查：全库 `*.md` 同名 x 个（0 或已加后缀说明）
- 提醒：Obsidian 内 `Ctrl+R` 刷新；收件箱笔记记得周五清单确认

## 7. 红线与常见失败

- 未经用户明确要求：不删除、不批量移动、不覆盖大量文件、不 `robocopy /MIR`、不改已归档正文。
- `wl` 超时 120 秒：大文件先 `--timeout 60`，更长用 `--no-wait` + `job status/logs`。
- 中文/emoji 文件名（如 `🏠 仓库说明.md`）一律 `-LiteralPath` + `-Encoding UTF8`，不要用通配。
- 落盘后 `Test-Path` 为 False → 读 `job logs`，不要盲重跑有副作用的移动。
