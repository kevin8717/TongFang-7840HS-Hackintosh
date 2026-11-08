# 驱动状态明细

> 采集日期：2026-11-08
> 系统：macOS 26.7.2 (Darwin 25.6.0, 25G332)
> 机型：TongFang / Ryzen 7 7840HS / Radeon 780M

图例：✅ 正常　⚠️ 部分可用　❌ 不可用　⏸️ 已禁用　— 不适用

---

## 1. 已启用的 kext（config.plist 实际加载）

| 顺序 | Kext | 版本 | 作用 |
|---:|---|---|---|
| 1 | Lilu.kext | 1.7.3 | 内核补丁框架 |
| 2 | VirtualSMC.kext | 1.3.8 | SMC 模拟 |
| 3 | AMFIPass.kext | 1.4.1 | AMFI 放行 |
| 4 | PhxBringup.kext | 1.0.0 | **自研**：Radeon 780M 显示加速 |
| 5 | SMCBatteryManager.kext | 1.3.8 | 电池（含 100 W ACID 修改） |
| 6 | AppleALC.kext | 1.9.9 | HDA 音频 |
| 7 | SimpleHDA.kext | 1.0.0 | **自研**：ALC257/SN6140 音频辅助 |
| 8 | AppleMCEReporterDisabler.kext | 1.2 | 禁用 MCE 报告器 |
| 9 | RestrictEvents.kext | 1.1.7 | CPU 名称/内存修正 |
| 10 | VoodooPS2Controller.kext | 2.3.8 | PS/2 键盘 |
| 11 | VoodooPS2Keyboard.kext | 2.3.8 | 键盘插件 |
| 12 | VoodooInput.kext | 2.9.1 | 触控输入插件 |
| 13 | VoodooI2CServices.kext | 2.9.1 | I2C 服务 |
| 14 | VoodooGPIO.kext | 2.9.1 | **自研修改**：AMD GPIO |
| 15 | VoodooI2C.kext | 2.9.1 | I2C 控制器 |
| 16 | VoodooI2CHID.kext | 1.0 | **自研修改**：I2C HID |
| 17 | AirportMtkWlan.kext | 2.0.0 | **自研**：MT7921 Wi-Fi |
| 18 | BlueToolFixup.kext | 2.7.2 | 蓝牙补丁 |
| 19 | SimpleMtkBtFw.kext | 1.1.0 | **自研**：MT7921 蓝牙固件 |
| 20 | SimpleMtkBtPatcher.kext | 1.1.0 | **自研**：蓝牙补丁 |
| 21 | USBPorts.kext | 1.0 | USB 端口映射 |
| 22 | PhxXhciKick.kext | 1.0.0 | **自研**：XHCI ring 1 中断修复 |
| 23 | PhxGpioProbe.kext | 1.0.0 | **自研**：GPIO 诊断 |

## 2. 已禁用（保留在 Kexts 目录）

| Kext | 版本 | 原因 |
|---|---|---|
| WhateverGreen.kext | 1.7.1 | 780M 由 PhxBringup 接管 |
| NootedRed.kext | 0.8.10 | 同上，避免冲突 |
| AMDRyzenCPUPowerManagement.kext | 0.7.2 | 与 SMC 方案冲突 |
| SMCAMDProcessor.kext | 1.0.1 | 同上 |
| GenericUSBXHCI.kext | 1.3.0b2 | 原生 XHCI 已可用 |
| USBInjectAll.kext | 1.0 | 已改用 USBPorts |
| BrightnessKeys.kext | 1.0.3 | 亮度由用户态桥接处理 |

## 3. ACPI

| SSDT | 状态 | 作用 |
|---|---|---|
| SSDT-EC-USBX-LAPTOP.aml | ✅ | EC + USBX 电源属性 |
| SSDT-PLUG-ALT.aml | ✅ | CPU 电源管理 |
| SSDT-ENTP.aml | ✅ | 键盘/触摸板相关 |
| SSDT-PHXBKL.aml | ✅ | **自研**：背光 |
| SSDT-BAT.aml | ✅ | **自研**：电池 `_BIX`/`_BST` |
| SSDT-CPUR.aml | ⏸️ | 未启用 |
| SSDT-XOSI.aml | ⏸️ | 未启用（Win 判定由其他方式处理） |
| SSDT-USBX.aml | ⏸️ | 未启用（已并入 EC-USBX） |
| SSDT-RMNE.aml | ⏸️ | 未启用 |
| SSDT-PNLF.aml | ⏸️ | 未启用（背光走 PHXBKL） |
| SSDT-TPDINI.aml | ⏸️ | 未启用（触控板调试用） |

