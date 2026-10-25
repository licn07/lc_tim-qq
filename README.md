# Lc 登录管理工具

> 面向多账号场景的轻量安卓管理工具 —— 上号 / 抓号 / 挂机 / 验号，一套搞定。
>
> 作者：**Lc** · 界面：液态玻璃 · 分发：粉丝群，非商用上架
>
> 工程有两个同构变体，源码逐字一致，仅包名不同：**TIM 版**（`com.tencent.tim`）/ **QQ 版**（`com.tencent.mobileqq`）。

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
   ├─ 检测通过 → 圆环变对勾 → 直连主界面
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
                      ├─ 三个源并发竞速，谁先成功用谁
                      │    ghproxy.net（反代，兜底）
                      │    raw.githubusercontent.com
                      │    cdn.jsdelivr.net
                      │
                      └─ 全部失败 → 读本地缓存 → 再失败才算检测未通过
```

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
  "versionCode": 207,
  "versionName": "2026.10.18",
  "minVersionCode": 0,
  "changelog": "1. 修复闪屏\n2. 优化启动动画",
  "downloads": [
    { "name": "移动云盘", "url": "https://yun.139.com/..." },
    { "name": "迅雷云盘", "url": "https://pan.xunlei.com/..." }
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

1. 改 `versionCode +1`、`versionName` 改成当天日期
2. 编译 APK，传网盘，拿分享链接
3. 编辑 `version.json`：版本号 / changelog / 网盘链接
4. commit + push

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

**处理**：内置 `GlassSafeMode`，默认**开启安全模式**（关闭全部 GPU 特效，只保留基础渲染色）。另外做了自愈机制：

- 用 SharedPreferences 记 `started` / `ui_alive`
- 上次启动开了头却没活到首帧 → 本次自动强制降级

**手动切换**：长按主界面右上角感叹号 → 开发者测试面板 → 玻璃特效降级开关 → 立即重启界面。

### 更新检测依赖网络

断网或未授权联网时无法完成版本校验，会停在失败页。这是设计如此 —— 不是 bug。

本地缓存能在离线时兜底一次（前提是之前成功拉取过）。

---

## 日志位置

排查问题时看这几个文件：

| 文件 | 内容 |
|---|---|
| `/storage/emulated/0/1LICN/启动日志.txt` | 每次启动的完整过程、结论、`[trace]` 打点 |
| `/storage/emulated/0/1LICN/crash_log.txt` | Java 未捕获异常堆栈 |
| `/storage/emulated/0/1LICN/shizuku_debug.log` | 提权通道执行记录 |

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
