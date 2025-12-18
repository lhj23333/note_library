### 0. 学习参考：

-  [一张图掌握 Linux platform 平台设备驱动框架！【建议收藏】-CSDN博客](https://blog.csdn.net/qq_16504163/article/details/118562670)
- [5_10-1.驱动进化之路_总线设备驱动模型_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1w4411B7a4?spm_id_from=333.788.player.switch&vd_source=e71be9ebbb70e681992d6bfd2b2fb088&p=108)
-  [Linux驱动开发—平台总线模型详解-CSDN博客](https://blog.csdn.net/weixin_46999174/article/details/140933712)

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/platform%20%E6%80%BB%E7%BA%BF%E6%A8%A1%E5%9E%8B.png"/>
</div>

### 1. 从体系角度理解 platform 总线：

在嵌入式 Linux 开发中，设备驱动模型和 `platform` 总线模型是实现设备与驱动匹配和管理的关键。`platform` 总线模型相较于传统的设备驱动模型，进一步优化了设备和驱动的关联方式。

1. 设备驱动模型：
   在 Linux 驱动模型中，设备（`device`）和驱动（`driver`）之间的关联是通过设备驱动模型来实现的。通常，这种关联方式是基于硬件的抽象和设备类型的匹配来完成的，最典型的例如字符设备驱动模型。设备驱动模型提供了基础的设备与驱动匹配机制，但它在硬件信息封装、资源管理和多设备复用方面存在一定的局限性，尤其是在嵌入式系统的环境下。

2. `platform` 总线模型提出：
   `platform` 总线模型的出现，正是为了在设备驱动模型基础上，提供更为灵活和高效的设备驱动匹配机制。其核心思想是在设备与驱动之间通过一条虚拟的总线进行关联，从而实现对硬件设备的高效管理和驱动加载。它特别适用于嵌入式系统中内存资源有限的环境。

即， `platform` 总线模型是基于设备驱动模型的优化，它通过在设备与驱动之间引入一个虚拟的总线层，实现了硬件信息的封装、驱动的复用和内存优化。相较于传统设备驱动模型，它提高了系统的灵活性、可维护性，并为嵌入式开发提供了更高效的资源管理方式。通过这种机制，硬件信息被有效隔离与管理，硬件层和应用层之间的接口也得到了更好的封装和抽象。

### 2. 关键结构体与函数：

#### 2.1 关键结构体：

-  `struct resource`：
```c
struct resource{
	resource_size_t start;
	resource_size_t end;
	unsigned long flags;
	struct resource *parent, *sibling, *child;
};
```

- `struct platform_device`：
```c
struct platform_device{
	const char *name;		// 设备名称
	int id;
	bool id_auto;
	struct device dev;
	u32 num_resource;
	struct resource *resource;
	
	const struct platform_device_id *id_entry;
	char *driver_override;	// 用户态可指定的驱动名称
	
	/* MFD cell pointer */
	struct mfd_cell *mfd_cell;
	
	/* arch specific addition */
	struct pdev_archdata archdata;
};
```

- `struct platform_driver`：
```c
struct platform_driver{
	int (*probe)(struct platform_device *);
	int (*remove)(struct platform_device *);
	void (*shutdown)(struct platform_device *);
	int (*suspend)(struct platform_device *, pm_message_t state);
	struct device_driver driver;				// 核心驱动结构
	const struct platform_device_id *id_table;	// 设备 ID 表
	bool prevent_deferred_probe;
};
```

- `struct bus_type platform_bus_type`： 
```c
struct bus_type platform_bus_type{
	.name = "platform",
	.dev_groups = platform_dev_groups,
	.match = platform_match,
	.uevent = platform_uevent,
	.pm = &platform_dev_pm_ops,
};
```


#### 2.2 `match()` 函数：
- `platform_match()`：匹配机制实现函数
```c
static int platform_match(struct device *dev, struct device_driver *drv){
	struct platform_device *pdev = to_platform_device(dev);	// 将 device 转为 platform_device
	struct platform_driver *pdrv = to_platform_driver(drv); // 将 driver 转为 platform_driver
	
	// 1.检查 driver_override 字段，优先匹配
	if(pdev->driver_override && strcmp(pdev->driver_override, drv->name) == 0){
		return 1;	
	}
	
	// 2.检查 id_table，次优匹配
	if(pdrv->id_table){
		for(id = pdrv->id_table; id->name[0]; id++){
			if(strcmp(pdev->name, id->name) == 0){
				return 1;
			}
		}
	}
	
	// 3.直接比较 driver->name 和 device->name
	return strcmp(pdev->name, drv->name) == 0;
}
```
- 参数解释：
	- `dev`：设备对象（类型为 `struct device` ）
	- `drv`：设备驱动对象（类型为 `struct device_driver`）
- 返回值：
	- 匹配成功返回 1
	- 匹配失败返回 0 

#### 2.3 `probe()` 函数：

- 函数签名：
```c
int (*probe)(struct device *dev);
```
- 主要作用：
	- 初始化硬件
	- 配置设备的资源
	- 启动设备的功能
	- 注册中断处理程序、设备文件等
	- 将设备的私有数据结构与设备关联

- 函数示例：
```c
static struct file_operations example_fops = {
	.owner = THIS_MODULE;
	.open = my_open;
	.read = my_read;
	.write = my_write;
	.ioctl = my_ioctl;
};

static int my_device_probe(struct platform_device *pdev){
	int ret;
	struct my_device *dev_data;
	struct resource *res;
	void __iomem &regs;
	
	// 初始化内核资源
	dev_data = devm_kzalloc(&pdev->dev, size(*dev_data), GFP_KERNEL);
	if(!dev_data){
		return -ENOMEM;
	}
	
	// 获取设备的资源：内存映射地址
	res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
	if(!res){
		dev_err(&pdev->dev, "Memory resource not found\n");
		return -ENODEV;
	}
	
	// 请求内存区域
	regs = devm_ioremap_resource(&pdev->dev, res);
	if(IS_ERR(regs)){
		dev_err(&pdev->dev, "Failed to map memory\n");
		return PTR_ERR(regs);
	}
	
	// 初始化设备硬件
	// 例如：设置某个寄存器的值
	
	// 设备初始化(如字符设备等)
	cdev_init(&dev_data->cdev, &my_fops);
	ret = cdev_add(&dev_data->cdev, dev_data->devno, 1);
	if(ret){
		return ret;
	}
	
	// 设备注册
	device_create(example_class, &pdev->dev, dev_data->devno, NULL, "my_device");
	
	dev_info(&pdec->dev, "Device initialized successfully");
}
```


### 3. 结构体关系性分析：

-  `struct paltform_device` 与 `struct platform_driver` 匹配机制：
- **浅层理解**：
- 在不带设备树的 `platform` 总线模型中，`platform_device` 与 `platform_driver` 的匹配主要依赖于 `name`，匹配流程遵循如下优先级：
- `driver_override` 机制（最高优先级）：
	- 若 `platform_device` 结构体中的 `driver_override` 字段非空，则仅匹配 `platform_driver` 结构体中 `struct device_driver` 的 `name`，若 `name == driver_override`，则将该 `platform_device` 与 `platform_driver` 进行关联
	- 该机制允许用户态或内核动态指定驱动覆盖默认匹配逻辑。
- `id_table` 机制（次优先级）：
	- 若 `driver_override` 为空，则遍历所有 `platform_driver`，查找其 `id_table` (`platform_device_id` 数组，支持设备列表 )
	- 若 `platform_device->name = id_table.name`，则将该 `platform_device` 与 `platform_driver` 进行关联
- `name` 直接匹配（最低优先级）（**最常用**）：
	- 若 `id_table` 匹配失败，则继续比较 `platform_device->name` 与 `platform_driver->driver.name`
	- 若匹配成功，则绑定对应 `platform_device` 与 `platform_driver`

> [!NOTE] 带设备树的 `platform` 总线模型
> 在带设备树的 `platform` 总线模型中，`platform_device` 与 `platform_driver` 的匹配机制通过设备树 `device tree` 中的 `compatible` 属性与驱动中的 `drive->of_match_table` 来实现
> 具体流程如下：
> - 设备的匹配：
> 	- 设备通常是通过设备树（`.dts` 文件）来进行描述，设备树中包含设备的硬件信息，其中 `compatible` 字段是用以描述设备与支持该设备的驱动兼容性的标识符
> - 驱动的匹配：
> 	- 驱动在内核中会通过 `of_match_table` 字段来定义支持设备（匹配设备列表）。`of_match_table` 是一个结构体数组，数组中每个元素都包含 `compatible` 字段，用于指定驱动支持的设备类型
> - 匹配过程：
> 	- 内核在启动时，会对每个设备（通过平台总线注册）和所有的驱动进行匹配
> 	- 对于平台设备，内核会查找器设备树节点中的 `compatible` 属性
> 	- 如果该属性与驱动的 `driver->of_match_table` 中的某一项兼容，驱动就会与设备绑定

- **深层理解**：
- 上述匹配机制是在设备或设备驱动进行注册 `platform_device_register()/platfrom_driver_register()` 时，依赖于 `platform_bus_type` 中的函数成员 `platform_match` 实现的。
	- `platform_match` 实现可以参考前面函数原型进行理解
	- 下面对整个模型框架流程中该匹配机制实现进行理解 (以 `platform_device_register` 为例)
- 函数层级分析（简化版的哈哈）：
- `platform_device_register()`：
```c
int platform_device_register(struct platform_device *pdev){
	int ret;
	
	if(ret = platform_device_add(pdev)){
		return ret;
	}
	
	return 0;
}
```
- 当调用 `platform_device_register()` 函数对平台设备 `platform_device` 进行注册时，该函数实际会调用 `platform_device_add()` 函数将设备添加到内核设备模型中
- 返回值：
	- 若设备添加失败，`platform_device_register()` 会返回错误码；
	- 若添加成功，则会返回 0，表示设备已经被成功注册

- `platform_device_add()`：
```c
int platform_device_add(struct platform_device *pdev){
	int ret;
	
	// 将设备添加到设备模型
	if(ret = device_add(&pdev->dev)){ 	
		return ret;
	}
	
	// 进行驱动与设备匹配
	if(ret = bus_probe_device(&pdev->dev)){
		device_del(&pdev->dev);		// 匹配失败，删除设备
	}
	
	return ret;
}
```
- 当调用 `platform_device_add()` 函数时，该函数会调用 `device_add()` 函数将平台设备添加到设备总线中，并调用 `bus_probe_device()` 函数启动驱动与设备的匹配过程
	- `device_add(&pedv->dev)`：
		- 将设备添加到设备模型（`device tree` 中），设备开始作为一个内核对象参与管理
		- 设备将注册成一个 `struct device` 类型的对象，其中包含了设备的各类信息，包括驱动程序、设备名称、资源等
	- `bus_probe_device(&pdev->dev)`：
		- 该函数会触发设备的驱动匹配过程。通过总线类型的 `match()` 函数，遍历与设备匹配的驱动列表，进行驱动的匹配操作
- 返回值：
	- 如果设备添加和驱动匹配都成功，`platform_device_add()` 返回 0
	- 如果设备添加或驱动匹配失败，`platform_device_add()` 会返回错误代码。同时如果匹配失败，设备会从设备模型中删除

- `bus_probe_device()`：
```c
int bus_probe_device(struct device *dev){
	struct device_driver *drv;
	struct bus_type *bus = dev->bus;
	
	// 遍历所有与该设备总线类型匹配的驱动
	list_for_ench_entry(drv, &bus->drivers, node){
		if(drv->probe){
			// 调用 bus_type 中的 match 函数进行设备与启动的匹配
			if(bus -> match(dev, drv)){
				// 如果匹配成功，调用驱动的 probe 函数进行设备初始化
				return drv->probe(dev);
			}
		}
	}
	
	return -ENODEV;	// 如果没有匹配的驱动，返回错误
}
```
- 当调用 `bus_probe_device()` 时，它会遍历与当前设备总线类型 (`dev->bus`) 相关的所有驱动，并调用其所属总线类型的 `match()` 函数（即 `platform_match()`）, 判断当前总线是否存在驱动可以与当前设备进行匹配
	- `list_for_each_entry(drv, &bus->driver, node)`：遍历与设备总线类型（`dev->bus`）相关的所有驱动，并检查该驱动是否具有 `probe()` 函数
		- `&bus->driver`：驱动列表
	- `bus->match()`：调用当前总线类型的 `platform_match()` 函数来判断设备能否与当前设备匹配。
		- 如果设备与驱动匹配成功，`match()` 返回 1，则调用 `drv->probe` 函数来初始化设备
	- 返回值：
		- 如果找到匹配的驱动，并且驱动的 `probe()` 函数被成功调用，`bus_probe_device()` 返回 `probe()` 函数返回值
		- 如果没有找到匹配的驱动，`bus_probe_device()` 返回 `-ENODEV`（表示未找到匹配的驱动）

### 4. 模型代码实现流程：

1. 分配、设置以及注册 `platform device` 结构体：
	- 定义`struct resource` 设备资源
	- 指定设备名字 `name`
2. 分配、设置以及注册 `platform drive` 结构体：
	- 在 `probe` 函数中，分配、设置以及注册 `struct operations` 结构体
	- 从 `platform device` 中确定所需的硬件资源
	- 指定驱动名字 `name`