## 4. UEFI 驱动

| 驱动 | 作用 |
|---|---|
| apfs_aligned.efi | APFS 读取（对齐修复） |
| HfsPlus.efi | HFS+ 读取 |
| OpenRuntime.efi | OpenCore 运行时 |
| OpenCanopy.efi | 图形启动菜单 |
| ResetNvramEntry.efi | 菜单内重置 NVRAM |

## 5. Boot Arguments

```text
-v keepsyms=1 revpatch=cpuname,sbvmm ipc_control_port_options=0
-amfipassbeta amfi=0x80 amfi_get_out_of_my_way=0x1
phxdebug=1 alcid=21 -nohe
```

| 参数 | 说明 |
|---|---|
| `-v keepsyms=1` | 详细日志 + 保留符号 |
| `revpatch=cpuname,sbvmm` | RestrictEvents 修正 CPU 名称与虚拟机标志 |
| `ipc_control_port_options=0` | 修复部分 IPC 崩溃 |
| `-amfipassbeta amfi=0x80 amfi_get_out_of_my_way=0x1` | AMFI 放行 |
| `phxdebug=1` | PhxBringup 调试输出 |
| `alcid=21` | AppleALC layout 21 |
| `-nohe` | 关闭 HE（硬件加密）相关 |

## 6. 未解决项与根因

### 6.1 Photo Booth 黑屏

- 菜单栏相机预览正常、拍照可取图 → UVC/解码链路正常。
- 主窗口黑屏，崩溃栈：
  `libvulkan_radeon.dylib radv_compute_pipeline_create +1132` → `VTMetalTransferSessionTransferImageSync`。
- RADV 在 `radv_compute_pipeline_compile()` 中未检查 `radv_shader_create()` 返回 NULL，
  随后 `radv_get_user_sgpr_info()` 解引用 `NULL+0x162`。
- 结论：显示栈问题，**已按用户要求停止跟进**。

### 6.2 USB4 / Thunderbolt

- NHI：`1022:1669`，ACPI `\_SB.PCI0.GP19.NHI1`。
- `AppleThunderboltNHI.kext` 只有 Info.plist、无可执行文件，且 x86 personality 仅匹配 Intel `0x0880`/`0x8A00`。
- `AppleSynopsysUSB40XHCI.kext` 同样无二进制，只匹配 Apple ARMIODevice。
- 结论：需独立 AMD NHI 内核驱动；SSDT 无法解决。USB-C 的 USB3 部分仍由 XHC 正常工作。

### 6.3 内置麦克风

- 内置麦走 AMD ACP DMIC（`62:00.5 [1022:15e2]`，Linux `snd_pci_ps` / `acp63`）。
- macOS 下该 PCI 设备只是 `IOPCIDevice`，无 ACP 音频驱动。
- 结论：需自写 ACP DMA 驱动 + HAL 插件，工作量大。

### 6.4 触控板

- `VoodooI2C` / `VoodooGPIO` / `VoodooI2CHID` 已加载，`VoodooI2CDeviceNub` 存在。
- 但多点触控未上报；GPIO 访问与 HID 描述符读取仍在逆向中。

## 7. 验证命令

```bash
# 硬件
system_profiler SPHardwareDataType
system_profiler SPDisplaysDataType

# 显卡
ioreg -l | grep -i -E 'IOAccel|MetalPlugin'

# 电池 / 电源
pmset -g batt
pmset -g custom
system_profiler SPPowerDataType

# 音频
system_profiler SPAudioDataType

# Wi-Fi / 蓝牙
system_profiler SPAirPortDataType
system_profiler SPBluetoothDataType

# kext 加载
kmutil showloaded --list-only | grep -i -E 'Phx|Voodoo|AppleALC|Mtk'
```
