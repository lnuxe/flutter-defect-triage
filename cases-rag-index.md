# Flutter Defect Triage — RAG 检索索引

> **用途：** `cases.md` 变长后，AI **先读本索引** 定位 Case ID，再打开 [cases.md](cases.md) 对应章节。  
> **维护：** 每新增/关闭 Case 或合入 fix 时，同步更新「速查表」「症状索引」「按业务域」。  
> 下列 Case 1–3 是虚构示例。

---

## 按业务域 → Case

| biz | 案例 ID | 一句话 |
|-----|---------|--------|
| **channel-detail** | 1, 2 | Tab 灰底；职位标签列表侧 |
| **im-message** | 2 | 气泡/浮层职位标签、merge 丢 sender |
| **im-input-voice** | 3 | 按住说话、Android WebView 权限 |
| **host-bridge** | 3 | 宿主录音权限 |
| **wx-mini** | — | （示例未覆盖） |
| **web-routing** | — | （示例未覆盖） |

**业务包：** Case 1–3 → `sample-club-web`

---

## AI 检索流程

```
1. 定 biz → 查「按业务域」或 businesses/README.md
2. 用户描述 / 工单 / 日志 → 查「症状 → Case」
3. gate / 运行端 / 热点 → businesses/<slug>/pack.md
4. 宿主细节 → hosts/*.md
5. 输出坑点对齐表 → 与用户确认（SKILL.md）
6. 文件路径 → 查「文件 → Case」
7. Case ID → cases.md ## Case N
8. git log/show + blame + rg
9. 合入后更新本文件（含 biz 列）
```

---

## 症状 → Case

| 关键词 / 现象 | Case | biz | Gate | Commit |
|---------------|------|-----|------|--------|
| Tab 行右侧灰底、成员列表空白 | **1** | channel-detail | 全端 | `demo-sha-0001` |
| 职位标签缺失（消息/头像浮层）、merge 丢 sender | **2** | im-message | `isTeamScene` | `demo-sha-0002` |
| 按住说话 0:00、不弹麦克风权限 | **3** | im-input-voice | Android 宿主 bridge | `demo-sha-0003` |

---

## 文件 → Case

| 路径 | Case |
|------|------|
| `lib/channel/widgets/channel_tab_bar.dart` | 1 |
| `lib/im/team_label_resolver.dart` | 2 |
| `lib/im/widgets/message_bubble.dart` | 2 |
| `lib/channel/person_card.dart` | 2 |
| `lib/im/widgets/chat_input_bar.dart` | 3 |
| `lib/bridge/host_bridge.dart` | 3 |

---

## 速查表

| Case | 短标题 | biz | 平台 | gate | commit |
|------|--------|-----|------|------|--------|
| **1** | Tab 右侧灰底 | channel-detail | 全端 | 无（同一 widget） | `demo-sha-0001` |
| **2** | 战队职位标签分裂 | im-message | App WebView + Native | `isTeamScene` | `demo-sha-0002` |
| **3** | Android 按住说话权限 | im-input-voice | Android 宿主 WebView | `isAndroidWebUA && isInHostApp` | `demo-sha-0003` |
