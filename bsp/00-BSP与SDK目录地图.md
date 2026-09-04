# 第 0 课：BSP 与 SDK 目录地图

目标：建立「BSP 管什么」的整体图，能在飞凌 OK1126B SDK 里找到入口文件。

---

## 1. BSP 是什么

**BSP（Board Support Package）** = 让「这颗芯片 + 这块板」跑起来的软件粘合层。在 Rockchip Linux SDK 里大致包括：

1. **启动固件**（DDR、SPL、Trust、U-Boot）
2. **内核 + 设备树**（外设怎么接）
3. **根文件系统**（用户空间）
4. **打包 / 分区 / 烧录**（变成可刷的镜像）

`external/`（MPP、RKAIQ、NPU…）属于平台能力，入门阶段先不深挖。

---

## 2. SDK 树怎么读

```text
OK1126B-linux-source/
├── device/rockchip/     ← 板级「总控」
│   ├── common/          ← build.sh、rkflash.sh、通用脚本
│   └── rv1126b/         ← 本芯片板级 defconfig、parameter.txt
├── rkbin/               ← 闭源：DDR/BL31/BL32、RKBOOT ini
├── u-boot/              ← U-Boot
├── kernel-6.1/          ← Linux 6.1（根目录 kernel → 这里）
├── buildroot/ / debian/ ← 根文件系统两种路线
├── external/            ← MPP、RKAIQ、RGA、NPU…
├── docs/rv1126b/        ← 官方文档
├── tools/               ← 主机侧工具
└── Makefile / build.sh  ← 软链到 common
```

本板常用配置名：**OK1126B-S**（另有练习配置 **OK1126B_PRACTICE**）。

---

## 3. 板级入口：defconfig

文件：`device/rockchip/rv1126b/OK1126B_S_buildroot_defconfig`

关键字段：

| 变量 | 示例值 | 含义 |
|------|--------|------|
| `RK_UBOOT_CFG` | `OK1126B-S` | U-Boot 配置名 |
| `RK_KERNEL_CFG` | `OK1126B-S-linux_defconfig` | 内核 defconfig |
| `RK_KERNEL_DTS_NAME` | `OK1126B-S-linux` | 设备树名 |
| `RK_KERNEL_PREFERRED` | `6.1` | 内核版本 |
| `RK_WIFIBT_CHIP` | `RTL8821CS` | WiFi/BT 芯片 |
| `RK_USE_FIT_IMG` | `y` | boot 使用 FIT |

同目录还有：

- `OK1126B_S_debian_defconfig` → Debian 根文件系统
- `OK1126B_PRACTICE_buildroot_defconfig` → 练习沙盒（DTS 指向 practice）

---

## 4. 软链接 vs 依赖（易混）

根目录这些是**软链接**（`ls -l` 能看到 `->`）：

| 名字 | 指向 |
|------|------|
| `Makefile` | `device/rockchip/common/Makefile` |
| `build.sh` | `device/rockchip/common/scripts/build.sh` |
| `rkflash.sh` | `device/rockchip/common/scripts/rkflash.sh` |
| `kernel` | `kernel-6.1` |

`configs`、`.chip`、`output`、`parameter.txt` 是运行时**读写/依赖**，不是软链接。

查看真实路径：

```bash
cd /path/to/OK1126B-linux-source
ls -l Makefile build.sh rkflash.sh kernel
readlink -f Makefile build.sh
```

---

## 5. 上电直觉

```text
MaskROM → Loader(DDR+SPL) → U-Boot → Kernel(+DTB in boot) → Rootfs
```

---

## 课后练习

1. 用 `ls -l` / `readlink -f` 确认 `Makefile`、`build.sh`、`rkflash.sh` 指向哪里。
2. 列出 `device/rockchip/rv1126b/` 下两个正式 `*_defconfig` 的区别。
3. 打开 `parameter.txt`，说出 `boot` 与 `rootfs` 分区各自放什么（不是“启动地址”）。
4. 从 buildroot defconfig 写出：U-Boot 配置名、DTS 名、WiFi 芯片。

## 下一课

[第 1 课：启动链 rkbin 与 U-Boot](./01-启动链rkbin与U-Boot.md)
