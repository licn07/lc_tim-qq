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
