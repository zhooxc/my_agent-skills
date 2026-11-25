# 分支管理规范（dsh-skills）

## 1. 分支一览

| 分支 | 用途 | 来源 | 合入目标 | 能否直接 push |
| --- | --- | --- | --- | --- |
| `main` | 稳定可发布，用户直接取用 | 从 `dev` 合入 | 打 tag 发布 | 否（保护） |
| `dev` | 日常集成，跑通后再发版 | 从 `main` 切出，长期存在 | `main`（发版时） | 否（PR） |
| `feature/<scope>-<short>` | 新 Skill / Skill 大改 | 从 `dev` 切 | `dev` | 是（个人分支） |
| `hotfix/<issue>` | 已发布 Skill 的紧急修复 | 从 `main` 切 | `main` + `dev`（都要合） | 是（个人分支） |

命名示例：`feature/obsidian-jira-name`、`feature/omnicom-batch`、`hotfix/obsidian-shared-path`。

## 2. 日常流程

新增/改 Skill（最常用）：

```bash
git checkout dev && git pull
git checkout -b feature/<scope>-<short>
# 改 skills/<name>/SKILL.md，本地用 skill 工具重载验证
git add skills/<name>/SKILL.md
git commit -m "<name>: <一句话说明>"
git push -u origin feature/<scope>-<short>
# 在 GitHub 开 PR：feature/... -> dev，写清验证步骤
```

发版（攒一批后）：

```bash
git checkout dev && git pull
git checkout main && git merge --no-ff dev
git tag -a v0.x.y -m "v0.x.y: <本次内容>"
git push origin main --tags
# dev 如落后 main，反向同步一次
git checkout dev && git merge --no-ff main && git push
```

紧急修复：

```bash
git checkout main && git pull
git checkout -b hotfix/<issue>
# 修完分别 PR/merge 到 main 和 dev
```

## 3. GitHub 建库后必做的 3 个设置

仓库：`zhooxc/dsh-skills`，默认分支 `main`。

1. **默认分支**：Settings → General → Default branch → `main`。
2. **分支保护 `main`**：Settings → Branches → Add rule → Branch name pattern `main`：
   - 勾选 Require a pull request before merging
   - 勾选 Require approvals（1 个即可，个人库可设 0，但建议保留 PR 流程）
   - 勾选 Dismiss stale pull request approvals when new commits are pushed
   - 不要勾选 Allow force pushes / Allow deletions。
3. **分支保护 `dev`**（轻量）：同上对 `dev` 加一条规则，只勾 Require a pull request before merging。

## 4. Skill 目录规约（新增 Skill 时）

```text
skills/<kebab-case-name>/SKILL.md
```

- 目录名必须 kebab-case（`^[a-z0-9]+(?:-[a-z0-9]+)*$`）。
- `SKILL.md` 顶部 frontmatter 必须有 `name`（与目录同名）与一句话 `description`（路由用）。
- 正文写清：何时用、前置检查、一步步操作、失败处理、红线。
- 大改 SKILL.md 后，用 DSH 的 `skill` 工具重载验证可加载，再提交。

## 5. 当前版本线

- `v0.1.0`（计划）：`obsidian-vault-save` + `win-omnicom` 首次入库，
  对应 `main` 首个 tag。建库推送完成后打 tag。
