# Harbor Club · 需求 & Case 文档（示例 overlay）

> 复制本目录到目标仓库 `.cursor/skills/flutter-defect-triage/`。  
> 全局流程仍走 `~/.cursor/skills/flutter-defect-triage/`。

## 文档分工

| 文件 | 用途 | 何时更新 |
|------|------|----------|
| [REPOS.md](REPOS.md) | 多仓库路径（国内/海外、发布 slot） | 路径或 slot 变更时 |
| [requirements.md](requirements.md) | 需求清单 | 每接一个新需求 **先加一行** |
| [cases-rag-index.md](cases-rag-index.md) | 检索索引 | 落地或踩坑后加索引行 |
| [cases.md](cases.md) | 案例全文 | 需求完成 / 合入前写全文 |

AI 协作：**先读 `cases-rag-index.md` → 再按需打开 `cases.md`。**

## 新需求接入清单

```
- [ ] REPOS.md：路径与发布 slot 仍正确
- [ ] requirements.md：新增需求行
- [ ] cases-rag-index.md：需求索引 + 症状/文件/biz 行
- [ ] cases.md：追加 ## Case N 全文
- [ ] 全局 pack（可选）：「进行中需求」→ 合入后移到「已完成」
```

## 发布规范速查（示例）

- 开发分支：优先 `feature-xxx`（用 `-`，不用 `/`）
- 若已有 `fix/xxx` 带斜杠分支，config 路径**保留斜杠为子目录**
- 版本号：默认复用本仓已提交、已进白名单的号，不要每个分支都 `+1`
- 仅在导师/负责人明确要求、旧号下线、或必须隔离实验包时才升号
