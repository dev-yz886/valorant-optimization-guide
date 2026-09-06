# 《无畏契约》开火对枪瞬卡掉帧排查：清理 %LocalAppData%\VALORANT 缓存与 10GB 专用着色器池扩容

**核心排查结论：** 《无畏契约》开火瞬卡根因是残留着色器坏块。清空%LocalAppData%\VALORANT与DXCache并在显卡扩容10GB着色器即可根除。

---

## 一、 底层机理排查与关键参数对比表

| 缓存类型 | 物理存储路径 | 典型故障表现 | 清理处置收益 |
| --- | --- | --- | --- |
| 虚幻引擎 PSO 局部着色器 | %LocalAppData%\VALORANT\Saved\Config | 开火交战瞬间画面冻结 0.2 秒 | 强制重新构建纯净图形管线 |
| DirectX 驱动层着色器 | %LocalAppData%\NVIDIA\DXCache | 幽影封烟、炼狱大招等粒子爆炸时严重掉帧 | 消除驱动层编译错位与坏块 |
| Riot Client 客户端日志 | %LocalAppData%\Riot Games\Riot Client\Data | 长时间连续游戏后内存泄漏越玩越卡 | 释放占用句柄，降低物理内存开销 |

> [!WARNING]
> **安全提示：** 清理 `%LocalAppData%\VALORANT` 缓存前，请务必退出游戏及 Riot 客户端；清理动作不会删除您的键位与灵敏度配置（云端自动同步），但会清空本地临时图形缓存与诊断日志。

---

## 二、 核心排查与实操修复步骤

### 1. 步骤 1：PowerShell 脚本一键清空旧版着色器与运行时坏块
以管理员身份打开 PowerShell，执行以下清理脚本强制剔除残留缓存文件：

```powershell
Remove-Item -Path "$env:LOCALAPPDATA\VALORANT\Saved\Config\*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "$env:LOCALAPPDATA\VALORANT\Saved\Logs\*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "$env:LOCALAPPDATA\NVIDIA\DXCache\*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "$env:LOCALAPPDATA\D3DSCache\*" -Recurse -Force -ErrorAction SilentlyContinue
```

### 2. 步骤 2：NVIDIA 控制面板扩容 10GB 专用着色器缓存池
驱动默认的 4GB 缓存极易被多个游戏填满并反复重置。进入 NVIDIA 控制面板 ->【管理 3D 设置】->【全局设置】，找到【着色器缓存大小 (Shader Cache Size)】，修改为【10 GB】并应用。

```cmd
NVIDIA 控制面板设置：
着色器缓存大小 -> 10 GB
```

### 3. 步骤 3：客户端启动校验与本地文件完整性修复
打开 Riot Client，点击右上角个人头像 ->【设置】->【无畏契约】-> 点击【修复 (Repair)】按钮，验证系统核心资源哈希一致性。

```cmd
操作完毕后重启电脑，进入靶场随便切枪射击 3 分钟，让显卡驱动顺畅完成初次着色器后台预编译。
```


---

## 三、 常见故障排查与深度 FAQ

#### Q: 刚清理完缓存进入第一把游戏为什么会有轻微掉帧？
> **A:** 这是正常现象。清理后显卡驱动需要对地图材质进行初次预编译与管线写入，建议进入正式排位前先在训练场释放一遍全英雄技能预热着色器。

#### Q: AMD 显卡用户如何清理着色器？
> **A:** 在 AMD 驱动面板中点击【设置】->【显卡】-> 展开【高级】，点击【重置着色器缓存 (Reset Shader Cache)】即可。


---

## 四、 站内相关深度排查推荐

- [无畏契约彻底关闭 GameDVR 与全屏优化 FSO：锁定原生独占全屏实战](https://www.479qk.com/val/val_02_disable_gamedvr_fso_exclusive.html)
- [无畏契约微秒级输入延迟压缩指南：NVIDIA Reflex 与系统队列调优实操](https://www.479qk.com/val/val_01_input_latency_reflex_tuning.html)

---
*本文由 479qk.com 电竞技术研究室独家实测原创，定位于合规的电竞外设调试、掉帧排查与客户端崩溃修复。*
