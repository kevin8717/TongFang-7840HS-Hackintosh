# 来源与许可证（CREDITS）

本仓库的 EFI 是在多个开源项目基础上整合、修改而成。**除下表标注为"本地编写/本地实现"的项目外，其余 kext 均为第三方作品**，
版权归原作者所有。

> 采集/核对日期：2026-11-08。许可证取自各上游仓库 `LICENSE` 文件或 GitHub API。

---

## 1. 显示 / GPU

| 组件 | 上游 | 许可证 | 本地改动 |
|---|---|---|---|
| `PhxBringup.kext` | [luuumi-dev/nullarc](https://github.com/luuumi-dev/nullarc)<br>本机取自 [nerux-real/nullarc-760M](https://github.com/nerux-real/nullarc-760M) | 多许可：MIT / GPL-2.0 / PolyForm Noncommercial（按组件） | 仅配置层：PCI ID `0x15bf1002` 匹配、`phxdebug=1`、构建 33 |
| `PhxAccel.kext`（`/L/E`，不在 EFI 内） | 同上 | 同上（`src/phxaccel` 为 PolyForm Noncommercial） | 无 |
| `NVMTLDriver.bundle` | 同上 | PolyForm Noncommercial | 曾修改 `-[NVMTLTexture usage]`（已按用户要求停止） |
| `WhateverGreen.kext` | [acidanthera/WhateverGreen](https://github.com/acidanthera/WhateverGreen) | BSD-3-Clause | 无（已禁用） |
| `NootedRed.kext` | [ChefKissInc/NootedRed](https://github.com/ChefKissInc/NootedRed) | 见上游（NOASSERTION） | 无（已禁用） |

> nullarc 的许可证说明：`src/phxbringup`、`tools/*` 为 MIT 或 GPL-2.0；`src/phxaccel`、`src/nmplugin`、
> `tools/nvmtl-patches` 为 PolyForm Noncommercial 1.0.0（源自 NullMoth 的 macOS 驱动）；`mesa-patches-phoenix` 为 MIT。

## 2. 音频

| 组件 | 上游 | 许可证 | 本地改动 |
|---|---|---|---|
| `AppleALC.kext` 1.9.9 | [acidanthera/AppleALC](https://github.com/acidanthera/AppleALC) | BSD-3-Clause | 加入 Conexant SN6140（`14F1:1F87`）资源，layout 21 |
| `SimpleHDA.kext` 1.0.0 | [laobamac/SimpleHDA](https://github.com/laobamac/SimpleHDA) | 未声明（使用前请向作者确认） | 无 |
| `SSDT-PHXBKL.aml` | — | 本项目 MIT | **本地编写**，配合 PhxBringup 背光 |

## 3. 网络 / 蓝牙

| 组件 | 上游 | 许可证 | 本地改动 |
|---|---|---|---|
| `AirportMtkWlan.kext` 2.0.0 | [Arxusion/AirportMtkWlan](https://github.com/Arxusion/AirportMtkWlan)<br>（另有 [mnnobre](https://github.com/mnnobre/AirportMtkWlan)、[lincyppq/AirportMtkWlan-RZ616](https://github.com/lincyppq/AirportMtkWlan-RZ616) 参考） | GPL-2.0 | **4 个文件**：`makeBeaconUpdate` 有效位改回 1/1/1；teardown 与 deep-sleep 顺序修正；新增 15 秒后台扫描节流；`lastWclScanMs` 成员 |
| `BlueToolFixup.kext` 2.7.2 | [acidanthera/BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM) | GPL-2.0 | 无 |
| `SimpleMtkBtFw.kext` 1.1.0 | [laobamac/SimpleMtkBtFw](https://github.com/laobamac/SimpleMtkBtFw) | GPL-3.0 | 删除 `IOMatchCategory`；`0e8d:0608` 的 `IOProbeScore` 4000 → 94000 |
| `SimpleMtkBtPatcher.kext` 1.1.0 | [laobamac/SimpleMtkBtFw](https://github.com/laobamac/SimpleMtkBtFw) | GPL-3.0 | 无 |

## 4. USB / 输入

| 组件 | 上游 | 许可证 | 本地改动 |
|---|---|---|---|
| `PhxXhciKick.kext` 1.0.0 | 思路来自 [thanhnndev/ga403wm-hackintosh](https://github.com/thanhnndev/ga403wm-hackintosh) 的 `docs/USB-XHCI-LAG.md`（`ItlXhciKick`） | 本项目 MIT（实现部分） | **本地独立实现**（`_work/s8-interrupt-kick/PhxXhciKick.cpp`） |
| `USBPorts.kext` | 本地生成 | 本项目 MIT | **本地编写**（端口映射） |
| `USBInjectAll.kext` | [RehabMan/USBInjectAll](https://github.com/RehabMan/USBInjectAll)（原仓库已归档/不可用） | 见上游 | 无（已禁用） |
| `VoodooPS2Controller.kext` 2.3.8 | [acidanthera/VoodooPS2](https://github.com/acidanthera/VoodooPS2) | 见上游（NOASSERTION） | 无 |
| `VoodooI2C.kext` 2.9.1 | [alexandred/VoodooI2C](https://github.com/alexandred/VoodooI2C) | GPL-3.0 | 本机分支 [kevin8717/VoodooI2C-htix5288](https://github.com/kevin8717/VoodooI2C-htix5288) |
| `VoodooGPIO.kext`（插件） | 同上 | GPL-3.0 | **本地补丁**：spurious IRQ 掩码位（`INTERRUPT_MASK_OFF`）修正 |
| `VoodooI2CHID.kext` | [alexandred/VoodooI2CHID](https://github.com/alexandred/VoodooI2CHID) | GPL-3.0 | **本地补丁**：RESET 后补 `SET_POWER(ON)`，轮询路径 |
| `VoodooInput.kext` | [acidanthera/VoodooInput](https://github.com/acidanthera/VoodooInput) | GPL-2.0 | 无 |
| `PhxGpioProbe.kext` 1.0.0 | — | 本项目 MIT | **本地编写**（只读诊断探针） |

## 5. 核心 / SMC / 电池

| 组件 | 上游 | 许可证 | 本地改动 |
|---|---|---|---|
| `Lilu.kext` 1.7.3 | [acidanthera/Lilu](https://github.com/acidanthera/Lilu) | BSD-3-Clause | 无 |
| `VirtualSMC.kext` 1.3.8 | [acidanthera/VirtualSMC](https://github.com/acidanthera/VirtualSMC) | BSD-3-Clause | 无 |
| `SMCBatteryManager.kext` 1.3.8 | [acidanthera/VirtualSMC](https://github.com/acidanthera/VirtualSMC) | BSD-3-Clause | **二进制修改**：`ACID` 常量 60 W → 100 W |
| `AMFIPass.kext` 1.4.1 | [dhinakg/AMFIPass](https://github.com/dhinakg/AMFIPass)（仓库可能已迁移） | 见上游 | 无 |
| `RestrictEvents.kext` 1.1.7 | [acidanthera/RestrictEvents](https://github.com/acidanthera/RestrictEvents) | BSD-3-Clause | 无 |
| `AppleMCEReporterDisabler.kext` 1.2 | [xlnc/AppleMCEReporterDisabler](https://github.com/xlnc/AppleMCEReporterDisabler) | 见上游 | 无 |
| `SSDT-BAT.aml` | — | 本项目 MIT | **本地编写**（基于原始 DSDT 的 `_BIX`/`_BST` 重写） |
| `AMDRyzenCPUPowerManagement.kext` 0.7.2 | [trulyspinach/AMDRyzenCPUPowerManagement](https://github.com/trulyspinach/AMDRyzenCPUPowerManagement) | 见上游 | 无（已禁用） |
| `SMCAMDProcessor.kext` 1.0.1 | [trulyspinach/SMCAMDProcessor](https://github.com/trulyspinach/SMCAMDProcessor) | BSD-3-Clause | 无（已禁用） |
| `BrightnessKeys.kext` 1.0.3 | [acidanthera/BrightnessKeys](https://github.com/acidanthera/BrightnessKeys) | BSD-3-Clause | 无（已禁用） |

## 6. 引导 / ACPI

| 组件 | 上游 | 许可证 |
|---|---|---|
| OpenCore（`OpenCore.efi`、`BOOTx64.efi`、`OpenRuntime.efi`、`OpenCanopy.efi`、`ResetNvramEntry.efi`） | [acidanthera/OpenCorePkg](https://github.com/acidanthera/OpenCorePkg) | BSD-3-Clause |
| `HfsPlus.efi` | [acidanthera/OcBinaryData](https://github.com/acidanthera/OcBinaryData) | 见上游 |
| `apfs_aligned.efi` | 见 `01-EFI备份` 来源记录 | 见上游 |
| `SSDT-EC-USBX-LAPTOP.aml` / `SSDT-PLUG-ALT.aml` / `SSDT-ENTP.aml` | 社区通用 SSDT（Dortania 等） | 见上游 |

## 7. 本项目自己编写的内容

以下内容为本项目原创（MIT，见根目录 `LICENSE`）：

- 所有 Markdown 文档（`README.md`、`DRIVERS.md`、`INSTALL.md`、本文件）
- `PhxXhciKick.kext` 源码实现（思路来自 thanhnndev 文档，代码独立编写）
- `PhxGpioProbe.kext` 源码
- `SSDT-BAT.aml` / `SSDT-PHXBKL.aml` 源码
- `USBPorts.kext` 端口映射
- `AirportMtkWlan` / `VoodooGPIO` / `VoodooI2CHID` 的本地补丁（补丁部分）

## 8. 分发注意

- **GPL-2.0**（`AirportMtkWlan`、`BlueToolFixup`、`VoodooInput`）：分发二进制时须提供对应源码或书面要约。
- **GPL-3.0**（`VoodooI2C` 全家、`SimpleMtkBtFw`/`Patcher`）：同上。
- **PolyForm Noncommercial**（nullarc 的 `phxaccel` / `nmplugin`）：仅限非商业用途。
- **BSD-3-Clause**（Acidanthera 系列）：保留版权声明与免责声明。
- `SimpleHDA` 未声明许可证，公开分发前请先联系作者。

如发现署名或许可证有误，请开 issue 指正。
