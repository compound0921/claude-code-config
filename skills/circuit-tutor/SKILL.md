---
name: circuit-tutor
description: 电路分析、模拟电路、数字电路的截图识题与规范求解、出题/抽题、仿真验证和错题归因。用户发来电路题目截图、要求解题/出题/抽题、或要求用 Multisim/ngspice 验证电路时触发。
---

# 电路辅导

项目根目录 `D:\CircuitStudy`：题库在 `bank\`，错题本 `mistakes.md`，网表在 `circuits\`，每次运行产物放 `runs\<题号>\`。

## 路由

| 用户意图 | 动作 |
|---|---|
| 发截图/文字题 | 「截图解题」 |
| 要一道题/抽题 | 「出题与抽题」 |
| 要验证电路 | 「仿真验证」 |
| 报错题/复盘 | 更新 `mistakes.md` |

## 截图解题

1. 完整提取：题干、元件符号与数值、节点、电源方向、受控源、开关状态、所求量。
2. 先输出「识图清单」。参数看不清就写「不确定」并问用户，不硬猜。
3. 判科目（电路分析/模电/数电）和具体题型。
4. 按 `references/solution-template.md` 求解：已知与所求 → 选用定律/模型及理由 → 分步推导（保留单位）→ 结果与校核。
5. 需要时按「仿真验证」复核；手算与仿真不一致，先查建模假设和网表，再改答案。
6. 结尾给 1–3 个易错点和一句话方法总结。
7. 需要图文讲解时，按「解答文件」的 spec 管线生成 HTML；PDF 先问用户。

## 出题与抽题

1. 读 `bank\index.json` 和对应科目文件。
2. 按科目、章节、题型、难度筛选。用户没指定时，优先选最近错题同类题，再按难度递增。
3. 出题时先只给题目，不给答案；用户作答后再讲评。
4. 新题写入题库，字段见 `references/bank-schema.md`，并附可自检的网表。
5. 自编题标 `"source": "自编"`，真题标来源。

## 仿真验证

使用已安装的 `multisim-spice` skill。最小步骤：

1. 写 `<题号>.cir` 到 `circuits\`：首行标题，末行 `.end`，节点 0 为地。
2. ngspice 自检：
   ```
   powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Users\Compound\.claude\skills\multisim-spice\scripts\check.ps1" "<cir>" "<meas 命令>"
   ```
   末行必须是 `CHECK: PASS`。
3. 打开 Multisim：
   ```
   powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Users\Compound\.claude\skills\multisim-spice\scripts\find-multisim.ps1" -Open "<cir>"
   ```
   让用户在 Multisim 里 Save As 成 `.ms14`，再用 `patch-ms14.ps1` 写回模型和分析设置，重开按 F5。
4. 确认焦点后发送 F5（焦点不是 Multisim 时脚本拒绝发送）：
   ```
   powershell -NoProfile -ExecutionPolicy Bypass -File "D:\CircuitStudy\scripts\multisim-keys.ps1" -Keys "{F5}" -WaitSec 5
   ```
5. 截图（默认只截 Multisim 窗口）：
   ```
   powershell -NoProfile -ExecutionPolicy Bypass -File "D:\CircuitStudy\scripts\capture-window.ps1" -Out "D:\CircuitStudy\runs\<题号>\sim.png"
   ```
6. 手算值、ngspice 值、Multisim 值三者对齐；不一致要说明原因。

## 图文题面（出题时必做）

- **不要用 ASCII 字符画电路图。**
- 在 `runs\<题号>\` 生成自包含 HTML：题干 + 内嵌 SVG 规范图 + Multisim 截图（base64）。
- PDF 先问用户；需要时再用 Edge headless 转：
  ```
  "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --headless=new --disable-gpu --no-pdf-header-footer --print-to-pdf="<pdf>" "file:///<html>"
  ```
- 题面文件不放答案；答案留在题库，用户作答后再讲评。
- 模板：`D:\CircuitStudy\scripts\make_ca002_html.py`。

## 答题纸

- 每道题在 `runs\<题号>\answer-sheet.html` 生成可填写答案的页面。
- 字段用端口术语：\(V_{oc}\)、\(R_o\)、\(V_o\)、\(I_o\)，加一个解题过程文本框。
- 提供「复制答案」「下载 .md」「自检」「清空」；自检只显示对错，不显示正确答案。
- 提供「查看答案」按钮，打开同目录的 `solution.html`。
- localStorage 自动保存。
- 模板：`D:\CircuitStudy\scripts\make_ca003_answer_sheet.py`。

## 出题后自动流程

- 生成题面和答题纸后，**自动用 `Start-Process` 打开答题纸 HTML**。
- 自动生成 `solution.html`；是否渲染 `solution.pdf` 先问用户。
- 解答仍按下面「解答文件」的版式规则；把 `solution_sheet` 写进题库。

## 解答文件

- 用户作答后，在 `runs\<题号>\solution.html` 生成完整解答；PDF 先问用户。
- 内容：答案汇总、原电路、分步推导、变换图、等效电路。
- **不在聊天里重复整篇解答**；聊天只给文件链接和一句结论。
- 排版：
  - 图内变量用 MathJax 叠加标签，不要用普通 SVG 文本写 \(I/V/R\)（大写 I 容易看成 l/1）。
  - 图片和正文适当缩小（图宽 420–480px，正文 15px），避免一页只有少量内容。
  - 标题用 `break-after: avoid`，保证与下段同页。
  - 得到新电路时正文写“得到图 x 所示电路”或“如图 x 所示”。
  - 图注紧跟图正下方，只写“图 x + 几个字”，不复述图中参数。
  - 不单独设“校核”小节；必要的核对写成正文一句话。
  - 易错点用 `.annotation` 红框插在对应步骤后，不集中放文末。
  - 仿真验证放最后，用证据块（`CHECK: PASS` + 数值），不用表格。
- 新版模板：先写 `specs\<id>.yaml`，再运行 `python scripts\build_explain.py specs\<id>.yaml --check`；PDF 先问用户，要才加 `--pdf`。图元库 `scripts\circuit_kit.py`，基础模板 `templates\explain\`。
- 旧模板 `D:\CircuitStudy\scripts\make_ca003_solution.py` 保留，仅作参考。
- 涉及电路变换（断开负载、独立源置零、等效化简、接回负载）时，**必须附变换后的电路图**，不能只用文字描述。
- 变换图用内嵌 SVG；每张图标注关键量（\(V_{oc}\)、\(R_o\)、\(I_t\)、a/b），图注写清变换条件。
- CSS 用 `figure > svg` 限定尺寸，避免把 MathJax 生成的行内公式 SVG 也拉大。

## 数学排版

- 行内公式用 `\(...\)`，独立公式用 `\[...\]`。
- 分数用 `\frac{}{}`，乘号 `\times`，单位 `\text{A}`、`\Omega`。
- 不要用 `(a-b)/(c+d)` 这类纯文本除法。
- PDF 用 MathJax（`D:\CircuitStudy\assets\mathjax\tex-svg.js`）+ Puppeteer 打印。
- 打印命令：`node D:\CircuitStudy\tools\puppeteer\render-pdf.js <http-url> <pdf>`。
- 需要本地 HTTP 服务指向项目根目录；`file://` 下 MathJax 加载不可靠。

