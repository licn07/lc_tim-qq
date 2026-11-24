# Lc 登录管理工具

> 面向多账号场景的轻量安卓管理工具 —— 上号 / 抓号 / 挂机 / 验号，一套搞定。
>
> 作者：**Lc** · 界面：液态玻璃 · 分发：粉丝群，非商用上架
>
> 工程有两个同构变体，源码逐字一致，仅包名不同：**TIM 版**（`com.tencent.tim`）/ **QQ 版**（`com.tencent.mobileqq`）。到底该装哪个，见下方「[两个版本怎么选](#两个版本怎么选)」。

---

## 两个版本怎么选

仓库每次发版放两个 APK，**功能完全一样，代码一模一样，只有安装包名不同**。
选哪个，只看你的手机上还要不要留着官方 QQ 正常用。

| | **TIM 版** | **QQ 版** |
|---|---|---|
| APK 文件名 | `免root隐藏QQ版本_<版本号>.apk` | `卸载QQ使用版本_<版本号>.apk` |
| 安装包名 | `com.tencent.tim` | `com.tencent.mobileqq` |
| 占用了谁的包名 | 官方 **TIM**（手机上极少有人装） | 官方 **QQ** |
| 装之前要不要卸载官方 QQ | **不用**，直接装 | **必须卸载**，否则装不上 |
| 主界面「QQ」分组 | 显示 | 隐藏 |
| 内置「伪装卸载 QQ / 一键恢复」 | 有 | 无 |
| 怎么处理官方 QQ | 用软件里的开关临时隐藏，随时一键还原（数据不丢） | 已经不存在了，没什么可处理 |

### 三种情况直接对号入座

**① 这台手机我还得正常聊 QQ**
→ 装 **TIM 版**（`免root隐藏QQ版本`）。
官方 QQ 原地不动。要上号时进主界面「QQ」分组点「伪装卸载 QQ」，软件通过 Root 的 `pm hide` 或 Shizuku 的 `pm uninstall -k --user 0` 把 QQ 藏起来，登录聊天记录全保留；用完点「一键恢复」原样回来。

**② 这是专门的跑号机，手机上不需要 QQ**
→ 装 **QQ 版**（`卸载QQ使用版本`）。
它自己就顶着 `com.tencent.mobileqq` 这个包名，官方 QQ 装不上也不会碍事，省掉每次手动隐藏的步骤。

**③ 我在软件里上号，但不想动官方 QQ 一个字节**
→ 还是 **TIM 版**，但别点「伪装卸载 QQ」，只做上号。隐藏只是可选动作，不是必须步骤。

### 两个都装行不行

技术上可以（包名不同、签名相同），但**不建议**：
两个版本共用同一份数据目录 `/storage/emulated/0/1LICN/数据号/` 和同一份日志、同一份账号文件，同时开着会互相抢写入，排查问题时日志也混在一起分不清。

### 别搞错的三件事

1. **官方 TIM 也装不了** —— TIM 版占着 `com.tencent.tim`。虽然用 TIM 的人少，但手机上如果本来有 TIM，得先卸掉。
2. **换版本要重装** —— 两个包名不同，从 TIM 版换到 QQ 版不是覆盖升级，是重新安装。账号数据在 `/1LICN/数据号/` 里，不受影响。
3. **升级必须同签名** —— 两个包都是同一把 keystore 签的，所以 TIM 版 → TIM 版、QQ 版 → QQ 版之间可以直接覆盖安装，不用卸载。

### 主界面那个「QQ」分组为什么有时不出现

软件会拿 `getPackageName()` 和自己比：不是 `com.tencent.tim` 就把「QQ」整组连同「伪装卸载 QQ / 一键恢复」按钮一起 `GONE`。
因为 QQ 版自己就占着 QQ 的包名，没有"再隐藏一次"的余地 —— 隐藏自己等于把自己干掉。所以那一组在 QQ 版里按设计就是隐藏的，不是坏了。

---

## 这是什么

一套用 Java + Android 原生 View 从零写的账号管理工具，把「多开上号」这件事里最烦的几步自动化掉：

- 一个个切号登录太慢 → **上号模式**批量处理
- 手动抄授权链接太蠢 → **抓号模式**直接提取
- 挂着要人盯 → **挂机模式**托管
- 号是不是活的心里没底 → **验号模式**直连官方校验

界面没有用任何 UI 框架（没有 Material、没有 ConstraintLayout、没有 Compose），全部自绘。液态玻璃特效也是手写的 —— 包括 RenderEffect、AGSL 着色器、模糊、高光、内阴影。

---

## 功能

### 上号模式
- 支持卡密登录与授权链接导入
- 内置 OP 格式转换器（H5，本地 assets 加载，不联网）
- 登录页支持自定义背景（相册选图 / 内置图 / 遮罩浓度可调）
- 横屏操作，贴合 QQ 授权页习惯

### 抓号模式
- 从 WebView 会话里提取登录态
- 支持模板化 URL 拼装
- 结果落盘到 `/1LICN/数据号/<游戏名>/N号.txt`

### 挂机模式
- 常驻托管，配合通知栏状态

### 验号模式
- 直接对 `graph.qq.com`（QQ Connect）发起校验
- 不经过任何中间服务器，不埋点、不上报
- 逐个校验结果落日志

### 工具页
- **OP 格式转换** —— 标准授权链接转上号器 OP 格式
- **验号** —— 独立的账号体检入口
- **冻结工具** —— 一键冻结 / 解冻目标应用
- **伪装卸载 / 一键恢复** —— 卸载 QQ 但保留数据，可完整还原

### 液态玻璃界面
- 五套主题，跟随明暗模式
- 玻璃卡片：半透明底 + 顶部高光 + 渐变描边 + 内层柔光
- 底部 Tab、按钮、滑块、开关全部玻璃化
- **安全模式**：检测到 GPU 后端不稳定时自动降级为纯色渲染，防 native 崩溃（详见下方「已知问题」）

---

## 技术栈

| 项 | 说明 |
|---|---|
| 语言 | Java 17 + 少量 Kotlin（Shizuku 桥接） |
| 最低支持 | Android 8.0（API 26） |
| 编译目标 | SDK 36 / buildTools 36.0.0 |
| UI | 纯手写原生 View，零 UI 框架依赖 |
| 提权 | Root（`su`）优先，Shizuku 兜底 |
| 依赖 | 仅 `dev.rikka.shizuku:api` + `:provider` |

体积刻意压到最小：正式包约 **240 KB**。

---

## 启动流程

```
SplashGateActivity（启动闸门）
   │  ① 显示背景（自定义图 / 内置图 / 光晕）
   │  ② 蓝色缺口圆环旋转 +「正在检测更新，请稍等」
   │
   ├─ 检测通过 → 圆环停止 → 内容层淡出放大「推远消失」→ 直连主界面
   │
   └─ 检测未通过（断网 / JSON 异常）→ 红叉失败页
            │  明确写出原因
            └─ 「重新检测」按钮，不提供跳过通道
```

**为什么要有这道闸门**：软件必须完成一次版本校验才允许进入功能页。这是硬性要求，不是可选项。

首次安装的用户还会经过全屏引导（欢迎页 → 权限页 → 口令验证）。

---

## 更新检测

不依赖服务器、不调 GitHub API、不需要 token。

```
APP 启动 ──GET──> GitHub 仓库根目录的 version.json
                      │
                      ├─ 五个源并发竞速，谁先成功用谁
                      │    ① cdn.jsdmirror.com        （国内可用）
                      │    ② gh-proxy.com             （国内可用）
                      │    ③ ghproxy.net              （反代，兜底）
                      │    ④ raw.githubusercontent.com
                      │    ⑤ cdn.jsdelivr.net
                      │
                      └─ 全部失败 → 读本地缓存 → 再失败才算检测未通过
```

单源超时 8 秒，整轮预算 13 秒。前两个是国内镜像，因为 GitHub 的官方域名在国内基本全线阻断（raw 被 DNS 污染、jsDelivr 连接重置），加上它们之后华为/荣耀这类机型才不再误报「没网」。

校验通过后写 `update_pass.flag`，7 天内不再联网检测；**只有真实联网拿到 JSON 才刷新这个时间戳**，走本地缓存放行不续期。

### 更新卡片

检测到新版本后弹出**强制更新**卡片：

- 不可取消，点外部 / 返回键均无效
- **没有「跳过」按钮**
- 只有「去网盘下载」一个出口 → 点了打开网盘并放行
- 多个下载源竖排，按 JSON 里的顺序

### version.json 格式

放在仓库**根目录**，文件名用英文，分支 `main`：

```json
{
  "versionCode": 211,
  "versionName": "2026.10.26",
  "minVersionCode": 0,
  "changelog": "1. 修复闪屏\n2. 优化启动动画",
  "downloads": [
    { "name": "移动云盘（TIM 版）", "url": "https://yun.139.com/..." },
    { "name": "移动云盘（QQ 版）", "url": "https://yun.139.com/..." }
  ],
  "size": 0,
  "md5": ""
}
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `versionCode` | 是 | 整数，必须大于本地才提示，每次发版递增 |
| `versionName` | 是 | 显示给用户看的版本号 |
| `changelog` | 否 | 更新说明，换行必须写成 `\n` |
| `downloads` | 是 | 下载源数组，几项就显示几个按钮 |
| `minVersionCode` | 否 | 预留 |
| `size` | 否 | 字节数，>0 才在卡片里显示体积 |

**注意**：`changelog` 里的换行必须是 `\n` 两个字符，直接敲回车会破坏 JSON；数组最后一项后面不能有逗号。

### 发版流程

1. 改 `versionCode +1`、`versionName` 改成当天日期（两个工程都要改）
2. 编译两个 APK，按 `免root隐藏QQ版本_<版本号>.apk` / `卸载QQ使用版本_<版本号>.apk` 命名，传网盘，拿分享链接
3. 编辑 `version.json`：版本号 / changelog / 两个网盘链接放进 `downloads`
4. commit + push

`downloads` 里请**把两个版本都列出来，名字里标明「TIM 版 / QQ 版」**，否则用户不知道该下哪个。

`raw.githubusercontent` 推送即生效；jsDelivr 有约 12 小时 CDN 缓存（代码里带时间戳破缓存）。

---

## 构建

### 环境要求

- JDK 17
- Android SDK：platform `android-36`、build-tools `36.0.0`
- Gradle 8.13
- **aapt2 必须是 arm64 原生版**（x86_64 的 aapt2 在 arm 设备上跑不起来）

如果 aapt2 报 `Daemon startup failed`，在 `gradle.properties` 里加：

```properties
android.aapt2FromMavenOverride=/path/to/arm64/aapt2
```

### 命令

```sh
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-arm64
export ANDROID_HOME=/path/to/android-sdk
export GRADLE_USER_HOME=/path/to/.gradle

cd "tim版测试包-Lc上号器"   # 或 "qq版测试包-lc上号器"
gradle clean :app:assembleRelease
```

产物：`app/build/outputs/apk/release/app-release.apk`

> `clean` 别省。AGP 的 up-to-date 判定偶尔抽风，增量编译会给你一个旧包。

### 签名

正式包使用自建 keystore，V1 关、V2/V3/V4 开。

> ⚠️ **keystore 与密码必须永久备份**。丢失后所有已发布版本都无法覆盖更新，用户只能卸载重装（数据全丢）。

---

## 目录结构

```
app/src/main/java/com/tencent/tim/
├── SplashGateActivity.java   启动闸门：更新检测 + 加载动画
├── GuideActivity.java        首次引导 + 口令验证
├── MainActivity.java         主界面（约 3000 行，功能主体）
├── UpdateChecker.java        更新检测（三源竞速 + 本地缓存）
├── GlassSafeMode.java        玻璃特效安全模式判定
├── StartupLog.java           启动日志（含崩溃前打点）
├── CrashHandler.java         崩溃捕获（双落点）
├── SignatureGuard.java       签名校验（防重打包）
├── ThemeManager.java         主题/配色
├── GlassCardDrawable.java    玻璃卡片绘制
├── CenterCropDrawable.java   等比铺满 + 居中裁剪背景
└── ...

com/liquidglass/java/miba/
├── LiquidGlassView.java      玻璃视口
├── RuntimeShader.java        AGSL 着色器封装
├── Platform.java             后端能力探测（含安全模式开关）
└── ...（20+ 个玻璃组件）

com/tencent/open/agent/
├── AgentActivity.java        上号/抓号/挂机会话
└── AgentStyle.java           会话页独立背景
```

---

## 已知问题

### 部分机型液态玻璃闪烁 / 闪退

**症状**：荣耀等机型上验证完口令直接闪退，`crash_log.txt` 为空、无 Java 堆栈。

**原因**：液态玻璃依赖的 `RenderEffect` / `RuntimeShader`(AGSL) / `RenderNode` 在个别 GPU 上会在 native 层直接崩进程 —— 不产生 Java 异常，所以崩溃捕获抓不到。

**处理**：内置 `GlassSafeMode`，按机型能力给默认值 ——

- **Android 13（SDK 33）及以上**默认**完整特效**，这些机型 RuntimeShader/AGSL 齐全，一刀切降级只会让好硬件跑得丑
- **Android 12 及以下**默认**安全模式**，老 GPU 后端缺失，强开会在 native 层崩
- 用户手动切换过一次后，以用户的选择为准，不会被默认值覆盖

自愈机制（只在真崩时才降级）：

- 用 SharedPreferences 记 `stage_began` / `ui_alive`
- 探测窗口收窄到「**真的开始创建玻璃视图** → 首帧活下来」这一段（`enterGlassStage()`）
- 启动闸门页、引导页本来就不用玻璃特效，**不参与判定** —— 在闸门页遇到断网、划掉进程、走引导流程都不会被判成崩溃
- 只有「上次进了玻璃阶段却没活到首帧」才强制降级

**手动切换**：长按主界面右上角感叹号 → 开发者测试面板 → 玻璃特效降级开关 → 立即重启界面。

### 更新检测依赖网络

断网或未授权联网时无法完成版本校验，会停在失败页。这是设计如此 —— 不是 bug。

本地缓存能在离线时兜底一次（前提是之前成功拉取过）。

---

## 日志位置

排查问题时看这几个文件：

| 文件 | 内容 |
|---|---|
| `/storage/emulated/0/1LICN/数据号/上号器日志/启动日志.txt` | 每次启动的完整过程、结论、`[trace]` 打点、各更新源实测结果 |
| `/storage/emulated/0/1LICN/数据号/上号器日志/crash_log.txt` | Java 未捕获异常堆栈 |
| `/storage/emulated/0/1LICN/数据号/上号器日志/shizuku_debug.log` | 提权通道执行记录 |
| `/storage/emulated/0/1LICN/数据号/上号器日志/update_pass.flag` | 7 天免检标志（空文件，mtime 即通过时刻） |
| `/storage/emulated/0/1LICN/数据号/上号器日志/调试模式` | 存在则显示调试面板（手动建空文件） |
| `/storage/emulated/0/1LICN/数据号/上号器日志/bg_cache/` | 内置背景图缓存 |

软件自己产出的文件全部收纳在 `数据号/上号器日志/` 下，`/1LICN/` 根目录不再有散落文件。

老版本（≤209）升级上来时会自动搬迁一次：`启动日志.txt`、`crash_log.txt`、`shizuku_debug.log`、`update_pass.flag`、`数据耗/调试模式`、`bg_cache/` 六项挪进新目录，其它文件一律不动。搬迁用 `renameTo` 保留 mtime，所以 `update_pass.flag` 的 7 天计时不会被刷新。目标位置已存在同名文件时不覆盖。

启动日志里的 `[trace]` 是无条件写入的，即使启动结论已经落盘也会继续追加。它的用途是定位「进程在 onCreate 之后凭空消失」这类非 Java 层崩溃 —— 有几个 trace，就证明活到了哪一步。

---

## 开发者测试面板

长按主界面右上角**感叹号**进入：

| 功能 | 说明 |
|---|---|
| 玻璃特效降级 | 现场切换安全模式 / 完整特效 |
| 立即重启界面 | 让开关生效（不杀进程） |
| 设备后端能力 | 显示 RenderNode / RenderEffect / RuntimeShader 支持情况与实际启用档 |
| 检查更新 | 手动检测，失败给明确错误 |
| 启动日志 | 弹窗查看日志尾部 |
| 清空启动日志 | 干净复现用 |
| 恢复默认设置 | 清除自愈标记与开关 |

---

## 免责声明

本项目为个人学习与技术研究作品，仅供交流使用。

- 使用本工具产生的一切后果由使用者自行承担
- 请勿用于任何违反相关法律法规或平台服务条款的用途
- 因违规使用导致的账号封禁、数据丢失等问题，作者不承担任何责任

作者：**Lc** · 版权所有 © 2026

---
## 发版待办（内部记录，对外发布前可删）

<!-- 发版待办 -->

### 待办 1（未完成）：更新内容要区分 TIM / QQ
现在 `version.json` 两个版本共用同一份，`changelog` 只能写成通用文案。
以后发版要把两个版本各自的更新内容分开写，二选一：

- **方案 A（零代码改动）** 字符串内分节：
  ```json
  "changelog": "【TIM 版】\n1. ...\n2. ...\n\n【QQ 版】\n1. ..."
  ```
  缺点：两边都会看到对方的条目。

- **方案 B（要改代码）** 新增 `changelog_tim` / `changelog_qq` 两个字段，
  客户端按运行时包名取对应字段，缺省回落 `changelog`。
  要改 `UpdateChecker.parse()` 和弹窗取值处，`UpdateInfo` 加两个字段。

发版前先确认走哪个，没明确指示就默认 B（干净，且旧客户端读不到新字段会自动回落 `changelog`，兼容安全）。
### 待办 2（未完成）：大版本更新提醒
用户要求下一次**大版本**更新时主动提醒。
当前：`versionCode = 212`，触发点定为 **versionCode 进入 300** 时（或用户另行声明的大版本号）。
届时发版前先提醒一句，并把待办 1 一并落实。

### 待办 3（未完成）：免 Root 安装走 content:// 交给系统安装器（2026-11-14 记）

**背景**：215 版已修复「system_server 读不了 FUSE 路径」的问题（改为先 `dd` 到
`/data/local/tmp/lc_install/base.apk` 再 `pm install`）。但仍有机器连这条路都不通，
表现为：`/data/local/tmp` 写不进去，或 ROM 对 shell 身份的 `pm install` 全量拦截。

**现状兜底**：已做成弹窗引导用户手动安装（`showRestoreManualDialog`），
弹窗里给出备份 APK 完整路径 + 「不会删除聊天记录、不会退出登录」的说明 + 复制路径按钮。

**待接入方案**（用户要求先记着，暂不实现）：
不再依赖 shell 身份执行 `pm install`，改为完全由本应用发起安装会话：

1. 用 `PackageInstaller`（`getPackageManager().getPackageInstaller()`）
   `createSession()` → `openWrite()` 把 APK 字节流写进会话 → `commit()`；
2. 这条路由**本应用自己**作为 installer 身份，`system_server` 读的是会话里的数据，
   不经过 FUSE 路径，绕开 SELinux 限制；
3. 需要用户手动确认安装界面（系统会弹「是否安装」），点一次即可；
4. 注意点：
   - `targetSdk = 29`，`PackageInstaller` 在 API 21+ 可用，无需提权；
   - 会话写入要用 `SessionParams.MODE_FULL_INSTALL`；
   - 若 ROM 要求 `REQUEST_INSTALL_PACKAGES` 权限，需在 Manifest 声明并引导用户开启「安装未知应用」；
   - QQ 版（`com.tencent.mobileqq`）伪装成 QQ，此路径同样适用。
**评估**：比现在的手动弹窗体验好（用户只需点一次确认，不用自己找文件），
但需要新增 Manifest 权限 + 引导页新增「安装未知应用」权限项，工作量约半天。
**触发条件**：若 215 版仍有用户反馈装不上，就上这个方案。

### 待办 4（已记，未决策）：免 Root「伪装卸载」要不要改成 disable-user（2026-11-14 记）

**背景**：免 Root 通道的「伪装卸载QQ」本质是**真卸载**：
`backupApk()` → `pm uninstall -k --user 0 com.tencent.mobileqq`。

`-k` 会保留 `/data/data/<pkg>` 和 `/sdcard/Android/data/<pkg>`，所以聊天记录**不会因为这一步丢**。
但重装之后如果 uid 变了，旧数据目录属主不符会被系统当作孤儿，效果等同数据丢失。

**已排除的错误方案**：原代码想用 `pm install --uid <n>` 把 uid 装回去。
实测 AOSP `cmd package install` 的参数表里**根本没有 `--uid`**（该参数只存在于
`pm list packages` / `pm dump` 的过滤条件中）。217 版已删除这条无效尝试。

**可选替代方案（需用户拍板）**：
用 `pm disable-user --user 0 <pkg>` 代替卸载：
- 桌面图标消失、应用不可启动，视觉上等同「卸载了」
- **包还在、uid 不变、数据绝对安全**，随时 `pm enable` 秒恢复
- 代价：在「设置 → 应用管理 → 显示系统应用」里仍能看到 QQ，伪装不完全

**取舍**：要绝对的数据安全就选 `disable-user`；要伪装彻底就维持真卸载（现在有完整性校验兜底）。
用户未表态前**保持现状不动**。

### 实测结论：`pm uninstall -k` 后重装，uid 不变（2026-11-14 实测）

用户明确要求「必须是真正的卸载，但绝对不要删除数据与登录状态」。为此做了三轮真机实验，
用**独立测试包 `com.lc.uidprobe`**（与 QQ 无关，实验后已彻底清理），全程未触碰 QQ 数据。

| 实验 | 结果 |
|---|---|
| 安装，记录 uid | uid=10587 |
| 写入 `marker.txt` / `shared_prefs/login.xml` / `databases/msg.db` | — |
| `pm uninstall -k com.lc.uidprobe` | 数据目录与全部文件**保留** |
| 重装 | **uid 仍是 10587** ✅ |
| 文件回读 | `marker.txt`、`login.xml`（含 uin/skey）、`msg.db` **全部完好** ✅ |
| 再来一轮卸载+重装 | **uid 仍是 10587**，文件依然完好 ✅ |
| 对照组：`pm uninstall`（不带 `-k`） | 数据目录**被删**，重装后 uid 变成 **10589** ❌ |

**机制**：`-k` 卸载后 `packages.xml` 保留了该包的 uid 记录，重装时 PackageManager
把原 uid 还给它，因此 `/data/data/<pkg>` 的属主仍然匹配，聊天记录与登录态可直接读取。

**结论：`-k` 这个参数是数据安全的命根子，绝不能省，也绝不能换成不带 `-k` 的卸载。**

### 218 版的五道保险

1. **备份完整性**：必须读到源与备份的字节数且完全一致，否则中止、不卸载
2. **卸载前取 uid**：取不到就中止（无法保证重装后数据可读）
3. **只允许 `pm uninstall -k`**：代码里不存在不带 `-k` 的卸载路径
4. **卸载后验证**：立刻 `ls -ld /data/data/<pkg>`，目录不在就报「严重」并停止
5. **恢复后比对 uid**：装完比对 `currentUid()` 与数据目录属主，不一致明确告警，
   绝不谎报「恢复成功」
**已知边界**：若 ROM 在卸载后清掉了 uid 记录（实测未出现），重装会分配新 uid，
此时第 5 道保险会触发告警，但无法自动修复 —— 数据仍在磁盘上，只是读不到。

### 223 版：UserService 绑定永久卡死（真机日志定位）

粉丝日志（Redmi chopin / Android 13 / TIM 222）暴露一个致命 bug：
**`bindUserService` 发起 24 次，`onServiceConnected` 回调 0 次，`onNullBinding` 0 次，异常 0 次。**
三个回调一个都没触发，全程静默。

反编译 `api-13.1.5` 找到根因：

```java
// Shizuku.bindUserService 字节码
invokeinterface IShizukuService.addUserService(...)  // 返回 int 错误码
pop                                                  // ← 直接丢弃，不检查！
```

服务端拒绝绑定时的错误码被 `pop` 掉了，客户端既看不到原因、也不触发任何回调。

**由此产生两个连锁 bug：**

1. **`bindState` 永久卡在 `STATE_BINDING`**
   `ensureService()` 开头有 `if (bindState == STATE_BINDING) return false`，
   状态永远不退出 → 之后每次重绑都被自己挡掉。
   `waitForService()` 的半程补绑条件也要求 `STATE_IDLE`，同样永不成立。
   → 表现为「重启后绑一次，然后永久无通道」。

2. **缓存清理逻辑永不执行**
   原清理条件 `if (bindFailCount > 0)`，而 `bindFailCount` 只在 `catch` 里自增。
   服务端静默拒绝不抛异常 → 计数恒为 0 → 清理不执行 → 每次复用同一个死连接。

**223 的修复：**

| 修复 | 说明 |
|---|---|
| 绑定超时自愈 | 新增 `BINDING_TIMEOUT_MS = 8000`，卡住超时即强制 `unbindUserService(remove=true)` 后重来 |
| 无条件清缓存 | 每次绑定前一律 `unbindUserService(args, conn, true)`，不再依赖 `bindFailCount` |
| 半程补绑条件放宽 | 从「仅 `STATE_IDLE`」改为「`IDLE` 或 `BINDING` 且已超时」 |
| **newProcess 第二条腿** | `execShizukuByProcess()` 反射调用 `Shizuku.newProcess()`，完全不经过 UserService |
| `destroy` code 兼容 | `IUserService` 原来只认 `16777114`，补上标准 AIDL 的 `code == 2` |
| 环境快照 | 每次绑定记录 `Shizuku.getVersion()` / `getUid()` / 组件包名，便于远程定位 |

**`newProcess` 是关键退路**：签名 `private static ShizukuRemoteProcess newProcess(String[], String[], String)`，
返回 `java.lang.Process` 子类。proguard 里已有 `-keep class rikka.shizuku.** { *; }`，反射可取。
权限与 UserService 完全相同（都是 Shizuku 服务端 uid），但少一层「另起 Android Service 进程」，
成功率明显更高。**UserService 彻底绑不上的机器靠这条能救回来。**

### 223 版：取不到 QQ 的 uid

粉丝反馈「取不到 QQ 的 id 所以无法卸载」。原 `currentUid()` 只认一条命令：

```kotlin
val out = exec("pm list packages -U $pkg")   // 单点失败 → 返回 -1 → 直接中止
```

实测本机 QQ（uid=10723）四条路径都能拿到，现改为逐条兜底：

| 路径 | 命令 | 实测输出 |
|---|---|---|
| 1 | `pm list packages -U <pkg>` | `package:... uid:10723,99610723,...` |
| 2 | `stat -c '%u' /data/data/<pkg>` | `10723` |
| 3 | `dumpsys package <pkg>` | `appId=10723` |
| 4 | `ls -ld /data/data/<pkg>` | `u0_a723` → 10000+723 |

**路径 2 最可靠**：QQ 若曾被 `pm uninstall -k` 卸载过，包不在「已安装」列表里，
路径 1 会返回空 —— 但数据目录还在，属主就是原 uid，而这也正是重装后要对上的那个号。

### 223 版：`detectPermissionBlocked` 假阳性

原实现把 `exec()` 返回的错误文本当成了 `dumpsys` 输出：

```kotlin
if (out.contains("system-excidle") || out.contains("system,") || out.length > 20) {
    inWhitelist = out.contains(selfPkg)   // "[错误] 未检测到稳定通道..." 长度 > 20 → 判为不在白名单
}
```

粉丝日志里每次都打印 `白名单=false 分桶=-1 shellOk=false` —— 全是假数据。
后果是把真正的「UserService 绑定失败」误报成「电池优化限制」，让用户白关一堆设置。

**修复**：`exec` 返回含 `[错误]` 时一律视为数据无效（`whitelistKnown=false`）；
且 `shellOk=false` 不再作为「被系统拦」的证据（那是循环论证 —— 通道没通本来就是待解释的现象）。

### 223 真机验证结果（粉丝日志，Redmi chopin / Android 13）

223 装上后拿到实测数据，**核心修复确认生效，但同时暴露两个新问题**。

**✅ 已验证成功**

```
Shizuku 环境: 服务端版本=13 / uid=2000(shell) / 组件包=com.tencent.tim
execShizukuByProcess 成功（newProcess 通道）：uid=2000(shell) gid=2000(shell) ...
```

| 指标 | 结果 |
|---|---|
| `newProcess` 通道成功次数 | **222 次，失败 0 次** |
| `detectChannelNow: newProcess 通道可用` | **111 次，0 次不可用** |
| 8 秒超时自愈是否触发 | **触发 2 次**，死绑定被正确解开 |
| UserService 回调次数 | 仍是 **0**（绑定 83 次） |

**这排除了三个此前的猜测**：Shizuku 版本不老（13）、uid 正常（2000）、组件包名没拼错。

**❌ 暴露的问题一：每条命令白等 3 秒**

223 期间 116 次 `exec`，间隔中位 **3230ms**（正好是 `waitForService(3000)` 的超时）。
分布：`1500-3500ms` 占 171/270。

原因：`detectChannelNow()` 每次都先跑 `waitForService(3000)`，必然失败后才走 newProcess。

**❌ 暴露的问题二：界面显示与实际状态矛盾**

启动日志写 `UserService 绑定失败（点「重新连接」）`，但 newProcess 通道完全可用。
`diagnose()` 只判 UserService，不认 newProcess，导致**界面误导用户以为功能坏了**。

**❌ 暴露的问题三：并发重复探测**

日志里大量行同毫秒成对出现（`exec > id`、`detectChannelNow`、`execShizukuByProcess`），
说明多线程同时调用 `exec()`，探测开销被翻倍。

**❌ 发现：223 期间用户从未成功触发过伪装卸载**

223 时段执行的命令只有三种：`dumpsys deviceidle whitelist`(42) / `id`(37) / `am get-standby-bucket`(37)，
**没有一次 `pm uninstall` / `cp -f` / `backupApk`**。4 次伪装卸载全部发生在 08:47–09:13（222 时期）。

即：用户升级后**根本没点成那个按钮**，全被 `detectPermissionBlocked` 的反复探测耗掉了。

### 224 版的修复

| # | 问题 | 修法 |
|---|---|---|
| 1 | 每条命令白等 3 秒 | 新增 `userServiceBroken` 标志，一旦确认 UserService 废了就**永久跳过** `waitForService` |
| 2 | 每条命令重探通道 | `cachedChannel` + `CHANNEL_CACHE_MS=30000`，30 秒内复用结论 |
| 3 | 界面误报绑定失败 | `diagnose()` 增加 newProcess 探测，可用即返回 `WHY_OK`；`channelDetail()` 显示「进程通道」 |
| 4 | 并发重复探测 | `synchronized(channelLock)` 双检锁，并发调用只让一个真探 |
| 5 | 用户连点 | `tryEnterPrivilegeAction()` 3 秒闸门 + 按钮置灰 + 提示剩余秒数 |

**缓存失效时机**（必须同时清理，否则会卡在旧结论）：
Binder 就绪 / Binder 断开 / `resetUserService()` / `exec` 执行失败。

**防连点实现细节**：`AtomicLong` 存上次点击时间 + CAS 抢占，`AtomicBoolean privilegeRunning` 防并发，
`finally { exitPrivilegeAction() }` 保证异常也能恢复按钮（否则会永久卡灰）。

### Root 通道加固（2026-11-16，版本号仍为 224）

Shizuku 通道验证通过后回头审 Root 通道，发现 6 处隐患。**均不影响用户操作流程**（按钮、提示、交互全不变）。

| # | 隐患 | 后果 | 修法 |
|---|---|---|---|
| 1 | `pm hide` 只试一种写法 | ColorOS/OriginOS/部分 HyperOS 必须 `--user 0`，失败后**无退路** | 依次试 `pm hide` → `pm hide --user 0` → `pm disable-user --user 0` |
| 2 | Root 分支不备份 APK | 数据被清/包损坏时恢复无兜底 | `runCatching { backupApk() }`，失败不影响主流程 |
| 3 | `pm unhide` 无回退、`pm enable` 返回值被忽略 | `disable-user` 兜底过的机器启用不回来 | 两种 unhide 写法都试 + 检查 enable 结果 |
| 4 | `isRootAvailable()` 永久缓存 | Magisk 补授权后仍认为无 Root，只能重启 App | 成功才永久缓存；失败 5 分钟后自动重测 + `resetRootCache()` |
| 5 | `execRootStrict` 无超时 | su 授权弹窗不响应 → `waitFor()` 永久阻塞，日志无痕迹 | `ROOT_CMD_TIMEOUT_MS = 25000`，超时 `destroyForcibly()` |
| 6 | 串行读 stdout/stderr | 子进程写满 64KB 管道缓冲即死锁（`pm install` 长错误栈易触发） | stderr 独立线程并行抽干（照抄 `UserService.java` 写法） |

**关键设计：`isPackageHiddenRoot()` 按真实状态复核**

不同 ROM 的 `pm hide` / `pm disable-user` 输出文案不一致：

```
AOSP:     "Package xxx new hidden state: true"
部分 ROM: "Package xxx new state: disabled"
还有的:   只返回 Success 或空
```

原来只靠字符串匹配，匹配不上就报「失败」——**但包其实已经隐藏了**。用户看到失败会重复点、换通道，反而把状态搞乱。

现在三道判据任一成立即认为已隐藏：
1. `pm list packages <pkg>` 查不到 → 已隐藏
2. `dumpsys package` 里 `enabled=false` → 已禁用
3. 输出关键字匹配

**返回值格式保持兼容**：`伪装卸载QQ成功 [...]` / `恢复成功 [...]` 前缀不变，
`MainActivity` 里 `startsWith("恢复失败 [免Root·安装被系统拒绝]")` 的判断不受影响。

### 全量查 bug：修掉 5 处（2026-11-16，版本号仍为 224）

Root 通道加固后又做了一次全量审查，**发现 2 个严重 bug（一直存在，非本轮引入）+ 3 个中等问题**。

#### 🔴 严重 ①：`ls -l` 大小解析抓错了数字

原代码：`Regex("\\s(\\d+)\\s").find(lsOut)` —— 取第一个「空格+数字+空格」。

真实输出：

```
-rw-rw---- 1 u0_a309 media_rw 2696 2026-09-18 17:25 /a/x.apk
           ↑                        ↑
        硬链接数 1               真正的文件大小
```

**第一个匹配到的是硬链接数 1，不是大小。** 影响三处：

| 位置 | 后果 |
|---|---|
| `installApkRobust` 预检 | 日志永远「待装 APK：（**1 字节**）」；`size<=0` 检查失效 |
| `stageApkToTmp` 校验 | `got` 恒为 1，`got == size` 永不成立 → **暂存必然失败** |
| `fileSizeOf` 兜底 | 同上 |

第 2 条最致命：暂存到 `/data/local/tmp` 是绕开 FUSE SELinux 限制的关键路径，
校验永远失败意味着**免Root 安装只能退回「从 /storage 直接装」，而那条路必被 SELinux 拦**。
→ 这很可能就是粉丝反馈「免Root 装不上」的真正根因。

**修法**：新增 `parseLsSize()`，按空白切分，权限位那行第 5 列（index 4）才是大小。
已用 8 个用例验证（普通文件/大 APK/目录/不存在/total 行/通道错误/空/多行混合）全部正确。

#### 🔴 严重 ②：`isPackageHiddenRoot` 把「通道失败」误判成「已隐藏」

```kotlin
val visible = exec("pm list packages $pkg")
if (!visible.contains("package:$pkg")) return true   // 判为已隐藏
```

`exec` 失败返回 `[错误] 未检测到稳定通道…`，它不含 `package:xxx`，于是被判成「已隐藏」。

**后果：通道挂了却告诉用户「伪装卸载QQ成功」—— 谎报成功，这是最不能犯的错。**

修法：先检查返回是否含 `[错误]`，拿不到真实数据就返回 `false`（当作未隐藏）。

#### 🟡 中等 ③④：uid 正则从错误信息里抓数字

```kotlin
Regex("(\\d{4,})")   // currentUid 路径 2
Regex("(\\d+)")      // restoreFromBackup 读 uid.txt
```

`stat` / `cat` 失败时返回的路径里若含数字（如 `com.lc.uidprobe2026`），正则会抓到 `2026` 当 uid。

修法：改成整行锚定 `^\s*(\d{4,})\s*$` + 必须 `>= 10000`。

#### 🟡 中等 ⑤：`restoreFromBackup` 的 `installed` 判断

`pm list packages` 失败 → 判成「未安装」→ 走进「按备份重装」分支。
虽然 `-r` 覆盖安装本身安全，但日志会误导排查。已加错误标记检查。

#### 已验证不是 bug 的项

| 检查项 | 结果 |
|---|---|
| `fileSizeOf` 第一段正则 `^(\d+)\s*$` | ✅ 报错文本不含纯数字，安全 |
| `isPackageHiddenRoot` 递归风险 | ✅ `detectChannelReal` 不调用它，无死循环 |
| 防连点 CAS 逻辑 | ✅ 正确 |
| 所有 `exec` 命令清单 | ✅ 无危险操作 |



---

### 字段契约（parse() 实际读取，名字不能改）

#### ⚠️ 下载链接是固定的，写配置时不要动

**这条是踩过坑的：我在 223 时把链接从「移动云盘 + 迅雷云盘」误改成「蓝奏云 + 百度网盘」，224 又照抄了错的。**

唯一正确写法（`downloads` 数组，顺序也不要变）：

```json
"downloads": [
  { "name": "移动云盘（不限速·推荐）", "url": "https://yun.139.com/shareweb/#/w/i/2rJV7GtJAta1c/*p90JUKLVJRVm:/" },
  { "name": "迅雷云盘", "url": "https://pan.xunlei.com/s/VOjfAF-1eQp0GgVEkeNvvICAA1?pwd=ngqb#" }
]
```

发新版时**只改 `versionCode` / `versionName` / `changelog` / `size` / `md5`**，`downloads` 一个字都不要动。
用户是往同一个网盘链接里覆盖上传新包，链接本身不变。

历史存档佐证：`version_221.json` / `version_222.json` 就是这两条链接。
`version_223.json` / `version_224.json` 早期版本里的「蓝奏云 + 百度网盘」是**错误内容**，不要参考。

`downloadUrl` 这个单字段**不写**（代码里是 `downloads` 为空时的兜底，写了反而多一个要同步的地方）。

错误版本已留档：`version.json.bak_wronglink_20261116`
| 字段 | 类型 | 备注 |
|---|---|---|
| `versionCode` | int | 必填，≤0 直接判格式错误 |
| `versionName` | string | 仅用于显示，不参与更新判断 |
| `minVersionCode` | int | 目前恒 0 |
| `changelog` | string | 也兼容 `updateLog` |
| `downloads[]` | array | 每项 `{name, url}`，url 为空的项会被跳过 |
| `size` / `md5` | long / string | 目前恒 0 / 空 |
| `downloadUrl` | string | 兼容 `apkUrl`，仅在 downloads 为空时生效 |

更新判断只看 `versionCode > 本地 code`，所以 `versionName` 带不带 `-TIM/-QQ` 不影响检测。