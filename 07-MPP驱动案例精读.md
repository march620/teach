# 第 7 课：MPP 驱动案例精读

目标：用前六课的概念，读懂 Rockchip **MPP（Media Process Platform）** 内核驱动的分层与数据路径。

主路径：`kernel-6.1/drivers/video/rockchip/mpp/`  
用户态库：`external/mpp`（若 SDK 中存在）

---

## 1. MPP 在系统中的位置

```
ffmpeg / 相机管道 / 测试程序
        │
        ▼
  libmpp（用户态）
        │  open / ioctl / dma-buf
        ▼
  /dev/mpp_service          ← mpp_service.c 注册的字符设备
        │
        ├─ 任务队列 / session
        │
        ▼
  子设备驱动：rkvdec / rkvenc / jpgdec …
        │
        ▼
  硬件 VPU + IOMMU + 时钟复位电源域
```

设备树（SoC）：

```dts
mpp_srv: mpp-srv {
	compatible = "rockchip,mpp-service";
	rockchip,taskqueue-count = <3>;
	rockchip,resetgroup-count = <3>;
	status = "disabled";  /* 板级改为 okay */
};

rkvdec: rkvdec@22140100 {
	compatible = "rockchip,rkv-decoder-rv1126b", ...;
	rockchip,srv = <&mpp_srv>;
	/* reg, clocks, resets, iommus, power-domains ... */
};
```

---

## 2. 用五件套看 mpp_service.c

| 件套 | 位置（概念） |
|---|---|
| `of_device_id` | `compatible = "rockchip,mpp-service"` |
| `platform_driver` | `mpp_service_driver` |
| `probe` | `mpp_service_probe` |
| `remove` | `mpp_service_remove` |
| 注册 | `module_platform_driver` |

`probe` 大意：

1. `devm_kzalloc` 分配 `mpp_service`  
2. `class_create("mpp_class")`  
3. 读 `rockchip,taskqueue-count` 等，创建 `kthread_worker`  
4. `mpp_register_service` → `/dev/mpp_service`  
5. 按 Kconfig 宏注册子驱动：`MPP_REGISTER_DRIVER(... rkvdec ...)`

子驱动注册本质是：`platform_driver_register(&rockchip_rkvdec_driver)`。

---

## 3. 服务层 vs 子设备层

| 层 | 文件 | 职责 |
|---|---|---|
| 服务 | `mpp_service.c` | 字符设备、session 列表、procfs、挂子驱动 |
| 公共 | `mpp_common.c/h` | 打开/ioctl 分发、任务、时钟、iommu 辅助 |
| IOMMU | `mpp_iommu.c` | DMA 映射、iova |
| 子设备 | `mpp_rkvdec.c`、`mpp_rkvenc.c`… | 具体寄存器与硬件能力 |

用户只打开 **一个** `/dev/mpp_service`；通过 ioctl 声明 client 类型，再绑定到某个 `mpp_dev`。

---

## 4. 字符设备与 ioctl（第 4 课落地）

```c
alloc_chrdev_region → cdev_init(&rockchip_mpp_fops) → cdev_add → device_create
```

常见命令族（见 proc 或头文件）：

- `MPP_CMD_QUERY_*`：查询硬件能力  
- `MPP_CMD_INIT_CLIENT_TYPE`：指定解码/编码类型  
- 后续：送任务、等完成、管理 buffer  

`copy_from_user` / dma-buf fd 传递帧缓冲，避免巨量拷贝。

---

## 5. 资源获取（第 2 课落地）

子设备 probe（在 `mpp_common` 辅助下）典型会：

- `platform_get_irq` + `request_irq`（第 5 课）  
- `devm_ioremap` 映射 `reg`  
- `devm_clk_get` + enable（按 `clock-names`）  
- `devm_reset_control_get`  
- 绑定 IOMMU、`power-domains`  
- 读 `rockchip,taskqueue-node` 把自己挂到服务的某个队列  

多段寄存器用 `reg-names`（如 `"regs"`, `"link"`）。

---

## 6. 任务与并发直觉

```
多个进程 open /dev/mpp_service
    → 各有 session
    → 任务进入 mpp_taskqueue
    → kthread_worker 串行/分组喂给硬件
    → 中断完成 → 唤醒等待的 ioctl/poll
```

`taskqueue-count` / `resetgroup-count` 用于多硬件实例时的调度与复位隔离。

调试：

```bash
ls -l /dev/mpp_service
ls /proc/mpp_service/          # 若启用 CONFIG_ROCKCHIP_MPP_PROC_FS
# 版本、session、支持的 cmd 等
```

模块参数：`mpp_dev_debug`（`mpp_service.c` 顶部 `module_param`）可开调试打印。

---

## 7. 建议阅读顺序（源码）

1. `mpp_service.c`：全文（约 550 行，先读完）  
2. `mpp_common.h`：关键结构体 `mpp_service` / `mpp_dev` / `mpp_session`  
3. `mpp_common.c`：`open` / `ioctl` 入口、任务提交  
4. 选一个子驱动（如 `mpp_rkvdec2.c`）看 `probe` 与中断  
5. 对照 `rv1126b.dtsi` 里 `mpp_srv` + `rkvdec` 节点  
6. （可选）用户态 `external/mpp` 里搜索 `mpp_service` 或 ioctl 封装  

---

## 8. 与前六课对照表

| 课 | 在 MPP 中的体现 |
|---|---|
| 0 用户/内核 | libmpp ↔ `/dev/mpp_service` |
| 1 platform 骨架 | `mpp_service_driver` + 子 `platform_driver` |
| 2 设备树资源 | reg/clk/reset/irq/iommu/自定义 rockchip,* |
| 3 GPIO | MPP 本身少用 GPIO；同属 platform 资源思维 |
| 4 字符设备 | cdev + fops + ioctl |
| 5 中断与等待 | 解码完成 IRQ + 队列/等待 |
| 6 I2C | 不同总线；对比“服务+子设备”分层思想 |

---

## 9. 检查题

1. 为什么应用不直接 open `rkvdec` 字符节点？→ 统一由 service 做会话与调度。  
2. `MPP_REGISTER_DRIVER` 宏在做什么？→ 条件编译下 `platform_driver_register` 子驱动。  
3. 帧数据为何常用 dma-buf？→ 零拷贝、多设备共享缓冲。

---

## 10. 练习

1. 在板级确认 `mpp_srv`、`rkvdec` 的 `status`。  
2. 画出：open → INIT_CLIENT → 送任务 → IRQ → 返回，的序列图。  
3. 读 `mpp_service_probe`，用注释标出：分配、class、队列、cdev、子驱动注册。  
4. （进阶）跟一次简单解码 demo，用 `mpp_dev_debug` 或 ftrace 看 ioctl 路径。

---

## 课程结束之后可以做什么

- 写一个最小 **platform + misc** 驱动，挂到 practice dts，完成 ioctl 点灯。  
- 给真实 I2C 传感器写客户端驱动或完善板级 DT。  
- 深入 Camera/ISP（V4L2），文档见 SDK `docs/cn/` 下 Camera / VI 指南。  
- 阅读 `Documentation/driver-api/` 与 Rockchip `docs/cn/Common/MPP`。

返回目录：[README.md](./README.md)
