# Windows 安全中心：彻底关闭与完整恢复最佳实践

> 实测日期：2026-09-25
> 系统：Windows 11 24H2（Build 26100）
> 结论：关闭与恢复流程同构，均需「UI 关篡改保护 + 提权写组策略/MpPreference/防火墙 + SYSTEM 计划任务写服务 Start」三步配合；删组策略键同样需要 SYSTEM 权限。

---

## 一、彻底关闭（按序执行）

### 1. 手动关闭篡改保护（必须 UI，程序化无法关闭）
- 路径：Windows 安全中心 → 病毒和威胁防护 → 管理设置 → 关闭「篡改保护」（弹确认时点「是」）
- 原因：篡改保护开启时，任何进程（管理员/SYSTEM/TrustedInstaller）都无法修改 Defender 配置或停止服务
- 验证：`Get-MpComputerStatus | Select-Object IsTamperProtected` 期望 `False`

### 2. 组策略 + MpPreference（提权 PowerShell）
```powershell
# 组策略：禁用防病毒
New-Item -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender' -Force
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender' -Name 'DisableAntiSpyware' -Value 1 -Type DWord

# 组策略：禁用实时防护各组件
New-Item -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection' -Force
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection' -Name 'DisableRealtimeMonitoring' -Value 1 -Type DWord
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection' -Name 'DisableBehaviorMonitoring' -Value 1 -Type DWord
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection' -Name 'DisableOnAccessProtection' -Value 1 -Type DWord
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection' -Name 'DisableIOAVProtection' -Value 1 -Type DWord

# 组策略：禁用云保护
New-Item -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\SpyNet' -Force
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\SpyNet' -Name 'DisableBlockAtFirstSeen' -Value 1 -Type DWord
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\SpyNet' -Name 'SubmitSamplesConsent' -Value 2 -Type DWord

# PowerShell：关闭各防护模块
Set-MpPreference -DisableRealtimeMonitoring $true -DisableBehaviorMonitoring $true -DisableIOAVProtection $true -DisableBlockAtFirstSeen $true -SubmitSamplesConsent 2 -MAPSReporting 0
```

### 3. 关闭防火墙
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

### 4. 禁用计划任务
```powershell
Get-ScheduledTask -TaskPath '\Microsoft\Windows\Windows Defender\' | Disable-ScheduledTask
```
> 注：之后会被系统自动恢复为 Ready，不影响防护已关闭的事实。

### 5. SYSTEM 权限写服务 Start=4（关键步骤）
- 管理员直接写 `Start=4` 会被系统自动还原为默认值（3/2），**必须 SYSTEM 权限**通过计划任务执行
- 对象：`SecurityHealthService`（界面服务）、`wscsvc`（状态服务）
- 方法：`schtasks /Create /TN ... /RU SYSTEM /RL HIGHEST /F` + `schtasks /Run`，脚本内 `Set-ItemProperty ... -Name 'Start' -Value 4`
- `Stop-Service` 即使 SYSTEM 也会报「无法打开服务」（WdFilter 内核驱动拦截），设 `Start=4` 后依赖重启或延迟停止

### 6. 重启
- 服务按 `Start=4` 不启动；通知中心弹出「启用 Windows 安全中心服务 / Windows 安全中心服务已关闭」即成功
- WinDefend 服务进程仍运行（内核级自我保护 WdFilter.sys 拦截 OpenService），空转但不执行防护，属正常

---

## 二、完整恢复（按序执行）

### 1. 删除组策略键（必须 SYSTEM 权限！）
- 实测：管理员 `Remove-Item` 报「尝试执行未经授权的操作」删除失败；子键与值其实已被删掉，仅剩空壳根键
- 正确：SYSTEM 计划任务执行
```powershell
Remove-Item 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender' -Recurse -Force
```

### 2. 恢复 MpPreference（提权）
```powershell
Set-MpPreference -DisableRealtimeMonitoring $false -DisableBehaviorMonitoring $false -DisableIOAVProtection $false -DisableBlockAtFirstSeen $false -SubmitSamplesConsent 1 -MAPSReporting 2
```

### 3. 恢复防火墙
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
```

### 4. 启用计划任务
```powershell
Get-ScheduledTask -TaskPath '\Microsoft\Windows\Windows Defender\' | Enable-ScheduledTask
```

### 5. SYSTEM 权限恢复服务 Start（同关闭方法）
- `SecurityHealthService` Start=3（Manual）、`wscsvc` Start=2（Automatic）

### 6. 重启 + 必要时手动启动
- 实测：重启后 `SecurityHealthService` 自动 Running；`wscsvc` 可能仍 Stopped，可 `Start-Service wscsvc` 手动拉起（实测成功）
- 若仍失败：正常模式多次重启（2-3 次），**勿进安全模式**（无网络）；仍不恢复再试 `sfc /scannow`、`DISM /Online /Cleanup-Image /RestoreHealth`

### 7. UI 重新开启篡改保护（最后一步）
- 路径同关闭：Windows 安全中心 → 病毒和威胁防护 → 管理设置 → 开启「篡改保护」
- 实测：本机重启后篡改保护自动恢复为 True，无需手动；如未恢复则手动开启

---

## 三、关键经验（含本会话新发现）

1. **篡改保护是总开关**：开启时一切程序化修改被拦截；必须先 UI 关闭，`Set-MpPreference -DisableTamperProtection` 返回 `0x80004001 (E_NOTIMPL)`，注册表 `TamperProtection` 值无法写入（即使 SYSTEM）
2. **服务 Start 值必须 SYSTEM 权限写**（schtasks /RU SYSTEM），管理员写入会被系统自动还原
3. **删组策略键也要 SYSTEM 权限**（管理员报「尝试执行未经授权的操作」）；但值的删除实际已生效，仅空壳根键需 SYSTEM 清理
4. Stop-Service / Start-Service 对 SecurityHealthService、wscsvc 会被 WdFilter 内核驱动拦截，依赖重启；恢复后 wscsvc 可手动 `Start-Service` 拉起
5. **PS 5.1 脚本必须 UTF-8 BOM 或纯 ASCII**，否则中文注释导致解析错误（本会话脚本均为 UTF-8 BOM + CRLF，执行成功）
6. 计划任务被禁用后会被系统自动恢复为 Ready，属正常现象

---

## 四、验证命令

```powershell
# Defender 防护状态（期望按需 True/False）
Get-MpComputerStatus | Select-Object RealTimeProtectionEnabled, BehaviorMonitorEnabled, IoavProtectionEnabled, IsTamperProtected

# 防火墙
Get-NetFirewallProfile | Select-Object Name, Enabled

# 服务
Get-Service WinDefend, SecurityHealthService, wscsvc, WdNisSvc | Select-Object Name, Status, StartType

# 计划任务
Get-ScheduledTask -TaskPath '\Microsoft\Windows\Windows Defender\' | Select-Object TaskName, State
```

---

## 五、风险提示

- 关闭后系统无病毒防护、无防火墙、无安全中心监控；勿运行来源不明的程序或打开可疑附件
- 建议完成需要的操作后尽快按第二节恢复防护
- WinDefend 进程仍在运行但无防护，不要误以为系统仍受保护
