# Business Pack · sample-club-web（虚构示例）

> **仓库：** `harbor_club/flutter_web` + `harbor_club/miniapp`  
> **biz 覆盖：** channel-detail · im-message · im-input-voice · wx-mini · host-bridge · web-routing  
> **注意：** 这是开源仓库里的**演示包**。路径、类名、gate 都是编的，不要当成真实项目。

---

## 运行端矩阵

| 口头说法 | 实际 | Gate 倾向 |
|----------|------|-----------|
| 「App 里」 | 自有 App WebView 加载 Flutter Web | `kIsWeb && isInHostApp()` |
| 「浏览器」 | 直接打开 H5 | `kIsWeb && !isInHostApp()` |
| 「小程序」 | 微信小程序 WebView | `PlatformUtils.isWxMini` |
| 「原生页」 | Flutter Native（非 Web 产物） | `!kIsWeb` |

**原则：** 口头「App 里」不等于某一个 UA。先确认是 WebView 还是 Native，再下 gate。

---

## Gate 速查

| Gate | 适用 | biz |
|------|------|-----|
| `kIsWeb` | 所有 Web 布局 / TextField / safe area | 多数 UI |
| `isTeamScene` | 战队频道（示例业务 flag） | im-message · channel-detail |
| `isInHostApp()` | 需要 JSBridge 的能力 | host-bridge · im-input-voice |
| `isAndroidWebUserAgent() && isInHostApp()` | Android WebView 录音权限 | im-input-voice |
| `PlatformUtils.isWxMini` | 小程序分享 / 底 inset | wx-mini |

**窄 gate 示例：** Android WebView 录音必须 `isAndroidWebUserAgent() && isInHostApp()`，不能扩成泛 `kIsWeb`（浏览器没有这套 bridge）。

---

## Hotspots（按 biz）

| biz | Area | Files（示例路径） |
|-----|------|-------------------|
| **channel-detail** | Tab / 成员列表 | `lib/channel/widgets/channel_tab_bar.dart` |
| **im-message** | 气泡 / 职位标签 | `lib/im/widgets/message_bubble.dart` |
| **im-input-voice** | 输入框 / 按住说话 | `lib/im/widgets/chat_input_bar.dart` |
| **host-bridge** | 权限 / 跳转 | `lib/bridge/host_bridge.dart` |
| **wx-mini** | 分享 / 安全区 | `miniapp/pages/webview/` |
| **web-routing** | SPA 回栈 | `lib/router/app_router.dart` |

---

## 刻意不修 / 已否决

- 不要用 magic 系数去「猜」WebView 键盘高度；先读 [hosts/android-webview-keyboard.md](hosts/android-webview-keyboard.md)
- 不要把 `isInHostApp()` 当成所有 Web 问题的 gate
- 战队职位标签和俱乐部用户标签是两条数据路径，不要混修

---

## 对接文档

- [hosts/android-webview-keyboard.md](hosts/android-webview-keyboard.md)
