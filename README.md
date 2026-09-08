# RV1126B Linux 学习课程

以 Rockchip RV1126B SDK（正点原子 `atk_dlrv1126b_linux6.1_sdk` / 飞凌 `OK1126B-linux-source`）为案例。

本仓库包含两条线：

1. **BSP 入门**（SDK / 启动 / 编译 / 烧录 / Rootfs / 媒体·NPU 平台栈）→ [`bsp/`](./bsp/)
2. **设备驱动**（Platform / DTS / GPIO / 字符设备 / 中断 / I2C·SPI / MPP）→ 下方第 0–7 课

建议顺序：先完成 [BSP 课](./bsp/README.md)（含第 7 课平台栈），再进入驱动第 0 课。

---

## A. BSP 入门（飞凌 OK1126B 对话课）

详见 **[bsp/README.md](./bsp/README.md)**。

| 课次 | 文件 | 主题 |
|------|------|------|
| 第 0 课 | [bsp/00-BSP与SDK目录地图.md](./bsp/00-BSP与SDK目录地图.md) | BSP、SDK 树、软链接 |
| 第 1 课 | [bsp/01-启动链rkbin与U-Boot.md](./bsp/01-启动链rkbin与U-Boot.md) | Loader / FlashData·FlashBoot |
| 第 2 课 | [bsp/02-板级配置三件套.md](./bsp/02-板级配置三件套.md) | defconfig / parameter / DTS |
| 第 3 课 | [bsp/03-编译与产物.md](./bsp/03-编译与产物.md) | make、firmware、`.config` |
| 第 4 课 | [bsp/04-设备树改GPIO.md](./bsp/04-设备树改GPIO.md) | gpio 三元组、practice 沙盒 |
| 第 5 课 | [bsp/05-烧录与串口日志.md](./bsp/05-烧录与串口日志.md) | rkflash、串口 115200 |
| 第 6 课 | [bsp/06-Rootfs与Buildroot.md](./bsp/06-Rootfs与Buildroot.md) | overlay、加程序 |
| 6.1 | [bsp/06.1-交叉编译与file命令.md](./bsp/06.1-交叉编译与file命令.md) | 交叉编译、`file` |
| 第 7 课 | [bsp/07-平台栈总览.md](./bsp/07-平台栈总览.md) | Rockit / MPP / RGA / RKAIQ / NPU |
| 7.1–7.5 | [bsp/07.1](./bsp/07.1-Rockit-MPI.md) … [07.5](./bsp/07.5-NPU.md) | 分站精读 |
| 答疑 | [bsp/对话答疑与易错点.md](./bsp/对话答疑与易错点.md)、[bsp/07-平台栈答疑](./bsp/07-平台栈答疑与易错点.md) | 课堂踩坑 |
| 速查 | [bsp/嵌入式Linux常用命令速查.md](./bsp/嵌入式Linux常用命令速查.md) | 命令按用途/频率 |

教材路径示例：`.../FL/OK1126B-linux-source`。

---

## B. 设备驱动课程

| 课次 | 文件 | 主题 | 难度 |
|------|------|------|------|
| 第 0 课 | [00-驱动入门与最简形态.md](./00-驱动入门与最简形态.md) | 驱动是什么、用户态/内核态、GPIO LED | ★ |
| 第 1 课 | [01-Platform驱动骨架.md](./01-Platform驱动骨架.md) | of_match / probe / module_platform_driver | ★ |
| 第 2 课 | [02-设备树与资源获取.md](./02-设备树与资源获取.md) | reg / irq / clk / reset / gpio / 自定义属性 | ★★ |
| 第 3 课 | [03-GPIO与LED实战.md](./03-GPIO与LED实战.md) | gpiod API、pinctrl、亮灯调用链 | ★★ |
| 第 4 课 | [04-字符设备.md](./04-字符设备.md) | cdev、file_operations、ioctl、/dev 节点 | ★★ |
| 第 5 课 | [05-中断与等待队列.md](./05-中断与等待队列.md) | request_irq、上下半部、wait_event | ★★★ |
| 第 6 课 | [06-I2C与SPI客户端驱动.md](./06-I2C与SPI客户端驱动.md) | 总线客户端、传感器常见套路 | ★★★ |
| 第 7 课 | [07-MPP驱动案例精读.md](./07-MPP驱动案例精读.md) | mpp_service、子驱动、session、用户态接口 | ★★★★ |

## SDK 路径对照

| 内容 | 正点原子 | 飞凌 OK1126B |
|------|----------|--------------|
| 内核 | `.../atk_dlrv1126b_linux6.1_sdk/kernel-6.1` | `.../FL/OK1126B-linux-source/kernel-6.1` |
| SoC DT | `arch/arm64/boot/dts/rockchip/rv1126b.dtsi` | 同左 |
| 板级 DT 示例 | `rv1126b-alientek.dtsi` | `FET1126B-S.dtsi` / `OK1126B-practice-linux.dts` |
| GPIO LED 驱动 | `drivers/leds/leds-gpio.c` | 同左 |
| MPP | `drivers/video/rockchip/mpp/` | 同左 |
| 厂商文档 | `docs/cn/Common/`（GPIO、I2C、MPP…） | SDK 自带 docs |

两套 SDK 内核均为 Linux 6.1 + Rockchip 补丁，驱动路径与 API 通用。

## 学习建议

1. 先 BSP、后驱动；每课末尾有检查题与练习。
2. 能上板时优先做烧录 / 串口 / sysfs / `/dev` 观察。
3. 读驱动源码时先找「五件套」：匹配表 → `platform_driver` → `probe` → `remove` → `module_platform_driver`。
4. MPP 精读（驱动第 7 课）前建议至少完成驱动第 0–4 课与 BSP 第 0–4 课。

## 许可证说明

本课程文档为学习笔记，引用的内核代码遵循原仓库 GPL/MIT 等许可证；请勿将未授权的 SDK 二进制随仓库分发。
