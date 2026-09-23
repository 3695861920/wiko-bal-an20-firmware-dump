# WIKO Hi畅享70 Pro 5G (BAL-AN20) 固件分区提取

> 本仓库提供 WIKO Hi畅享70 Pro 5G (BAL-AN20) 的固件分区镜像，供研究、恢复、适配使用。
> 所有文件从设备完整 `flash.bin` 中按 GPT 分区边界精确提取，不包含任何个人数据。

---

## 设备信息

| 项目 | 值 |
|------|-----|
| 设备型号 | BAL-AN20 |
| 设备名称 | WIKO Hi畅享70 Pro 5G |
| 处理器 | MediaTek MT6833 (Dimensity 700) |
| 系统版本 | Android 14 (14.0.0.140 C12E7R2P2) |
| 内核版本 | Linux 4.14.186+ |
| 分区方案 | **A-only**（无 `_a` / `_b` 后缀） |
| Boot header | **v2**，page_size 2048 |
| 内核加载地址 | `0x40080000` |
| 内核基址（虚拟） | `0xffffff8008080000` |

---

## 仓库内容

### 文件说明

| 文件 | 用途 |
|------|------|
| `boot.img` | 内核 + header，可用于分析、打补丁 |
| `Image` | 解压后的 ARM64 内核，用于符号恢复、漏洞适配 |
| `ramdisk.img` | 独立 ramdisk 分区，与 `boot` 分离 |
| `vbmeta*.img` | AVB 元数据，包含分区哈希与签名 |
| `dtbo.img` | 设备树覆盖，用于硬件适配 |
| `recovery.img` | 恢复模式镜像 |
| `lk.img` | 联发科引导程序 |

---

## 提取方式

1. 通过 BROM 模式从设备读取完整 `flash.bin`（119 GB）。
2. 解析 `flash.bin` 内的 GPT 分区表（**4096 字节逻辑扇区**，GPT 头位于 LBA1）。
3. 按每个分区的起始 LBA 与大小，用 `dd` 精确切出。
4. 对切出的镜像做结构校验（header magic、gzip 流完整性、cpio 完整性）。

---

## 关键发现

- **A-only 布局**：全表 63 个分区，没有任何 `_a` / `_b` 后缀。
- **无 `init_boot` 分区**：ramdisk 位于独立的 `ramdisk` 分区，而非 `init_boot`。
- **`boot` 为 v2 header**：`page_size = 2048`，`kernel_addr = 0x40080000`，`ramdisk_size = 0`。
- **`vbmeta` 签名**：`SHA256_RSA2048` + `Flags: 0`，BL 锁未解时修改 `boot` 后无法通过校验。
- **内核无 BTF / DWARF**：结构体偏移需手动反汇编推导。
- **内核符号表完整**：`kallsyms` 含 105485 个符号，可恢复出带符号的 `vmlinux`。

---

## 使用说明

- **分析**：可用 `binwalk`、`avbtool`、`vmlinux-to-elf` 等工具进一步分析。
- **适配**：可用于 TWRP 适配、内核研究、漏洞分析。
- **打补丁**：如需修改 `boot`，必须同时处理 `vbmeta` 校验，否则设备无法启动。
- **恢复**：可用于救砖，但需配合正确的刷写工具（如 MTKClient）。

> ⚠️ 修改任何分区前，请确保已备份原镜像，并理解设备的安全机制。

---

## 不包含的内容

为保证隐私，本仓库**不包含**以下文件：

- `flash.bin`（完整 119 GB 全盘镜像，含 IMEI、MAC、序列号）
- `hwparam.json`（含 ME_ID、SOC_ID、CID）

---

## 免责声明

- 本仓库所有文件仅用于**研究、恢复、适配**目的。
- 使用者需自行承担刷机、修改分区带来的风险。
- 请遵守当地法律法规，不得用于非法用途。

---

## 致谢

感谢所有为 WIKO 机型玩机做出贡献的人。

---

## 附：如何生成 Image 与 vmlinux_sym.elf

从 `boot.img` 中提取内核并解压：

```bash
dd if=boot.img bs=1 skip=$((0x800)) count=$((0xF3213A)) 2>/dev/null | gzip -dc > Image
