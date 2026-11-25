# dsh-skills

DeepSeek Harness（DSH）个人 Skill 仓库：存放可在多设备、多会话复用的 Skill。
每个 Skill 即一个目录（含 `SKILL.md`），遵循 DSH 本地 Skill 规范（kebab-case 目录名）。

## 目录

```text
dsh-skills/
  skills/
    obsidian-vault-save/   # 当前回答总结成 md，按库规约存入 Windows Obsidian 仓库
    win-omnicom/           # 经 WinLinuxBridge 操作 Windows 串口工作台 OmniCOM
  docs/
    BRANCHING.md           # 分支管理规范（必读）
```

## Skill 一览

| Skill | 用途 | 来源 |
| --- | --- | --- |
| `obsidian-vault-save` | 把当前回答提炼成 md，经 `wl` 中转存入 `D:\Ian.ZHAO\Documents\Quectel_MCU`，按模组/客户/流程归属、命名、标签、索引规则落地 | 本仓库 `skills/obsidian-vault-save/SKILL.md` |
| `win-omnicom` | 在 Linux 侧经 WinLinuxBridge 操作 OmniCOM：发现端口/窗口、收发/monitor/session、传文件、DTR/RTS、GUI 兜底 | 本仓库 `skills/win-omnicom/SKILL.md` |

## 安装（取用）

全局生效（推荐，当前两 Skill 的存放方式）：

```bash
cp -r skills/obsidian-vault-save ~/.dsh/skills/obsidian-vault-save/
cp -r skills/win-omnicom ~/.dsh/skills/win-omnicom/
```

仅当前项目生效：

```bash
mkdir -p <projectRoot>/.dsh/skills
cp -r skills/<name> <projectRoot>/.dsh/skills/<name>/
```

DSH 本地 Skill 发现优先级：`project-dsh (.dsh/skills)` > `user-dsh (~/.dsh/skills)`。
同名 Skill 以高优先级目录的为准。

## 分支管理（摘要）

- `main`：稳定可发布分支，只接受从 `dev` 的 PR，不直接 push。
- `dev`：集成分支，日常合入点。
- `feature/<scope>-<short>`：新 Skill / 大改从 `dev` 切，改完 PR 回 `dev`。
- `hotfix/<issue>`：线上 Skill 坏了从 `main` 切，修完同时合回 `main` + `dev`。
- 发布：`main` 上打 tag（`v0.1.0` 起，semver），每个 Skill 目录变更在 PR 里写明验证步骤。

完整规范见 [docs/BRANCHING.md](docs/BRANCHING.md)。
首次在 GitHub 建库后，请按该文档把 `main` 设为默认分支并打开分支保护（`docs/BRANCHING.md` 有截图路径级步骤）。
