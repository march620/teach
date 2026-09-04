# RV1126B BSP 入门课程（飞凌 OK1126B）

以飞凌 SDK `OK1126B-linux-source` 为教材，从「目录地图 → 启动链 → 板级配置 → 编译 → 设备树 → 烧录 → Rootfs」顺序学习 BSP。

**建议 SDK 路径：**

```text
/home/m3588/rv1126b/FL/OK1126B-linux-source
```

本目录对应一次完整教学对话的整理稿；练习板配置名：`OK1126B_PRACTICE` / `OK1126B-practice-linux`。

## 课程目录

| 课次 | 文件 | 主题 |
|------|------|------|
| 第 0 课 | [00-BSP与SDK目录地图.md](./00-BSP与SDK目录地图.md) | BSP 是什么、树怎么读 |
| 第 1 课 | [01-启动链rkbin与U-Boot.md](./01-启动链rkbin与U-Boot.md) | MaskROM → Loader → U-Boot |
| 第 2 课 | [02-板级配置三件套.md](./02-板级配置三件套.md) | defconfig / parameter / DTS |
| 第 3 课 | [03-编译与产物.md](./03-编译与产物.md) | make / build.sh / firmware |
| 第 4 课 | [04-设备树改GPIO.md](./04-设备树改GPIO.md) | gpio 写法、practice 沙盒 |
| 第 5 课 | [05-烧录与串口日志.md](./05-烧录与串口日志.md) | rkflash、115200、验证 model |
| 第 6 课 | [06-Rootfs与Buildroot.md](./06-Rootfs与Buildroot.md) | overlay、加程序、刷 rootfs |

另见：[对话答疑与易错点.md](./对话答疑与易错点.md)（课堂里踩过的坑集中整理）。

学完本 BSP 课，建议接主目录的 [驱动课程第 0 课](../00-驱动入门与最简形态.md)。
