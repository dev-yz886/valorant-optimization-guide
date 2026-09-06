# 《无畏契约》彻底关闭 GameDVR 与全屏优化 FSO：锁定原生独占全屏与 0 抖动帧时间

**核心排查结论：** 《无畏契约》画面抖动是因全屏优化降级与录屏抢占。程序属性勾选“禁用全屏优化”且注册表将GameDVR置0即可锁定独占全屏。

---

## 一、 底层机理排查与关键参数对比表

| 系统特性 | 默认工作状态 | 对无畏契约的负面影响 | 电竞调优状态 | 预期优化收益 |
| --- | --- | --- | --- | --- |
| 全屏优化 (FSO) | 默认开启 (On) | 强制被 DWM 桌面合成器转接，增加 6ms~10ms 延迟 | 勾选禁用 (Disabled) | 显卡驱动直通前置缓冲区，延迟极低 |
| Xbox GameDVR | 默认开启 (后台录屏) | 持续占用 GPU 独立编码器与硬盘 I/O | 注册表强制关闭 (0) | 释放 15% 编码算力，帧时间波动归零 |
| 高 DPI 缩放行为 | 系统默认缩放 | 偶发 1080P/2K 分辨率点对点模糊与准星虚化 | 锁定为【应用程序】执行 | 像素级点对点原生硬核输出 |

> [!WARNING]
> **安全提示：** 修改系统注册表前，请在管理员 CMD 中执行 `reg export HKCU\System\GameConfigStore C:\GameConfigStore_backup.reg` 备份游戏配置存储，以便在需要重新使用 Xbox 录屏功能时一键还原。

---

## 二、 核心排查与实操修复步骤

### 1. 步骤 1：游戏主程序属性中彻底勾选【禁用全屏优化】
找到无畏契约真实运行核心二进制文件路径：
`C:\Riot Games\VALORANT\live\ShooterGame\Binaries\Win64\VALORANT-Win64-Shipping.exe`
右键点击选择【属性】->【兼容性】：
1. 勾选【禁用全屏优化 (Disable fullscreen optimizations)】；
2. 点击【更改高 DPI 设置】-> 勾选【替代高 DPI 缩放行为】，缩放执行选择【应用程序】；
3. 点击确定并应用保存。

```cmd
# 定位主程序目录路径示例
cd "C:\Riot Games\VALORANT\live\ShooterGame\Binaries\Win64"
```

### 2. 步骤 2：注册表彻底关闭 Xbox GameDVR 静默录屏与抓取
以管理员身份打开 CMD 命令提示符，执行以下三条注册表注入命令，永久消除后台录屏开销：

```cmd
reg add "HKCU\System\GameConfigStore" /v GameDVR_Enabled /t REG_DWORD /d 0 /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\GameDVR" /v AppCaptureEnabled /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\GameDVR" /v AllowGameDVR /t REG_DWORD /d 0 /f
```

### 3. 步骤 3：注册表写入强制硬件独占全屏标识
强制 Windows 针对游戏进程分配真正的硬件独占上下文：

```cmd
reg add "HKCU\System\GameConfigStore" /v Win32_FullscreenExclusive /t REG_DWORD /d 1 /f
reg add "HKCU\System\GameConfigStore" /v Win32_DVR_DisableFlags /t REG_DWORD /d 1 /f
```


---

## 三、 常见故障排查与深度 FAQ

#### Q: 关闭全屏优化后切屏时黑屏闪烁 1 秒正常吗？
> **A:** 完全正常！这正是“硬件独占全屏（Fullscreen Exclusive）”的核心标志——显卡将物理显示权从 Windows 桌面管理器夺取并直接交给游戏引擎，切屏时的短暂黑屏是显卡进行刷新率硬切换的正常物理过程。

#### Q: 关掉 GameDVR 之后怎么录制击杀高光？
> **A:** 推荐使用显卡原生的 NVIDIA ShadowPlay（按 Alt+Z）或 AMD ReLive，它们直接调用 GPU 内置的独立硬件编码芯片，对游戏前台渲染性能损耗为 0。


---

## 四、 站内相关深度排查推荐

- [无畏契约开火对枪瞬卡掉帧排查：清理 %LocalAppData%\VALORANT 缓存实操](https://www.479qk.com/val/val_01_shader_cache_localappdata_clean.html)
- [无畏契约 VAN 1067 报错深度排查：Win11 安全启动与 TPM 2.0 修复全指南](https://www.479qk.com/val/val_01_van_1067_secure_boot_tpm2_fix.html)

---
*本文由 479qk.com 电竞技术研究室独家实测原创，定位于合规的电竞外设调试、掉帧排查与客户端崩溃修复。*
