---
name: web-vuln-analysis
description: Web 漏洞分析四阶段工作流。当用户提出 Web 漏洞分析、漏洞排查、漏洞扫描、渗透测试、安全评估或复测指令，或要求“看看这个站有什么漏洞”时使用；按信息收集→漏洞分析→人工验证→结果整理推进，强制执行授权范围、Kali 执行、非破坏性验证与证据留存。
---

# Web 漏洞分析

对已授权 Web 目标做非破坏性漏洞分析。四阶段顺序执行，前一阶段产物是下一阶段输入，不跳级。

## 0. 启动门禁

动手前输出并确认四项；缺一项不得发包：

| 项 | 要求 |
|---|---|
| 目标 | 具体 IP / 域名 / URL，确认在授权范围内 |
| 边界 | 允许与禁止的操作、时间窗口 |
| 环境 | 默认 Kali VM，经 `scripts/kali.sh` 执行 |
| 交付物 | 目录、格式、证据要求 |

- 无授权或目标不明：停止，先问用户。
- 有副作用、写入、爆破、对外发包：必须先确认。
- 只读、本地、可逆操作可按明示假设继续，并在输出中标注假设。

## 1. 信息收集

目标：形成完整资产清单，不做漏洞利用。

- 全端口 TCP + 常用 UDP；服务、版本、OS 识别。
- HTTP 指纹、响应头、Cookie、CORS、错误页、公开页面。
- 按需做子域、DNS、证书、公开情报。
- 端点优先从源码、jar、JS、接口文档静态提取，少用目录爆破。
- 只对公开 GET 做非破坏性状态验证；不读取或导出敏感数据。

交付：`01-recon/asset-inventory.md`、`asset-inventory.csv`、原始输出、`scope.md`。每条结论附证据文件路径。

## 2. 漏洞分析

目标：产出**疑似漏洞线索清单**，不是结论清单。

按类型读取专项方法（路径 `~/.claude/skills/web-security/skills/<name>/SKILL.md`，用 Read 执行，不要用 Skill 工具）：

| 类型 | 专项 skill |
|---|---|
| SQL 注入 | `exploiting-sql-injection-vulnerabilities`；`exploiting-sql-injection-with-sqlmap` 只确认注入点，禁止 dump |
| XSS | `testing-for-xss-vulnerabilities`、`testing-for-xss-vulnerabilities-with-burpsuite` |
| 越权 / IDOR | `testing-for-broken-access-control`、`exploiting-idor-vulnerabilities`、`testing-api-for-broken-object-level-authorization`、`exploiting-broken-function-level-authorization` |
| 信息泄露 | `exploiting-excessive-data-exposure-in-api`、`performing-api-inventory-and-discovery` |
| 文件上传 | 按 `performing-web-application-penetration-test` 的 WSTG 流程手工验证 |
| 逻辑漏洞 | `testing-for-business-logic-vulnerabilities`、`exploiting-race-condition-vulnerabilities` |
| 配置问题 | `performing-security-headers-audit`、`testing-cors-misconfiguration` |
| 线索分级 | `performing-web-application-vulnerability-triage` |

边界：允许在 Kali 内做注入探测与访问控制测试；禁止数据 dump、写数据库、口令/目录爆破、反弹 shell、上传 webshell。探测保持非破坏性；复现与人工判定留给第 3 阶段。

每条线索必填：ID（`VULN-001` 递增）、类型、端点/参数、疑似证据（脱敏，无证据标“仅特征推测”）、原理、CVSS 3.1（向量 + 分数）、修复建议、优先级、状态（`待人工验证` / `已确认` / `误报`）。

优先级：
- **P0**：有明确证据的高风险（未授权数据暴露、认证绕过、RCE、任意文件读写）。
- **P1**：强特征但缺直接证据（可疑参数、错误回显、版本匹配）。
- **P2**：仅指纹/版本推测、低影响或利用条件苛刻。
- 不求数量：高风险 + 有证据 > 低风险 + 推测。最终结论由第 3 阶段人工验证确定。

