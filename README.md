<div align="center">

# flutter-defect-triage

> 先对齐坑，再写修复。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Cursor](https://img.shields.io/badge/Cursor-Compatible-black)](https://cursor.sh)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

</div>

Cursor / Claude Code 用的缺陷分诊 skill。面向 **Flutter Web、App WebView、小程序 H5、多宿主 JSBridge** 这类高坑密度场景。

不是「帮你改 bug 的 prompt」。它强制 agent 在动代码前做四件事：

1. **选业务包** — 运行端、gate、热点文件先定位
2. **扫坑** — 先读 RAG 索引，再打开对应 Case
3. **坑点对齐** — 把机制表摊给你确认，不确定的必须问
4. **窄修复** — Scope Gate 四维写清；合入后再回填 Case

```
用户报缺陷
    → 选 businesses/<slug>/pack.md
    → cases-rag-index.md 按 biz / 症状 / 文件检索
    → 输出坑点对齐表（用户可纠正）
    → 代码 + git timeline 证明机制
    → 最小 diff
    → 回填 cases.md
```

[安装](#安装) · [使用](#使用) · [为什么不是直接修](#为什么不是直接修) · [仓库结构](#仓库结构) · [接到真实项目](#接到真实项目)

---

## 为什么不是直接修

Flutter Web / WebView 缺陷经常不是「写错一行 CSS」。常见翻车：

| 翻车 | 实际发生的事 |
|------|----------------|
| 口头「App 里」 | 可能是 WebView，也可能是 Native；gate 挂错边 |
| 一处有标签、一处没有 | 列表 / 气泡 / 浮层走了三条数据路径 |
| 修了 A 机、B 机爆 | 用系数猜宿主键盘 / 安全区 |
| 长按没反应 | 手势链中间 `await`，pointer 已经丢了 |
| 同样的坑第三次出现 | 修完没记 Case，下次 agent 从零猜 |

本 skill 把这些收成可执行流程：**未对齐前不写修复代码。**

---

## 安装

### Cursor（推荐）

```bash
curl -fsSL https://raw.githubusercontent.com/lnuxe/flutter-defect-triage/main/setup | bash
```

默认装到 `~/.cursor/skills/flutter-defect-triage/`。装完新开一轮对话即可。

已克隆到本地时：

```bash
git clone https://github.com/lnuxe/flutter-defect-triage.git
cd flutter-defect-triage && bash setup
```

### Claude Code

同一条 `setup` 会同时尝试写入 `~/.claude/skills/flutter-defect-triage/`（目录存在或已安装 Claude Code 时）。

### 手动

把本仓库放到：

| 工具 | 路径 |
|------|------|
| Cursor 全局 | `~/.cursor/skills/flutter-defect-triage/` |
| Cursor 项目 | `<repo>/.cursor/skills/flutter-defect-triage/` |
| Claude Code | `~/.claude/skills/flutter-defect-triage/` |

`SKILL.md` 必须在该目录根上。

---

## 使用

直接说缺陷即可，不必先打指令：

```
频道成员列表 Tab 右侧有一块灰的
Android WebView 里按住说话没权限框
这个是不是安全区的问题？
帮我 triage 一下这条
```

Agent 应先给出**坑点对齐表**，而不是直接改文件。你纠正「这个坑不存在 / 那个才是」之后，它才进调查和修复。

合入后让它「回填 Case」：更新 `cases.md` + `cases-rag-index.md`。

---

## 双层结构

| 层 | 放哪 | 放什么 |
|----|------|--------|
| **全局 skill** | `~/.cursor/skills/flutter-defect-triage/` | 流程、模板、跨仓 Case、业务包 |
| **项目 overlay** | `<repo>/.cursor/skills/flutter-defect-triage/` | 本仓路径、需求表、双仓对照、本仓 Case |

全局文件保持短、保持业务无关。产品路径、协议、发布 slot 放 overlay。完整范文：[examples/project-overlay](examples/project-overlay/)。

---

## 仓库结构

```
flutter-defect-triage/
├── SKILL.md                          # 流程（agent 入口）
├── templates.md                      # Case / pack / 对齐表模板
├── cases-rag-index.md                # 先读索引
├── cases.md                          # 虚构示例 Case 1–3
├── businesses/
│   ├── README.md                     # biz taxonomy
│   └── sample-club-web/              # 虚构业务包
│       ├── pack.md
│       └── hosts/android-webview-keyboard.md
├── examples/project-overlay/         # 项目内 stub 范文
├── setup
├── VERSION
└── LICENSE
```

示例产品 **Harbor Club** 是编的，用来演示 Tab 收缩、数据路径分裂、宿主权限这三类真坑。接到真实仓库后换掉 slug 和 Case。

---

## 接到真实项目

1. `bash setup` 装全局 skill
2. 复制 `examples/project-overlay/` → 目标仓库 `.cursor/skills/flutter-defect-triage/`
3. 新建 `businesses/<你的 slug>/pack.md`，登记到 `businesses/README.md`
4. 删掉或替换 Harbor Club 示例 Case
5. 第一个真实缺陷走完整：对齐 → 修 → 回填

**不要**把内部仓库地址、AppId、真实协议、工单号、员工路径写进你 fork 的公开仓库。本仓库开源的是流程，不是某家公司的业务包。

---

## 脱敏说明

本仓库从一套生产环境的 Cursor skill **抽出方法论**后重写。已去掉：

- 真实产品名、活动 ID、AppId
- 内部 Git / 工单 / 流水线地址
- 真实协议名、道具 ID、支付通道
- 真实 Case、commit、MR、版本白名单
- 个人机器路径与公司邮箱

示例 Case 只保留可公开的机制：Flutter 布局收缩、展示路径 ≠ 数据路径、WebView 权限必须走宿主 bridge。

---

## 升级

```bash
git -C ~/.cursor/skills/flutter-defect-triage pull --ff-only
```

或重新跑 `setup`。

---

## License

[MIT](LICENSE) © lnuxe
