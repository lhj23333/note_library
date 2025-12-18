
## 0. 学习资料（感谢感谢!!!）

- [【ESP32教程】第二章: 低功耗蓝牙BLE相关概念及用法_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ad4y1d7AM?buvid=Y546F35A18C2B25141C2AE0B6966431E7B80&from_spmid=search.search-result.0.0&is_story_h5=false&mid=9zSgGOW3ZIOVICV5QXCJgA%3D%3D&plat_id=116&share_from=ugc&share_medium=iphone&share_plat=ios&share_session_id=9A164FC9-AAAC-4646-92F3-7324DD3BB1E3&share_source=WEIXIN&share_tag=s_i&spmid=united.player-video-detail.0.0&timestamp=1751722468&unique_k=JsbyokM&up_id=604244921&vd_source=e71be9ebbb70e681992d6bfd2b2fb088)
- [ESP32蓝牙教程一: 开篇介绍_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1fs4y1G7eu?buvid=Y546F35A18C2B25141C2AE0B6966431E7B80&from_spmid=main.space-contribution.0.0&is_story_h5=false&mid=9zSgGOW3ZIOVICV5QXCJgA%3D%3D&plat_id=116&share_from=ugc&share_medium=iphone&share_plat=ios&share_session_id=A5EF64EA-7DD0-4B46-8B91-22E91CFAEF6C&share_source=WEIXIN&share_tag=s_i&spmid=united.player-video-detail.0.0&timestamp=1751722567&unique_k=2pMGhlL&up_id=1338335828&vd_source=e71be9ebbb70e681992d6bfd2b2fb088)

## 1.  基础概念

#### 1.1 蓝牙广播

##### 1.1.0 基础概念

- 低通道蓝牙拥有 40 个信道（0~39），频段范围为：2402 MHz 到 2480 MHz
- 其中 37、38、39 为广播信道，其余信道为数据信号
	- 广播信道不相邻原因：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250705223535552.png" 
  height="300 px"/>
</div>

##### 1.1.1 广播数据包

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250705224034406.png" 
  height="300 px"/>
</div>

- 广播数据包由 37 字节数据组成，其中前 6 位字节为 `MAC` 地址，其余 31 位字节可供用户使用
- 其中用户使用数据包部分，由一个个广播数据结构（`AD Structure`）组成（如若 `AD Structure` 长度总和不足 31 位字节，其余位由全 `0` 字节补齐）
---
- `AD Structure` 解析：
- 1 个 `AD Structure` 由长度（ `Length`） 、类型 （`Type` ）和数据（`Data`）三部分顺序组成
- 长度（`Length`）:
	- 长度（`Length`）长度 = 类型（`Type`）长度 + 数据（`Data`）长度
		- 长度：1 位
		- 类型：1 位
		- 数据：用户自定义
- 类型（`Type`）：
	- 定义数据的含义

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250705230021795.png" 
  height="175 px"/>
</div>

- 数据（`Data`）:
	- 用户依据类型 `Type` 进行定义
		- 其中中文采用 `UTF-8` 编码，一个 `UTF-8` 编码占据 3 个字节


##### 1.1.2 扫描响应

- 蓝牙广播类型

|  广播类型   |          作用          | 广播内容  |  可被连接  | 扫描响应 |
| :-----: | :------------------: | :---: | :----: | :--: |
| 可连接非定向  |       最常用的广播方式       | 正常广播包 |   是    |  支持  |
|  可连接定向  |   主要用于已配对设备中的快速连接    | 定向广播包 | 指定设备连接 | 不支持  |
| 不可连接非定向 |  蓝牙信标、蓝牙传感器所使用广播方式   | 正常广播包 |   否    | 不支持  |
|  可扫描定向  | 在不可连接非定向基础之上添加扫描响应功能 | 正常广播包 |   否    | 不支持  |

- 广播与扫描响应：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250705231534438.png" 
  height="300 px"/>
</div>

- 广播（`adveertises`）：
	- 蓝牙从机（设备）主动广播其数据包信息
- 扫描响应（`scan response`）：
	- 蓝牙主机（`Host`）向从机发送扫描请求（`scan request`），从机接受请求后再将数据包信息作为扫描响应（`scan response`）发送给主机

---
- 当从机设备所需发送的数据包大小超过用户数据包限制（31 字节）时，可通过在广播数据（`advertise data`）之后附加扫描响应数据（`respon data`）进行补充。（扫描响应数据是非必需的）
- 扫描响应数据的格式与广播数据相同，且仅在接收到相应请求后才会发送。

```python
## 广播数据发布（MicroPython）
BLE.gap_advertise(interval_us, adv_data=None, *, resp_data=None, connetable=True)

# adv_data : 广播数据
# resp_data : 扫描响应数据
```


#### 1.2 蓝牙状态

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250706222119113.png" 
  height="300 px"/>
</div>

- 蓝牙设备运行通常可分为 5 种主要状态：就绪态（`Standby`）、广播态（`Advertising`）、扫描态（`Scanning`）、发起态（`Initiating`）和连接态（`Connected`）
---
- **就绪态**（`Standby State`）：
	- 设备处于**未参与任何蓝牙通信**时的待命状态。此时无线电功能关闭，不进行广播、扫描或连接等操作。蓝牙处于**最低功耗状态**

