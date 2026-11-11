# HxLockScreen

> 一键锁屏 —— 点击桌面图标，屏幕立即熄灭。

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![Android](https://img.shields.io/badge/android-5.0%2B-green.svg)](https://developer.android.com/)
[![Java](https://img.shields.io/badge/java-8%2B-orange.svg)](https://www.oracle.com/java/)
[![Release](https://img.shields.io/github/v/release/hxcan/HxLockScreen?label=%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC)](https://github.com/hxcan/HxLockScreen/releases)
[![Build](https://github.com/hxcan/HxLockScreen/actions/workflows/ci.yml/badge.svg)](https://github.com/hxcan/HxLockScreen/actions/workflows/ci.yml)

一个极简的 Android 锁屏工具。桌面上只有一个图标，点一下，屏幕就灭了。

---

## 为什么会有这个应用

手机的**电源键弹不起来 / 按下去没反应**（硬件老化、按键失灵、系统卡死）的时候，你可能遇到过这种情况：

> 屏幕亮着，却怎么也熄不掉。

HxLockScreen 就是为这个场景做的——不依赖任何物理按键，点一下图标就能熄屏。

顺带也适用于这些场景：

- 人一离开屏幕就该灭（临时离开、专注工作）
- 手机放在桌上不需要一直亮屏
- 屏幕意外点亮（比如放在包里），随手点掉

---

## 特性

- **零等待**：点击图标 → 立即熄屏，目标延迟 < 200ms
- **零弹窗**：正常情况下不弹任何权限对话框
- **无主界面**：不显示启动页、不显示确认框，点完就消失
- **极小体积**：APK < 100 KB
- **多级兜底**：4 级降级策略，总有一路能锁上

---

## 4 级锁屏策略

不同厂商的 ROM 对熄屏 API 的限制差别很大，所以本应用按优先级依次尝试：

| 优先级 | 策略 | 实现方式 | 说明 |
|---|---|---|---|
| **A** | 系统级熄屏 | `PowerManager.goToSleep()` | 首选，效果最佳，无需权限 |
| **B** | 模拟电源键 | `ACTION_CLOSE_SYSTEM_DIALOGS` 广播 | 部分 ROM 有效 |
| **C** | 设备管理器 | `DevicePolicyManager.lockNow()` | 需授权设备管理器（只弹一次） |
| **D** | 全屏遮罩 | 黑屏 Activity + 阻止触摸 | 最终兜底，任何环境都可用 |

每次实际命中的策略会记录在日志中：

```bash
adb logcat -s HxLockScreen
```

```
I/HxLockScreen: 方案A-goToSleep: succeeded
I/HxLockScreen: lock() cost=87ms
```

### 方案 D 的安全设计

遮罩模式会显示全屏黑屏并阻止触摸，因此内置了**双重保险**，绝不会把手机锁死：

1. 监听 `ACTION_USER_PRESENT` / `ACTION_SCREEN_ON` —— 用户一解锁，遮罩立即消失
2. **30 秒超时兜底** —— 即使收不到广播，也强制移除遮罩

---

## 使用方法

1. 安装 APK（[Releases](https://github.com/hxcan/HxLockScreen/releases) 页下载）
2. 桌面上会出现「一键锁屏」图标
3. 点它，屏幕即灭

### 可选：授权设备管理器

如果你的机型需要策略 C 才能锁屏（会在日志里显示 `方案C-DeviceAdmin: not authorized`），可以到：

> 设置 → 应用 → 特殊应用权限 → 设备管理器 → 启用「一键锁屏」

授权后立即生效，只需授权一次。

---

## 权限说明

| 权限 | 用途 | 是否必需 |
|---|---|---|
| `WAKE_LOCK` | 熄屏瞬间保持唤醒，避免被立刻唤醒 | 建议 |
| `DISABLE_KEYGUARD` | 配合设备管理器锁屏 | 可选（策略 C） |
| `SYSTEM_ALERT_WINDOW` | 方案 D 全屏遮罩 | 可选（策略 D） |

应用**不收集任何数据**，不联网，无统计、无广告。

---

## 系统要求

- Android 5.0（API 21）及以上
- 无需 root

---

## 构建

```bash
git clone https://github.com/hxcan/HxLockScreen.git
cd HxLockScreen
./gradlew assembleDebug
```

产物：`app/build/outputs/apk/debug/app-debug.apk`

需要 JDK 17+、Android SDK Platform 34。

---

## 贡献

欢迎提交 Issue 和 Pull Request！详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

遇到 Bug 请提交 [Issue](https://github.com/hxcan/HxLockScreen/issues)，选择 Bug 模板并填写：
- 手机型号
- Android 版本 / ROM 名称
- `adb logcat -s HxLockScreen` 的输出

---

## 免责声明

本应用仅调用 Android 公开的电源管理接口用于熄屏屏幕，不涉及任何系统修改。

- 本应用不会绕过系统锁屏（PIN / 图案 / 指纹）
- 熄屏后请正常使用系统解锁方式解锁
- 使用前请确认您的设备状况符合当地法规

---

## License

[Apache License 2.0](LICENSE)

## 相关链接

- 主项目：https://app.ppnew.cn
- GitHub：https://github.com/hxcan/HxLockScreen
