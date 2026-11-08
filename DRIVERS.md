# 驱动状态明细

> 采集日期：2026-11-08
> 系统：macOS 26.7.2 (Darwin 25.6.0, 25G332)
> 机型：TongFang / Ryzen 7 7840HS / Radeon 780M

图例：✅ 正常　⚠️ 部分可用　❌ 不可用　— 不适用

---

## 1. 已启用的 kext（config.plist 实际加载）

| 顺序 | Kext | 版本 | 作用 |
|---:|---|---|---|
| 1 | Lilu.kext | 1.7.3 | 内核补丁框架 |
| 2 | VirtualSMC.kext | 1.3.8 | SMC 模拟 |
| 3 | AMFIPass.kext | 1.4.1 | AMFI 放行 |
| 4 | PhxBringup.kext | 1.0.0 | **nullarc**（luuumi-dev / nerux-real）：Radeon 780M 显示加速 |
| 5 | SMCBatteryManager.kext | 1.3.8 | 电池（含 100 W ACID 修改） |
| 6 | AppleALC.kext | 1.9.9 | HDA 音频 |
| 7 | SimpleHDA.kext | 1.0.0 | **laobamac**：恢复 Tahoe 下的 AppleHDA |
| 8 | AppleMCEReporterDisabler.kext | 1.2 | 屏蔽 Apple MCE Reporter |
| 9 | RestrictEvents.kext | 1.1.7 | CPU 名称/内存修正 |
| 10 | VoodooPS2Controller.kext | 2.3.8 | PS/2 键盘 |
| 11 | VoodooPS2Keyboard.kext | 2.3.8 | 键盘插件 |
| 12 | VoodooInput.kext | 2.9.1 | 触控输入插件 |
| 13 | VoodooI2CServices.kext | 2.9.1 | I2C 服务 |
| 14 | VoodooGPIO.kext | 2.9.1 | **alexandred** 上游 + 本地补丁：AMD GPIO |
| 15 | VoodooI2C.kext | 2.9.1 | I2C 控制器 |
| 16 | VoodooI2CHID.kext | 1.0 | **alexandred** 上游 + 本地补丁：I2C HID |
| 17 | AirportMtkWlan.kext | 2.0.0 | **Arxusion/AirportMtkWlan** + 本地 4 处修正 |
| 18 | BlueToolFixup.kext | 2.7.2 | 蓝牙补丁 |
| 19 | SimpleMtkBtFw.kext | 1.1.0 | **laobamac**（GPL-3.0）：MT7921 蓝牙固件 |
| 20 | SimpleMtkBtPatcher.kext | 1.1.0 | **laobamac**（GPL-3.0）：蓝牙补丁 |
| 21 | USBPorts.kext | 1.0 | USB 端口映射 |
| 22 | PhxXhciKick.kext | 1.0.0 | **本地实现**，思路来自 thanhnndev 的 `ItlXhciKick`：XHCI ring 1 中断修复 |
| 23 | PhxGpioProbe.kext | 1.0.0 | **本地编写**：GPIO 只读诊断探针 |

## 2. ACPI

| SSDT | 作用 |
|---|---|
| SSDT-EC-USBX-LAPTOP.aml | EC + USBX 电源属性 |
| SSDT-PLUG-ALT.aml | CPU 电源管理 |
| SSDT-ENTP.aml | 键盘/触摸板相关 |
| SSDT-PHXBKL.aml | **本地编写**：背光（配合 nullarc 的 PhxBringup） |
| SSDT-BAT.aml | **本地编写**：电池 `_BIX`/`_BST` |

## 3. UEFI 驱动

| 驱动 | 作用 |
|---|---|
| apfs_aligned.efi | APFS 读取（对齐修复） |
| HfsPlus.efi | HFS+ 读取 |
| OpenRuntime.efi | OpenCore 运行时 |
| OpenCanopy.efi | 图形启动菜单 |
| ResetNvramEntry.efi | 菜单内重置 NVRAM |

## 4. Boot Arguments

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

## 5. 未解决项与根因

### 5.1 Photo Booth 黑屏

- 菜单栏相机预览正常、拍照可取图 → UVC/解码链路正常。
- 主窗口黑屏，崩溃栈：
  `libvulkan_radeon.dylib radv_compute_pipeline_create +1132` → `VTMetalTransferSessionTransferImageSync`。
- RADV 在 `radv_compute_pipeline_compile()` 中未检查 `radv_shader_create()` 返回 NULL，
  随后 `radv_get_user_sgpr_info()` 解引用 `NULL+0x162`。
- 结论：显示栈问题，**已按用户要求停止跟进**。

### 5.2 USB4 / Thunderbolt

- NHI：`1022:1669`，ACPI `\_SB.PCI0.GP19.NHI1`。
- `AppleThunderboltNHI.kext` 只有 Info.plist、无可执行文件，且 x86 personality 仅匹配 Intel `0x0880`/`0x8A00`。
- `AppleSynopsysUSB40XHCI.kext` 同样无二进制，只匹配 Apple ARMIODevice。
- 结论：需独立 AMD NHI 内核驱动；SSDT 无法解决。USB-C 的 USB3 部分仍由 XHC 正常工作。

### 5.3 内置麦克风

- 内置麦走 AMD ACP DMIC（`62:00.5 [1022:15e2]`，Linux `snd_pci_ps` / `acp63`）。
- macOS 下该 PCI 设备只是 `IOPCIDevice`，无 ACP 音频驱动。
- 结论：需自写 ACP DMA 驱动 + HAL 插件，工作量大。

### 5.4 触控板

- **现状：轮询模式工作正常，多点触控可用**（双指缩放正常）。
- `VoodooI2C` / `VoodooGPIO` / `VoodooI2CHID` 均已加载；`VoodooI2CDeviceNub` 的 `Interrupt Mode` 显示 `GPIO`、`gpioPin=8`、`gpioIRQ=8`。
- 实测 `VoodooI2CHIDDevice` 的 `PhxInputCalls` 持续增长（约 89.9 万次），说明输入上报路径在跑。
- **未解决：GPIO 中断模式**。中断路径尚未打通，当前依赖轮询兜底；`PhxLastReportLen=0` 是中断路径未取到数据的表现。
- 待办：GPIO spurious IRQ 掩码位修正（`INTERRUPT_MASK_OFF`）需要在整包重建 VoodooI2C 全家后验证。

## 6. 验证命令

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

## 7. 来源与许可证

本 EFI 中绝大多数 kext 来自开源项目，不是本项目原创。完整上游列表、许可证和本地改动见 [CREDITS.md](CREDITS.md)。

- 本仓库根目录 `LICENSE`（MIT）**仅适用于本项目自己编写的文档、SSDT 和 `PhxXhciKick` / `PhxGpioProbe` 源码级实现**。
- `AirportMtkWlan` 为 GPL-2.0，`VoodooI2C` 全家为 GPL-3.0，`SimpleMtkBtFw` / `SimpleMtkBtPatcher` 为 GPL-3.0，
  `SimpleHDA` 许可证需向作者确认；分发这些二进制时请遵守对应许可证。
- `PhxBringup.kext` 来自 nullarc，其许可证为多许可（MIT / GPL-2.0 / PolyForm Noncommercial，按组件区分），
  详见 `_work/nullarc-src-20261106/LICENSE` 或上游仓库。
