---
name: design-router
description: 前端设计任务的统一路由与裁决层。当任务是公开展示页、产品界面/后台、文档或原生，或需要同时使用 taste / impeccable / apple-skills 时加载；当用户说「设计界面」「重新设计界面」「改版」「重做 UI」而没有指明是页面还是产品界面时，必须先加载本 skill 判定表面。判定表面类型，指定唯一主包与细节层，并裁决质量门、动效、方向、素材、评审的优先级。表面明确时再直接加载对应主包。
---

# Design Router

三套设计包各有主场。本 skill 只做三件事：判定表面、指定主包、裁决冲突。不复制任何包的设计规则。

## 1. 判定表面

依次问四个问题，先命中先定：

1. 访客是来被说服、决定或体验作品？（营销页、落地页、作品集、活动页、产品官网）→ **Page**
2. 访客是来完成任务？（后台、设置、编辑器、收件箱、表单、管理工具、登录后界面）→ **Product**
3. 访客是来理解内容？（文档、文章、帮助中心、changelog）→ **Read**
4. 目标是 iOS / Android / React Native / Expo 原生界面？→ **Native**
5. 只改动效或手机端细节，不改整体设计？→ **Motion/Mobile fix**

判定前先看当前目录和目标文件：路由名、页面路径（`app/`、`pages/`、`src/`）、组件名、是否登录后才可见。能从代码上下文判定就直接判定；看不出、且用户没有指明表面时，问一个问题再选，不要同时加载两个主包。

同一个项目可以按路由分别处理：营销首页走 Page，登录后的界面走 Product。按**路由或文件**判定，不按项目整体判定。

## 2. 路由表

| 表面 | 主包 | 细节层（按需） | 禁止加载 |
|---|---|---|---|
| Page | `design-taste-frontend` | apple `animate` | impeccable 的 `new-work`/`concept-seed`/`bolder`/`quieter`/`delight`/`overdrive`；taste 的独立 app-screen 流程 |
| Product | `impeccable` | apple `animate`、`mobile-native` | taste 的 `draw.mjs`、house rules、`inspect.mjs` |
| Read | `impeccable`（Read mode） | — | taste 全部 |
| Native | `impeccable` native refs | apple `animate-expo`、`write-swift` | taste 全部 |
| Motion/Mobile fix | apple 对应 skill | — | 另两套 |

一个任务最多两个细节层。主包决定方向和验收；细节层只给实现质量。

## 3. 加载纪律

- 一个任务只有一个主包。主包确定后，不加载另一个主包的 SKILL.md。
- 需要跨包能力时，读具体 reference 文件，不读另一个包的主 SKILL.md。
- 页面里嵌产品截图：仍由 taste 主，读 `references/app-screens.md` 的片段规则；不加载 impeccable `operate`。
- 产品界面里有一段营销式空状态或引导页：仍由 impeccable 主；不加载 taste。
- 不确定表面时，先问用户一个问题，不要同时加载两套。

## 4. 冲突裁决

按优先级从高到低：

1. **质量门**：Page 只认 taste `inspect.mjs` / `lint.mjs` + `critique.md`；Product/Read/Native 只认 impeccable `detect` / `audit` + `craft-floor` + `critique`。两套不叠加。
2. **动效数量与方向**：Page 由 taste 决定（≥8 个移动元素、场景、签名动作、每节入场）；Product 由 impeccable 的 motion thesis 决定。
3. **动效实现质量**：两个表面都由 apple `animate` / `animate-expo` 决定（缓动、属性、reduced-motion、频率、性能）。apple 的「高频不动画/不确定就删」只用于 Product 和低频交互，不用来否决 Page 必需的动效层。
4. **方向生成**：Page 只用 taste `draw.mjs`；Product 只用 impeccable `concept-seed`；组件级多版本只有用户明确要求时才用 apple `prototype`。一个任务只掷一次。
5. **素材**：Page 用 taste `assets.mjs`（真实图 + 版权）；Product comp 用 impeccable `visualize`（生成图）。同一任务不混用。
6. **评审与评分**：Page 用 taste `critique.md`；Product 用 impeccable `critique`；动效实现用 apple `review-animations` 十标准。`emil-design-eng` 的 Before/After 表只作问题输出格式，不参与评分。
7. **状态文件**：Page 项目写 `.taste-skill/`；Product 项目写 `PRODUCT.md` / `DESIGN.md` / `.impeccable/`。互不创建、互不覆盖。

## 5. 禁止组合

- taste house rules 与 impeccable craft-floor 同时作为质量门
- taste `draw.mjs` 与 impeccable `concept-seed` 在同一次方向决策中使用
- taste `critique` 与 impeccable `critique` 对同一产物各打一次分
- taste `assets.mjs` 与 impeccable `visualize` 在同一次素材生产中混用
- apple `prototype` 与 taste `draw.mjs` 在同一次整页方向决策中使用
- 对 Product 表面运行 taste `inspect.mjs`
- 对 Page 表面运行 impeccable `new-work` 或 `concept-seed`

## 6. 交接产物

| 从 | 到 | 交接内容 |
|---|---|---|
| taste | apple `animate` | 方向表里的 signature move、场景清单、动效预算（≥8 元素） |
| impeccable | apple `animate` | motion thesis：焦点时刻、continuity、feedback、budget |
| impeccable | apple `mobile-native` | 目标设备、断点、需要修的触控或视口问题 |
| taste | impeccable | 仅当项目同时有 Page 与 Product 路由：页面方向与品牌约束 |
| apple `break-ui` | impeccable `harden` | worst-case fixture 与已发现的边界；harden 补 i18n、网络、API |

## 7. 干跑用例

| 输入 | 应选 |
|---|---|
| 「做一个 SaaS 的营销首页」 | Page → taste + apple animate |
| 「改一下这个后台的表格和设置页」 | Product → impeccable + apple mobile-native |
| 「这个文档站的排版有点乱」 | Read → impeccable |
| 「给这个 Expo App 加手势和转场」 | Native → impeccable native + apple animate-expo |
| 「这个按钮的动画很卡」 | Motion fix → apple animation-performance |
| 「营销页里放一张我们产品的截图」 | Page → taste，`app-screens` 片段规则 |
