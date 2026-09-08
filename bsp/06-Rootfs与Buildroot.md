# 第 6 课：Rootfs 与 Buildroot

目标：理解根文件系统从哪来，会用 overlay 往系统里加文件/脚本。

---

## 1. Rootfs 是什么

**Rootfs = 内核挂上的整棵 `/`**：程序、库、`/etc`、启动脚本都在这里。

本练习板走 Buildroot：

```text
OK1126B_*_buildroot_defconfig
  RK_BUILDROOT_BASE_CFG="ok1126b-s"
        ↓
buildroot/configs/rockchip_ok1126b-s_defconfig
        ↓
make rootfs / make buildroot
        ↓
output/buildroot/images/rootfs.ext4 → rootfs 分区
```

---

## 2. 内容从哪来（三层）

```text
① Buildroot 编出的基础系统（busybox、库、已选包）
② 板级 / SDK overlay（打包时拷进根文件系统）
③ 你的自定义文件或自研程序
```

板级 Buildroot 配置中常见：

```text
BR2_ROOTFS_OVERLAY="board/rockchip/common/base
                    board/rockchip/rv1126b/fs-overlay/
                    ..."
```

SDK 文档还推荐自定义文件放：

```text
device/rockchip/common/overlays/rootfs/default/
```

主机路径按**板上绝对路径**摆放，例如：

```text
.../overlays/rootfs/default/usr/bin/practice_hello.sh
→ 板上 /usr/bin/practice_hello.sh
```

---

## 3. 三种加程序方式

| 办法 | 适合 | 做法 |
|------|------|------|
| **A. Overlay** | 脚本、配置、现成二进制 | 放进 overlay，再 `make rootfs` |
| **B. 打开 BR2 包** | SDK/上游已有软件 | `make buildroot-config` 或改 defconfig |
| **C. 做成 package** | 长期维护的工程 | `buildroot/package/` 或 `app/` + `BR2_PACKAGE_xxx` |

入门优先 A；SDK 已有 `app/forlinx`、`BR2_PACKAGE_FORLINX` 等可作 C 的参考。

---

## 4. 与前面几课的对应

| 改动 | 编译 | 烧录 |
|------|------|------|
| DTS / 驱动 | `make kernel` | **boot** |
| 用户程序 / overlay | `make rootfs` | **rootfs** |
| 两者都改 | 两个都 make | boot + rootfs 或整包 |

`make kernel` **不会**把 overlay 里的脚本打进系统。

---

## 5. 无板练习：加一个 hello 脚本

```bash
cd /path/to/OK1126B-linux-source
mkdir -p device/rockchip/common/overlays/rootfs/default/usr/bin

cat > device/rockchip/common/overlays/rootfs/default/usr/bin/practice_hello.sh <<'EOF'
#!/bin/sh
echo "Hello from OK1126B practice rootfs overlay"
EOF
chmod +x device/rockchip/common/overlays/rootfs/default/usr/bin/practice_hello.sh

# 耗时可能较长
make rootfs
ls -l output/buildroot/images/rootfs.ext4
```

有板：

```bash
./rkflash.sh rootfs
# 板上执行
practice_hello.sh
```

---

## 课后练习

1. 只加 shell 脚本：应 `make` 什么、刷哪个分区？
2. Overlay `.../default/usr/bin/xxx` 对应板上哪条路径？
3. `make kernel` 会不会带上 `practice_hello.sh`？为什么？

## 加餐

交叉编译 C 程序再放进 overlay：[06.1 交叉编译与 file 命令](./06.1-交叉编译与file命令.md)

## 学完之后

- 继续本仓库驱动课：[第 0 课 驱动入门](../00-驱动入门与最简形态.md)
- 或第 7 课平台栈：[07-平台栈总览](./07-平台栈总览.md)
