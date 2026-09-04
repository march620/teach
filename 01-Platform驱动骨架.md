# 第 1 课：Platform 驱动骨架

目标：看懂几乎所有片上外设驱动的「标准五件套」，并能在 `leds-gpio` 和 `mpp_service` 里对上号。

---

## 1. 为什么叫 Platform 驱动？

SoC 里的外设（GPIO、UART、VPU…）一般**挂在总线上、地址写死在芯片手册里**，不像 USB 那样热插拔枚举。

Linux 用 **platform 总线** 管这类设备：

| 角色 | 是什么 | 谁创建 |
|---|---|---|
| `platform_device` | “板上有这个硬件” | 设备树解析出来 |
| `platform_driver` | “我会驱动这类硬件” | 你写的 `.c` |
| 匹配成功 | 调用 `probe()` | 内核自动做 |

设备树里：

```dts
leds {
	compatible = "gpio-leds";   /* ← 匹配钥匙 */
	work {
		gpios = <&gpio0 ...>;
	};
};
```

内核读到这段 → 造出 `platform_device` → 找 `compatible = "gpio-leds"` 的驱动。

---

## 2. 标准五件套

```
① of_device_id 匹配表      ← 声明“我能驱动谁”
② platform_driver 结构体   ← 挂上 probe / remove
③ probe()                  ← 匹配成功：申请资源、初始化
④ remove() / shutdown()    ← 卸载或关机清理
⑤ module_platform_driver() ← 模块加载时注册驱动
```

---

## 3. ① 匹配表

`drivers/leds/leds-gpio.c`：

```c
static const struct of_device_id of_gpio_leds_match[] = {
	{ .compatible = "gpio-leds", },
	{},
};
MODULE_DEVICE_TABLE(of, of_gpio_leds_match);
```

要点：

- `compatible` 必须和 DT **完全一致**。
- 表必须以空项 `{ }` 结尾。
- `MODULE_DEVICE_TABLE` 支持按 DT 自动加载模块。

MPP 同套路：

```c
static const struct of_device_id mpp_dt_ids[] = {
	{ .compatible = "rockchip,mpp-service", },
	{ },
};
```

路径：`drivers/video/rockchip/mpp/mpp_service.c`。

---

## 4. ②③⑤ platform_driver + 注册

LED：

```c
static struct platform_driver gpio_led_driver = {
	.probe		= gpio_led_probe,
	.shutdown	= gpio_led_shutdown,
	.driver		= {
		.name	= "leds-gpio",
		.of_match_table = of_gpio_leds_match,
	},
};
module_platform_driver(gpio_led_driver);
```

MPP：

```c
static struct platform_driver mpp_service_driver = {
	.probe = mpp_service_probe,
	.remove = mpp_service_remove,
	.driver = {
		.name = "mpp_service",
		.of_match_table = of_match_ptr(mpp_dt_ids),
	},
};
module_platform_driver(mpp_service_driver);
```

| 字段 | 含义 |
|---|---|
| `.probe` | 找到设备后初始化 |
| `.remove` / `.shutdown` | 清理 |
| `.driver.name` | sysfs / 日志名 |
| `.of_match_table` | 和 DT 对暗号 |
| `module_platform_driver` | 代替手写 module_init/exit |

---

## 5. ③ probe 里一般做什么

```
probe(pdev)
  1. 分配私有数据  devm_kzalloc(...)
  2. 从 DT 拿资源  (GPIO、IRQ、时钟、寄存器…)
  3. 初始化硬件 / 注册子系统接口
  4. platform_set_drvdata(pdev, priv)
  5. return 0;  或返回负的 errno
```

LED 精简逻辑：

```c
static int gpio_led_probe(struct platform_device *pdev)
{
	priv = gpio_leds_create(pdev);  /* 读 DT、拿 GPIO、注册 LED class */
	platform_set_drvdata(pdev, priv);
	return 0;
}
```

一对 API：

- `platform_set_drvdata(pdev, priv)` — probe 里存  
- `platform_get_drvdata(pdev)` — remove / 业务里取  

大量使用 **`devm_*`**：设备注销时自动释放资源。

---

## 6. 开机到亮灯时序

```
开机
  → 解析 DT，创建 platform_device(compatible="gpio-leds")
  → 驱动已注册
  → compatible 匹配
  → gpio_led_probe()
  → /sys/class/leds/work/
  → heartbeat 周期性调 gpio_led_set()
  → GPIO 翻转 → 灯闪
```

读任意 platform 驱动的习惯：**先找匹配表和 probe，再往下钻。**

---

## 7. 最小纸上模板

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>

static int my_probe(struct platform_device *pdev)
{
	dev_info(&pdev->dev, "probe ok\n");
	return 0;
}

static int my_remove(struct platform_device *pdev)
{
	dev_info(&pdev->dev, "removed\n");
	return 0;
}

static const struct of_device_id my_of_match[] = {
	{ .compatible = "myvendor,mydev" },
	{ }
};
MODULE_DEVICE_TABLE(of, my_of_match);

static struct platform_driver my_driver = {
	.probe  = my_probe,
	.remove = my_remove,
	.driver = {
		.name = "mydev",
		.of_match_table = my_of_match,
	},
};
module_platform_driver(my_driver);
MODULE_LICENSE("GPL");
```

DT：

```dts
mydev {
	compatible = "myvendor,mydev";
	status = "okay";
};
```

---

## 8. 检查题

1. DT 写 `gpio-led`（少 s），能 probe 吗？→ **不能**  
2. `set_drvdata` / `get_drvdata` 成对使用。  
3. `shutdown` vs `remove`：都是清理，触发路径不同。  
4. 打开 `mpp_service.c` 末尾，标出五件套位置。

---

## 9. 课后练习

```bash
ls /sys/bus/platform/drivers/leds-gpio/
realpath /sys/class/leds/work/device/driver
```

对照：

- `drivers/leds/leds-gpio.c`（末尾 50 行）
- `drivers/video/rockchip/mpp/mpp_service.c`（末尾 60 行）

---

## 下一课

[第 2 课：设备树与资源获取](./02-设备树与资源获取.md)
