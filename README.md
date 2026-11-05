# CleanTV-With-ChildLock

> 专为国产电视及 Android 5.0+ 设备打造的高性能纯净桌面，深度集成儿童模式多规则防沉迷管控（单次限时强制退出、护眼冷却、日总额限制、大人儿童双模式）。

---

## 🌟 核心特性

### 1. 纯净 tvOS 极简美学
- **Top Shelf 顶部画卷**：光标移动到应用时，顶部大字展示名称，背景动态泛起与图标色调自适应的环境流光（Ambient Glow）。
- **液体玻璃焦点（Fluid Glass Focus）**：按方向键时光标呈阻尼弹簧物理回弹，告别生硬方框。
- **单排横向排列**：首屏 5 张大卡片占满横排，右侧自带 `+`（添加应用）卡片，支持长按弹出【移动 / 去除 / 替换 / 卸载】。
- **Apple TV 航拍屏保（Aerial）**：内置 114 段官方 4K/2K 实景航拍，闲置自动淡入，2K AVC 硬解不卡顿。

### 2. 内置儿童模式多规则防沉迷管控（核心突破）
彻底摒弃容易崩溃的第三方无障碍服务，在桌面系统内部内生实现：
- **开机管控闸门**：仅在命中已配置的儿童管控时段开机/唤醒时，弹出【👨‍👩‍👧 大人模式 / 👶 儿童模式】大卡片双选（不自动跳转，等待主动选择）；非管控时段直进大人模式，零密码、零打扰。
- **大人模式**：输入 4 位 PIN 码进入，无时长无应用限制，本次开机期间空闲不自动复位。
- **儿童模式三道防线**：
  1. **允许时段检查**：不在设定时间窗内禁止进入；
  2. **每日总时长检查**：累计时长达到上限提示明日再来；
  3. **单次限时与强制冷却退出（关键杀手锏）**：
     - 单次看满设定时长（默认 30 分钟）时，通过**系统 Home 意图抢占前台 + 强掐音频焦点 + 延迟 0.8s 杀死后台进程 + 启动全屏护眼休息屏保**；
     - 护眼屏保展示大号倒计时（默认冷却 60 分钟），吞掉所有按键防退出；
     - 支持家长输入 4 位密码临时放行 15 分钟或直接切换回大人模式。
- **多规则时段自由配置**：支持按星期（上学日/周末）、多时段、单次时长、冷却时长、每日总额自由组合。
- **时间防篡改保护**：默认开启，检测到系统时间被异常回拨时强制触发锁定。

### 3. 双通道配置（遥控器 + 手机扫码）
- **电视遥控器端**：在桌面控制中心直接打开「儿童模式与家长设置」，支持遥控器修改密码、启停规则、清除冷却。
- **手机/电脑 Web 局域网控制台**：电视展示二维码，手机扫码即可直接打字投屏、开关儿童锁、切换模式、管理规则。

---

## 🛠️ 构建与安装

### 编译 Release APK
```bash
cd CleanTV-With-ChildLock
./gradlew assembleRelease
# 产物位置：app/build/outputs/apk/release/app-release.apk
```

### 通过无线 ADB 安装到电视
```bash
adb connect <电视IP>:5555
adb install -r app/build/outputs/apk/release/app-release.apk

# 设为默认桌面并启动
adb shell cmd package set-home-activity com.cleantv.launcher/.MainActivity
adb shell am start -n com.cleantv.launcher/.MainActivity
```

---

## 🔄 版本更新：TV 端直接覆盖安装

从 v1.0.0 起，更新**不需要先卸载旧版**，直接在电视上覆盖安装即可（儿童锁规则、桌面排序等本地数据都会保留）：

```bash
adb connect <电视IP>:5555
adb install -r app/build/outputs/apk/release/app-release.apk
```

为保证覆盖安装长期可用，每次发版必须同时满足：

| 条件 | 说明 |
| --- | --- |
| `applicationId` 不变 | 固定 `com.cleantv.launcher` |
| 签名证书不变 | 所有版本使用同一张证书 |
| `versionCode` 递增 | 每次发版在 `app/build.gradle` 里 +1（当前为 `1`），否则系统会判定为降级并拒绝安装 |

签名配置位于 `app/build.gradle`：`release` 与 `debug` **共用** `signingConfigs.stable`，避免开发包与正式包在电视上签名冲突。证书文件为 `signing/cleantv-release.jks`（PKCS12，alias `cleantv`），该文件被 `.gitignore` 排除、**不会提交到公开仓库**，请务必自行备份——一旦丢失，只能卸载后重装一次，之后才恢复可覆盖安装。

> 若仓库里没有该密钥（例如他人 clone 后自行构建），Gradle 会自动回退到本机 debug 签名，产物可正常安装，但**不能用于覆盖官方 Release 包**。

发版前建议校验签名证书指纹是否一致：

```bash
apksigner verify --print-certs app/build/outputs/apk/release/app-release.apk
# SHA-256 应为 15:26:C6:AE:B9:2B:7C:6C:26:55:FC:B6:03:CD:6E:28:6A:58:53:D8:46:F4:7C:B7:07:1F:1A:9C:74:32:A4:E1
```
