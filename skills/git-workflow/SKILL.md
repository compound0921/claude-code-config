---
name: git-workflow
description: >-
  处理 Git 提交、分支、推送、合并、标签、Release，或提到标准化/规范化/整理提交消息时使用。提交与推送前须列计划并获许可，多次提交一次列全；新功能用 feat/，修复与优化用 fix/，简单项目只用 develop 和 main；提交信息用 `type: 内容`；Release 说明与版本号按 references/ 规范。
---

# Git 工作流

## 何时启用

- 用户要求提交、推送、建/切/合并分支、打标签、发 Release。
- 用户要求写或修改 Release 说明、整理提交历史、拆分提交。
- 用户提到“标准化提交消息”“规范化提交消息”“整理提交信息”，或要求改写已有提交信息。

## 硬性规则

1. **提交、推送、打标签、发布 Release 前必须得到用户明确许可。** 用 `AskUserQuestion` 询问，不用普通文本问“可以吗”。
2. 一次许可只覆盖当次列出的计划。文件、分支、提交信息或提交数量有变，停下重新确认。
3. 要拆成多次提交时，**一次性列出全部提交的基本信息**，等用户同意后再按顺序执行，不要边提交边问。
4. 提交前先看 `git status`、`git diff`、当前分支和最近提交风格；只读命令（`status`/`diff`/`log`/`show`）无需许可。
5. 不跳过钩子（`--no-verify`）、不强推共享分支、不改写已推送历史，除非用户明确要求。
6. 钩子失败、测试失败、推送被拒时如实报告，不隐瞒，不自行绕过。

## 分支模型

### 复杂项目

| 分支 | 用途 |
|---|---|
| `main` | 随时可发布，只接受来自 `develop` 的合并 |
| `develop` | 集成分支，日常汇总 |
| `feat/<slug>` | 新功能，从 `develop` 切出，完成后合回 `develop` |
| `fix/<slug>` | 修复、优化、性能、重构、文档、构建等非新功能改动，从 `develop` 切出 |

- `<slug>` 用小写 kebab-case，描述改动内容，不加日期和人名。
- 一个分支只做一件事；混入无关改动时拆分支或拆提交。
- 新功能 → `feat/`；其余改动（修 bug、优化、重构、文档等）→ `fix/`。判断不了时按“用户能否感知到新能力”区分，仍不确定就问用户。
- 发布：`develop` 合入 `main`，在 `main` 打标签。

### 简单项目

- 只保留 `main` 和 `develop` 两个分支。
- 日常改动**直接提交到 `develop`**，不拉 `feat/`、`fix/`；提交同样需要许可。
- 发布时 `develop` 合入 `main`。
- 仓库已有 `feat/`、`fix/` 或多人协作流程时，沿用复杂模型，不要降级。

### 建分支前

- 先用 `git branch -a` 确认现状。缺少 `develop` 时，先说明再征得许可后创建，不要擅自建。
- 切分支前确认工作区状态；有未提交改动时先处理，不带着乱状态切分支。

## 提交许可流程

1. **只读检查**：当前分支、`git status --short`、`git diff`、最近 20 条提交的信息风格。
2. **设计拆分**：按逻辑单元拆，每个提交能独立理解、独立回滚；不按文件数硬拆。
3. **列出计划**（多次提交必须用表格列全）：

| # | 分支 | 文件 | 类型 | 提交信息 | 内容摘要 |
|---|---|---|---|---|---|
| 1 | `feat/login` | `src/auth.ts` | feat | `feat(auth): 增加密码登录` | 新增 `login()` 与校验 |
| 2 | `feat/login` | `src/app.ts` | feat | `feat(auth): 挂载登录路由` | 注册 `/login` |

4. **询问许可**：用 `AskUserQuestion`，选项为「按计划提交（推荐）／调整计划／暂不提交」。
5. **执行**：按列出的文件逐个 `git add`，再逐个 `git commit -m`；不用 `git add .`。
6. **汇报**：给出每个提交的 hash 和标题。
7. **问推送**：汇报后用 `AskUserQuestion` 给出推送选项，不要只在文字里说“没推送”，也不要默认推送。

单次提交也要列出文件和信息再询问，只是不必用表格。

### 推送询问

