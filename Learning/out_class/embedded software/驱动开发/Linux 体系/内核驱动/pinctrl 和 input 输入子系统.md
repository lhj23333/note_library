### 0. 学习参考：

-  [【深度】韦东山：GPIO和Pinctrl子系统的使用-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1709010)

### 1. `Pinctrl` 子系统

- `Pinctl` 子系统主要用于配置引脚复用、驱动、功能以及引脚的其他硬件特性
- 主要功能：
	- **引脚复用**：`pinctrl` 子系统允许配置每个引脚的功能
	- **电压/电流控制**：`pinctrl` 还可以配置引脚的驱动能力、电流和电压，确保硬件的稳定性
	- **硬件功能启用**：`pinctrl` 可以启用特定硬件功能，比如将引脚设置为 `SPI`  总线的数据线、`UART` 串口的TX/RX等
	- **`GPIO` 控制**：

#### 1. 1 `Pinctrl` 子系统模型对象：

- `pin controller`：
	- 提供服务端：进行引脚复用以及引脚配置

``` c
pincontroller{
	state_0_node_a {
		function = "uart0";					// 复用功能（复用为 uart0 功能）
		groups = "u0rxtx", "u0rtscts";		// 引脚群组
	};
	
	state_1_node_a {
		groups = "u0rxtx", "u0rtscts";		// 引脚群组
		output-high;						// 引脚配置（输出高电平）
	};
};
```
- `groups`：引脚节点状态
- `functions`：复用功能配置
- `state_0_node_a`：`Generic pin multiplexing` 通用引脚复用
- `state_1_node_b`：`Generic pin configuration node` 通用引脚配置
- `pin controller` 节点格式没有统一的标准!!! 每家芯片不一样

- `client device`：
	- 服务客户端：声明设备状态

```c
// For a client device requiring named states
devcie {
	pinctrl-names = "default", "sleep";			// 描述设备状态
	pinctrl-0 = <&state_0_node_a>;				// 设备状态 default 引脚功能配置
	pinctrl-1 = <&state_1_node_a>;				// 设备状态 sleep 引脚功能配置
};
```
- `pinctrl-names = 'default', 'sleep';`： 设备存在 `default` 和 `sleep` 两种状态
- `pinctrl-0 = <&state_0_node_a>`：第一种状态 `default` 在 `pinctrl-0` 中定义，调用 `state_0_node_a` 引脚节点状态实现
- `pinctrl-1 = <&state_1_node_a>`：第二种状态 `sleep` 在 `pinctrl-1` 中定义，调用 `state_1_node_a` 引脚节点状态实现

#### 1.2 `pinctrl` 子系统调用：

- emmm, 一般不调
	- 在嵌入式开发过程中，在硬件配置和功能已经确定的情况下，其对应引脚的配置需求也基本确定，基本上不会进行动态的调整，同时，`pinctrl` 的实现在内核已经封装完善，当设备切换状态时，对应的 `pinctrl` 就会被调用，所以并不需要手动调用`pinctrl`接口来切换引脚配置。
- 其实现是在 `platfrom_device` 和 `platform_driver` 枚举过程中实现的：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250221105008590.png" width = "700px"/>
</div>

- 动态切换引脚配置：
	- 当需要在运行时切换引脚的配置时，`pinctrl`提供了相应的接口来动态修改引脚的状态。
		- 主要流程：
			- 获取设备 `pinctrl` 句柄
			- 切换引脚配置

```c
struct pinctrl *pinctrl
struct pinctrl_state *gpio_state, *uart_state

// 获取 pinctrl 句柄
pinctrl = devm_pinctrl_get(dev);
if(IS_ERR(pinctrl)){
	pr_err("Failed to get pinctrl\n");
	return PTE_ERR(pinctrl);
}

// 查找 GPIO 和 UART 模式的配置
gpio_state = pinctrl_lookup_state(pinctrl, "gpio");
uart_state = pinctrl_lookup_state(pinctrl, "uart");

if(IS_ERR(gpio_state) || IS_ERR(uart_state)){
	pr_err("Failed to lookup pinctrl states\n");
	return -EINVAL;
}

// 切换 GPIO 模式
pinctrl_select_state(pinctrl, gpio_state);

// 进行 GPIO 操作

// 切换 UART 模式
pinctrl_select_state(pinctrl, gpio_state);

// 进行 UART 操作
```

### 2. `GPIO` 子系统

#### 2.1 基础概念与流程

- `GPIO` 是一个通用的输入/输出接口，通常用于控制微控制器（`MCU`）或处理器的引脚。`GPIO` 引脚可以配置为输入、输出、或者某些特定的功能 (如中断)，`GPIO` 子系统负责管理这些引脚的状态和配置
- 主要功能：
	- 输入/输出控制      ->      输入、推挽输出、开漏输出模式
	- 中断管理               ->      配置中断触发
	- 电平控制               ->      设置 `GPIO` 引脚电平

