# 第 6 课：I2C 与 SPI 客户端驱动

目标：分清 **总线控制器驱动** 与 **外设客户端驱动**，掌握传感器类驱动的常见写法。

---

## 1. 两层结构

```
┌─────────────────────────────┐
│  客户端驱动（传感器、EEPROM） │  ← i2c_driver / spi_driver
│  知道芯片寄存器含义           │
└──────────────┬──────────────┘
               │ i2c_transfer / regmap / spi_write
┌──────────────▼──────────────┐
│  控制器驱动（SoC I2C/SPI IP）│  ← platform_driver（如 i2c-rk3x）
│  只负责时序与中断             │
└──────────────┬──────────────┘
               │
            SCL/SDA 或 CLK/MOSI...
```

你写温湿度、IMU、PMIC 时，多半是 **客户端**；`i2c-rk3x.c` 是控制器，一般不用改。

---

## 2. 设备树怎么写

控制器（SoC，已存在）：

```dts
i2c0: i2c@21100000 {
	compatible = "rockchip,rv1126b-i2c";
	reg = <0x21100000 0x1000>;
	interrupts = <GIC_SPI 48 IRQ_TYPE_LEVEL_HIGH>;
	clocks = <&cru CLK_I2C0>, <&cru PCLK_I2C0>;
	clock-names = "i2c", "pclk";
	#address-cells = <1>;
	#size-cells = <0>;
	status = "okay";

	/* 客户端挂在总线节点下 */
	accel@18 {
		compatible = "vendor,my-accel";
		reg = <0x18>;          /* 7-bit I2C 地址 */
		interrupt-parent = <&gpio0>;
		interrupts = <RK_PBx IRQ_TYPE_EDGE_FALLING>;
	};
};
```

注意：客户端的 `reg` 是 **I2C 地址**，不是 MMIO。

SPI 类似，用 `spi@...` 控制器 + 子节点 `reg = <片选号>`。

---

## 3. I2C 客户端驱动骨架

```c
#include <linux/i2c.h>
#include <linux/module.h>
#include <linux/regmap.h>

struct my_accel {
	struct i2c_client *client;
	struct regmap *map;
};

static int my_accel_probe(struct i2c_client *client, const struct i2c_device_id *id)
{
	struct my_accel *priv;
	u8 chip_id;

	priv = devm_kzalloc(&client->dev, sizeof(*priv), GFP_KERNEL);
	priv->client = client;

	/* 推荐 regmap，少手写 i2c_transfer */
	priv->map = devm_regmap_init_i2c(client, &my_regmap_config);
	regmap_read(priv->map, REG_CHIP_ID, &chip_id);

	i2c_set_clientdata(client, priv);
	/* 注册 input / iio / 字符设备 … */
	return 0;
}

static void my_accel_remove(struct i2c_client *client)
{
	/* 清理；devm 资源可自动释放 */
}

static const struct of_device_id my_accel_of_match[] = {
	{ .compatible = "vendor,my-accel" },
	{ }
};
MODULE_DEVICE_TABLE(of, my_accel_of_match);

static struct i2c_driver my_accel_driver = {
	.driver = {
		.name = "my-accel",
		.of_match_table = my_accel_of_match,
	},
	.probe_new = my_accel_probe,  /* 内核版本 API 名可能为 probe */
	.remove = my_accel_remove,
};
module_i2c_driver(my_accel_driver);
MODULE_LICENSE("GPL");
```

与 platform 对比：

| | platform | i2c 客户端 |
|---|---|---|
| 匹配 | `of_device_id` | 同样 + 可选 `i2c_device_id` |
| probe 参数 | `platform_device *` | `i2c_client *` |
| 注册宏 | `module_platform_driver` | `module_i2c_driver` |
| 通信 | 自己 ioremap 寄存器 | `i2c_client` / regmap |

---

## 4. SPI 客户端要点

```c
static int my_probe(struct spi_device *spi)
{
	spi->mode = SPI_MODE_0;
	spi->max_speed_hz = 1000000;
	spi_setup(spi);
	/* spi_write / spi_read / spi_sync */
	return 0;
}

static struct spi_driver my_driver = {
	.driver = {
		.name = "my-spi-dev",
		.of_match_table = my_of_match,
	},
	.probe = my_probe,
};
module_spi_driver(my_driver);
```

---

## 5. 本 SDK 中去哪找例子

| 类型 | 路径提示 |
|---|---|
| I2C 控制器 | `drivers/i2c/busses/i2c-rk3x.c` |
| SPI 控制器 | `drivers/spi/spi-rockchip*.c` |
| 传感器/编解码器 | `drivers/iio/`、`sound/soc/codecs/`、`drivers/input/` |
| 板级挂载 | `arch/arm64/boot/dts/rockchip/*rv1126b*`、`OK1126B*.dtsi` |

搜板级 dts：`compatible` + `reg = <0x..>` 在 `&i2cX` 下的节点，再反查驱动。

---

## 6. 用户态两条路

1. **内核驱动 + 标准子系统**：IIO、Input、hwmon → `/sys`、`/dev/iio:deviceN`  
2. **用户态直接打 I2C**：`/dev/i2c-N` + `i2c-tools`（`i2cget`）—— 适合调试；量产复杂设备仍建议内核驱动

---

## 7. 检查题

1. I2C 子节点 `reg = <0x18>` 是 MMIO 吗？→ **不是，是从设备地址**  
2. 客户端还要自己 `platform_get_irq` 控制器中断吗？→ **一般不要**；数据就绪可用 GPIO 中断  
3. 为何推荐 regmap？→ 统一读写、支持缓存/锁、少重复代码

---

## 8. 练习

1. 在 `rv1126b` 相关 dts 里找一个真实 I2C 子设备，记下 compatible 与地址。  
2. 用 `git grep` / 搜索该 compatible，定位驱动 `probe`。  
3. 板上：`ls /dev/i2c-*`，`i2cdetect -y 0`（注意总线号与权限）。

---

## 下一课

[第 7 课：MPP 驱动案例精读](./07-MPP驱动案例精读.md)
