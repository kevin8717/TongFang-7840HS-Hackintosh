# 安装说明

## 1. 准备

1. 备份现有 EFI（整个 `EFI` 文件夹）。
2. 准备一个 FAT32 U 盘，把本仓库的 `EFI/` 复制到 U 盘根目录。
3. 确认 BIOS：关闭 Secure Boot，开启 Above 4G Decoding / Resizable BAR（如可用）。

## 2. 填写三码（必须）

本仓库 `config.plist` 的 `PlatformInfo → Generic` 中以下字段为空：

```text
SystemSerialNumber = ""
MLB                = ""
SystemUUID         = 00000000-0000-0000-0000-000000000000
ROM                = <empty>
```

用 GenSMBIOS 生成 `MacBookPro16,1` 三码后填入；`ROM` 填真实网卡 MAC（6 字节）。

> 注意：`plutil -data` 需要 base64；用 ProperTree 直接编辑十六进制更稳妥。

## 3. 首次启动

1. U 盘启动，进入 OpenCore 菜单。
2. 选择 `Reset NVRAM`（仅首次，或改 SMBIOS 后）。
3. 再选择 macOS 安装盘 / 系统盘。

## 4. 安装后

- 把 U 盘 EFI 复制到硬盘 ESP（`disk0s1`），确认可脱离 U 盘启动。
- 检查 `pmset -g custom`：`hibernatemode 0`、`proximitywake 0`。
- 电池校准：完整充放电一次。

## 5. 常见问题

| 现象 | 排查 |
|---|---|
| 卡 `[EB|#LOG:EXITBS:START]` | 检查 `Booter/Quirks`、`ProvideCurrentCpuInfo` |
| 黑屏但系统在跑 | 检查 `PhxBringup` 是否加载、`phxdebug=1` 日志 |
| 电池不显示 | 确认 `SSDT-BAT.aml` 与 `SMCBatteryManager` 均启用 |
| 无 Wi-Fi | 确认 `AirportMtkWlan` 与 `itlwm` 版本匹配 |
| iMessage 不可用 | 三码/ROM 不匹配，重新生成并 Reset NVRAM |

## 6. 回滚

用备份的 `EFI` 覆盖回 ESP 即可；OpenCore 本身不写固件。
