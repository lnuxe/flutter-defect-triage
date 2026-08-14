# Harbor Club · 需求清单（示例）

> 每接一个新需求 **先在本表加一行**，完成后关联 [cases.md](cases.md) Case ID。

## 状态图例

| 状态 | 含义 |
|------|------|
| 开发中 | feature 分支开发 / 流水线验证 |
| 待合 dev | 验收通过，待 MR |
| 已合 dev | 集成测试 |
| 已上线 | 白名单 / 全量 |

## 需求表

| ID | 产品编号 | 标题 | biz | 分支 | 国内 config | 海外 config | 状态 | Case |
|----|----------|------|-----|------|-------------|-------------|------|------|
| REQ-001 | 3.1.1 | 成员列表 Tab 铺满宽度 | channel-detail | `feature-channel-tab-width` | `0.0.10` | `0.0.4` | 已合 dev | [Case 1](cases.md#case-1-成员列表-tab-行右侧灰底) |
| REQ-002 | — | 战队职位标签在气泡/浮层对齐 | im-message | `feature-team-role-label` | `0.0.10`（复用） | — | 开发中 | [Case 2](cases.md#case-2-战队场景--职位标签在消息里缺失) |
| BUG-001 | — | Android WebView 按住说话权限 | im-input-voice | `fix/android-hold-to-talk` | `0.0.11` | `0.0.5` | 待合 dev | [Case 3](cases.md#case-3-按住说话-000--android-webview-不弹权限) |

## 待接需求（模板）

<!--
| REQ-00x | 3.x.x | 标题 | biz-xxx | feature-xxx | 0.0.NNN | 待开发 | — |
-->