- **广播态**（`Advertising State`）：
	- 设备**主动发送广播数据包**，同时**公开自身信息以供其他设备发现**，并**可在接收到扫描请求后发送扫描响应，等待连接请求的到来**

- **扫描态**（`Scanning State`）：
	- 设备**开启接收功能**，**监听信道上其他设备发出的广播包**，并**可根据需要发送扫描请求以获取更多信息**。用于发现和识别周边设备

- **发起态**（`Initiating State`）：
	- 设备在**接收到目标设备广播后主动发送连接请求**，尝试与其建立连接（是建立蓝牙链路的起始阶段）

- **连接态**：
	- 设备**已成功与目标设备建立连接，双方可进行稳定的数据交互**，直到连接断开或主动退出


---
**对比表：**

| 状态  |         主要作用         | 是否发送广播 | 是否接收广播  | 是否能发起连接 | 是否能接收连接 |
| :-: | :------------------: | :----: | :-----: | :-----: | :-----: |
| 就绪态 |         空闲待命         |   否    |    否    |    否    |    否    |
| 广播态 | 主动发送广播数据包，并可被发现并建立连接 |   是    |    否    |  否（一般）  |    是    |
| 扫描态 |   接收广播数据并可发送扫描响应请求   |   否    |    是    |    否    |    否    |
| 发起态 |  接收目标设备广播后主动发送连接请求   |   否    | 是（监听广播） |    是    |    否    |
| 连接态 |     连接建立，通信数据交互      |   否    |    否    |    否    |    否    |

### 1.3 服务与特性

> [!NOTE] `UUID`
> 通用唯一标识码 (`University Unique Identifier`, "`UUID`") 是一个 128 位（16 字节）的标识符，用于蓝牙通信中唯一标识一个服务或特征（`Characteristic`）。每一个蓝牙服务或特征都通过 `UUID` 来进行标识，以确保在不同设备或平台间不会冲突。
> 
> 蓝牙 `UUID` 可分为三类长度
> - 128 位 `UUID`：标准 `UUID` 长度，如 `0000180F-0000-1000-8000-00805F9B34FB`, 用于自定义服务，且其中 `0000xxxx-0000-1000-8000-00805F9B34FB` 部分称为 `UUID` 基地址
> - 16 位 `UUID`：与 128 位 `UUID` 相同，用于标准蓝牙服务。为缩减版 `UUID`，即允许用户使用 16 为 `UUID` 与 `UUID` 基地址进行拼接，以创建蓝牙服务或特性
> - 32 位 `UUID`：较少使用
>   
>   `UUID` 作用：
>   - 标识服务（`Service`）:
> 	  - 在 `GATT`（通用属性配置文件）架构中，每一个服务都有一个唯一的 `UUID`
>   - 标识特征值（`Characteristic`）：
> 	  - 每个服务内部可包含多个特征，每个特征都有自己的 `UUID`
>   - 保证唯一性和互操作性：
> 	  - 使用 `UUID` 可以确保不同厂商定义的服务不会发生冲突，以便于标准化设备间的兼容通信
>   - 实现自定义功能：
> 	  - 开发者可通过自定义 128 位 `UUID` 来实现专属服务，以满足特定应用需求

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250706223241977.png" 
  height="350 px"/>
</div>

##### 1.3.1 应用层：

- 蓝牙应用层的开发主要基于 `GATT`（`Generic Attribute Profile`）架构，它定义了设备如何通过服务和特征来组织、传输和访问数据。
- 在开发蓝牙应用程序时，通过围绕服务、特性、权限等概念进行功能设计

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250706232358856.png" 
  height="450 px"/>
</div>

- 服务（`Service`）:

- 特征（`Characteristic`）：

- 描述符（`Descriptor`）：

```python
import ubluetooth
for ubluetooth import BLE

# 定义服务特性 UUID
SERVER_1_UUID = ubletooth.UUID(0x9011)
CHAR_A_UUID = ubletooth.UUID(0x9012)
CHAR_B_UUID = ubletooth.UUID(0x9013)

# 设定特性读写权限
CHAR_A = (CHAR_A_UUID, ubluetooth.FLAG_READ | ubluetooth.FLAG_NOTIFY, )
CHAR_B = (CHAR_B_UUID, ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE, )

# 设定服务内容
SERVER_1 = (SERVER_1_UUID, (CHAR_A, CHAR_B, ), )

# 设定设备服务集合
SERVICES = (SERVER_1, )

ble = BLE()	# 创建蓝牙对象
ble.active(True) # 开启蓝牙
((char_a, char_b), ) = ble.gatts_register_services(SERVICES) # 注册服务到 GGATTS

# 开始广播
ble.gap_advertise(100, adv_data = b '\x02\x01\x05\x05\x09\x42\x69\x62\x69')

```

##### 1.3.2 协议层：

##### 1.3.3 物理层：


