# Flutter Defect Triage · 缺陷分诊

<div align="center">

> **先对齐坑，再写修复。Align the pits first. Then fix.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Cursor](https://img.shields.io/badge/Cursor-Compatible-black)](https://cursor.sh)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

</div>

Cursor / Claude Code 用的缺陷分诊 skill。面向 **Flutter Web、App WebView、小程序 H5、多宿主 JSBridge** 这类高坑密度场景。

A defect triage skill for Cursor / Claude Code, built for **Flutter Web, App WebView, mini-program H5, and multi-host JSBridge** — environments where pit density is high and naive fixes routinely fail.

不是「帮你改 bug 的 prompt」。它强制 agent 在动代码前做四件事：

This is **not** a "fix my bug" prompt. It forces the agent to do four things before touching any code:

1. **选业务包 / Select a business pack** — 运行端、gate、热点文件先定位 / identify the runtime target, gate, and hotspot files
2. **扫坑 / Scan for known pits** — 先读 RAG 索引，再打开对应 Case / read the RAG index first, then open the matching Case
3. **坑点对齐 / Pit alignment** — 把机制表摊给你确认，不确定的必须问 / present a mechanism table for your confirmation; ask about anything uncertain
4. **窄修复 / Narrow fix** — Scope Gate 四维写清；合入后再回填 Case / write a four-dimension Scope Gate; backfill the Case after merge

```
用户报缺陷 / User reports a defect
    → 选 businesses/<slug>/pack.md
    → cases-rag-index.md 按 biz / 症状 / 文件检索
    → 输出坑点对齐表（用户可纠正）
    → 代码 + git timeline 证明机制
    → 最小 diff
    → 回填 cases.md
```

[安装 / Installation](#安装--installation) · [使用 / Usage](#使用--usage) · [为什么不是直接修 / Why Not Just Fix It](#为什么不是直接修--why-not-just-fix-it) · [仓库结构 / Repository Structure](#仓库结构--repository-structure) · [接到真实项目 / Onboarding a Real Project](#接到真实项目--onboarding-a-real-project)

---

## 为什么不是直接修 / Why Not Just Fix It

Flutter Web / WebView 缺陷经常不是「写错一行 CSS」。常见翻车：

Flutter Web / WebView defects are rarely "one wrong line of CSS." Common failure modes:

| 翻车 / What you hear | 实际发生的事 / What's actually happening |
|---------------------|----------------------------------------|
| 口头「App 里」 | 可能是 WebView，也可能是 Native；gate 挂错边 / Could be WebView, could be Native; gate is on the wrong side |
| 一处有标签、一处没有 | 列表 / 气泡 / 浮层走了三条数据路径 / List / bubble / overlay use three different data paths |
| 修了 A 机、B 机爆 | 用系数猜宿主键盘 / 安全区 / Using magic coefficients to guess host keyboard / safe area |
| 长按没反应 | 手势链中间 `await`，pointer 已经丢了 / An `await` in the gesture chain dropped the pointer |
| 同样的坑第三次出现 | 修完没记 Case，下次 agent 从零猜 / Fixed without recording a Case; next agent starts from zero |

本 skill 把这些收成可执行流程：**未对齐前不写修复代码。**

This skill turns these into an executable workflow: **no fix code before alignment.**

---

## 安装 / Installation

### Cursor（推荐 / Recommended）

```bash
curl -fsSL https://raw.githubusercontent.com/lnuxe/flutter-defect-triage/main/setup | bash
```

默认装到 `~/.cursor/skills/flutter-defect-triage/`。装完新开一轮对话即可。

Installs to `~/.cursor/skills/flutter-defect-triage/` by default. Start a new conversation after installation.

已克隆到本地时 / If you've already cloned the repo locally:

```bash
git clone https://github.com/lnuxe/flutter-defect-triage.git
cd flutter-defect-triage && bash setup
```

### Claude Code

同一条 `setup` 会同时尝试写入 `~/.claude/skills/flutter-defect-triage/`（目录存在或已安装 Claude Code 时）。

The same `setup` script also writes to `~/.claude/skills/flutter-defect-triage/` when the directory exists or Claude Code is installed.

### 手动 / Manual

把本仓库放到 / Place this repository at:

| 工具 / Tool | 路径 / Path |
|-------------|-------------|
| Cursor 全局 / global | `~/.cursor/skills/flutter-defect-triage/` |
| Cursor 项目 / project | `<repo>/.cursor/skills/flutter-defect-triage/` |
| Claude Code | `~/.claude/skills/flutter-defect-triage/` |

`SKILL.md` 必须在该目录根上。 / `SKILL.md` must be at the root of that directory.

---

## 使用 / Usage

直接说缺陷即可，不必先打指令：

Just describe the defect — no need to invoke a command first:

```
频道成员列表 Tab 右侧有一块灰的
Android WebView 里按住说话没权限框
这个是不是安全区的问题？
帮我 triage 一下这条
```

Agent 应先给出**坑点对齐表**，而不是直接改文件。你纠正「这个坑不存在 / 那个才是」之后，它才进调查和修复。

The agent should first present a **pit alignment table**, not jump straight to editing files. You correct it ("this pit doesn't exist / that one is the real cause"), and only then does it proceed to investigation and fix.

合入后让它「回填 Case」：更新 `cases.md` + `cases-rag-index.md`。

After the fix lands, tell it to "backfill the Case": update `cases.md` + `cases-rag-index.md`.

---

## 双层结构 / Two-Layer Architecture

| 层 / Layer | 放哪 / Location | 放什么 / Contents |
|------------|-----------------|-------------------|
| **全局 skill / Global** | `~/.cursor/skills/flutter-defect-triage/` | 流程、模板、跨仓 Case、业务包 / Workflow, templates, cross-repo Cases, business packs |
| **项目 overlay / Project** | `<repo>/.cursor/skills/flutter-defect-triage/` | 本仓路径、需求表、双仓对照、本仓 Case / Repo paths, requirement tables, dual-repo mappings, repo-specific Cases |

全局文件保持短、保持业务无关。产品路径、协议、发布 slot 放 overlay。完整范文：[examples/project-overlay](examples/project-overlay/)。

Keep global files short and business-agnostic. Product paths, protocols, and release slots go in the overlay. See the full example: [examples/project-overlay](examples/project-overlay/).

---

## 仓库结构 / Repository Structure

```
flutter-defect-triage/
├── SKILL.md                          # 流程（agent 入口）/ Workflow (agent entry point)
├── templates.md                      # Case / pack / 对齐表模板 / Case / pack / alignment table templates
├── cases-rag-index.md                # 先读索引 / Read the index first
├── cases.md                          # 虚构示例 Case 1–3 / Fictional sample Cases 1–3
├── businesses/
│   ├── README.md                     # biz taxonomy
│   └── sample-club-web/              # 虚构业务包 / Fictional business pack
│       ├── pack.md
│       └── hosts/android-webview-keyboard.md
├── examples/project-overlay/         # 项目内 stub 范文 / In-project stub template
├── setup
├── VERSION
└── LICENSE
```

示例产品 **Harbor Club** 是编的，用来演示 Tab 收缩、数据路径分裂、宿主权限这三类真坑。接到真实仓库后换掉 slug 和 Case。

The sample product **Harbor Club** is fictional, used to demonstrate three real-world pit categories: Tab shrinkage, data path divergence, and host permissions. Replace the slug and Cases when onboarding a real repository.

---

## 接到真实项目 / Onboarding a Real Project

1. `bash setup` 装全局 skill / install the global skill
2. 复制 `examples/project-overlay/` → 目标仓库 `.cursor/skills/flutter-defect-triage/` / Copy to your target repo
3. 新建 `businesses/<你的 slug>/pack.md`，登记到 `businesses/README.md` / Create and register your business pack
4. 删掉或替换 Harbor Club 示例 Case / Delete or replace the Harbor Club sample Cases
5. 第一个真实缺陷走完整：对齐 → 修 → 回填 / Run your first real defect through the full pipeline: align → fix → backfill

**不要**把内部仓库地址、AppId、真实协议、工单号、员工路径写进你 fork 的公开仓库。本仓库开源的是流程，不是某家公司的业务包。

**Do not** commit internal repository URLs, App IDs, real protocols, ticket numbers, or employee paths to your public fork. This repository open-sources the **workflow**, not any company's business pack.

---

## 脱敏说明 / Desensitization Note

本仓库从一套生产环境的 Cursor skill **抽出方法论**后重写。已去掉：

This repository was rewritten after extracting the **methodology** from a production Cursor skill. The following have been removed:

- 真实产品名、活动 ID、AppId / Real product names, campaign IDs, App IDs
- 内部 Git / 工单 / 流水线地址 / Internal Git / ticket / pipeline URLs
- 真实协议名、道具 ID、支付通道 / Real protocol names, item IDs, payment channels
- 真实 Case、commit、MR、版本白名单 / Real Cases, commits, MRs, version allowlists
- 个人机器路径与公司邮箱 / Personal machine paths and company email addresses

示例 Case 只保留可公开的机制：Flutter 布局收缩、展示路径 ≠ 数据路径、WebView 权限必须走宿主 bridge。

Sample Cases retain only publicly shareable mechanisms: Flutter layout shrinkage, display path ≠ data path, and WebView permissions must go through the host bridge.

---

## 升级 / Upgrading

```bash
git -C ~/.cursor/skills/flutter-defect-triage pull --ff-only
```

或重新跑 `setup`。 / Or re-run `setup`.

---

## License

[MIT](LICENSE) © lnuxe