## 术语

- 输入端口：输入电压 \(V_i\)、输入电流 \(I_i\)、输入电阻 \(R_i\)。
- 输出端口：输出电压 \(V_o\)、输出电流 \(I_o\)、输出电阻 \(R_o\)。
- 戴维南/诺顿等效：从负载端看进去的等效电阻称**输出电阻 \(R_o\)**（可注明 \(R_o=R_{th}\)）；开路电压称**开路输出电压 \(V_{oc}\)**（可注明 \(V_{oc}=V_{th}\)）。
- 负载用 \(R_L\)；负载上的电压、电流在输出端称**输出电压 \(V_o\)**、**输出电流 \(I_o\)**。
- 不要只写「等效电阻」「两端电压」而不说明端口；第一次出现时写全称和符号。

## 输出规范

- 中文，先结论后步骤；公式用 LaTeX，数值带单位。
- 每步写依据，不跳步；不确定处明确标注。
- 讲完题给「这题考什么 / 易错点 / 下次怎么快速识别」。

## 数据

- 题库索引：`D:\CircuitStudy\bank\index.json`
- 科目题库：`circuit-analysis.json`、`analog.json`、`digital.json`
- 错题本：`D:\CircuitStudy\mistakes.md`
- 网表：`D:\CircuitStudy\circuits\`
- 运行产物：`D:\CircuitStudy\runs\`
