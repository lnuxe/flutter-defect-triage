---
name: flutter-defect-triage
description: >-
  Triage Flutter Web / WebView / Native UI and integration bugs with collaborative
  pit alignment before fixing. Works for any business module: scan code/git/cases
  for pits (坑), present a pit map, ask about unknown pits, then investigate
  and fix with minimal scope. Combines pit alignment, code trace, git timeline,
  and fix scoping. Business-specific gates/hotspots live in businesses/<slug>/pack.md.
  Lookup: cases-rag-index.md → cases.md Case N. Update cases after commits land.
  Use when the user reports a defect, asks for triage, pit alignment, or root-cause
  analysis on Flutter Web, WebView, mini program H5, or multi-host integration.
---

# Flutter Defect Triage

跨仓库复用的缺陷分诊流程。业务细节进 `businesses/<slug>/pack.md`，不要堆回本文件。

| 层 | 路径 | 何时读 |
|----|------|--------|
| 流程 | 本 `SKILL.md` | 每次缺陷 / 需求 |
| 业务目录 | [businesses/README.md](businesses/README.md) | 选业务包 |
| 业务包 | `businesses/<slug>/pack.md` | Scope Gate / gate / 热点 |
| 宿主对接 | `businesses/<slug>/hosts/*.md` | bridge / 键盘 / 分享 |
| 检索 | [cases-rag-index.md](cases-rag-index.md) | 扫坑第一步 |
| 案例全文 | [cases.md](cases.md) | 命中 Case ID 后 |
| 模板 | [templates.md](templates.md) | 新 Case、新业务接入 |

**未对齐前不写修复代码。**

---

## 选业务包（第一步）

1. 读 [businesses/README.md](businesses/README.md)
2. 按症状域或仓库命中 `businesses/<slug>/pack.md`
3. 无现成包 → 用 [templates.md](templates.md) 新建
4. 案例写入 `cases.md` + `cases-rag-index.md`（带 `biz` 列）

本仓库自带一份虚构示例包： [sample-club-web](businesses/sample-club-web/pack.md)。接到真实项目后换掉它。

---

## Principles

