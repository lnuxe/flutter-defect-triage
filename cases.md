# Case Studies

> **AI 检索：** 先读 [cases-rag-index.md](cases-rag-index.md)，再打开本文件对应 `## Case N`。  
> 下面 3 个 Case 是**虚构示例**，用来演示模板。接到真实项目后删掉或整表替换。

## Fix scoping 约定（Flutter Web）

报告入口可能是「某某 App 里」，实际跑的常常是 **flutter_web 编译产物**（WebView）。排查时：

| 错误收敛 | 问题 | 正确收敛 |
|----------|------|----------|
| 某一个宿主 UA / bridge 探测 | 只覆盖该 App | 漏掉其它 WebView |
| 某一个宿主的 iOS 探测 | 更窄 | 漏掉其它 iOS WebView 同类问题 |
| **`kIsWeb`**（必要时再加 `PlatformUtil.isIOS`） | 平台级 Web 行为 / TextField / safe area | 所有 WebView + 浏览器一致处理 |

**原则：** 「这是 Flutter Web 问题」→ 修复条件优先用 **`kIsWeb`**，不要默认挂某一个宿主的业务 flag，除非机制明确只存在于那套 bridge。

**例外（Case 3）：** Android WebView 录音需要宿主权限 + JSBridge，gate 必须为 **`isAndroidWebUserAgent() && isInHostApp()`**，不能扩到泛 `kIsWeb`。

---

## Case 1: 成员列表 Tab 行右侧灰底

**Report:** 标题行修好之后，筛选 Tab 右侧出现空白/灰块。  
**biz:** channel-detail  
**pack:** sample-club-web

**Scope Gate：**

| 维度 | 在范围 | 不在范围 |
|------|--------|----------|
| 运行端 | Flutter Web + Native（同一套 widget） | 小程序原生 Tab |
| 业务场景 | 频道成员列表 | 聊天页 Tab |
| 入口路径 | 频道详情 → 成员 | 其它列表 |
| 修复边界 | Tab 外层 Container 宽度 | 不改标题行、不改主题色 |

**坑点地图：**

| 坑 | 机制 | 证据 | 状态 |
|----|------|------|------|
| P0 无宽度 Container | `Column(crossAxisAlignment: start)` 下 Container 收缩 | widget 树 | 确认 |
| P1 页面底色透出 | 灰块其实是父页 `#F7F9FA` | 对比标题行已设 `screenWidth` | 确认 |
| P2 业务 flag | 曾怀疑角色等级导致 | 与 `hasRoleLevel` 无关 | 排除 |

**Verification：**

1. 标题：显式 `width: ScreenUtils.screenWidth`
2. Tab：外层 `Container` 无 width；子级 `SingleChildScrollView` + `Row(mainAxisSize: min)` 收缩
3. 近期 refactor 把 Tab 改成可横滑，引入 shrink；标题 fix 让对比变明显

**Root cause：** 无宽度约束的 Tab 容器在 `crossAxisAlignment: start` 下按内容收缩，右侧露出页面底色。

**Fix scoping：** 全端同一 widget；`width: double.infinity`，不加平台 gate。

**Fix：**

| 文件 | 改动 |
|------|------|
| `lib/channel/widgets/channel_tab_bar.dart` | Tab 外层 Container `width: double.infinity` |

**Commit：** `demo-sha-0001`（示例）

**刻意不改：** 标题行宽度、主题灰、横滑逻辑。

**回归面：**

| # | 场景 | 预期 |
|---|------|------|
| 1-1 | 成员列表 Tab | 行铺满，右侧无灰块 |
| 1-2 | Tab 项较少 / 较多需横滑 | 仍可滑，无溢出 |

---

## Case 2: 战队场景 · 职位标签在消息里缺失

**Report:** 成员列表有「副队长」标签，聊天气泡和点头像浮层没有。  
**biz:** im-message · channel-detail  
**pack:** sample-club-web

**Scope Gate：**

| 维度 | 在范围 | 不在范围 |
|------|--------|----------|
| 运行端 | 自有 App WebView（主报）+ 同 gate Native | 泛 `kIsWeb` 非战队 |
| 业务场景 | `isTeamScene` 战队 | 俱乐部 / 公共频道 |
| 入口路径 | 聊天气泡 + 点头像浮层 | 成员列表（已正确） |
| 修复边界 | 战队 `positionName` 映射 | 不改队长-only 撤回逻辑；不改俱乐部 `userChatLabel` |

