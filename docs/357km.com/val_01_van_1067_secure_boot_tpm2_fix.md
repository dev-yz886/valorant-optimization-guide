# 《无畏契约》VAN 1067 报错深度排查：Win11 安全启动 Secure Boot 与 TPM 2.0 状态修复全指南

**核心排查结论：** 《无畏契约》报VAN 1067根因是未开安全启动或TPM2.0。进BIOS关CSM并开UEFI安全引导与重置TPM即可修复。

---

## 一、 底层机理排查与关键参数对比表

| 排查检查项 | 故障检测指令 / 路径 | 正常状态返回值 | 异常表现与处置优先级 |
| --- | --- | --- | --- |
| TPM 2.0 物理就绪 | PowerShell 执行 Get-Tpm | TpmPresent: True, TpmReady: True | 返回 False 表示 BIOS 中 AMD fTPM / Intel PTT 被关闭（最高） |
| Secure Boot 安全启动 | PowerShell 执行 Confirm-SecureBootUEFI | 返回 True | 返回 False 表示主板安全引导未开启（最高） |
| 磁盘分区表格式 | CMD 执行 diskpart -> list disk | Gpt 栏位标注 * 符号 | 若未标注表示为 MBR 格式，需转换为 GPT 才能开启安全启动（高） |
| CSM 兼容支持模块 | BIOS 高级启动菜单 | Disabled (已关闭) | 开启 CSM 会强制阻断 UEFI 安全启动生效（最高） |

> [!WARNING]
> **安全提示：** 修改主板 BIOS 引导模式前，请务必确认当前 Windows 系统磁盘为 GPT 分区格式；若为旧式 MBR 分区，请先在管理员终端执行 `mbr2gpt /convert /allowFullOS` 转换，切勿直接关闭 CSM 导致系统无法开机引导。

---

## 二、 核心排查与实操修复步骤

### 1. 步骤 1：PowerShell 诊断 TPM 2.0 与安全启动就绪状态
以管理员身份打开 PowerShell，分别执行以下两条系统级诊断指令，查看底层安全模块返回状态：

```powershell
Get-Tpm | Select-Object TpmPresent, TpmReady, ManufacturerId
Confirm-SecureBootUEFI
```

### 2. 步骤 2：BIOS 中关闭 CSM 兼容模块并启用 UEFI 纯净安全引导
重启电脑狂按 Del 或 F2 进入主板 BIOS，导航至【Boot (启动)】菜单：
1. 找到【CSM (Compatibility Support Module)】，将其切换为【Disabled (禁用)】；
2. 进入【Secure Boot (安全启动)】，将【OS Type (操作系统类型)】设为【Windows UEFI Mode】；
3. 将【Secure Boot Mode】设为【Standard (标准)】，按 F10 保存并重启。

```cmd
# 验证磁盘 GPT 格式指令（必须确认系统盘带 * 标记）
diskpart
list disk
exit
```

### 3. 步骤 3：重置与清除 TPM 平台信任模块
按 Win+R 输入 `tpm.msc` 打开 TPM 管理工具，查看右侧状态是否显示【TPM 已就绪，可以使用】。若显示异常，点击右侧【清除 TPM】并根据屏幕提示重启完成硬件身份初始化。

```cmd
tpm.msc
```


---

## 三、 常见故障排查与深度 FAQ

#### Q: 为什么开启安全启动后电脑直接黑屏进不去系统？
> **A:** 根本原因是系统盘属于旧版 MBR 分区。纯 UEFI 启动必须依赖 GPT 分区与 ESP 引导分区。若发生此情况，需在主板 BIOS 重新开启 CSM 进系统，使用 mbr2gpt 工具完成无损转换后再开启安全启动。

#### Q: Win10 用户玩无畏契约也必须开 TPM 2.0 吗？
> **A:** Windows 10 系统目前对 TPM 2.0 无硬性校验，但依然强烈建议开启以防反作弊驱动偶发报错；而 Windows 11 环境下 Vanguard 则强制要求 TPM 2.0 与 Secure Boot 双开。


---

## 四、 站内相关深度排查推荐

- [无畏契约提示 VAN 9003 无法启动排查：主板 UEFI 模式转换与开机安全引导配置教程](https://www.357km.com/val/val_02_van_9003_uefi_key_fix.html)
- [无畏契约微秒级输入延迟压缩指南：NVIDIA Reflex 与系统队列调优实操](https://www.357km.com/val/val_01_input_latency_reflex_tuning.html)

---
*本文由 357km.com 电竞技术研究室独家实测原创，定位于合规的电竞外设调试、掉帧排查与客户端崩溃修复。*
