# Defect Triage · Templates

用于：**新建 Case**、**扩展 RAG 索引**、**接入新业务包**。  
流程见 [SKILL.md](SKILL.md) · 业务目录见 [businesses/README.md](businesses/README.md)。

---

## 1. 新业务接入清单

```
- [ ] 定 pack slug（例：sample-club-web / voice-square-h5）
- [ ] 定 biz 分类（见 businesses/README.md taxonomy）
- [ ] mkdir businesses/<slug>/hosts
- [ ] 写 businesses/<slug>/pack.md（骨架见下）
- [ ] 登记 businesses/README.md「已接入业务包」
- [ ] rag-index：「按业务域」+ 症状行 + 速查表（含 biz 列）
- [ ] 第一个缺陷走完整：坑点对齐 → Case → 合入回填
- [ ] 不把业务细节写回 SKILL.md
```

### businesses/<slug>/pack.md 骨架

```markdown
# Business Pack · <产品名>

> **仓库：** `<path>` · **biz 覆盖：** …

## 运行端矩阵
| 口头说法 | 实际 | Gate 倾向 |

## Gate 速查
| Gate | 适用 | biz |

## Hotspots（按 biz）
| biz | Area | Files |

## 刻意不修 / 已否决
- …

## 对接文档
- [hosts/xxx.md](hosts/xxx.md)
```

---

## 2. Case 全文模板（追加到 cases.md）

```markdown
## Case N: <一句话标题>

**Report:** <用户/工单原始现象>
**biz:** <channel-detail / im-message / …>
**pack:** <sample-club-web / …>

**Scope Gate：**

| 维度 | 在范围 | 不在范围 |
|------|--------|----------|
| 运行端 | | |
| 业务场景 | | |
| 入口路径 | | |
| 修复边界 | | |

**坑点地图：**

| 坑 | 机制 | 证据 | 状态 |
|----|------|------|------|
| 0 | | | 确认 / 排除 / 待验证 |

**Verification（代码级）：**
1. …
2. …

**Root cause：** <mechanism，非猜测>

**Fix scoping：** <gate 表达式 + 为何窄>

**Fix：**
| 文件 | 改动 |
|------|------|
| | |

**Commit：** `pending` / `<sha>`

**刻意不改：** …

**回归面：**

| # | 场景 | 预期 |
|---|------|------|
| N-1 | | |
```

---

## 3. RAG 索引行模板（追加到 cases-rag-index.md）

### 按业务域

```markdown
| **<biz>** | N, M, … | <一句话> |
```

### 症状 → Case

```markdown
| <关键词1>、<关键词2> | **N** | <biz> | <gate> | <sha> |
```

### 文件 → Case

```markdown
| `<path>` | N, … |
```

### 速查表行（含 biz）

```markdown
| **N** | <短标题> | <biz> | <平台> | <gate> | <commit> | <关联> |
```

---

## 4. 坑点对齐表（会话输出，勿省略）

```markdown
## 坑点对齐 · [缺陷简述]

**Scope：** 运行端 · 场景 · 入口 · 修复边界
**biz / pack：** <biz> · <slug>

| 坑 | 机制（一句话） | 证据 | 置信度 | 你的理解？ | 待问你 |
|----|----------------|------|--------|------------|--------|
| 0 | | | | ？ | |

**本次倾向叠加的坑：**
**明确排除的坑：**
**还需你确认：**
```

---

## 5. 合入后回填清单

```
- [ ] cases.md：Commit / biz / pack / 坑状态
- [ ] cases-rag-index.md：按业务域 + 症状 + 速查表（biz 列）
- [ ] businesses/<slug>/pack.md：新 gate / 热点
- [ ] businesses/<slug>/hosts/*.md：协议变更
```

---

## 6. 项目 stub（可选，放仓库 .cursor/skills/）

全局 skill 已安装时，仓库内只保留快捷入口。完整示例见 [examples/project-overlay](examples/project-overlay/)。

```markdown
---
name: flutter-defect-triage
description: Project stub → global flutter-defect-triage
---

# 本项目 · Defect Triage 快捷入口

**全局 skill：** ~/.cursor/skills/flutter-defect-triage/
**业务包：** sample-club-web

## 本仓库路径
- Flutter Web：lib/
- 小程序：miniapp/
```

---

## 7. 新 pack 接入（无需复制整份 skill）

1. `mkdir -p ~/.cursor/skills/flutter-defect-triage/businesses/<slug>/hosts`
2. 写 `pack.md` + 登记 `businesses/README.md`
3. Case 写入全局 `cases.md`（带 `pack:` 字段）
4. 目标仓库可选加 §6 stub
