# 嵌入式 Linux 常用命令速查

面向 RV1126B / 飞凌 OK1126B SDK 学习场景。按**用途分组**，组内大致按**日常使用频率**从高到低排列。

路径约定：SDK 根目录多为 `OK1126B-linux-source`。

---

## 一、目录与文件查看（最高频）

| 频率 | 命令 | 用途 | 示例 |
|------|------|------|------|
| ★★★ | `cd` | 切换目录 | `cd ~/rv1126b/FL/OK1126B-linux-source` |
| ★★★ | `ls` / `ls -l` | 列目录；`-l` 看权限与软链 `->` | `ls -l Makefile build.sh` |
| ★★★ | `pwd` | 当前绝对路径 | `pwd` |
| ★★★ | `cat` | 打印小文件全文 | `cat parameter.txt` |
| ★★★ | `less` / `more` | 分页看大文件 | `less output/.config` |
| ★★☆ | `head` / `tail` | 看头/尾几行；日志常用 `tail -f` | `tail -f /var/log/messages` |
| ★★☆ | `file` | 看文件类型/CPU 架构（交叉编译必用） | `file practice_hello` → 应含 `ARM aarch64` |
| ★★☆ | `readlink` / `readlink -f` | 解析软链接真实路径 | `readlink -f Makefile` |
| ★★☆ | `find` | 按名搜索（尽量在 SDK 内，别从 `/` 盲搜） | `find . -name '*defconfig'` |
| ★☆☆ | `tree` | 树形看目录（若已安装） | `tree -L 2 device/rockchip` |
| ★☆☆ | `stat` | 看时间戳/大小细节 | `stat output/firmware/boot.img` |
| ★☆☆ | `du` / `df` | 目录占用 / 磁盘剩余 | `df -h`；`du -sh output` |

---

## 二、文本搜索与编辑

| 频率 | 命令 | 用途 | 示例 |
|------|------|------|------|
| ★★★ | `grep` / `rg` | 内容搜索（`rg` 更快，若已装） | `grep RK_KERNEL_DTS_NAME output/.config` |
| ★★☆ | `grep -n` / `-R` | 带行号 / 递归 | `grep -n encode_put_frame mpi_enc_test.c` |
| ★★☆ | `grep -E` | 扩展正则 | `grep -E 'FlashData\|FlashBoot' rkbin/RKBOOT/*.ini` |
| ★★☆ | `vi` / `vim` / `nano` | 编辑源码、配置 | `vi OK1126B-practice-linux.dts` |
| ★☆☆ | `sed` / `awk` | 批量改文本、切列（脚本里常用） | `sed -n '1,20p' file` |
| ★☆☆ | `diff` / `diff -u` | 比两个文件差异 | `diff -u a.dts b.dts` |

---

## 三、SDK 编译与配置（BSP 核心）

| 频率 | 命令 | 用途 | 示例 |
|------|------|------|------|
| ★★★ | `make <目标>` | 编内核/uboot/rootfs/打包 | `make kernel`；`make rootfs`；`make updateimg` |
| ★★★ | `make *_defconfig` | 选中板级配置（写入 `output/.config`） | `make OK1126B_PRACTICE_buildroot_defconfig` |
| ★★☆ | `make help` | 列出 SDK 可用目标 | `make help \| head -40` |
| ★★☆ | `./build.sh <子命令>` | 与 make 同源脚本；可更新 `output/defconfig` 软链 | `./build.sh kernel` |
| ★☆☆ | `make clean` / `clean-kernel` 等 | 清理（慎用 `cleanall`） | `make clean-kernel` |
| ★☆☆ | `make kconfig` / `buildroot-config` | 进内核/Buildroot 菜单配置 | `make kconfig` |

**改什么编什么（记口诀）：**

```text
只改 DTS/驱动 → make kernel     → 刷 boot
只改用户程序  → make rootfs     → 刷 rootfs
大改/出厂包   → make updateimg  → 刷 update.img
```

**确认当前配置以谁为准：**

```bash
grep RK_KERNEL_DTS_NAME output/.config
# 不要只看 output/defconfig 软链（可能滞后）
```

---

## 四、交叉编译与二进制检查

| 频率 | 命令 | 用途 | 示例 |
|------|------|------|------|
| ★★★ | `aarch64-none-linux-gnu-gcc` | 编 Linux 用户态 ARM64 程序 | 见 `app/practice_hello/Makefile` |
| ★★★ | `file` | 确认是否 `ARM aarch64`（非 x86-64） | `file usr/bin/practice_hello` |
| ★★☆ | `make -C <dir> install` | 在工程目录编译并装进 overlay | `make -C app/practice_hello install` |
| ★☆☆ | `aarch64-none-linux-gnu-readelf` / `objdump` | 看依赖、符号 | `readelf -d practice_hello \| grep NEEDED` |
| ★☆☆ | `strip`（交叉版） | 去掉符号减小体积 | 量产可选 |

详见：[06.1 交叉编译与 file 命令](./06.1-交叉编译与file命令.md)

---

## 五、烧录与固件

