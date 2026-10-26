# 休息提醒 · 安卓 APK 工程

护眼/久坐休息提醒的**原生安卓版**。功能与网页版完全一致（时间段内每小时间隔提醒、跳过节假日、仅工作日、单次不重复、休息倒计时、声音+震动），但**用系统闹钟驱动**，因此：

- **锁屏、杀进程、Doze 省电模式下，到点仍会精确唤醒并全屏弹出提醒**（网页版做不到）。
- 节假日数据随网页版同源：内置 2026，并可在 App 内“联网更新”拉取（数据源：开源 holiday-cn，自动抓取国务院公告）。
- 完全离线可用；联网仅用于可选“节假日更新”。无广告、无账号、免费。

## 工程结构
```
app/src/main/
  assets/index.html        ← 与网页版同一份 UI/逻辑
  java/com/restreminder/
    MainActivity.kt        ← WebView 承载网页 + JS 桥接
    SchedulePrefs.kt       ← 精确闹钟排程（AlarmManager.setExactAndAllowWhileIdle）
    AlarmReceiver.kt       ← 闹钟广播 → 弹出提醒
    BootReceiver.kt        ← 重启后自动重排
    ReminderActivity.kt    ← 全屏提醒界面（全自动：10 秒自动收起 + 自动休息计时）
```

## 提醒是全自动的（与网页版一致，无需任何点击）

| 时间点 | 自动发生的事 |
|---|---|
| 到点（如 10:00） | 全屏弹出「该休息一下了」+ 响铃 + 震动；屏幕亮起（锁屏也会弹） |
| 到点 +0s | **自动开始“休息 N 分钟”倒计时**，界面直接显示 mm:ss |
| 到点 +10s | **自动收起界面**（回到桌面/锁屏），响铃同时停止；休息计时在后台继续 |
| 到点 +N 分钟（默认 5） | 休息结束，界面自动关闭 |
| 下一个间隔点（如 11:00） | 系统闹钟自动触发下一次提醒，循环往复 |

- 全程**不需要点任何按钮**；界面上唯一的「知道了（提前收起）」只是让你提前收掉界面，**不影响休息计时，也不影响下一轮提醒**。
- 返回键等于「提前收起」，不会打断休息计时。
- 下一轮提醒由 `AlarmManager` 链式闹钟在**下一个间隔点**独立触发（默认 60 分钟一次），与休息是否结束无关，所以即使系统把后台进程回收了也照样响。
- 小提示：休息期间界面已收起、没有常驻通知（保持零通知、零打扰）；休息是否到点不影响准时提醒。

## 三种构建方式（选其一）

### 方式 A：GitHub Actions 云构建（最轻量，推荐）
**无需在本机安装任何软件**，但需要一个**免费 GitHub 账号**（这是唯一需要的账号）。
1. 注册 github.com（免费）。
2. 新建一个仓库，把本工程目录 `RestReminderAPK/` 下的所有文件推上去（含 `.github/workflows/build.yml`）。
3. 仓库 → **Actions** → 左侧 `Build APK` → **Enable workflow**。
4. 在 Actions 页点 **Run workflow**（或推送一次代码），等待约 2–4 分钟。
5. 运行完成后，在 `Artifacts` 里下载 `rest-reminder-apk`（含 `app-debug.apk`）。
6. 手机：设置 → 关于手机 → 连点“版本号”开启**开发者选项** → 开启 **USB 调试**；把 apk 传到手机直接安装（或电脑 adb install）。无需 Google Play 账号，侧载即可。

> 调试版（debug）APK 已自带签名，可直接安装使用。

### 方式 B：本机 Android Studio
**不需要注册任何账号**，但需要下载安装 **Android Studio（约 1GB+，含 SDK）**。
1. 下载安装 Android Studio（developer.android.com）。
2. 打开本工程目录（`RestReminderAPK/`）。
3. 等待 Gradle 同步完成（首次会下载依赖）。
4. 菜单 **Build → Build Bundle(s) / APK(s) → Build APK(s)**。
5. 在 `app/build/outputs/apk/debug/` 得到 `app-debug.apk`，传到手机安装。

### 方式 C：其他云构建（Expo EAS / Codemagic）
同样需要注册对应服务的**免费账号**，本环境无法替你触发（需你自己的凭据）。流程与 A 类似，上传工程后由其云端出 APK。

## ⚠️ 关键：国产 ROM 必须做白名单
华为/小米/OPPO/vivo 等会激进杀后台、限制闹钟。请务必：
- 设置 → 应用 → 休息提醒 → **电池/耗电管理 → 设为“无限制/允许后台”**。
- 设置 → 应用 → 休息提醒 → **闹钟和提醒 → 允许**（Android 12+ 精确闹钟权限）。
- 设置 → 应用 → 启动管理 → 允许自启动/关联启动。
否则即使闹钟设好，系统也可能不响。

## 说明
- 本工程为可编译源码 + 一键 CI 配置；当前环境无 Android SDK，无法在此直接产出 `.apk` 二进制，请按上述方式构建。
- 应用 ID：`com.restreminder`；最低支持 Android 7.0（API 24）。
- 如需改包名/图标，改 `app/build.gradle` 的 `applicationId` 与 `app/src/main/res/drawable/ic_launcher.xml` 即可。
