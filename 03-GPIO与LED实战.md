# 第 3 课：GPIO 与 LED 实战

目标：弄清 **pinctrl 与 GPIO 的分工**、**gpiod API**，以及从 DT 到亮灯的完整调用链。

---

## 1. 两个子系统，别混

| 子系统 | 管什么 | 典型 DT | 典型 API |
|---|---|---|---|
| **pinctrl** | 管脚复用：这根脚当 GPIO / UART / I2C？上拉？ | `pinctrl-0 = <&work_led_gpio>` | 框架在 probe 前自动 `select` |
| **GPIO** | 当 GPIO 之后：读/写高低电平、中断 | `gpios = <&gpio0 RK_PA3 ...>` | `gpiod_set_value` |

常见流程：

```
pinctrl 把脚配成 GPIO 功能
        ↓
GPIO 子系统按 gpios 属性申请该脚
        ↓
驱动 gpiod_set_value(1/0) 控制灯
```

只配 `gpios`、漏了 pinctrl，有时仍能亮（默认已是 GPIO），有时会和别的功能冲突——板级 dtsi 里常两者都写。

---

## 2. descriptor 风格 GPIO（现代写法）

旧 API：`gpio_request(编号)` —— 依赖全局编号，不推荐新代码。  
新 API：`struct gpio_desc *`（descriptor）。

常用：

```c
/* 设备节点上的 "xxx-gpios" 或 "gpios" */
gpiod = devm_gpiod_get(dev, "enable", GPIOD_OUT_LOW);

/* LED 子节点场景 */
gpiod = devm_fwnode_get_gpiod_from_child(dev, NULL, child,
					 GPIOD_ASIS, NULL);

gpiod_direction_output(gpiod, 0);
gpiod_set_value(gpiod, 1);
gpiod_set_value_cansleep(gpiod, 1);  /* 可能睡眠的 GPIO（如 I2C 扩展） */
val = gpiod_get_value(gpiod);
```

`ACTIVE_LOW`：逻辑“开”时硬件是低电平；`gpiod_set_value(desc, 1)` 仍表示逻辑开，驱动框架会翻转。

---

## 3. leds-gpio 调用链（结合 practice_led）

假设板级：

```dts
&leds {
	practice_led {
		label = "practice_led";
		gpios = <&gpio0 RK_PC0 GPIO_ACTIVE_HIGH>;
		default-state = "off";
	};
};
```

链路：

```
1. DT 解析 → platform_device(compatible=gpio-leds)
2. leds-gpio probe → gpio_leds_create()
3. 遍历子节点 practice_led
4. devm_fwnode_get_gpiod_from_child(...) 得到 gpiod
5. create_gpio_led() → 注册 led_classdev
6. 用户: echo 1 > /sys/class/leds/practice_led/brightness
7. LED core → brightness_set → gpio_led_set()
8. gpiod_set_value(gpiod, 1) → 硬件亮
```

`linux,default-trigger = "heartbeat"` 时，由 trigger 内核线程周期性调 brightness，无需用户 echo。

---

## 4. sysfs 常用操作

```bash
ls /sys/class/leds/

# 关掉自动心跳，改手动
echo none > /sys/class/leds/work/trigger
echo 1 > /sys/class/leds/work/brightness
echo 0 > /sys/class/leds/work/brightness

# 看可用 trigger
cat /sys/class/leds/work/trigger

# 反查设备树
cat /sys/class/leds/work/device/of_node/compatible
ls /sys/class/leds/work/device/of_node/
```

---

## 5. 按键？同一套 GPIO

输入侧常用 `gpio-keys`：

```dts
gpio-keys {
	compatible = "gpio-keys";
	key-user {
		gpios = <&gpio0 RK_PAx GPIO_ACTIVE_LOW>;
		linux,code = <KEY_POWER>;
		label = "user";
	};
};
```

驱动：`drivers/input/keyboard/gpio_keys.c`。  
套路仍是：compatible 匹配 → probe 拿 gpiod → 注册 input 子系统 → 可产生按键事件。

---

## 6. 和字符设备的边界

| 需求 | 推荐 |
|---|---|
| 标准灯/按键 | `gpio-leds` / `gpio-keys`（已有驱动） |
| 业务自定义（多路 IO 组合协议） | 自己写 platform + 字符设备（第 4 课） |
| 仅调试翻转脚 | `libgpiod` 用户态（`gpioset`）也可，不经你的驱动 |

能复用内核标准驱动就不要重复造轮子。

---

## 7. 检查题

1. pinctrl 和 GPIO 谁负责“这脚是不是 UART”？→ **pinctrl**  
2. DT 写了 `GPIO_ACTIVE_LOW`，`gpiod_set_value(d, 1)` 表示逻辑开还是硬件高？→ **逻辑开**（硬件可能为低）  
3. `cansleep` 版本何时用？→ GPIO 获取可能睡眠时（总线扩展芯片等）

---

## 8. 练习

1. 在 practice dts 增加第二颗 LED 子节点，重编 dtb，确认 `/sys/class/leds/` 出现新名字。  
2. 阅读 `leds-gpio.c` 中 `gpio_leds_create` 与 `gpio_led_set` 全文。  
3. （可选）在板级用 `gpio-keys` 挂一个按键，用 `evtest` 看事件。

---

## 下一课

[第 4 课：字符设备](./04-字符设备.md)
