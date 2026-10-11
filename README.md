# 中国足球小将球员 · 国字号入选与国内外比赛数据库

本库记录与民间青训项目 **“中国足球小将”** 有关（或被称有关）的球员，内容包括国字号集训入选、赛事出场，以及国内外比赛的首发、进球、助攻。每条事实都能追溯到来源。泛指意义上的“足球小将”不在收录范围内。

> ⚠️ `npm run validate:data` 只检查数据结构和内部一致性（技术校验），**不等于事实核验**。事实是否准确，要看每条记录列出的来源与摘录。

## 口径（摘要）

- **集训/名单 ≠ 出场**：“未首发、是否登场未知”不算出场。
- **俱乐部比赛（含 B 队、梯队）永远不计入国字号统计**，页面上单独展示。
- **未知值为空（—）**：只有完整的进球/助攻名单能支持时才记 0。合计只累加已知值，并标出未知条数；“+”表示下限值。
- **上场分钟**：来源写明时才填写，目前全部为空。
- **媒体报道不标为官方**：转载按原始媒体归类，同时记录转载平台。参考站 timkok.github.io 只作线索，不作证据。
- **核实状态由代码计算**：证据链 A（与小将的关系有可靠来源）和证据链 B（官方名单或核读过的国字号出场）都成立为“已确认”，成立一条为“部分确认”，只有弱线索为“待核实”，证据链字段有未解决冲突为“有争议”。
- 待核实球员不计入确认/入选指标与图表；没有出生年份的球员不进入出生年份图。
- “由出场推定”的赛事入选，其日期是首次出场的当地日期，不是名单公布日期。

详见 `docs/data-model.md` 和站内 `/methodology` 页面。

## 来源优先级

1. 中国足协官方（必须是 thecfa.cn 域名，`official_cfa`）
2. 球队/俱乐部官方（`official_team` / `club`）
3. 中国足球小将官方（`xiaojiang_official`）
4. 主流媒体（`mainstream_media`，如新华社、第一财经、直播吧、懂球帝数据）
5. 社交/自媒体（`social`，仅作线索，不支持任何证据链）
6. 参考站/未知（`unknown`，仅作线索）

## 目录结构

```
data/            players / sources / relations / selections / matches / appearances / conflicts / gaps / changelog .json
src/lib/core.js  状态计算、聚合、筛选（前端、校验、测试共用）
src/lib/csv.js   CSV 导出
src/views.js     各页面渲染；src/charts.js 自制 SVG 图表；src/router.js History 路由；src/app.js 应用壳
scripts/validate-data.mjs  数据校验
scripts/mobile-check.mjs   无头 Chrome 截图与横向溢出检查
tests/           vitest + jsdom 测试
docs/data-model.md  数据模型说明
screenshots/     移动端/桌面端截图
```

页面：`/` 总览，`/players` 球员（可筛选、导出 CSV），`/players/<slug>` 球员详情，`/matches` 比赛（可筛选、导出 CSV），`/sources` 来源与冲突，`/methodology` 口径与证据缺口，`/changelog` 更新日志。

## 常用命令

```bash
npm run dev            # 开发服务器
npm run validate:data  # 数据校验（出错时列出文件、记录ID、问题和修复建议，并以非零码退出）
npm test               # 单元测试 + 渲染测试
npm run build          # 先校验数据，再构建到 dist/（同时把 data/*.json 复制到 dist/data/）
npx vite preview --port <空闲端口>   # 预览（自带 SPA fallback）
npm run build:pages    # GitHub Pages 构建：BASE_PATH=/xiaojiang-db-grokbot/，输出到 dist-pages/（含 404.html 深链接回退与 .nojekyll）
BASE_URL=http://127.0.0.1:<端口> npm run check:mobile   # 截图 390x844 / 1440x900，并检查横向溢出
```

## 部署与基础路径

基础路径由环境变量 `BASE_PATH` 决定，默认为 `/`。视图中的站内链接一律写作应用内路径（如 `/players`），由 `src/lib/base.js` 在渲染时统一加上基础路径；路由匹配前会先去掉基础路径。`SITE_ORIGIN` 用于生成 og:url、canonical 和 JSON-LD 中的绝对 URL。

线上地址：https://timkok.github.io/xiaojiang-db-grokbot/ 。部署仓库只包含构建产物和 README，源码、数据与脚本保存在本地项目目录中。GitHub Pages 对深链接返回 404.html：HTTP 状态码是 404，但页面会正常渲染应用。

## 如何添加球员

1. 在 `sources.json` 中添加来源：填写 `source_type`、`verification_status`、日期；如果是转载，还要填 `original_publisher` 和 `republished_on`。
2. 在 `players.json` 中添加球员：ID 按 `P00xx` 顺延，`slug` 用小写拼音。未知字段填 `null`；有值的字段必须同时填写对应的 `*_source_ids`。
3. 在 `relations.json` 中添加该球员与小将的关系记录（至少一条，弱说法用 `claimed_weak`）。
4. 在 `selections.json` 中添加入选记录：每期名单每人一条，`unit_as_listed` 和 `name_as_listed` 照抄名单原文。
5. 运行 `npm run validate:data`，按提示把 `verification_status` 改成计算出的值。

## 如何添加比赛/出场

1. 在 `matches.json` 中添加比赛：分别填写当地日期和北京时间，写明 `date_basis` 和 `date_precision`。俱乐部比赛 `team_kind=club`，并且 `national_level=null`。
2. 在 `appearances.json` 中每名球员添加一条记录。不确定是否首发时用 `appeared_start_unknown` + `started:null`；只有全部进球/助攻都有归属时才写 0；首发是推出来的要写 `start_basis: derived_*`。
3. 运行校验：它会检查比分、个人进球之和不超过球队进球、同一球员同一场是否重复等。

## 如何更改状态

不要直接手改状态，而是补充或修正证据（relations / selections / appearances / conflicts），再运行校验。校验会给出计算后的状态，把 `players.json` 中的 `verification_status` 改成这个值即可。登记 `chain: "A"` 或 `"B"` 且状态为 open 的冲突后，该球员会变为“有争议”。

## 如何纠错

1. 如果来源之间矛盾，在 `conflicts.json` 中登记各方说法，写明采用哪个值及理由，不要直接删掉其中一方。
2. 修正数据后，在 `changelog.json` 中追加一条真实的记录（日期、改动、依据）。
3. 运行 `npm run validate:data && npm test`。

## 当前覆盖（2026-10-11）

10 名球员；26 个来源（21 个已核读）；36 条入选记录；15 场比赛（国字号 13 场、俱乐部 2 场）；31 条出场记录；6 条冲突；36 条证据缺口。各球员的具体缺口见 `/methodology#gaps`。
