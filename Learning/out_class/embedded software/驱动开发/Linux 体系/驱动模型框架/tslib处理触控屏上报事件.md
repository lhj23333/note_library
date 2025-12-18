
- 在一套触摸屏显示系统中，通常由 LCD 屏和触摸屏两部分组成，其中 LCD 屏负责图形和像素的显示，而触摸屏用于感知用户的触控操作，实现交互功能。
	- `LCD` 屏在 `Framebuffer` 驱动模型中有进行讲解：[[FramBuffer 驱动框架|FramBuffer 驱动框架]]
	- 触摸屏则分为电阻屏、电容屏两种，它们的工作原理和应用场景有所不同。
	- 在触摸屏显示系统中，触摸屏的尺寸一般会设计的与显示屏的尺寸相同，覆盖在其之上

- 本文主要讲解电阻屏和电容屏的触控原理及其物理实现，并介绍如何通过 `tslib` 库处理触控屏产生的触控事件

### 0. 基础概念

1. 电阻屏：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250306205422.png" width = "450 px">
</di>

- 电阻屏主要由以下几层结构组成：
	- 上下层导电薄膜
	- 绝缘点（间隔层 ）：用于分隔上下两层，避免未触摸时接触
- 电阻屏的核心原理：压力感应
	- 当用手指或触控笔按压屏幕时，上下两层的导电薄膜接触，会形成一个闭合回路
	- 通过测量接触点的电压值，可以计算出触摸点的 `X、Y` 坐标
	- 控制芯片解析坐标值数据，并将其传输给操作系统

- 例如，我们可以将电阻屏简单看成是一个滑动变阻，当我们用手指按压屏幕时，上下层导电薄膜接触，形成闭合回路，此时控制芯片会通过检测触摸点出电压 $V_{o}$ 来计算触摸点坐标，以单坐标方向为例：
$$
x = L * \frac{V_{o}}{V}
$$
- 其中：
	- $x$：触摸点坐标位置
	- $L$：屏幕长度
	- $V_{o}$：触摸点处电压值
	- $V$：该闭合回路总电压值

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250306203300.png" width = "300 px"/>
</div>

- 在调试触摸屏时，通常需要进行触控校准。校准过程基于上述原理，通过测量当前触摸点的电压与实际位置，并计算其线性关系，从而实现精准校准。
$$
\frac{V}{x_{o}} = \frac{V}{L}
$$

---
2. 电容屏：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250306212426.png" width = "450 px">
</di>

- 电容屏主要由以下几层结构组成：
	- 保护玻璃层
	- `ITO` 透明导电层
	- 驱动电极层
- 电容屏的核心原理：人体电流感应
	- 电容屏表面覆盖着 `ITO` 层，会形成一个固定电场
	- 当手指触摸屏幕时，手指会与屏幕形成一个微小的电容，使得触摸点的电场发生变化
	- 触控 `IC` 通过检测电流变化，计算手指的 `X、Y` 坐标

> [!Tip] 电容屏原理：
> 这部分实际比较复杂，但也不需要学的比较深，了解一下即可。emm，在韦老师课里面是这么描述的：
> 电容屏中有一个控制芯片，会周期性产生控制信号，驱动电极接收到电信号，会测量电荷大小。当电容屏被按下时，相当于接入了新的电容，进而影响接收电极收到的电荷大小。触控 `IC` 会根据电荷大小计算触点位置

- 在实际开发中，使用电容触控屏时，只需通过 `IIC` 读取触控屏 IC 解析出的位置信息。
- 在适配过程中，需要根据 `LCD` 屏的尺寸调整触控屏 IC 的相关参数，以确保触控坐标与显示区域匹配。
---
- 相较于电阻屏，电容屏支持多点触控、触摸体验较好，但成本较高


### 1.  触控事件上报

- 在 Linux 系统中，电容屏和电阻屏的触控事件通过**输入子系统（Input Subsystem）** 来进行管理，并最终通过 `edev` 事件设备 (如 `/dev/input/eventX`) 上报给用户空间。

#### 电阻屏触控事件上报