1. **协作式坑点对齐优先** — 先扫坑、再和用户对齐、再动手。
2. **Reproduce in code before trusting the report** — 用户描述是症状；必须独立证明机制。
3. **Confirm scope before coding** — 运行端 / 场景 / 入口 / 修复边界未说清就先问。
4. **Check git timeline** — 症状可能因 sibling fix 迁移。
5. **Separate display path from data path** — 列表、气泡、浮层、卡片常不同源。
6. **Smallest fix at root cause** — 对齐已有 pattern；禁止堆叠补丁。
7. **Minimal diff on submit** — 只动修复必需行。
8. **Scope platform deliberately** — 用**窄 gate**；具体 gate 见当前业务包。
9. **Learn from same-type fixes before coding** — 见 [同类型修复检索](#同类型修复检索修前先学)。
10. **合入后维护 skill** — 更新 `cases.md` + `cases-rag-index.md`。
11. **禁「猜状态」式修复** — 见 [反模式](#反模式猜状态)。

---

## 反模式：猜状态

| 反模式 | 症状特征 | 错在哪 |
|--------|----------|--------|
| **`await` 打断手势 / 关键路径** | 松手失效、首按无反应、双弹窗 | pointer 链中间 await，事件已丢 |
| **无 gate 的 heuristic / magic 系数** | 空白条、两倍高度、偶发错位 | 用系数「猜」宿主行为 |
| **单机型结论上泛平台** | 一边好一边坏 | 缺窄 gate |
| **修显示不修数据源** | 一处有标签一处无 | display path ≠ data path |

**修之前必须写清：** 宿主行为模型 · 窄 gate · 首帧/冷启动边界。

---

## 协作式坑点对齐（新需求 / 新缺陷必做）

### 何时触发

- 新缺陷、新需求、或「是不是 X 导致的」
- 症状与已有 Case 部分相似
- 高坑密度：IM / bridge / 多运行端 / 异步时序 / 分享 / 路由回栈

### 你的职责（AI）

1. **扫坑**（改代码前）
   - [cases-rag-index.md](cases-rag-index.md) → 按 **biz** + 症状 / 文件 / 工单
   - [cases.md](cases.md) 对应 Case 的坑点地图
   - `git log` / `git blame` 热点文件近 14 天
   - `rg` 同 API / gate / 调用链
   - 当前 `businesses/<slug>/pack.md` 热点与 gate 表
2. **产出坑点对齐表**（[templates.md](templates.md)）
3. **向用户提问** — 不确定或偏业务上下文的坑必须问
4. **对齐后再进 Investigation Loop**

### 对齐完成标志

- [ ] Scope Gate 四维已确认
- [ ] 坑点对齐表已输出，用户有机会纠正
- [ ] ≥ 2 个竞争假设，证据淘汰
- [ ] 「修什么 / 不修什么」已共识

---

## 同类型修复检索（修前先学）

1. [cases-rag-index.md](cases-rag-index.md) — 先按 **biz** 过滤
2. [cases.md](cases.md) — 机制链、Fix scoping、已否决方案
3. Git — 热点文件近期 fix
4. 代码 — 同 API / 同 gate / 同页面用法
5. `businesses/<slug>/pack.md` — 可复用 gate / 热点

---

## Scope Gate（修改前必问）

| 维度 | 要问清 | 填法 |
|------|--------|------|
| **运行端** | Native / App WebView / 浏览器 / 小程序 | 业务包「运行端矩阵」 |
| **业务场景** | 哪个业务 flag / 哪个模块 | 例：`isTeamScene` |
| **入口路径** | 从哪进、有无底层页 | 例：聊天 → 管理 → 创建 |
| **修复边界** | 只修一端？全端？刻意不修？ | 写进 Case「不修什么」 |

**红旗：** 口头平台名含糊 → 必须确认；同一模块多子缺陷 → **分开提交**。

---

## Investigation Loop

```
- [ ] 选业务包：businesses/README → pack.md
- [ ] 坑点对齐：对齐表 + 用户确认 / 排除
- [ ] Scope Gate：四维已确认
- [ ] Restate symptom（谁 / 哪页 / 哪行 / 何时）
- [ ] Find render widget + parent constraints（UI 类）
- [ ] Trace data source for visible state
- [ ] git log / blame（近 14 天 hotfix）
- [ ] 同类型修复检索（同 biz 优先）
- [ ] 竞争假设 + 证据淘汰
- [ ] 最小修复 + 回归面 +「不修什么」
- [ ] 合入后：更新 cases.md / cases-rag-index.md / 业务包
```

---

## 通用缺陷类（跨业务复用）

### UI：Column + 无宽度 Container

`Column(crossAxisAlignment: start)` 下无 `width` 的 `Container` 会收缩 → 右侧露页面底色。**优先：** `width: double.infinity`。

### 数据：展示路径 ≠ 数据路径

对齐表必须分开写「展示面 × 数据源」。

### 时序：自发送 / merge / 对象替换

`onCreate` 补丁字段在 success/recv merge 时被整对象替换抹掉 → 先修 merge 保留逻辑。

### 身份：多 openId / userId 源

匹配时双源兜底，并写清主源。

### 宿主：bridge / 权限 / 键盘 / 分享

必须先定宿主行为模型 + 窄 gate；对接细节读 `businesses/<slug>/hosts/`。

---

## Final report template

```markdown
## 问题复述
## 坑点对齐结论（biz: <slug>）
## 核查结论
## 证据
## 修复建议
## 回归面
```

---

## 新业务 / 新仓库接入

1. **直接开用** — 流程不变；无 Case 则新建。
2. **建业务包** — `businesses/<slug>/pack.md`，登记到 [businesses/README.md](businesses/README.md)。
3. **记 Case** — `cases.md` + rag-index（含 `biz` 列）。
4. **对接类** — `businesses/<slug>/hosts/*.md`。
5. **项目可选 stub** — 仓库内 `.cursor/skills/flutter-defect-triage/` 只放路径快捷链接与本仓 Case。见 [examples/project-overlay](examples/project-overlay/)。

不要把业务细节堆回本 `SKILL.md`；保持本文件为**流程层**。
