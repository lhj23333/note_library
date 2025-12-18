
`uboot` 也支持 `TAB` 键自动补全命令

### 1. 信息查询命令：

- `bdinfo`：用于查看 `board` 信息
```shell
bdinfo
```

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250225144335.png" width = "300 px"/>
</div>

- `printenv`：用于查看 `Uboot` 环境变量
```
printenv
```

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250225154529.png" width = "600 px"/>
</div>

- `version`：用于查看 `Uboot` 版本号
```
version
```

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250225154738.png" width = "500 px"/>
</div>

### 2. 环境变量操作命令

- `setenv`：修改（新建）`DRAM` 中的环境变量值：
```shell
setenv name_env xxx
```
- 参数解释：
	- `name_env`：环境变量名称
	- `xxx`：修改值

```shell
setenv bootdelay 5		# 修改 uboot 启动延时
setenv mybootemmc 'fatload mmc 1:1 80800000 zImage; fatload mmc 1:1 83000000 imx6ull- 14x14-emmc-7-1024x600-c.dtb;bootz 80800000 - 83000000'	# EMMC 启动方式
setenv mybootemmc 		# 置空，删除环境变量

saveenv
```

- `saveenv`：保存修改后的环境变量
```
saveenv
```

### 3. 内存操作命令



### 4. 网络操作命令


### 5. Boot 操作命令

- `bootz`：用于启动 Linux 内核镜像以及加载设备树文件
```
bootz [addr[initrd[:size]] [fdt]]
```
- 参数解释：
	- `addr`：Linux 内核镜像在 `DRAM` 中的首地址
	- `initrd`：`initrd` 文件在 `DRAM` 中的首地址，不使用用“-”来代替即可
	- `fdt`：设备树文件在 `DRAM` 中的首地址
- 说明：
	- 可以通过 `nfs` (网络文件系统) 或 `tftp` 命令将内核启动所需的镜像以及设备树文件，加载到开发板指定地址处
	- 如果内核启动所需镜像以及设备树文件存放于 `EMMC` 中，可通过 `fatload` 命令，将文件加载到 `DRAM` 中

```shell
fatload mmc 1:1 80800000 zImage
fatload mmc 1:1 83000000 imx6ull-14x14-emmc-7-1024x600-c.dtb
bootz 80800000 - 83000000
```

- `bootm`：用于启动 `uImage` 镜像文件
```
bootm [addr[initrd[:size]] [fdt]]
```
- 参数解释：
	- `addr`：`uImage` 镜像文件在 `DRAM` 中的首地址
	- `initrd`：`initrd` 文件在 `DRAM` 中的首地址，不使用用“-”来代替即可
	- `fdt`：设备树文件在 `DRAM` 中的首地址


- `boot`：读取环境变量 `bootcmd` 来启动 Linux 系统
```
boot
```
- 说明：
	- `bootcmd`：即该环境变量中存放 `boot`（引导）和 `cmd` (命令) ，就是引导命令的集合
	- `uboot` 倒计时 `bootdelay` 结束后就会启动 Linux 系统，就算执行 `bootcmd` 中的启动命令
	- 可通过修改 `bootcmd` 内容来实现 `boot` 启动所需的文件配置
```shell
setenv bootcmd 'fatload mmc 1:1 80800000 zImage; fatload mmc 1:1 83000000 imx6ull-14x14- emmc-7-1024x600-c.dtb; bootz 80800000 - 83000000'	# 加载内核镜像以及设备树
savenev		# 保存环境变量
boot		# 启动 Linux 系统
```

### 6. 其他命令：

- `reset`：复位重启 `Uboot`
```
reset
```

- `go`：跳到指定地址处执行应用
```
go addr [arg...]
```
- 说明：
	- 我们可以将裸机应用 `xxx.bin` 通过 `tftp` 或其他方式加载到开发板 `DRAM` 直接中，通过 `go` 命令启动裸机应用
```
tftp 87800000 printf.bin
go 87800000
```


- `run`：运行环境变量中定义的命令
```
run xxxx
```
- 说明：
	- `run` 更多的是运行我们自己定义的环境变量
		- 例如：在后面调试 Linux 系统的时候常常要在网络启动和 `EMMC/NAND` 启动之间来回切换，而 `bootcmd` 只能保存一种启动方式，我们就可以通过自定义变量的方式来实现不同的启动方式
```shell
setenv mybootemmc 'fatload mmc 1:1 80800000 zImage; fatload mmc 1:1 83000000 imx6ull- 14x14-emmc-7-1024x600-c.dtb;bootz 80800000 - 83000000'	# EMMC 启动方式

setenv mybootnand 'nand read 80800000 4000000 800000;nand read 83000000 6000000 100000;bootz 80800000 - 83000000'	# NAND 启动方式

setenv mybootnet 'tftp 80800000 zImage; tftp 83000000imx6ull-14x14-emmc-7-1024x600-c.dtb; bootz 80800000 - 83000000' 	# 网络启动方式

saveenv

run mybootemmc	# 以 EMMC 方式启动
run mybootnand	# 以 NAND 方式启动
run mybootnet	# 网络启动
```