- 电阻屏是压力感应的，其事件上报的主要时机为：
	- 触摸按下（`Press`）：手指施加压力，屏幕两层导电膜接触，形成闭合回路
		- 事件类型（`Type`）：`EV_KEY`
		- 具体事件（`Code`）：`BTN_TOUCH`
		- 参数值（`Value`）：1（按下）
	- 移动（`Move`）：手指在屏幕上滑动时，触点坐标变化，持续上报位置数据
		- 事件类型（`Type`）：`EV_ABS`
		- 具体事件（`Code`）：`ABS_X`、`ABS_Y`
		- 参数值（`Value`）：`xxx`（绝对位移值）
	- 抬起 (`Release`)：手指抬起，屏幕两层导电膜分离，系统检测无触点
		- 事件类型（`Type`）：`EV_KEY`
		- 具体事件（`Code`）：`BTN_TOUCH`
		- 参数值（`Value`）：0 (抬起)
	- 同时在每一次事件上报结束后会进行事件同步
		- 事件类型（`Type`）：`EV_SYNC`
		- 具体事件（`Code`）：0
		- 参数值（`Value`）：0
- Linux 驱动程序中，会上报触点的 `X`、`Y` 数据，
	- 注意：这并不是 `LCD` 屏的实际坐标值，需通过应用程序进行处理才能转换为 `LCD` 坐标值
- 在其事件设备 `edev` 中的 `input_event` 结构体中，其事件上报属性值如下

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250306221158.png" width = "600 px"/>
</div>

#### 电容屏触控事件上报

- 电容屏的触控事件上报的方式分为两种：
	- `Type-A`：
		- 该类型触摸屏无法分别具体触点，通过将所有触电触控数据串行一股脑式的进行上报，由软件来分辨这些数据属于哪个触点
	- `Type-B`：
		- 该类型触摸屏可以分辨具体触点，在上报数据时，会先上报触点 `ID`，再上报其数据
	- 在实际中，`Type-A` 类型的触摸屏一般不怎么使用，`Type-B` 类型触摸屏的应用较广，下面我们所提及到的电容屏都默认为 `Type-B` 类型触摸屏
---
- 电容屏通常通过 `MT`（`Multi-Touch`，多点触控）方式上报
- 电容屏是电场感应的，其事件上报的主要时机为：（下面以存在两个触点情况进行分析）
- 手指靠近屏幕时即可触发触摸事件（不需要施加压力），其事件上报数据为：
	- 创建触点对象：
		- 事件类型（`Type`）：`EV_ABS`
		- 具体事件（`Code`）：`ABS_MT_SLOT`
		- 参数值（`Value`）：`n` （触点对象编号）
	- 分配触点 `ID`：
		- 事件类型（`Type`）：`EV_ABS`
		- 具体事件（`Code`）：`ABS_MT_TRACKING_ID`
		- 参数值（`Value`）：`x` （触点 `ID`）
	- 触点信息：
		- 事件类型（`Type`）：`EV_ABS`
		- 具体事件（`Code`）：`ABS_MT_POSITION_X` `ABS_MT_POSITION_Y`
		- 参数值（`Value`）：`x[n]` `y[n]`

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250306223908.png" width = "650 px"/>
</div>

- 某一触点正在移动时，其事件上报数据为
	- 上报触点对象：
		- 事件类型（`Type`）：`EV_ABS`
		- 具体事件（`Code`）：`ABS_MT_SLOT`
		- 参数值（`Value`）：`n` （触点对象编号）
	- 触点信息：
		- 事件类型（`Type`）：`EV_ABS`
		- 具体事件（`Code`）：`ABS_MT_POSITION_X` `ABS_MT_POSITION_Y`
		- 参数值（`Value`）：`x[n]` `y[n]`

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250306224357.png" width = "700 px"/>
</div>

- 抬起释放某一触点：
	- 上报触点对象：
		- 事件类型（`Type`）：`EV_ABS`
		- 具体事件（`Code`）：`ABS_MT_SLOT`
		- 参数值（`Value`）：`n` （触点对象编号）
	- 释放触点 `ID`：
		- 事件类型（`Type`）：`EV_ABS`
		- 具体事件（`Code`）：`ABS_MT_TRACKING_ID`
		- 参数值（`Value`）：-1

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250306224603.png" width = "700 px"/>
</div>

- 同样在每次事件上报完毕后，都会进行事件同步
	- 事件同步：（报告数据上报完毕）
		- 事件类型（`Type`）：`EV_SYNC`
		- 具体事件（`Code`）：`SYN_REPORT`
		- 参数值（`Value`）：

---
- 同时在电容屏上报的事件数据中，为兼容老程序，也会进行上报 `ABS_X`、`ABS_Y` 数据
	- 电阻触控屏就是使用这类型的数据，所以基于电阻屏的应用程序，也能用在电容屏上
		- 事件数据会上报的对象应用程序。
		- 电容屏上报的数据中含有适用于电容屏应用程序的事件数据，所以基于电阻屏的应用程序能用在电容屏上

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250306225855.png" width = "700 px"/>
</div>


