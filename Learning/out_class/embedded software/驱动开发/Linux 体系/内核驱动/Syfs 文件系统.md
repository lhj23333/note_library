### 学习参考文献：

- [Linux驱动开发笔记（十三）Sysfs文件系统_linux sysfs-CSDN博客](https://blog.csdn.net/sincerelover/article/details/139882450)
- [[Linux 基础] -- sysfs 基本原理与使用方法_进程如何使用sysfs-CSDN博客](https://blog.csdn.net/u014674293/article/details/103480058)

### `sysfs` 个人理解：

> [!NOTE] `sysfs` 个人理解
> `sysfs` 是 Linux 内核的一个特殊的虚拟文件系统，它为用户空间提供了一个统一的访问内核对象以及设备信息的接口，即它将系统的内核参数 (`./kernel`)、驱动设备 (`./devices`)、驱动程序 （`./drivers`）以及模块 (`./module`) 等内容通过内核数据结构与文件系统映射的方式在文件系统中以文件、目录的形式提供给用户进行查看与修改的部分操作。
> 
> `sysfs` 文件系统与内核数据结构进行映射，并将文件系统挂载到`./sys`目录下，在其内依据不同的分类方式对系统信息进行分类：
> 设备驱动主要就以 `./devices` 、`./class ` 以及 `./bus` 几个目录为主，其中：
> - `/sys/devices`：根据设备的物理层次结构进行组织
> - `/sys/class`：根据设备的功能进行分类
> - `/sys/bus`：更具设备总线进行分类
>  但其实仅有 `/sys/devices` 目录下保存有所有设备驱动的真实对象，`./class` 以及 `./bus` 目录下设备驱动信息是通过内核数据结构链接到 `./sys/devices` 中的具体设备文件来实现的
>  
>    其余目录：
> - `/sys/kernel`：保存可调整内核参数信息
> - `/sys/module`：保存系统所有模块的信息
> - ....

### 深入理解以及补充：

#### `sysfs` 的设计原则：

- 一个属性文件只做一件事
	- `sysfs` 属性文件一般只有一个值，可以进行读取或写入

#### `sys` 目录文件内容详细讲解：

- `/device`：内核对系统中所有设备的分层表达模型，以设备本身的连接按层次组织。
- `/sys/dev`： 该目录主要维护字符设备和块设备的主次设备号 (`major:minor`) 链接到真实设备 (`sys/devices`) 的符号链接文件
- `/sys/bus`：内核设备按总线类型分层放置的目录结构
	- `device` 中所有设备都是连接于某种总线之下，在 `/bus` 目录下不同总线目录下可以找到连接该总线具体设备的符号连接
- `/sys/class`：按照设备功能分类的设备模型
	- 如系统所有输入设备都会出现在 `sys/class/input` 目录之下
- `/sys/block`：系统中当前所有块设备所在，按照功能来说应该放置于 `/sys/class` 下更核识，但由于历史遗留因素一直存在 `/sys/block`，但再 2.6.26 内核版本就已正式移到 `/sys/class/block` 下
	- 旧的接口 `/sys/block` 为了向后兼容保留存在，其中内容变为指向指向 `/sys/devices` 中真实设备对象的符号链接文件

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250214111211229.png" width = "600 px"/>
</div>

- `/sys/firmware`：系统加载固件机制对用户空间的接口
- `/sys/fs`：用于描述系统中所有的文件系统，包括文件系统本身和按文件系统分类存放的已挂载点
	- 目前只有 `fuse` 、`ext4` 等少数文件系统支持 `sysfs` 接口，一些传统的虚拟文件系统 (`VFS`) 层次控制参数仍然再 `sysctl` (`proc/sys/fs`) 接口中
- `/sys/kernel`：内核中可调整参数的位置
	- 有些内核可调整参数仍然位于 `sysctl` (`proc/sys/kernel`) 接口中
- `/sys/module`：系统中所有模块的信息

> [!NOTE] 模块信息
> 不论这些模块是以内联（`inlined`）方式编译到内核映像文件（`vmlinuz`）中还是编译为外部模块（`.ko`文件），都可能会出现在 `/sys/module` 中
> - 编译为外部模块（`.ko` 文件）在加载后会出现对应的`/sys/module//`，并且在这个目录下会出现一些属性文件和属性目录来表示次外部模块的一些信息，如版本号、加载状态、所提供的驱动程序等；
> - 编译为内联方式的模块则只在当它有非 0 属性的模块参数时会出现对应的`/sys/module/`，这些模块的可用参数会出现在`/sys/module//parameters/`中，如 `/sys/module/printk/parameters/time`这个可读可写参数控制着内联模块 `printk`在打印内核消息时是否加上时间前缀；所有内联模块的参数也可以由 "`.=`" 的形式写在内核启动参数上，如启动内核时加上参数 "`printk.time = 1`" 与向 "`/sys/module/printk/parameters/time`" 写入 1 的效果相同；没有非 0 属性参数的内联模快不会出现于此；

-  `/sys/power`：系统中的电源选项，该目录下有几个属性文件可用于控制整个机器电源状态


#### 设备模型基本结构：

| 类型                     | 所包含的内容                                                                    | 对应内核数据结构               | 对应 /sys 项                |
| ---------------------- | ------------------------------------------------------------------------- | ---------------------- | ------------------------ |
| 设备（`Devices`）          | 设备是此模型中最基本的类型，以设备本身的连接按层次组织                                               | `struct device`        | `/sys/devices/*/*/.../`  |
| 设备驱动（`Device Driver`）  | 在一个系统中安装多个相同设备，只需要一份驱动程序的支持                                               | `struct device_driver` | `/sys/bus/pci/driver/*/` |
| 总线类型（`Bus Types`）      | 在整个总线级别对此总线上连接的所有设备进行管理                                                   | `struct bus_type`      | `/sys/bus/*/`            |
| 设备类别（`Device Classes`） | 这是按照功能进行分类组织的设备层次树；如 USB 接口和 PS/2 接口的鼠标都是输入设备，都会出现在 `/sys/class/input/` 下 | `struct class`         | `/sys/class/*/`          |
#####  内核数据结构实现 -- 内核对象基本数据结构基类

1. `kobject` 结构体
- 在 Linux 内核中，`kobject` 是用于表示内核或其它内核对象的基本数据结构，是很多内核对象（如设备、设备驱动等）的基类
- 其结构原型：
```c
struct kobject{
	const char *name;
	struct list_head entry;
	struct kobject *parent;
	struct kset *kset;
	struct kobj_type *ktype;
	struct sysfs_dirent *sd;
	struct kref kref;
	unsigned int state_initialized:1;
	unsigned int state_in_sysfs:1;
	unsigned int state_add_uevent_sent:1;
	unsigned int state_remove_uevent_sent:1;
}
```
- 参数解释：
	- `name`：对象名称，用于标识对象
	- `entry`：`struct list_head` 类型，表示该 `kobject` 结构将作为一个对象，插入到父对象链表中
	- `parent`：指向该 `kobject` 父节点指针
	- `kset`：一个 `kset` 结构的指针，`kset` 是一种对象集合，用于组织和管理一组 `kobject` 对象。
	- `kobj_type`：`kobject` 类型，用于定义该 `kobject` 的行为
	- `sysfs_dirent`：`sysfs` 相关的字段，该字段指向与该 `kobjct` 相关的 `sysfs` 目录项
	- `kref`：`kref`是一个引用计数结构，用于管理该对象的生命周期，确保对象不会被提前销毁。
	- `state_initalized`：标志位，用于表示 `kobject` 是否已初始化
	- `state_in_sysfs`：标记该 `kobject` 是否已被导出到 `sysfs`
	- `state_add_uevent_sent`：标记是否已经发送了 `uevent` 事件
	- `state_remove_uevent_sent`：标记是否已经发送了删除 `uevent` 事件。
- 理解：
	- 每个内核对象的基类是由 `kobject` 类构建而成的
		- 驱动设备的信息（如设备号、操作集等）并不会直接存储在 `kobject` 中，而是通过 `kobject` 进行组织和管理。
		- 具体来说，`kobject` 为设备对象（如驱动、设备）提供了基本的框架和功能，但设备的具体信息等通过会存在于其他的结构体中，并通过 `kobject` 进行关联和管理。
	- `sysfs` 是 Linux 内核提供的一个虚拟文件系统，用于暴露内核对象（如驱动设备，内核参数，驱动程序等）的属性，并允许用户空间通过 `sysfs` 提供的用户空间与内核对象交互的接口对内核对象结构体 `kobject` 的信息进行访问
	- 其中参数：
		- `sysfs_dirent`：`sysfs_dirent` 字段指向一个与 `sysfs` 相关的目录项，这使得 `kobject` 能够在 `sysfs` 中公开其属性
		- `state_in_sysfs`：该标记表示该 `kobject` 是否已经被添加到 `sysfs` 文件系统中。如果该标记为 1，意味着该对象的属性已暴露在 `sysfs` 中，用户可以通过文件系统查看和操作这些属性。


> [!NOTE] `device` 与 `kobject` 的关系：
> - `kobject` 作为设备对象的基础类，负责管理设备的生命周期和层次结构
> - 每个设备结构体（`struct device`）通常会在内部包含一个 `kobject` 的结构体 (通过嵌套或继承)，从而使设备具备 `kobject` 的基本功能 (如父子关系，引用计数等)。
> - 流程：
> 	- 设备的 `kobject` 可以通过 `kobject_init()` 函数进行初始化，并将该设备的相关信息 (如设备号，操作集等) 与具体的设备结构 (如 `struct device`) 关联起来

> [!Tip] `sysfs` 与设备驱动模型
> 特别注意，特别注意!!!，这两玩意不能结合看，否则特别容易晕，因为它们做的事情是一样的（向用户空间提供程序接口），但实现的方法与表现形式不一样（一个是操作集，一个是文件系统），且没有什么太大的关联关系
> 
> 1. 设备驱动模型：
> - 模型流程（以字符设备 `cdev` 为参考）：
> 	- 依据功能编写操作集 `fops` 结构体成员函数 `my_write()`、`my_open()` 、`my_read()`
> 	- 实例化字符设备对象 `struct cdev`
> 	- 通过 `cdev_init()` 将字符设备对象与操作集 `ops` 相关联
> 	- 通过 `alloc_chrdev_region()` 为字符设备对象分配主次设备号
> 	- 通过 `cdv_add()` 在内核中注册设备，并在 `/dev` 下生成设备文件
> 
> 2. `sysfs` 虚拟文件系统：
> - 该虚拟文件系统的实现主要基于 `struct device` 结构体与 `kobject` 结构体，其中 `kobject` 是 `device` 的一个结构体成员，其内包含有 `parent` 与 `sysfs_dirent` 等成员参数，在设备进行注册的时候会 `device` 结构体会进行 `kobject_init()` 初始化，依据 `kobject` 来进行设备之间的层次关联，同时将设备信息暴露给 `sysfs`
>   
> 3. `struct device` 与 `struct cdev` 之间的区别与联系：
> - `device` 与 `cdev` 是相互独立的结构体类型。但两者都是对设备信息的描述，每一个设备（字符设备、块设备、网络设备等）都会有其对应的一个 `struct device` 的实例
> - `device` 主要负责设备的管理、生命周期以及其他内核组件的交互
> - `cdev` 主要作用是为用户空间提供设备文件操作接口函数（`fops`）
> - 当 `cdev` 进行 `cdev_add()` 注册的时候，函数调用 `cdev` 实例对象、`dev` 设备号参数将 `cdev` 添加到内核中并在文件系统 `/dev` 路径下生成设备文件，用户可通过 `fops ` 提供的操作函数对设备文件进行查看修改操作
> - 而 `sysfs` 是当设备设备号进行注册时，`device` 结构体会对其内结构体成员 `kobject` 进行 `kobject_init()`, 基于 `kobject` 进行设备层次结构的梳理。同时在 `kobject` 进行初始化时，其内部成员 `sysfs_dirent` 会将该设备文件信息暴露给 `sysfs`，即设备结构体数据与文件系统的映射实现

2. `kobj_attribute` 结构体：
- `kobj_attribute` 是一个用于 `kobject` (内核对象) 创建属性的结构体。这些属性可以通过 `sysfs` 文件系统进行读取或写入
- 结构体原型：
```c
struct attribute{
	const char *name; 		// 属性的名称，在 sysfs 文件系统中为文件名
	umode_t mode;			// 属性权限模式。定义了用户对该数据的权限
}
```

#### 相关函数：

1. `kobject_create_and_add()`：

```c
// 函数用于创建、初始化并将object添加到系统中
struct kobject *kobject_create_and_add(const char *name, struct kobject *parent);
```
- 参数解释：
	- `name`：创建 `kobj` 的名字
	- `parent`：指定的父 `kobject`,
		- 实际上就是在哪个目录下创建一个目录。比如 `kernel_kobj`, 函数将在 `/sys/kernel` 目录下创建目录，如果是 `NULL`, 将在 `/sys` 下创建
- 返回值：
	- 成功时返回新创建并添加的 `kobject` 结构体指针
	- 失败时返回 `NULL`

2. `kobject_put()`：
```c
// 用于减少 kobject 的引用次数，当引用次数降为零时会释放对象
void kobject_put(struct kobject *kobj);
```
- 参数解释：
	- `kobj`：指向要减少引用计数的 `kobject` 结构体的指针
- 返回值：
	- 成功时返回 `kobj` 指针
	- 如果 `kobj` 为 `NULL` 则返回 `NULL`

3. `kobject_get()`：
```c
// 用于增加 kobject 的引用次数
void kobject_put(struct kobject *kobj);
```
- 参数解释：
	- `kobj`：指向要增加引用计数的 `kobject` 结构体的指针
- 返回值：
	- 成功时返回 `kobj` 指针
	- 如果 `kobj` 为 `NULL` 则返回 `NULL`

4. `sysfs_create_file()`:

```c
// 创建一个文件
int sysfs_create_file(struct kobject *kobj, const struct attribute *attr);
```
- 参数解释：
	- `kobj`：已创建的 `kobject` 结构体
	- `attr`：属性描述
- 返回值：
	- 成功时返回 0
	- 失败时返回错误码

5. `sysfs_show()`
```c
ssize_t (*show)(struct kobject *kobj, struct kobj_attribute *attr, char *buf);
```
- 指向显示方法的函数指针，当读取属性是调用该方法

6. `sysfs_store()`
```c
ssize_t (*store)(struct kobject *kobj, struct kobj_attribute *attr, char *buf);
```
- 指向存储方法的函数指针，当写入属性时调用该方法。