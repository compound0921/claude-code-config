# claude-code-config

Claude Code 个人配置备份：自建 skills 与可公开的 `settings.json`。

> Personal Claude Code setup — self-authored skills plus a sanitized settings file.

## 结构

```
settings.json          权限白名单、模型、插件市场配置（已移除私有仓库相关的权限项）
plugins/               已装插件与市场清单（仅索引，不含插件本体）
skills/                自建技能，每个目录一个 SKILL.md
```

## 技能

| 技能 | 用途 |
|---|---|
| `circuit-tutor` | 电路截图识题、规范求解、出题抽题、仿真验证 |
| `design-router` | 前端设计任务的路由与裁决（需配合 taste / impeccable / apple 三个技能包） |
| `web-vuln-analysis` | Web 漏洞分析四阶段流程：信息收集 → 漏洞分析 → 人工验证 → 结果整理 |
| `confirming-requirements` | 动手前用 AskUserQuestion 逐轮确认需求 |
| `confirming-file-creation` | 写文件前先列出路径与内容并征得同意 |
| `avoiding-over-testing` | 只测关键行为，不为低风险改动新增测试 |
| `concise-markdown` | 精简 Markdown 写作规范 |
| `git-workflow` | 提交、分支、推送、合并、Release 的规范流程 |

## 使用

把需要的技能目录复制到 `~/.claude/skills/`：

```bash
cp -r skills/circuit-tutor ~/.claude/skills/
```

`settings.json` 里的临时目录路径（`//tmp/**`、`AppData/Local/Temp`）按自己的系统调整后再用。

## 未包含

- 第三方技能：`multisim-spice`（from `zuoliangyu/multisim-spice`）、`web-security`、`pick-ui-library` 等，版权与许可归原作者。
- 会话记录、持久记忆、计划文档：含个人与项目信息，只在本地保留。
- `plugins/cache/` 与插件本体。

`circuit-tutor` 依赖的 `multisim-spice` 需另行安装，且其中的绝对路径要改成自己的安装位置。