| 频率 | 命令 | 用途 | 示例 |
|------|------|------|------|
| ★★★ | `./rkflash.sh boot` | 只刷 boot（改 DTS/内核后） | 板子进下载模式后执行 |
| ★★☆ | `./rkflash.sh rootfs` | 只刷根文件系统 | 改 overlay/程序后 |
| ★★☆ | `./rkflash.sh update` | 刷整包 `update.img` | 首次或大改 |
| ★☆☆ | `./rkflash.sh uboot` / `all` 等 | 刷 U-Boot / 分区逐刷 | 见脚本内分支 |
| ★☆☆ | `upgrade_tool` | Rockchip 底层烧录工具（rkflash 会调） | 路径在 `tools/linux/...` |

WSL2 注意：USB 设备常需 Windows 侧工具或 `usbipd` 转发。

---

## 六、串口与板上调试

| 频率 | 命令 | 用途 | 示例 |
|------|------|------|------|
| ★★★ | `minicom` / `picocom` / `screen` | 串口终端（本板常 **115200**） | `sudo minicom -D /dev/ttyUSB0 -b 115200` |
| ★★★ | `dmesg` / `dmesg \| tail` | 内核日志 | `dmesg \| grep -i mpp` |
| ★★☆ | `ls /dev` | 看设备节点 | `ls /dev/video*`；`ls /dev/mpp*` |
| ★★☆ | `cat /proc/device-tree/model` | 确认当前 DTB 板型名 | 期望 Practice / 正式板 model |
| ★★☆ | `ls /sys/class/leds` | LED 类设备（gpio-leds） | `echo 1 > .../brightness` |
| ★★☆ | `lsmod` / `insmod` / `rmmod` / `modprobe` | 内核模块 | 驱动课常用 |
| ★☆☆ | `top` / `htop` / `ps` | 进程与 CPU | `ps \| grep rkipc` |
| ★☆☆ | `free` / `cat /proc/meminfo` | 内存 | `free -h` |
| ★☆☆ | `ifconfig` / `ip` / `ping` | 网络 | `ip addr`；`ping 8.8.8.8` |
| ★☆☆ | `mount` / `umount` | 挂载分区/镜像（分析 rootfs 时） | `mount -o loop rootfs.ext4 /mnt` |

---

## 七、权限与进程

| 频率 | 命令 | 用途 | 示例 |
|------|------|------|------|
| ★★★ | `sudo` | 提权（串口、挂载、部分烧录） | `sudo minicom ...` |
| ★★☆ | `chmod` | 改可执行权限 | `chmod +x practice_hello.sh` |
| ★★☆ | `chown` | 改所有者（部署脚本时） | `chown root:root app` |
| ★☆☆ | `kill` / `killall` | 结束进程 | `killall rkipc` |
| ★☆☆ | `export` | 设环境变量（交叉工具链 PATH） | `export PATH=...:$PATH` |

---

## 八、Git（课程仓库 / 自己的笔记）

| 频率 | 命令 | 用途 | 示例 |
|------|------|------|------|
| ★★☆ | `git status` | 看改动 | — |
| ★★☆ | `git add` / `git commit` | 暂存、提交 | — |
| ★★☆ | `git push` / `git pull` | 推送、拉取 | `git push -u origin main` |
| ★☆☆ | `git log` / `git diff` | 历史与差异 | `git log -3 --oneline` |
| ★☆☆ | `git clone` | 克隆仓库 | `git clone git@github.com:march620/teach.git` |

本课程仓库：https://github.com/march620/teach

---

## 九、媒体 / NPU 联调时常用（板上）

| 频率 | 命令 | 用途 |
|------|------|------|
| ★★☆ | `ls /dev/video*` | 相机/ISP 节点 |
| ★★☆ | `media-ctl -p`（若有） | 看 media 拓扑、实体名（给 RKAIQ） |
| ★☆☆ | 跑官方 demo | `rkisp_demo`、`rknn_*_demo`、`mpi_enc_test` 等（以 rootfs 是否集成为准） |
| ★☆☆ | `cat /sys/kernel/debug/...` | 部分驱动 debugfs（需内核开启） |

---

## 十、一张「按场景」速查

| 你想做的事 | 优先命令 |
|------------|----------|
| 看 SDK 软链指哪 | `ls -l`；`readlink -f` |
| 确认编的是哪块板 | `grep RK_KERNEL_DTS_NAME output/.config` |
| 改了 DTS | `make kernel` → `./rkflash.sh boot` |
| 加了 App/脚本 | 交叉编译 + `file` → overlay → `make rootfs` → 刷 rootfs |
| 串口看启动 | `minicom -b 115200` |
| 确认 practice DTB | `cat /proc/device-tree/model` |
| 二进制能否上板 | `file xxx` 必须是 aarch64 |
| 搜 API 在哪 | `grep -n` / `rg` 在 `external/` 或示例里 |

---

## 相关课程

- [BSP 目录](./README.md)
- [06 Rootfs](./06-Rootfs与Buildroot.md) / [06.1 交叉编译](./06.1-交叉编译与file命令.md)
- [05 烧录与串口](./05-烧录与串口日志.md)
- [07 平台栈](./07-平台栈总览.md)
