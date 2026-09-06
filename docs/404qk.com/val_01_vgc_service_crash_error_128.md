# 《无畏契约》vgc 服务无法启动与错误代码 128 修复：Windows 服务项权限与开机自启配置

**核心排查结论：** 《无畏契约》vgc服务报错128是杀软拦截驱动所致。管理员执行sc config vgk start= system并加白即可解决。

---

## 一、 底层机理排查与关键参数对比表

| 核心服务/驱动 | 物理文件路径 | 标准启动类型 | 故障报错表象 | 修复处置优先级 |
| --- | --- | --- | --- | --- |
| vgc (用户态服务) | C:\Program Files\Riot Vanguard\vgc.exe | Manual (按需) 或 Auto | 手动启动弹出错误 128 / 服务终止 | 最高（SC命令重新注册） |
| vgk.sys (内核驱动) | C:\Program Files\Riot Vanguard\vgk.sys | System (系统级启动) | 被火绒/360阻断导致签名无法验证 | 最高（解除驱动拦截并恢复System） |
| RiotClientServices | C:\Riot Games\Riot Client\RiotClientServices.exe | 应用级进程 | 启动器闪退，无法建立管道通信 | 高（重置LocalAppdata凭证） |

> [!WARNING]
> **安全提示：** 在修改注册表 Services 启动项前，请在管理员 CMD 中执行 `reg export HKLM\SYSTEM\CurrentControlSet\Services\vgc C:\vgc_backup.reg` 备份原服务配置，防止服务项参数损坏造成启动故障。

---

## 二、 核心排查与实操修复步骤

### 1. 步骤 1：使用管理员 CMD 重新注册并启动 vgc 与 vgk 核心服务
以管理员身份运行命令提示符（CMD），依次输入以下命令强制将反作弊内核驱动设置为 System 级别、服务项设为自启：

```cmd
sc config vgk start= system
sc config vgc start= demand
net start vgc
```

### 2. 步骤 2：注册表解除安全软件对 vgc 服务依赖项的锁死
按 Win+R 输入 `regedit`，导航至以下服务路径，确认关键数值：

```cmd
路径：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vgc

# 确认并修改以下键值：
"Start" = dword:00000003   # 3 代表 Manual，亦可设为 2 (Auto 自动)
"Type"  = dword:00000010   # 10 代表独立进程服务
```

### 3. 步骤 3：解除第三方安全软件的驱动主动防御拦截
将 `C:\Program Files\Riot Vanguard` 目录整体加入杀毒软件（如 360、火绒、腾讯电脑管家）的【信任区】与【排除项】，并在 Windows Defender 中允许其内核通信。

```powershell
Add-MpPreference -ExclusionPath "C:\Program Files\Riot Vanguard" -ErrorAction SilentlyContinue
```


---

## 三、 常见故障排查与深度 FAQ

#### Q: 执行 net start vgc 提示服务发生系统错误 1 怎么办？
> **A:** 错误 1 代表底层 vgk.sys 内核驱动未正确加载。请先执行 `sc start vgk`，若提示文件被占用，请重启电脑并在未打开任何第三方软件的前提下启动游戏。

#### Q: 为什么每次电脑开机 vgc 都自动变成停止状态？
> **A:** 这是正常现象。vgc 属于按需启动服务（demand），只有当《无畏契约》客户端主程序唤醒时才会拉起。若游戏启动后仍不运行，则需将其设为 `start= auto`。


---

## 四、 站内相关深度排查推荐

- [无畏契约对局中突发 VAN 84 与 VAN 6 错误代码排查：网络心跳包超时修复](https://www.404qk.com/val/val_02_van_84_van_6_heartbeat_timeout.html)
- [无畏契约开火对枪瞬卡掉帧排查：清理 %LocalAppData%\VALORANT 缓存实操](https://www.404qk.com/val/val_01_shader_cache_localappdata_clean.html)

---
*本文由 404qk.com 电竞技术研究室独家实测原创，定位于合规的电竞外设调试、掉帧排查与客户端崩溃修复。*
