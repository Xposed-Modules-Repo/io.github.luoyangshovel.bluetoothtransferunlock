# Bluetooth Transfer Unlock

**注意，本人只对Android系统有少量了解，项目中本人只提供想法，所有修改均为AIGC，使用前务必审查项目代码防止出现不可逆破坏。**

中文 | [English](README.en.md)

解除蓝牙发送与接收文件类型与配对界面以外不可见的限制，若无需强制所有文件类型分享时在面板内添加蓝牙功能，作用域仅选择蓝牙即可，无需选择system。

LSPosed 模块，现代 libxposed API（102）：没有 UI、没有设置页面，启用模块 → 勾选作用域 → 重启目标进程即可。

| 功能 | 一句话 |
| :- | :- |
| 一 | 接收端不再按 MIME 白名单拒绝任何文件类型 |
| 二 | 发送端不再因为没有类型 / 类型不认识而中断分享 |
| 三 | 永久可被搜索：蓝牙一开就自己断言 discoverable，不用进配对页面，也不会被超时降级 |
| 四 | 蓝牙出现在**任何** App 的分享面板里，不再只有 zip/图片/视频能看到 |

> **风险**：功能三让设备在蓝牙打开期间随时可被扫描到（不想被搜到就关掉蓝牙）；功能四是注入 system_server 的全局 hook，影响整机所有分享面板。

## 启用

作用域已预置在 APK 里，装完仍可在 LSPosed 里逐条勾选：

- `com.android.bluetooth` —— 功能一/二/三的全部实现。**只要这三项，勾这一条就够。**
- `system` —— 功能四的唯一开关（意图解析跑在 system_server）。不需要它就别勾。

重启：功能一/二/三重启蓝牙进程即可；**功能四必须重启整机**，system_server 不会因为蓝牙进程重启而重新加载模块。

## 诊断

```bash
adb logcat | grep BTUnlock   # 别用 -s BTUnlock：LSPosed 会把模块日志套在自己的标签里，整行只有内容含包名
adb shell setprop log.tag.BTUnlock VERBOSE   # 要在目标进程启动前设置
```

- `[com.android.bluetooth] hooks installed: receive=N, send=N, discoverable=N` / `[system_server] hooks installed: discoverable-fallback=N, share-sheet=M` —— 装没装上、装了几条，看这两行。
- 面板里没有蓝牙：连 `[system_server]` 这行都没有 → 作用域缺 `system`（勾上重启整机）。
- 搜不到：开机后应有一行 `discoverable: N through … is what this ROM takes for discoverable`，那是探测读回确认了编码；若 `asserted mode …` 反复出现并跟着每分钟一条 `discoverable: no encoding reached the stack …` → 三层写法都没把实时模式抬成 discoverable，这个 ROM 的 setter 换了名字或形状。

## 已知限制

1. 共享 App 自己不给类型时，补类型只发生在蓝牙进程内；面板里那条蓝牙是功能四加的，点开才走补类型逻辑。
2. 功能四在蓝牙未打开时不显示蓝牙（该 activity 本身是 disabled），是设计不是 bug。自带固定分享列表、根本不查解析的自研面板也会漏掉。
3. 功能三依赖蓝牙进程活着：重启期间最长 5 s 空窗。
4. 没有任何 UI 与通知。
5. 真机验证覆盖 Android 10 与 Android 16 各一台；11-15 是按 AOSP 源码推出的同一套形状匹配，未逐版验证。

源码与构建方式：<https://github.com/luoyangshovel/Bluetooth_Transfer_Unlock>
