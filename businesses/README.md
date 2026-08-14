# Business Catalog · Flutter Defect Triage

> 按**业务域（biz）**归纳 Case 与业务包。流程见 [../SKILL.md](../SKILL.md)。

本目录里的示例包是**虚构产品**，用来演示 taxonomy / pack 写法。接到真实仓库后：改 slug、改路径、清掉示例 Case。

---

## 业务域分类（biz taxonomy）

| biz slug | 中文 | 典型症状 | 典型 gate |
|----------|------|----------|-----------|
| **channel-detail** | 频道详情 / 成员列表 / Tab | Tab 灰底、顶空白、展开更多 | 场景 flag / 全端 |
| **im-message** | IM 消息 / 气泡 / 标签 | 职位标签缺失、撤回文案 | 场景 flag / `kIsWeb` |
| **im-input-voice** | 输入框 / 语音 / 键盘 / 安全区 | 贴 Home Indicator、松手失败、键盘白缝 | `kIsWeb` / 宿主 bridge |
| **channel-mgmt** | 频道管理 / 创建 / 分类 | 输入截断、选人、空态 | `kIsWeb` |
| **share** | 分享（App / 卡片） | 文案错、隐藏人数、入口不见 | 场景 flag / fallback |
| **wx-mini** | 微信小程序 H5 + 宿主 | 底贴 Home、分享无效、外链 | `PlatformUtils.isWxMini` |
| **web-routing** | Web SPA 路由 / 回栈 / URL | 子页回错页、history 重复 | `kIsWeb` |
| **host-bridge** | 宿主 JSBridge | 权限、跳转个人主页、外部 H5 | 窄 UA / bridge 探测 |
| **web-infra** | Web 基建 / CDN / 引擎 | 资源 404、CanvasKit 白屏 | 运维 / 宿主 |

**用法：** triage 时先锁定 1～2 个 `biz`，再在 [cases-rag-index.md](../cases-rag-index.md)「按业务域」表检索。

业务变了就改这张表。上面是社区 / IM / 小程序一类产品的常见切分，不是强制分类。

---

## 已接入业务包

| slug | 产品 | 说明 | pack |
|------|------|------|------|
| **sample-club-web** | Harbor Club（虚构示例） | Flutter Web + 小程序 H5 | [pack.md](sample-club-web/pack.md) |

### 新增业务包

```bash
mkdir -p ~/.cursor/skills/flutter-defect-triage/businesses/<slug>/hosts
# 复制 templates.md 骨架 → pack.md
# 在本 README「已接入业务包」加一行
# 第一个 Case 合入后更新 cases-rag-index.md（带 biz 列）
```

---

## biz × 运行端 速查矩阵

| | Native | 自有 App WebView | 浏览器 Web | 微信小程序 |
|--|--------|------------------|------------|------------|
| channel-detail | ○ | ● | ● | ○ |
| im-message | ● | ● | ● | ○ |
| im-input-voice | ○ | ●（bridge） | ● | ● |
| wx-mini | — | — | — | ● |
| web-routing | — | ● | ● | ○ |
| host-bridge | ○ | ● | ○ | ● |

● = 高频 · ○ = 偶发 · — = 通常不涉及
