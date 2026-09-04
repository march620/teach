# 第 1 课：启动链 rkbin 与 U-Boot

目标：弄清上电后谁先跑，以及 `FlashData` / `FlashBoot` 真正指什么。

---

## 1. 启动顺序

```text
MaskROM → Loader(DDR + SPL) → U-Boot → Kernel(FIT in boot 分区) → Rootfs
```

---

## 2. Loader 配方：`rkbin`

本芯片常用 ini：`rkbin/RKBOOT/RV1126BMINIALL.ini`

```ini
FlashData=bin/rv11/rv1126b_ddr_1332MHz_v1.06_115200.bin
FlashBoot=bin/rv11/rv1126b_spl_v1.05.bin
PATH=rv1126b_spl_loader_v1.06.105.bin
```

| 名字 | 实际含义 | 不是什么 |
|------|----------|----------|
| **FlashData** | **初始化 DDR** 的二进制 | 不是 Flash「数据分区」 |
| **FlashBoot** | **SPL**（二级加载，接着加载 U-Boot） | 不是 Flash「启动分区」 |
| **PATH** | 合成后的 `*_spl_loader_*.bin` | 烧录常用的 Loader 镜像 |

二进制在 `rkbin/bin/rv11/`。

名字带 `Flash`，是因为给 Flash 烧录工具用的 Loader 组件，**不是** `parameter.txt` 里的分区名。

---

## 3. U-Boot 配置如何对上

```text
device/.../OK1126B_S_buildroot_defconfig
    RK_UBOOT_CFG="OK1126B-S"
              ↓
u-boot/configs/OK1126B-S_defconfig
```

产物一般是 `uboot.img`，写入 `parameter.txt` 中的 **uboot** 分区。

配置里可见：`CONFIG_ROCKCHIP_RV1126B=y`、`CONFIG_TARGET_OK1126B_S=y`、`CONFIG_DEFAULT_DEVICE_TREE=...` 等。

---

## 4. 与分区表对应

| 阶段 | 主要镜像 | 分区 / 说明 |
|------|----------|-------------|
| Loader | `*_spl_loader_*.bin` | 由 MaskROM / 烧录工具写入 |
| U-Boot | `uboot.img` | `uboot` |
| 环境变量 | env | `env` |
| 内核+DTB | FIT `boot.img` | **`boot`**（不是叫 kernel 的分区） |
| 根文件系统 | `rootfs.img` | `rootfs` |

---

## 课后练习

```bash
cd /path/to/OK1126B-linux-source
ls u-boot/configs/OK1126B-S_defconfig
grep -E 'FlashData|FlashBoot|PATH=' rkbin/RKBOOT/RV1126BMINIALL.ini
```

用自己的话回答：

1. `FlashData` 和 `FlashBoot` 各干什么？
2. `OK1126B-S` 从 defconfig 到 `u-boot/configs/` 如何对应？

## 下一课

[第 2 课：板级配置三件套](./02-板级配置三件套.md)
