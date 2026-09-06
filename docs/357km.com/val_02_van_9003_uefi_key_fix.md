# 《无畏契约》提示 VAN 9003 无法启动排查：主板 UEFI 模式转换与开机安全引导配置教程

**核心排查结论：** 《无畏契约》报VAN 9003根因是安全启动密钥丢失。进BIOS密钥管理点击Restore Factory Keys恢复出厂密钥即可秒解。

---

## 一、 底层机理排查与关键参数对比表

| 主板品牌 | BIOS 快捷键 | Secure Boot 核心设置路径 | Key Management 关键动作 |
| --- | --- | --- | --- |
| 华硕 (ASUS) | Del / F2 | Advanced Mode -> Boot -> Secure Boot | OS Type 选 Windows UEFI, 恢复 Factory Default Keys |
| 微星 (MSI) | Del | Advanced -> Settings -> Security -> Secure Boot | Secure Boot Mode 设为 Standard，Enroll all factory default keys |
| 技嘉 (Gigabyte) | Del | BIOS Features -> Secure Boot | 关闭 CSM Support -> Secure Boot Mode 设为 Standard |
| 华擎 (ASRock) | F2 | Security -> Secure Boot | Secure Boot 开启 -> Install default Secure Boot keys |

> [!WARNING]
> **安全提示：** 在执行 Key Management（密钥管理）重置出厂密钥时，切勿随意点击“Clear Secure Boot Keys”，否则会导致平台密钥（PK）彻底清空使系统陷入死循环；请务必选择“Restore Factory Keys”或“Install Default Keys”。

---

## 二、 核心排查与实操修复步骤

### 1. 步骤 1：系统信息（msinfo32）诊断 BIOS 模式与安全启动状态
按 Win+R 输入 `msinfo32` 打开系统信息窗口，重点查看以下两项指标：

```cmd
检查项目指标：
1. 【BIOS 模式】必须显示为：UEFI（若显示为传统/Legacy，必须转换磁盘）
2. 【安全启动状态】必须显示为：开启（若显示为关闭，触发 VAN 9003）
```

### 2. 步骤 2：进入主板 BIOS 重建安全引导平台密钥（PK）
重启电脑进入 BIOS，找到【Key Management (密钥管理)】选项：
1. 点击【Restore Factory Keys (恢复出厂默认密钥)】或【Enroll Factory Keys】；
2. 弹出提示“Reset without saving?”选择【Yes】；
3. 确认 Platform Key (PK) 状态由 Unloaded 变更为 Loaded (已加载)；
4. 按 F10 保存并重启进入 Windows。

```cmd
# 验证安全引导状态命令
Confirm-SecureBootUEFI
```

### 3. 步骤 3：修复系统 BCD 引导记录与签名验证状态
以管理员身份打开 CMD 命令提示符，强制校验当前 Windows 引导配置数据并启用测试签名保护：

```cmd
bcdedit /set {current} integritychecks on
bcdedit /set {current} testsigning off
bcdedit /set {current} nointegritychecks off
```


---

## 三、 常见故障排查与深度 FAQ

#### Q: 技嘉主板开启 Secure Boot 提示需先关闭 CSM，重启后依然进不去游戏怎么办？
> **A:** 这是技嘉 BIOS 的经典逻辑：先关闭 CSM 保存并重启一次，再次进入 BIOS 安全启动菜单，将 Secure Boot Mode 由 Custom 改为 Standard，点击“Restore Factory Keys”生成 PK 密钥，保存后即可永久根治 VAN 9003。

#### Q: 提示系统未处于 UEFI 模式该如何处理？
> **A:** 说明安装系统时采用了旧式 MBR 引导，需使用 Windows 官方工具 mbr2gpt 进行无损转轨，或者备份数据后重新在 UEFI 模式下重装纯净系统。


---

## 四、 站内相关深度排查推荐

- [无畏契约 VAN 1067 报错深度排查：Win11 安全启动与 TPM 2.0 修复全指南](https://www.357km.com/val/val_01_van_1067_secure_boot_tpm2_fix.html)
- [无畏契约彻底关闭 GameDVR 与全屏优化 FSO：锁定原生独占全屏实战](https://www.357km.com/val/val_02_disable_gamedvr_fso_exclusive.html)

---
*本文由 357km.com 电竞技术研究室独家实测原创，定位于合规的电竞外设调试、掉帧排查与客户端崩溃修复。*