交付：`02-analysis/vuln-candidates.md` + 每条请求/响应证据。

## 3. 人工验证

目标：逐条核验。AI 只制定方案与判定结果，不代替发包。

1. **先给方案，命令行为主**：每个疑似项写成**可直接逐行执行的命令 runbook**，而不是散文式步骤。硬要求：
   - 一条命令占一行，不跨行、不用续行符；整段复制到 PowerShell 能顺序执行。
   - 每行命令后用 `# 预期:` 标注判据，执行者对照即可判断成败。
   - 请求、取证落盘（`-o <证据路径>`）、回滚都要有现成命令；不能让执行者自己拼参数。
   - 文字只保留前置条件、安全边界、成立判据与结果解释。
   - 浏览器只用于命令行覆盖不了的最终确认（如 DOM XSS 实际执行、验证码图片比对），并明确标注“唯一非命令行步骤”。
   - 模板：

     ```powershell
     # 1) <这一步验证什么>
     curl.exe -s -X POST "$B/api/xxx" -H "Content-Type: application/json" -d '{"k":"v"}' -o "$E\vuln-00X-response.txt" -w "%{http_code}\n"
     # 预期: 200，响应含 "success":true
     ```
2. **用户执行**：用户在浏览器或 Kali 内手动执行；AI 不运行针对目标的验证请求（`curl`、`sqlmap`、浏览器自动化等）。
3. **回填判定**：用户回填请求/响应、截图或输出后，AI 判 `确认` / `排除` / `需补充验证` / `疑似`。
4. **证据门槛**：可复现、影响明确、无其他解释才标 `确认`；其余保持 `疑似`。
5. **写操作保护**：写入、删除、重置、上传、并发竞态使用一次性账号/对象，验证前做数据库快照。
6. **证据留存**：归档到 `03-verification/evidence/`，报告脱敏。

交付：`verification-plan.md`、`results.md`、`evidence/`。

## 4. 结果整理

目标：基于**已确认**漏洞生成《渗透测试报告》（Markdown）。

报告结构：
1. 目标概述（授权范围、时间窗口、环境、测试限制）
2. 资产清单（引用第 1 阶段）
3. 漏洞汇总表：编号 / 名称 / 危害等级 / 位置 / 状态
4. 每个已确认漏洞：描述、影响、复现步骤、请求/响应证据、CVSS、修复建议
5. 按优先级排列的修复建议
6. 安全观察项
7. 附录：疑似项与未验证项

规则：
- 正文只收录 `确认` 漏洞；`疑似`、`需补充验证`、`排除` 一律放附录。
- 每条结论可追溯到 `03-verification/evidence/`。
- 口令、哈希、Token、手机号、邮箱脱敏。
- 输出 `04-report/report.md`；生成后人工复核再提交。

## 目录

```
engagements/<项目>/
├── scope.md
├── 01-recon/
├── 02-analysis/
├── 03-verification/
├── 04-report/
└── evidence/
```

已有项目可沿用 `recon/`、`scan/`、`exploit/`、`report.md`，对应 `01`–`04`。产物先落 Kali `/root/engagements/<项目>/`，再用 `get` 同步回本地。

## 硬性约束

- 所有扫描、枚举、注入、利用命令在 Kali VM 内执行，经 `scripts/kali.sh`；Windows 只编排、编辑、存档。
- 不在 Windows 安装/运行扫描器或攻击载荷，不用 Windows 原生工具向目标发包。
- 限速：`nmap -T3`；`masscan --rate <= 1000`；`hydra -t <= 4`；禁止 DoS 与 HTTP 泛洪。
- 非破坏性优先：读文件、回显、带外确认；不写 webshell、不改删数据。
- 保留原始命令、时间戳、完整输出与证据。
- 环境细节（Kali 地址、SSH、工具、靶场）见 `D:\AI-security\AGENTS.md`。
