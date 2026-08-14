# Harbor Club · 国内 / 海外仓库路径（示例）

> 演示双仓对照怎么写。路径和版本号都是假的。

## 对照

| 项 | 国内 | 海外 |
|----|------|------|
| Git 仓库 | `org/harbor-club` | `org/harbor-club-intl` |
| 本地 Git 根 | `~/src/harbor-club` | `~/src/harbor-club-intl` |
| 工程子路径 | `flutter_web/` | `flutter_web/`（相对路径相同） |
| 开发 `cd` | 上表工程子路径 | 同左 |
| 架构文档 | `docs/ARCHITECTURE.md` | 结构同源，换路径即可 |

## 路径映射（改文件时）

| 模块 | 国内 | 海外 |
|------|------|------|
| 主页 | `flutter_web/lib/home.dart` | 同相对路径 |
| 发布 config | `configs/` | `configs/`（**版本号各仓独立**） |
| bridge sender | `flutter_web/lib/bridge/` | 同相对路径 |

**保留海外差异：** 多语言、区域、上报；只移植缺陷相关逻辑。

## 发布 config（复用优先）

1. 默认复用已提交、已进白名单的版本号，不要每个新分支都 `+1`。
2. 同仓库内多 feature/fix 可共用同一版本号。
3. 国内 ↔ 海外版本号互不共用；但各自仓库内部优先复用本仓已提交号。
4. 分支含 `/` 时 config 路径保留子目录：  
   `fix/chat-input-keyboard` → `configs/fix/chat-input-keyboard_config_version.env`  
   不要把 `/` 换成 `-` 写成 flat 文件（流水线会 `Config file not found`）。

## 国内 → 海外同步

1. 国内合入 / 验收通过后，在海外 `dev` 拉**同名分支**
2. 按路径映射移植缺陷相关逻辑；保留海外差异
3. 新建海外 config，版本号优先复用本仓已提交基准
4. 海外独立 MR → 流水线
5. 不要把 A 产品的改动同步到 B 产品的海外仓