提交完成后的同一条回复里，必须用 `AskUserQuestion` 让用户选择，典型选项：

| 选项 | 含义 |
|---|---|
| 推送当前分支（推荐） | `git push`，无上游时用 `git push -u origin <branch>` |
| 推送并合并到 develop | 在 `feat/`、`fix/` 分支时，推送后合入 `develop` |
| 暂不推送 | 只保留本地提交 |

用户选“暂不推送”后不要再追问；下次提交或用户主动提起时再问。推送范围、分支与此时列出的一致，有变化重新确认。

### 提交信息

- 格式：`type(scope): 内容`。type 区分改动性质（`feat` / `fix` / `docs` / `refactor` / `perf` / `test` / `style` / `build` / `ci` / `chore` / `revert`），冒号后接具体内容；scope 可省略。详见 `references/commit-messages.md`。
- 仓库的第一个提交用 `Init` 开头，如 `Init: 初始化项目`；之后恢复 type 分类。
- 内容一句话说清做了什么，不加句号；禁止“更新代码”“修复问题”这类空话。
- 优先沿用仓库既有风格（看 `git log`）；没有约定时按上述格式。
- 一个提交只用一个 type，只做一件事。

### 标准化已有提交消息

用户要求标准化或规范化提交消息时：

1. `git log --oneline <范围>` 列出目标提交，逐条对照 `references/commit-messages.md`。
2. 用表格给出旧信息 → 新信息，并标出是否已推送。
3. 用 `AskUserQuestion` 确认后执行。只改写未推送的提交；已推送历史默认不动，用户明确要求时先说明影响和协作风险。
4. 最新一条用 `git commit --amend`；多条用 `git rebase` 的 reword（交互不可用时用环境支持的非交互方式）。
5. 改写前确认工作区干净；改写后确认文件内容未变，并汇报新旧 hash。

## 推送与合并

- `git push` 同样需要许可；每次提交完成后用 `AskUserQuestion` 把推送作为选项交给用户，不默认推送，也不用文字提问代替。
- 首次推送用 `git push -u origin <branch>`。
- 合并到 `develop` 或 `main` 需要许可；仓库使用 PR 时优先走 PR。
- 不向共享分支强推。用户要求强推时，先说明影响并要求二次确认。

## 发布与 Release

1. **定版本号**：按 `references/versioning.md`。用 Release 说明的小节反推——有「新增」或「变更」就升 MINOR，只有「修复」才是 PATCH；**PATCH 不得包含新功能或默认值变化**；配置不兼容、协议变化、提高系统要求才升 MAJOR。
2. **一个 MINOR 一个主题**：主题说不清就拆版；功能在 `develop` 上攒，不为每个 PR 发版。
3. **发布前检查**：工作区干净、构建和测试通过、`git log <上一个标签>..HEAD` 已梳理、没有遗漏的未合并分支。
4. **合并与打标签**：MINOR/MAJOR 由 `develop` 合入 `main`；PATCH 热修从 `main` 切 `fix/<slug>`，修完合回 `main` 再回合 `develop`。创建带注释标签 `vX.Y.Z`；这些操作也要许可。
5. **写说明**：按 `references/release-notes.md` 的格式起草，先展示给用户确认。
6. **发布**：`gh release create` 是对外操作，即使说明已确认，也要再次获得许可。可用 `--notes-file` 传入草稿；是否先建 `--draft` 按用户偏好。预发布用 `vX.Y.Z-beta.N`，不合并 `main`。

## 禁止

- 未经许可执行 `git commit`、`git push`、`git tag`、`gh release create`。
- 提交后用一句“没有推送”代替推送选项，或默认替用户推送。
- 把 `git add .` 当默认做法，或提交密钥、构建产物、无关改动。
- 在 `main` 上直接提交新功能或修复。
- 把多次提交拆成多轮询问，边做边问。
- 把 Release 说明写成 commit 列表、内部重构记录或营销文案。
- 用“提交完成”掩盖钩子失败或测试失败。

## 参考

- `references/commit-messages.md` — 提交信息的 type 分类、写法、示例与分支对应。
- `references/release-notes.md` — Release 说明的结构、写法、模板与检查清单。
- `references/versioning.md` — 版本号定义、反推方法、发布节奏、热修流程与反模式。
