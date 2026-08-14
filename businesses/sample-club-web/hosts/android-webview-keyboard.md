# Host · Android WebView 键盘 / 语音（虚构示例）

> 演示「宿主对接文档」怎么写。真实项目换成你们自己的 bridge 名和权限 API。

---

## 行为模型

Android 系统 WebView **不会**把 `RECORD_AUDIO` 自动转给 H5。按住说话要成功，必须：

1. 宿主 App 已在 Manifest 声明录音权限
2. 运行时由 **原生** 调系统权限框（不能只靠 `getUserMedia`）
3. H5 通过 JSBridge 触发上述请求，并等回调后再开始录音

浏览器里没有这套 bridge → gate 必须排除普通 Chrome。

---

## 推荐 gate

```dart
bool get needsHostAudioBridge =>
    kIsWeb && isAndroidWebUserAgent() && isInHostApp();
```

| 环境 | 走这条路径？ |
|------|----------------|
| 自有 App · Android WebView | 是 |
| 自有 App · iOS WebView | 否（iOS WKWebView 权限模型不同） |
| 桌面 Chrome | 否 |
| 微信小程序 WebView | 否（走小程序授权） |

---

## 时序坑

| 坑 | 机制 |
|----|------|
| 手势链里 `await` 权限 | `onLongPress` 中间 await → 后续 pointer up 丢失 → 松手不停止 |
| 权限回调后才 bind 录音 | 首按总是 0:00；应预热或把「开始」放到手势同步段 |
| 只探测 UA、不探测 bridge | 内置浏览器 UA 像 App，但没有方法 |

---

## 排查

1. 原生侧是否弹出过系统权限框
2. bridge 方法名、回调字段是否仍与 H5 约定一致
3. 失败时 H5 有没有把错误当成「用户取消」吞掉
