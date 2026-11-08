# TongFang 7840HS Hackintosh — OpenCore EFI

> 同方（TongFang）模具 / AMD Ryzen 7 7840HS + Radeon 780M 的 macOS 26 (Tahoe) OpenCore 配置。
> 本仓库只包含 **EFI 与文档**，不含任何真实序列号、MLB、SmUUID、网卡 MAC。

![platform](https://img.shields.io/badge/CPU-Ryzen%207%208040HS-ED1C24)
![gpu](https://img.shields.io/badge/GPU-Radeon%20780M-ED1C24)
![os](https://img.shields.io/badge/macOS-26.7.2-000000)
![oc](https://img.shields.io/badge/OpenCore-1.x-4B8BBE)

---

## 1. 设备信息

| 项目 | 值 |
|---|---|
| 机型 | 同方（TongFang）AMD 笔记本，16 英寸级 |
| CPU | AMD Ryzen 7 7840HS（8C/16T，Zen 4，3.8 GHz 基频） |
| iGPU | AMD Radeon 780M（`1002:15BF`，RDNA3，12 CU） |
| 独显 | 无（或已禁用，见 ACPI） |
| 内存 | 32 GB DDR5 |
| 屏幕 | 2880×1800 内建屏，HiDPI 1440×900 @ 120 Hz |
| 音频 Codec | Conexant SN6140（`14F1:1F87`，即 ALC257 系列） |
| 有线/无线 | MT7921（Wi-Fi 6 + BT 5.2，USB 蓝牙） |
| 固件 | UEFI，OpenCore 引导 |
| 目标系统 | macOS 26.7.2 (Darwin 25.6.0, build 25G332) |
| SMBIOS | `MacBookPro16,1`（仅型号标识符，三码已清空） |

> 设备识别依据：`system_profiler SPHardwareDataType` / `SPDisplaysDataType` / `SPAudioDataType`、`ioreg`、Linux `lspci` 与 codec dump。

## 2. 驱动现状

图例：✅ 正常　⚠️ 部分可用/有已知缺陷　❌ 不可用　⏸️ 已禁用

### 2.1 核心

| 组件 | 状态 | 说明 |
|---|---|---|
| OpenCore | ✅ | UEFI 引导，`OpenCanopy` 图形菜单 |
| Lilu | ✅ | 1.7.3 |
| VirtualSMC | ✅ | 1.3.8 |
| AMFIPass | ✅ | 1.4.1，配合 `amfi=0x80` |
| RestrictEvents | ✅ | 1.1.7，CPU 名称/内存信息修正 |
| AppleMCEReporterDisabler | ✅ | 关闭 MCE 报告器 |

### 2.2 显卡 / 显示

| 组件 | 状态 | 说明 |
|---|---|---|
| Radeon 780M 加速 | ✅ | 自研 `PhxBringup.kext`（Metal 3，4 GB VRAM，内建屏 2880×1800@120Hz） |
| 亮度调节 | ✅ | 用户态 `phxbacklight-bridge` + `BL_PWM_CNTL`（不在本 EFI 内） |
| HiDPI | ✅ | 1440×900 逻辑分辨率 |
| 外接显示 | ⚠️ | 未充分验证 |
| Photo Booth 主窗口合成 | ❌ | 菜单栏预览/拍照正常，主窗口黑屏；崩溃于 `radv_compute_pipeline_create`（RADV 未检查 `radv_shader_create` 返回 NULL）。**已停止跟进** |
| NootedRed / WhateverGreen | ⏸️ | 已禁用，780M 走自研 `PhxBringup` |

### 2.3 音频

| 组件 | 状态 | 说明 |
|---|---|---|
| 扬声器 / 耳机输出 | ✅ | `AppleALC` 1.9.9 + `SimpleHDA` 1.0.0，layout 21，`alcid=21` |
| 3.5mm 外接麦克风 | ⚠️ | ALC257 输入节点存在，但 AppleALC 无该 codec 原生表项，未验证录音 |
| 内置麦克风 | ❌ | 走 AMD ACP DMIC（`1022:15E2`），macOS 无 ACP 音频驱动 |

### 2.4 电池 / 电源

| 组件 | 状态 | 说明 |
|---|---|---|
| 电池电量 | ✅ | `SSDT-BAT.aml` + `SMCBatteryManager`，100% / 60 Wh / 138 循环 |
| 实时功率 | ✅ | `_BST` 用 EC `B1CR`×10 / `B1RC`×10，电压×电流可读 |
| 适配器功率 | ✅ | `SMCBatteryManager` 内 `ACID` 由 60 W 改为 100 W |
| `hibernatemode` | ✅ | 0（电池与 AC） |
| `proximitywake` | ✅ | 0（电池与 AC） |

### 2.5 USB / 输入

| 组件 | 状态 | 说明 |
|---|---|---|
| XHCI USB 2.0/3.x | ✅ | `PhxXhciKick` 修复 ring 1 中断导致的 2 s 输入延迟 |
| USB 端口映射 | ✅ | `USBPorts.kext` |
| 键盘 | ✅ | `VoodooPS2Controller` 2.3.8 |
| 触控板（I2C HTIX5288） | ❌ | `VoodooI2C`/`VoodooGPIO`/`VoodooI2CHID` 已加载但不上报多点触控，仍在逆向 |
| USB4 / Thunderbolt | ❌ | NHI `1022:1669`；`AppleThunderboltNHI` 无 x86 可执行文件且只匹配 Intel ID。USB-C 的 USB3 部分仍可用 |

### 2.6 网络 / 蓝牙

| 组件 | 状态 | 说明 |
|---|---|---|
| Wi-Fi（MT7921） | ✅ | `AirportMtkWlan` 2.0.0，itlwm 2.0.0，2.4/5 GHz 可用 |
| 蓝牙 | ✅ | `BlueToolFixup` + `SimpleMtkBtFw`/`SimpleMtkBtPatcher`，USB transport |
| 有线网口 | — | 本机无 |

### 2.7 其他

| 组件 | 状态 | 说明 |
|---|---|---|
| 睡眠 / 唤醒 | ⚠️ | 可睡眠唤醒，亮度恢复有边界问题（拖到 0 后唤醒可能变最亮） |
| 摄像头（UVC） | ✅ | 预览与拍照正常 |
| iMessage / FaceTime | ⚠️ | 需自行填写三码与 ROM 后才能验证 |

## 3. 目录结构

```
EFI/
├── BOOT/
│   └── BOOTx64.efi
└── OC/
    ├── ACPI/            # SSDT（EC/USB、PLUG、ENTP、PHXBKL、BAT）
    ├── Drivers/         # OpenRuntime / OpenCanopy / HfsPlus / ResetNvramEntry / apfs_aligned
    ├── Kexts/           # 30 个 kext（含自研 PhxBringup / PhxXhciKick / PhxGpioProbe）
    ├── Resources/       # OpenCanopy 主题与音频
    ├── Tools/
    ├── OpenCore.efi
    └── config.plist     # 三码已清空
```

## 4. 使用前必须做的事

本仓库的 `config.plist` **故意清空了 SMBIOS 三码和 ROM**，直接使用会无法登录 iCloud/iMessage，部分功能异常。请用自己的值填写：

```text
PlatformInfo → Generic → SystemSerialNumber   # 你的序列号
PlatformInfo → Generic → MLB                  # 主板序列号
PlatformInfo → Generic → SystemUUID           # SmUUID
PlatformInfo → Generic → ROM                  # 6 字节网卡 MAC（不是 Wi-Fi 私有地址）
```

推荐用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 生成与 `MacBookPro16,1` 匹配的三码，并用 [ProperTree](https://github.com/corpnewt/ProperTree) 编辑。

`ROM` 取真实网卡 MAC：

```bash
networksetup -getmacaddress en0     # 或 ioreg -l | grep IOMACAddress
```

注意 `ifconfig en0` 可能显示 macOS 的“私有 Wi-Fi 地址”（随机 MAC，首字节本地管理位为 1），不要用它。

## 5. 已知问题与边界

1. **Photo Booth 黑屏**：RADV compute pipeline 崩溃，属显示栈问题，本仓库不含修复。
2. **内置麦克风**：AMD ACP DMIC 无 macOS 驱动，需自写 ACP 音频驱动。
3. **USB4**：需独立 AMD NHI 内核驱动，ACPI/SSDT 无法解决。
4. **触控板**：I2C HID 已加载但无多点触控上报。
5. **三码**：仓库内为空，必须自填；请勿把填好的 config 再提交回公开仓库。

## 6. 免责声明

- 本项目仅供学习研究。使用 OpenCore 修改 SMBIOS 可能违反 Apple 软件许可协议。
- 刷写 EFI 有变砖风险，请先备份原 EFI 并准备 U 盘救援。
- 仓库不含任何 Apple 版权文件；`Resources/Audio` 等为 OpenCore 自带资源。

## 7. 致谢

Acidanthera（OpenCore / Lilu / VirtualSMC / AppleALC / VoodooI2C / VoodooPS2）、
ChefKiss（NootedRed 思路）、以及各 kext 原作者。
