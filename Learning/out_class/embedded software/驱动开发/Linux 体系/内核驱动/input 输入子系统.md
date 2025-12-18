### 0. 资料参考：

-  [linux输入子系统详解——看这一篇就够了-CSDN博客](https://blog.csdn.net/weixin_42031299/article/details/125111946)

### 1. 基础概念：

#### 1.1 简介：

- Linux 输入子系统 (`Input Subsystem`) 是**Linux 内核中用于处理输入设备**的框架，它**提供了一套通用接口**，使不同类型的输入设备能够以一致的方式向用户空间传递**事件**。

#### 1.2 系统架构：

- Linux 输入子系统的架构主要是由以下几个部分组成：（`Down to Top`）

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250307230706.png" width = "600 px"/>
</div>

- **1. 硬件设备（`Hardware Layer`）**：
	- 物理输入设备，如鼠标、键盘、触摸屏、游戏手柄等，连接到系统不同接口（`I2C`、`SPI` 、`USB` 、`PS/2` 等）

- **2. 输入设备驱动层（`Input Device Drivers`）**：
	- 设备驱动程序负责与具体的输入设备交互。
	- 设备驱动层分为两个部分：硬件特性 + 核心层注册接口
		- **硬件特性（`Hardware Specifics`）**：
			- 负责与底层硬件（如 `GPIO`、`I2C`、`SPI`、`USB` 等接口的输入设备）进行交互，实现具体的设备操作
			- 该部分驱动主要实现：
				- **硬件初始化：** 配置 `GPIO`、初始化对应接口
				- **数据读取：** 以轮询（`poll`）或中断（`interrupt`）的方式从设备读取输入数据，并对数据进行解析，转化为标准的 Linux 输入事件格式
				- **中断处理：** 使用中断（`IRQ`）来捕获输入事件
		- **核心层注册接口（Core Registration Interface）**：
			- 输入子系统为设输入设备提供向内核注册的接口，即为输入设备进行注册
			- 驱动程序会调用 `input_allocate_device()` 分配 `input_dev` 结构体，并通过 `input_register_device()` 注册到系统中。
			- 当设备产生输入时，驱动调用 `input_report_*()` 进行事件上报

- **3. 输入核心层（Input Core Layer）**：
	- 核心层是 Linux 输入子系统的中枢，起到承上启下的作用，负责协调输入事件在事件处理层与设备驱动层之间的传递
		- 主要作用：
			- 管理输入设备（`input_dev`）和事件处理程序 (`input_handler`) 的注册和匹配
			- 协调事件的传递，确保设备驱动上报输入数据正确传递到对应的事件处理程序

- **4. 事件处理层（`Event Handler Layer`）**：
	- 处理输入设备的输入数据，并将其转换为标准格式，并提供给用户空间
	- 对于向内核输入子系统注册的输入设备，会在 `sysfs` 创建设备节点，应用程序通过操作设备节点来获取输入事件
	- 事件处理层将输入事件划分为几大类，例如通用事件 `event`、鼠标事件 `mouse`、摇杆事件 `js` 等等，每个输入类设备在注册使需要指定属类


### 2. 重要结构体：

#### 2.1  `input_dev` ：输入设备结构体

- 每个输入设备在 `Linux` 内核中都对应一个 `struct input_dev` 结构体，它包含设备的基本信息，如支持事件类型、按键、坐标范围等
- 结构体原型：

```c
struct input_dev {
	const char *name;	/* 设备名称 */
	const char *phys;	/* 设备物理路径 */
	const char *uniq;	/* 设备唯一标识符 */
	struct input_id id;	/* 设备ID 与 input handler 匹配会用到*/
	
	unsigned long evbit[BITS_TO_LONGS(EV_CNT)];		// 设备支持的事件类型
	
	unsigned long keybit[BITS_TO_LONGS(KEY_CNT)];	// 设备支持的具体的按键、按钮事件
	unsigned long relbit[BITS_TO_LONGS(REL_CNT)]; 	// 户设备支持的具体的相对坐标事件
	unsigned long absbit[BITS_TO_LONGS(ABS_CNT)];	// 设备支持的具体的绝对坐标事件
	.....
	
	int sync;				// 在上次同步事件(EV_SYNC)发生后没有新事件产生，则被设置为1 
	
	int abs[ABS_CNT];		// 用于上报的绝对坐标当前值
	int rep[REP_MAX + 1];	// 记录自动重复按键参数的当前值
	
	int absmax[ABS_CNT];	// 绝对坐标的最大值
	int absmin[ABS_CNT];	// 绝对坐标的最小值
	....
	
	int (*open)(struct input_dev *dev);
	void (*close)(struct input_dev *dev);
	int (*flush)(struct input_dev *dev, struct file *file); 
	int (*event)(struct input_dev *dev, unsigned int type, unsigned int code, int value);
	
	struct input_handle *grab;
	
	struct device dev;
	struct list_head h_list;
	struct list_head node;
} 
```


#### 2.2  `struct input_handler`：输入处理结构体

- `input_handler` 负责处理来自 `input_dev` 设备的数据，并向用户空间提供接口

```c
struct input_handler {
	void *private;	// 由具体事件处理程序指定的私有数据
	
	void (*event)(struct input_handle *handle, unsigned int type, unsigned int code, int value);
	
	bool (*filter)(struct input_handle *handle, unsigned int type, unsigned int code, int value);
	bool (*match)(struct input_handler *handler, struct input_dev *dev);
	
	int (*connect)(struct input_handler *handler, struct input_dev *dev, const struct input_device_id *id);
	
	void (*disconnect)(struct input_handle *handle);
	
	void (*start)(struct input_handle *handle);
	
	const struct file_operations *fops;
	int minor;
	const char *name;
	
	const struct input_device_id *id_table;
	
	struct list_head h_list;
	struct list_head node;
}
```