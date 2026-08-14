---
name: flutter-defect-triage
description: >-
  Project overlay for Harbor Club (sample). New work: read requirements.md and
  REPOS.md first; scan pits via cases-rag-index.md → cases.md. Global process:
  ~/.cursor/skills/flutter-defect-triage/SKILL.md.
---

# 本项目 · Defect Triage 快捷入口（示例）

> 这是**项目内 overlay** 的写法示例。复制到你的仓库 `.cursor/skills/flutter-defect-triage/` 后，换成真实路径。

**全局流程：** `~/.cursor/skills/flutter-defect-triage/SKILL.md`  
**业务包：** `~/.cursor/skills/flutter-defect-triage/businesses/sample-club-web/pack.md`  
**双仓路径：** [REPOS.md](REPOS.md)

---

## 本仓库文档（优先读本地）

| 文档 | 何时读 |
|------|--------|
| [REPOS.md](REPOS.md) | 国内 / 海外路径、发布 slot |
| [requirements.md](requirements.md) | 新需求接入、查状态 / 分支 |
| [cases-rag-index.md](cases-rag-index.md) | 扫坑、按文件 / 症状检索 |
| [cases.md](cases.md) | 命中 Case ID 后读实现与回归 |

**新需求 workflow：** `requirements.md` 加行 → 开发 → `cases.md` + `cases-rag-index.md` 回填。

---

## 本仓惯例（示例）

新增 **H5 → Native** 协议时按三层拆分，不要在 UI 或 sender 里堆业务配置：

```
UI 组件
    ↓ 埋点、点击条件
业务 util（常量 / 拉数 / 转换 / 缓存）
    ↓ 组装协议字段
bridge sender（只发 JSON）
    ↓
Native
```

失败兜底：打日志 + 空结果，**不伪造 UI 文案**。
