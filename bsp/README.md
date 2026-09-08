# RV1126B BSP 入门课程（飞凌 OK1126B）

以飞凌 SDK `OK1126B-linux-source` 为教材，从「目录地图 → 启动链 → 板级配置 → 编译 → 设备树 → 烧录 → Rootfs → 媒体/NPU 平台栈」顺序学习 BSP。

**建议 SDK 路径：**

```text
/home/m3588/rv1126b/FL/OK1126B-linux-source
```

本目录对应完整教学对话整理稿；练习板配置名：`OK1126B_PRACTICE` / `OK1126B-practice-linux`。

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
| 6.1 | [06.1-交叉编译与file命令.md](./06.1-交叉编译与file命令.md) | 交叉 gcc、`file`、装进 overlay |
| 第 7 课 | [07-平台栈总览.md](./07-平台栈总览.md) | external 媒体/NPU 地图 |
| 7.1 | [07.1-Rockit-MPI.md](./07.1-Rockit-MPI.md) | Rockit MPI、Bind、VI |
| 7.2 | [07.2-MPP编解码.md](./07.2-MPP编解码.md) | MPP Frame/Packet |
| 7.3 | [07.3-RGA.md](./07.3-RGA.md) | im2d 缩放/转格式 |
| 7.4 | [07.4-RKAIQ.md](./07.4-RKAIQ.md) | ISP/3A、IQ |
| 7.5 | [07.5-NPU.md](./07.5-NPU.md) | Toolkit2 → RKNPU2 |

另见：

- [对话答疑与易错点.md](./对话答疑与易错点.md)（BSP 0–6 踩坑）
- [07-平台栈答疑与易错点.md](./07-平台栈答疑与易错点.md)（第 7 课踩坑）
- [嵌入式Linux常用命令速查.md](./嵌入式Linux常用命令速查.md)（按用途与频率）

学完本 BSP 课，建议接主目录的 [驱动课程第 0 课](../00-驱动入门与最简形态.md)。
