#### 文章参考：

- 【正点原子】I.MX 6 U 嵌入式 Linux 驱动开发指南 V 1.81. pdf
- [U-boot顶层Makefile分析及编译流程_uboot顶层makefile详解-CSDN博客](https://blog.csdn.net/xi_xix_i/article/details/134918576)

#### 说明：

- 代码内容过大，这里就不放置在文章中，代码文件位置位于：`/uboot/Makefile`
	- `/uboot` 为下载 `Uboot` 源码位置
- `Uboot` 顶层 `Makefile` 主要功能通过定义变量与目标，协调版本管理、编译流程和编译配置选项，实现多架构，多功能的灵活构建
- 本文将从上到下顺序理解 `Uboot` 的顶层 `Makefile` ，除了讲解一些通用配置、编译管理以外，主要分析以下几个过程：
	- `make xxx_defconfig`：
	- `Uboot` 的 `make` 过程

### 通用 `Makefile` 配置：

##### 版本号：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226140346838.png" width = "200 px"/>
</div>

``` makefile
VERSION = 2016
PATCHLEVEL = 03
SUBLEVEL =
EXTRAVERSION =
NAME =
```
- `VERSION` 为主版本号、`PATCHLEVEL` 为补丁版本号、`SUBLEVEL` 为次版本号
- 以上三个信息构成 `Uboot` 的版本号：当前 `Uboot` 版本号为 2016.03
- `EXTRAVERSION` 为附加版本号信息，`NAME` 为版本名称
	- 一般不使用这两个信息，赋空值

##### `MAKEFLAGS` 变量：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226140454741.png" width = "600 px"/>
</div>

- `make` 支持递归编译，即在父级 `Makefile` 中可通过 `make` 命令来执行子目录中的 `Makefile` 文件。例如，当前目录下存在一个 `subdir` 子目录，且该目录下存在对应的 `Makefile` 文件，那么在父级 `Makefile` 文件中就可以通过下述命令来调用子目录 `Makefile` 的 `make` 指令完成子目录 `subdir` 的编译

``` Makefile
$(MAKE) -C subdir
make -C subdir

# -C 指定子目录
```
- 在进行编译子目录时，有时候父级 `Makefile` 需要向子级 `Makefile` 传递变量，一般可以通过 `export` 以及 `unexport` 来进行变量的传递
```makefile
export VARIABLE ......		# 导出变量给子 Makefile（用的比较多）
unexport VARIABLE ......	# 不导出变量给子 Makefile
```
- 在这其中存在有两个特殊变量：`SHELL` 和 `MAKEFLAGS`，这两个变量除非使用 `unexport` 进行声明剔除，否则在整个 `make` 的执行过程中，它们的值始终都回自动的传递给子 `Makefile`
- 在 `Uboot` 的主 `Makefile` 中有如下代码：
```makefile
MAKEFLAGS += -rR --include-dir=$(CURDIR)

# Avoid funny character set dependencies
unexport LC_ALL
LC_COLLATE=C
LC_NUMERIC=C
export LC_COLLATE LC_NUMERIC

# Avoid interference with shell env settings
unexport GREP_OPTIONS

```
- 其在 `MAKEFLAGS` 中追加了一些值
	- `-rR`：禁止使用内置的隐含规则和变量定义
	- `--include-dir`：指明搜索路径
	- `$(CURDIR)`：表示当前的目录
		- `CURDIR` 是 `GNU Make` 中的一个内置变量，它会自动设置为当前工作目录的绝对路径

##### 命令输出：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226140546903.png" width = "400 px"/>
</div>

```makefile
ifeq ("$(origin V)", "command line")
  KBUILD_VERBOSE = $(V)
endif
ifndef KBUILD_VERBOSE
  KBUILD_VERBOSE = 0
endif
```
- 在 `uboot` 编译输出部分，可以通过设置变量 `V = 1` 来实现完整命令输出
- 在代码中通过 `origin` 方法获取变量 `V` 的来源是否为用户输入 `command line` (命令行输入)，如果为用户输入则将其赋值给 `KBUILD_VERBOSE = $(V)`，如果没有输入，默认为 0（`ifndef KBUILD_VERBOSE`）

```
ifeq ($(KBUILD_VERBOSE),1)
  quiet =
  Q =
else
  quiet=quiet_
  Q = @
endif
```
- 后续依据 `KBUILD_VERBOSE` 的值来设置是否输出完整命令
- 当 `KBUILD_VERBOSE = 1` 时，`quiet` 与 `Q` 取空，使得命令完整输出
- 当 `KBUILD_VERBOSE = 0` 时，`quiet = quiet_`，`Q = @`，取消编译命令的打印以及设置编译输出内容为输出短版本

- 例如：
```makefile
$(Q)$(MAKE)$(build) = tools
```
- 当命令展开为 `make $(build) = tools` 时，`make` 在执行时会默认在终端输出命令，但当命令（终端命令: `make`、`cd`、`mkdir`.....）前面加有 `@` 标识符时，在执行命令时就不会在终端输出命令。

```makefile
quiet_cmd_sym ?= SYM 	$@
cmd_sym ?= $(OBJDUMP) -t $< > $@
```
- `sym` 命令分为 `quiet_cmd_sym` 与 `cmd_sym` 两个版本，两个版本命令执行的功能是一样的，区别在于执行时输出的命令不同。`quiet_cmd_xxx` 命令输出的信息较少，也就是短命令。`cmd_xxx` 命令输出信息多，也就是完整命令
	- `quiet = ` 时，输出完整命令
	- `quiet = quiet_` 时，输出短命令
	- `quiet = silent_` 时，不输出命令

##### 静默输出：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226140954123.png" width = "550 px"/>
</div>

- 在 `Uboot` 顶层 `Makefile` 中，设置了 `uboot make` 的静默输出功能（即 `quiet = silent_`），在编译时使用 `make -s` 即可实现静默输出。
- 在实际命令中，`-s` 并不是 `make` 编译的可选项，而是作为标识符来提供 `Makefile` 进行判断是否进行静默输出处理

---
- 其中 `filter` 函数规则如下：
```
$(filter <pattern...>, <text>)
```
- `filter` 函数表示以 `pattern` 模式过滤 `text` 字符串中的单词，仅保留符合模式 `pattern` 的单词，可以有多个模式。

---
-  `firstword` 函数规则如下：
```
$(firstword <text>)
```
- `firstword` 函数用于取出 `text` 字符串中的第一个单词，函数返回值为获取到的单词。
- 当使用 `make -s` 编译时，`-s` 会作为 `MAKEFLAGS` 变量的一部分传递给 `Makefile`

---
- 所以首先，该部分代码首先对 `make` 工具版本号 `MAKE_VERSION` 进行提取，判断其是否为 4.0 及以上版本，依据不同版本进行相应的 `-s` 标识符提取操作
```makefile
ifneq($(filter 4.%, $(MAKE_VERSION)), )
```
- 若为 `4.0` 及以上版本，通过 `filter` 函数对 `MAKEFLAGS` 变量第一个单词进行提取 `s` 标识符，若存在标识符，则设置 `quiet = silent_` 静默输出状态
- 若为 `4.0` 以下版本，同理，也是对 `MAKEFLAGS` 变量进行提取，由于 `-s` 标识符在不同版本下的 `make` 的 `MAKEFLAGS` 保存形式不同，所以在这里直接对整段变量进行提取（`s% -s%`）

##### 设置编译结构输出目录：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226151733739.png" width = "700 px"/>
</div>

- 在 `Uboot`中，与上述命令输出 `—V` 部分同理，可通过 `make O=output_dir` 设置目标文件输出路径，即将编译产生文件输出到 `output_dir` 目录下，使得其与源文件分离开来
- 在代码中，通过 `ifeq("$(origion 0)","command line")` 判断 `O` 变量是否来自于命令行，再通过 `ifneq($(KBUILD_OUTPUT), )` 判断输出目录是否存在。若路径不存在，则通过调用 `$(shell mkdir -p $(KBUILD_OUPUT) && cd $(KBUILD_OUTPUT) && (/bin/pwd))` 在当前路径下创建 `outpot_dir` 目录，并将输出目录的路径赋值给 `KBUILD_OUTPUT`, 以供后续编译输出时调用

---
```Makefile
PHONY += $(MAKECMDGOALS) sub-make

$(filter-out _all sub-make $(SURDIR)/Makefile, $(MAKECMDGOALS)) _all: sub_make
	@:

sub-make: FORCE
	$(@)$(MAKE) -C $(KBUILD_OUTPUT) KBUILD_SRC=$(CURDIR) -f $
```

##### 代码检查：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226145437606.png" width = "650 px"/>
</div>

- 在 `Uboot` 中，与上述指定输出路径 `O=` 获取同理，可以通过使用命令 `make C=？` 是能代码进行检查
	- `make C=1`：检查那些需要重新编译的文件
	- `make C=2`：检查所有猿哀鸣文件

##### 模块编译：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226145815682.png" width = "650 px"/>
</div>

- 在 `Uboot` 中，同上述代码检查 `make C=` 获取同理，可以通过 `make M=dir` 来进行单独模块的编译
- 在代码 186 行处，通过 `ifdef SUBDIRS` 来判断 `SUBDIRS` 是否定义，若有定义则赋值给 `KBUILD_EXTMOD` 变量，以支持老语法 `make SUBDIRS=dir`
- 后续通过 `ifeq("$(origin M)", "command line") KBUILD_EXTMOD := $(M)` 来获取用户输入的模块变量 `M`
- 在代码 197 行处，通过 `ifeq ($(KBUILD_EXTMOD), )` 判断 `KBUILD_EXTMOD` 是否为空，若为空则目标 `_all` 依赖 `all` 目标，会先编译 `all` 目标。否则目标 `_all` 依赖于 `modules`。会先进行编译模块
- 一般情况下我们不会在 `uboot` 中编译模块 

##### 获取主机架构与系统：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226145907208.png" width = "600 px"/>
</div>
- 在这部分代码中，`Uboot` 会通过定义变量来获取系统的主机架构信息 `HOSTARCH` 与系统信息 `HOSTOS`

---
```makefile
HOSTARCH := $(shell uname -m | \
    sed -e s/i.86/x86/ \
        -e s/sun4u/sparc64/ \
        -e s/arm.*/arm/ \
        -e s/sa110/arm/ \
        -e s/ppc64/powerpc/ \
        -e s/ppc/powerpc/ \
        -e s/macppc/powerpc/\
        -e s/sh.*/sh/)
```
- 该代码通过 `shell uname -m` 获取电脑主机架构信息，并通过管道 `|` 将输出信息传递给 `sed` 命令
- `sed -e` 为替换命令，"`sed -e s/i.86/x86/`"表示将管道输入的字符串中的 `i.86` 替换成 `x86`，其他 `sed -e` 命令同理

---
```makefile
HOSTOS := $(shell uname -s | tr '[:upper:]' '[:lower:]' | \
        sed -e 's/\(cygwin\).*/cygwin/')
```
- 该代码通过 `shell uname -s` 获取电脑主机系统信息，并通过 `tr '[:upper:]' '[:lower:]'` 命令将信息中的大写字母替换成小写字母（`tr '[:lower:]' '[:upper:]'` 是将小写替换成大写）
- 最后通过 `sed -e` 命令，将 `cygwin.*` 替换为 `cygwin`

##### 设置目标架构、交叉编译器和配置文件：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226150039993.png" width = "450 px"/>
</div>

- 在这部分代码主要负责目标板架构和交叉编译器的默认设置
- 在实际编译 `uboot` 时，需要通过 `make ARCH=arm CROSS_COMPILE=arm-linux-gnueabinf` 来设置目标板架构 `ARCH` 以及交叉编译器 `CROSS_COMPILE`
	- 但每次编译 `uboot` 都需要在 `make` 命令后设置 `ARCH` 与 `CROSS_COMPILE` 会比较麻烦，可以通过在 `enif` 后添加 `ARCH` 与 `CROSS_COMPILE` 设置或编写 `shell` 脚本来简便操作

---
- 添加内容：

```makefile
ARCH ?= arm
CROSS_COMPILE ?= arm-linux-gnueabinf-
```

---
- `shell` 脚本：

``` shell
#!/bin/bash

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabinf- distclean
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabinf- mx6ull_alientek_emmc_defconfig
make V=1 ARCH=arm CROSS_COMPILE=arm-linux-gnueabinf- -j16
```

##### 调用 `scripts/Kbuild.include`：
<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226150158617.png" width = "600 px"/>
</div>

- 在 `uboot` 编译过程中会调用到 `scripts/Kbuild.include` 中的变量，在这里通过 `include` 读取该文件内容

##### 交叉编译工具变量设置：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226150305403.png" width = "750 px"/>
</div>

- 设置 `CROSS_COMPILE` 交叉编译工具变量

##### 导出其他变量：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226164423649.png" width = "700 px"/>
</div>

- 在这里顶层 `Makefile` 会到处很多变量以传递供子 `Makefile` 调用，其中大部分在前面都有进行定义，但在其中也存在几个变量找不到定义：
```
ARCH CPU BOARD VENDOR SOC CPUDIR BOARDDIR
```
- 以上有几个变量事实上来自于 `uboot` 根目录下的 `config.mk` 文件，在其中有它们的定义
---
<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226165136004.png" width = "700 px"/>
</div>

- 在第 25 行处，定义变量 `ARCH` 为 `ARCH := $(CONFIG_SYS_ARCH:"%"=%)` ，即提取 `CONFIG_SYS_ARCH` 里面双引号 `“”` 之间的内容作为 `ARCH` 的值
- 其余变量同理
- 代码 46 行处，运用 `sinclude` 方法读取指定文件内容。该方法与 `include` 的功能相似，但 `sinclude` 读取文件如果不存在的话，不会出现报错

---
- 该代码中同时也出现了新的变量 `CONFIG_SYS_ARCH`、`CONFIG_SYS_CPU` 、`CONFIG_SYS_BOARD` 、`CONFIG_SYS_VENDOR` 和 `CONFIG_SYS_SOC`，这五个变量在 `uboot` 根目录下的 `.config` 文件中进行定义

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226170433856.png" width = "300 px"/>
</div>

### 重要过程：

##### `make xxx_defconfig` 过程：

- `make xxx_defconfig` 命令主要功能为配置 `uboot`
	- 在顶层 `Makefile` 中并无以   为目标的规则，但存在下段代码：

- 在代码 422 、423 行处定义了变量 `version_h` 、`timestamp_h` ,

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226170850005.png" width = "700 px"/>
</div>

##### `Uboot` 的 `make` 过程：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250226170957873.png" width = "700 px"/>
</div>