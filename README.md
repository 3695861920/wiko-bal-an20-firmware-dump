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