- 要进行 `GPIO` 引脚操作时，要先把引脚配置为 `GPIO` 功能，这需要通过 `Pinctrl` 子系统实现，
- 然后就可以依据需求，设置引脚方向，读值 - 获取电平状态，写值 - 写入电平状态
- 具体流程步骤：
	- 在设备树中指定 `GPIO` 引脚
	- 在驱动代码中：
		- 使用 `GPIO` 子系统标准函数获取 `GPIO`，设置 `GPIO` 方向，读取/设置 `GPIO` 值

- `GPIO` 子系统 `GPIO controller`：
- 在设备树 `xxx.dts` 文件中， `GPIO Controller` 通常是由厂家定义好的
-  芯片厂商 `.dts`:
```dts
gpio1: gpio@0209c000{
	compatible = "fsl, imx6ul-gpio", "fsl, imx35-gpio";
	reg = <0x02x9c00 0x4000>;
	interrupts = <GIC_SPI 66 IRQ_TYPE_LEVEL_HIGH>,
				 <GIC_SPI 67 IRQ_TYPE_LEVEL_HIGH>,
				 
	gpio-controller;
	#gpio-cells = <2>;	
	
	interrupt-controller;
	#interrupt-cells = <2>;
	
	gpio-ranges = <&iomuxc  0 23 10>, <&iomuxc 10 17  6>,
				  <&iomuxc 16 33 16>;
};
```
- `gpio-controller`：表示该节点是一个 `GPIO Controller`, 其下会由很多引脚
- `#gpio-cells`：表示该控制器下每一个引脚要用多少位字符来进行描述

- 设备 `.dts`：
```dts
device {
	compatible = "goodix, gt9xx";
	reg = <0x5d>;
	status = "okay";
	interrupt-parent = <&gpio1>;
	interrupts = <5 IRQ_TYPE_EDGE_FALLING>;
	
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_tec_reset &pinctrl_touchscreen_int>;
	
	reset-gpios = <&gpio5 2 GPIO_ACTIVE_LOW>;
	irq_gpios = <&gpio1 5 IRQ_TYPE_EDGE_FALLING>;
}
```
- 在前面芯片厂商提供的 `gpio-controller` 下规定了 `gpio-cells = <2>`, 则在设备 `.dts` 中对于该 `gpio` 组进行设置时，`&gpiox` 后需添加两个元组 `2 GPIO_ACTIVE_LOW` 进行设置，一般格式 ：
```dts
name-gpios = <&gpiox num flags>;
```
- `name-gpios`：该规则下，`name` 为对设置 `GPIO` 的别名，以供后续驱动程序调用
- `gpiox`：芯片 `.dts` 内设定的 `GPIO` 群组
- `num`：该 `GPIO` 群组下的 `GPIO` 编号
- `flags`：`GPIO` 配置设置，常用 `flags`
	- `GPIO_ACTIVE_HIGH`：高电平有效
	- `GPIO_ACTIVE_LOW`：低电平有效


#### 2.2.  驱动代码调用 `GPIO` 子系统：

- `GPIO` 子系统接口：基于描述符的（`descriptor-based`）,
	- 使用 `gpio_desc` 结构体来表示一个引脚
- 该接口函数头文件 `#include <linux/gpio/consumer.h>`
- 常用函数：
	- 获取 `GPIO`:
		- `gpiod_get`
		- `gpiod_get_index`
	- 设置 `GPIO` 方向：
		- `gpiod_direction_input`
		- `gpiod_direction_ouput`
	- 读值、写值：
		- `gpiod_get_value`
		- `gpiod_set_value`
	- 释放 `GPIO`:
		- `gpiod_free`
		- `gpiod_put`
- 在 Linux 开发过程中，需要先申请 `GPIO`，再申请内存；如果内存申请失败，再返回之前需要先释放 `GPIO` 资源
- 示例：
- `device.dts`：
```dts
foo_device {
	compatible = "acme, foo";
	....
	led-gpios = <&gpio 15 GPIO_ACTIVE_HIGH>,
				<&gpio 16 GPIO_ACTIVE_HIGH>,
				<&gpio 17 GPIO_ACTIVE_HIGH>;
				
	power-gpios = <&gpio 1 GPIO_ACTIVE_LOW>;
};
```
- `driver.c`：
```c
struct gpio_desc *red, *green, *blue, *power;
red = gpiod_get_index(dev, "led", 0, GPIOD_OUT_HIGH);
green = gpiod_get_index(dev, "led", 1, GPIO_OUT_HIGH);
blue = gpiod_get_index(dev, 'led', 2, GPIO_OUT_HIGH;
power = gpiod_get(dev, "power", GPIOD_OUT_HIGH);
```


### 3. 驱动实现调用 `GPIO` 子系统：

#### 3.1 驱动实现流程：

- 定义、注册 `platform_driver`
- 在 `platform_driver -> probe` 内
	- 依据 `platform_device` 的设备树信息确定 `GPIO`
	- 定义、注册 `file_operations` 结构体
	- 在 `file_operations` 结构体内使用 `GPIO` 子系统接口函数操作 `GPIO`




