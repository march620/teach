# 第 4 课：设备树改 GPIO

目标：读懂 GPIO 三元组，会在练习板 DTS 里正确新建节点，并编出 practice 的 dtb。

---

## 1. GPIO 写法

板级真实例子（WiFi 复位）：

```dts
reset-gpios = <&gpio0 RK_PB2 GPIO_ACTIVE_LOW>;
```

| 字段 | 含义 |
|------|------|
| `&gpio0` | 第 0 组 GPIO 控制器 |
| `RK_PB2` | 该组 **PB2** 脚（不是 PB0） |
| `GPIO_ACTIVE_LOW` | 有效电平为低 |

这是「控制器 + 脚 + 极性」，不是「0 亮 1 灭」的口语描述。  
若 LED 且 `ACTIVE_LOW`，驱动认为“亮”时会把脚拉低。

当脚要当普通 GPIO 时，常在 `&pinctrl` 里声明复用，例如：

```dts
rockchip,pins = <0 RK_PB2 RK_FUNC_GPIO &pcfg_pull_none>;
```

---

## 2. 改哪里

| 改动 | 文件 |
|------|------|
| 本板新增 LED/按键节点 | 板级 `.dts`（练习：`OK1126B-practice-linux.dts`） |
| 多板共用基础外设 | `OK1126B-S-common.dtsi`（练习阶段先别动原厂） |

练习用独立 dts，避免改错原厂 common 导致引脚冲突、难启动。

---

## 3. 新建节点：`leds` 不要写成 `&leds`

**错误（引用不存在的标签）：**

```dts
&leds {
	compatible = "gpio-leds";
	...
};
```

`&leds` 表示修改已有标签 `leds:`。本板 common 里若没有该标签，DTC 会报 `Label or path leds not found`。

**正确（在根节点内新建）：**

```dts
/ {
	model = "OK1126B Practice Board";
	/* ... chosen ... */

	leds {
		compatible = "gpio-leds";

		practice_led {
			label = "practice-led";
			gpios = <&gpio0 RK_PC0 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};
	};
};
```

| 写法 | 含义 |
|------|------|
| `/ { leds { } }` | **新建** |
| `&leds { }` | **引用并修改**已有节点 |
| `&fiq_debugger { }` | 修改已有节点（正确用法） |

`RK_PC0` 为教学占位脚，真板需对照原理图；脚不对不一定变砖，但可能无效或冲突。

编辑时注意：**必须用英文分号 `;`，不能用中文全角 `；`**。

---

## 4. 编译练习板

```bash
cd /path/to/OK1126B-linux-source

make OK1126B_PRACTICE_buildroot_defconfig
grep RK_KERNEL_DTS_NAME output/.config   # 应为 OK1126B-practice-linux

make kernel
ls -l kernel-6.1/arch/arm64/boot/dts/rockchip/OK1126B-practice-linux.dtb
```

若只跑 `make kernel` 而 `.config` 仍是 S 板，会继续编 `OK1126B-S-linux.dtb`，**practice 改动不会进包**。

---

## 课后练习

1. 解释 `<&gpio0 RK_PB2 GPIO_ACTIVE_LOW>` 三个字段。
2. 为什么练习改 DTS 用 practice 文件而不是直接改 common？
3. 加上 `leds` 节点，切换 PRACTICE 配置后 `make kernel`，确认生成 `.dtb`。

## 下一课

[第 5 课：烧录与串口日志](./05-烧录与串口日志.md)
