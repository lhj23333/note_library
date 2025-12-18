## 1. 概念

- `U-boot` 全称：`Univerasl Boot Loader`, 是一个遵循 `GPL` 协议的开源软件，是一个裸机代码
- 一般不会直接使用 `u-boot` 官方源码，而是适用于自己板载芯片厂家所维护的 `u-boot` 版本

| 种类                     | 描述                                                      |
| ---------------------- | ------------------------------------------------------- |
| `uboot` 官方 `u-boot` 代码 | 由 `uboot` 官方维护开发的 `uboot` 版本，版本更新快，基本包含所有常用的芯片          |
| 半导体厂商的 `u-boot` 代码     | 半导体厂商维护的 `uboot` 版本，专门针对自家的芯片，再对自家芯片的支持上要比 `uboot` 官方的好 |
| 开发板厂商的 `u-boot` 代码     | 开发板厂商再半导体厂商提供的 `uboot` 基础上加入对自家开发板的支持                   |
## 2. `Uboot` 
### 2.1 `Uboot` 目录结构：

- 开发板厂商 `Uboot` 代码目录：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250215114038378.png" width = "500 px"/>
</div>
- 其与官方 `uboot` 代码的区别之处 :
	- `board`：开发板代码，移植 `uboot` 时，需在其对应厂商位置下新建开发板 `bsp`
	- `configs`：存放不同开发板的 `uboot` 配置文件，命名规则同一为 `xxx_defconfig`, `xxx` 表示开发板名称

### 2.2 系统启动流程：

> [!NOTE]
> 1. **硬件引导**：CPU 从预定义地址开始执行 `bootROM`，`bootROM` 会加载第一阶段的引导程序，通常是 `Uboot` 的第一阶段（`Stage 1`）
> 2. **加载 Stage 1 到内存**：`Uboot` 的 `Stage 1` 会初始化最基本的硬件，如内存、串口、时钟等，然后查找存储设备（如 `SPI Flash`、`SD` 卡等），将 `Uboot` 的第二阶段加载到内存中
> 3. **执行 Stage 2**：`Stage 2` 负责执行更复杂的硬件初始化，设置环境变量，并准备好加载操作系统
> 4. **加载操作系统**：根据环境变量中的配置，`Uboot` 会选择合适的方式来加载操作系统的内核镜像。常见的方式：从本地存储设备（如 SD 卡，`eMMC`、`NAND` 等）或通过网络（`TFTP`）来加载内核
> 5. **移交控制权**：当操作系统的内核镜像加载到内存后，`Uboot` 会将控制权交给操作系统内核，启动操作系统

1. 硬件引导（`Boot ROM`）:
	- CPU 执行 `Boot ROM`：
		- 在系统上电时，CPU 会从预定义的地址开始执行 `Boot ROM`（或 `Bootloader`），用于初始化 CPU 和基本硬件环境 (如时钟、内存控制器等)
			- `Boot ROM`:
				- 
2. 加载第一阶段引导程序 （`Bootloader Stage 1`）：
	- 加载 `Uboot Stage 1`：
		- `bootROM` 会从预定的存储设备（如 `SPI Flash` 或 `NAND Flash`）中加载 `Uboot` 的第一阶段的引导程序。`Uboot` 会初始化基本硬件，如内存、串口、时钟等，确保系统能继续启动
3. 加载第二阶段引导程序 （`Bootloader Stage 2`）：
	- 加载 `Uboot Stage 2`：
		- `Uboot` 第二阶段会进一步初始化硬件，配置系统环境，并加载操作系统的内核。它还会根据预定义的环境变量和硬件状态，选择合适的存储设备来加载操作系统
4. 加载操作系统内核：
	- 选择内核源：
		- `Uboot` 根据环境变量来选择内核加载方式，通常情况下，内核镜像存储在本地存储设备 (如 `eMMC`、`SD卡` ) 上，或者从网络通过 `TFTP` 等协议加载
	- 加载内核镜像源：
		- `Uboot` 会将 Linux 内核（如 `vmlinuz`）加载到内存中，准备将控制权交给内核
5. 内核初始化：
	- 解压内核镜像：
		- 内核加载到内存后，内核会解压自己并初始化内核空间，设置系统所需的各种内核数据结构和设备驱动程序
	- 加载 `initramfs`：
		- 内核会从指定的存储位置加载 `initramfs` (初始 RAM 文件系统) 并将其解压到内存中，这通常是一个小型的文件系统，包含了系统启动所需要的最基本文件与驱动
6. 挂载 `initramfs`（init RAM file system）：
	- 挂载 `initramfs`：
		- 内核会将 `initramfs` 作为临时根文件系统挂载，并开始执行其内含的 `init` 脚本，进行一些初步的初始化工作，如挂载必要的文件系统、配置网络、启动基础服务等。
7. 设备初始化：
	- 硬件识别和驱动加载：
		- `initramfs` 内的 `init` 脚本会通过 `udev` 等机制，自动识别和加载硬件设备所需的驱动程序
	- 挂载其他文件系统：
		- `initramfs` 会挂载其他所需的文件系统
8. 切换到真正的根文件系统：
	- 解析 `/etc/fstab`：`initramfs` 中的 `init` 脚本会解析 `/etc/fstab` 配置文件，确定真正的根文件系统位置
	- 使用 `switch_root`：通过 `switch_root` 命令，`initramfs` 会切换到新的根文件系统
9. 启动用户空间：
	- 执行 `/sbin/init`：
		- 在新的根文件系统中，`/sbin/init` 作为第一个用户空间进程启动。
			- `init` 进程是所有用户空间进程的祖先，它会负责进一步的系统初始化工作
	- 启动服务：
		- `init` 会根据系统配置，启动各个系统服务，如网络服务、日志记录服务、用户会话等
10. 完全启动，进入用户模式：

## 2.3 `Uboot` 的配置与编译

`Uboot` 的配置通常由 `Kconfig` 文件控制，通过修改该文件可以定制不同的功能
- `Uboot` 使用 `make menuconfig` 等命令来进行配置。通过配置，用户可以启用或禁用不同的功能，例如支持某种硬件、文件系统或网络协议

编译 `Uboot` 时，可以选择目标硬件架构，并生成相应的引导镜像
- `Uboot` 的编译通常使用 GNU 工具链，运行 `make` 命令会生成一个适合目标设备的引导镜像

### 2.4 `Uboot` 编译烧录流程

可参考：
- [配置uboot和kernel选项参数 | RDK DOC](https://developer.d-robotics.cc/rdk_doc/Advanced_development/linux_development/driver_development_x5/uboot_kernel_config)
- [linux系统移植篇（二）—— Uboot使用介绍_uboot官网-CSDN博客](https://blog.csdn.net/sinat_31039061/article/details/124321470)

