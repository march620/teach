# RV1126B Linux 设备驱动学习课程

以 Rockchip RV1126B SDK（正点原子 `atk_dlrv1126b_linux6.1_sdk` / 飞凌 `OK1126B-linux-source`）为案例的设备驱动教程。

## 课程目录

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

两套 SDK 内核均为 Linux 6.1 + Rockchip 补丁，本课程中的驱动路径与 API 通用。

## 学习建议

1. 按课次顺序阅读，每课末尾有检查题与练习。
2. 能上板时优先做 sysfs / `/dev` 观察实验。
3. 读源码时先找「五件套」：匹配表 → `platform_driver` → `probe` → `remove` → `module_platform_driver`。
4. 第 7 课前建议至少完成第 0–4 课。

## 许可证说明

本课程文档为学习笔记，引用的内核代码遵循原仓库 GPL/MIT 等许可证；请勿将未授权的 SDK 二进制随仓库分发。