**坑点地图：**

| 坑 | 机制 | 证据 | 状态 |
|----|------|------|------|
| P0 展示路径分裂 | 列表读 `positionName`；气泡只认队长 openId | 两套 API | 确认 |
| P1 merge 丢 sender | 自发送 success 整对象替换，补丁字段被抹掉 | `_mergeSelfMessage` | 确认 |
| P2 浮层清标签 | 战队分支 `userLabels.clear()` | person card model | 确认 |
| P3 双 openId | IM login 与 create patch 不是同一个 id 源 | 待验证 | 待验证 |

**Verification：**

1. 战队场景跳过俱乐部标签接口；消息标签走 `teamPositionLabelForSender`（改前仅 `isTeamCaptain`）
2. 拉成员列表时只缓存了队长 openId，没建 openId → positionName 映射
3. 浮层用另一套 mini-card 接口，战队分支把 labels 清掉

**Root cause：** 同一「职位标签」在列表 / 气泡 / 浮层走了三条数据路径；气泡路径只认识队长。

**Fix scoping：** `isTeamScene` 内补 openId→positionName；浮层传入列表已有 tags。不扩 `kIsWeb`。

**Fix：**

| 文件 | 改动 |
|------|------|
| `lib/im/team_label_resolver.dart` | 建 position 映射，气泡复用 |
| `lib/im/widgets/message_bubble.dart` | 用 mapping，不再只认队长 |
| `lib/channel/person_card.dart` | 战队浮层保留 tags |

**Commit：** `demo-sha-0002`（示例）

**刻意不改：** 撤回仍仅队长；俱乐部标签接口。

**回归面：**

| # | 场景 | 预期 |
|---|------|------|
| 2-1 | 战队成员列表 | 副队长标签仍在 |
| 2-2 | 副队长发消息 | 气泡有标签 |
| 2-3 | 点副队长头像 | 浮层有标签 |
| 2-4 | 俱乐部频道 | 行为与改前一致 |
| 2-5 | 自己发一条再回显 | merge 后标签还在 |

---

## Case 3: 按住说话 0:00 · Android WebView 不弹权限

**Report:** 在 Android 宿主 App 的 WebView 里按住说话，时长停在 0:00，系统权限框不出现。  
**biz:** im-input-voice · host-bridge  
**pack:** sample-club-web

**Scope Gate：**

| 维度 | 在范围 | 不在范围 |
|------|--------|----------|
| 运行端 | Android + 自有 App WebView | iOS、桌面浏览器、小程序 |
| 业务场景 | 频道聊天按住说话 | 语音消息播放 |
| 入口路径 | 输入栏长按 | 其它录音入口 |
| 修复边界 | 宿主 bridge 要权限 + 手势链不要 await | 不改 iOS；不把 gate 扩成 `kIsWeb` |

**坑点地图：**

| 坑 | 机制 | 证据 | 状态 |
|----|------|------|------|
| P0 权限在原生 | WebView 不会把 RECORD_AUDIO 给 H5 | 宿主对接文档 | 确认 |
| P1 手势链 await | `onLongPress` 里 await 权限 → pointer up 丢失 | 输入栏代码 | 确认 |
| P2 泛 kIsWeb | 若用 `kIsWeb` 会在 Chrome 走不通的 bridge | pack gate 表 | 排除（刻意不这么修） |

**Root cause：** 录音开始依赖宿主运行时权限，但请求被放进了长按手势的异步段，系统框没弹出来，松手事件也丢了。

**Fix scoping：** `isAndroidWebUserAgent() && isInHostApp()`。权限预热放到手势外；手势同步段只 start/stop。

**Fix：**

| 文件 | 改动 |
|------|------|
| `lib/im/widgets/chat_input_bar.dart` | 长按不再 await 权限 |
| `lib/bridge/host_bridge.dart` | 进入聊天页预热 `requestPermission('audio')` |

**Commit：** `demo-sha-0003`（示例）

**刻意不改：** iOS 路径、小程序授权、播放器。

**回归面：**

| # | 场景 | 预期 |
|---|------|------|
| 3-1 | Android 宿主 WebView 首次长按 | 先出系统权限；同意后能录 |
| 3-2 | 已授权后再长按 | 立即录音，松手停止 |
| 3-3 | 桌面 Chrome | 不走宿主 bridge，行为与改前一致 |
