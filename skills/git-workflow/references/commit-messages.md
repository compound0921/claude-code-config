# 提交信息命名

格式：`type(scope): 内容`。type 区分改动性质，冒号后接具体内容；scope 可省略。

**初始化提交**：仓库的第一个提交用 `Init` 开头，例如 `Init: 初始化项目`；之后恢复 `type: 内容` 格式。

## 类型

| type | 用途 | 示例 |
|---|---|---|
| `feat` | 新功能、新能力 | `feat(share): 增加窗口采集` |
| `fix` | 修复错误行为 | `fix: 修复托盘菜单无响应` |
| `docs` | 文档、注释、README | `docs: 补充端口映射说明` |
| `refactor` | 重构，不改变外部行为 | `refactor(capture): 拆分初始化逻辑` |
| `perf` | 性能优化 | `perf: 降低编码缓冲` |
| `test` | 测试 | `test(auth): 覆盖密码错误分支` |
| `style` | 格式调整，不影响逻辑 | `style: 统一缩进` |
| `build` | 构建脚本、依赖 | `build: 升级 ffmpeg 到 7.0` |
| `ci` | CI 配置 | `ci: 增加 Windows 构建` |
| `chore` | 杂项：依赖、配置、忽略文件、发布、脚本 | `chore(deps): 升级 vite 到 5.4` |
| `revert` | 回滚提交 | `revert: 回滚 1a2b3c4` |

## 写法

1. type 用小写；冒号用半角 `:`，后面空一格再接内容。
2. 内容一句话说清“做了什么”，用项目惯用语言；不加句号。
3. scope 可省略；写了就用小写模块名，如 `auth`、`capture`、`ui`。
4. 破坏性变更用 `type!:`，或在正文写 `BREAKING CHANGE:`。
5. 禁止“更新代码”“修复问题”“优化”这类空话；信息要能脱离 diff 读懂。
6. 一个提交只用一个 type；混合改动就拆提交。
7. 正文说明原因和影响，不重复 diff。

## chore 的常见范围

- 依赖与版本：`chore(deps): 升级 vite 到 5.4`
- 仓库配置：`chore(config): 调整 ESLint 忽略项`
- 忽略与清理：`chore: 清理未使用的资源文件`
- 发布与版本号：`chore(release): 发布 v1.8.0`
- 开发脚本：`chore(scripts): 增加本地构建脚本`
- 锁文件与工具链：`chore: 同步 pnpm-lock.yaml`

边界：影响构建产物用 `build`，CI 配置用 `ci`，纯文档用 `docs`，格式调整用 `style`。

## 与分支对应

| 分支 | 常用 type |
|---|---|
| `feat/<slug>` | `feat`，必要时 `test`、`docs` |
| `fix/<slug>` | `fix`、`refactor`、`perf`、`docs` |
| 热修 `fix/<slug>`（从 `main`） | `fix` |
| 简单项目的 `develop` | 按改动性质选 type |

## 示例

```
Init: 初始化项目
feat(auth): 增加密码登录
fix: 修复 HDMI 接入后无声音
docs: 补充局域网连接说明
refactor(capture): 拆分采集源初始化
perf: 降低推流端编码缓冲
chore: 升级 ffmpeg 到 7.0
```

反例：

```
update code                     # 没说改了什么
fix bug                         # 哪个 bug
feat: 增加登录并重构整个鉴权模块   # 两件事，拆开
```
