---
share_link: https://share.note.sx/ncgs0v0p#MiNVkJsZpZQZYwXluTTa7SYzSQAGZU883JAJI7W+sQM
share_updated: 2025-04-21T16:36:05+08:00
---
### 1. 基础概念：

- `Node-RED` 是一款基于 `Node.js` 的可视化低代码编程工具，主要用于连接硬件设备、`API` 和云服务，尤其适合物联网（`IOT`）、自动化和数据流处理等场合。
- `Node-Red` 提供基于 `Web` 网页的编程环境。通过拖拽定义`node`到工作区并用线连接`node`创建数据流来实现编程。

#### 1. 1 核心组件：

- **节点**（**`Nodes`**）：基本功能单元，分为输入（如传感器）、处理（如数据转换）和输出（如存储库存储）

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250401135604824.png" width = "700px"/>
</div>

- **流（Flows）**：通过拖拽节点并连线形成逻辑流程，代表数据流动路径

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250401135815408.png" width = "700px"/>
</div>

- **消息（Messages）**：节点间传递的数据，格式通常为 `JSON` 对象
	- `JSON` ：详见 [[Learning/out_class/IOT/网络层/JSON|JSON]]
		- `note` 笔记分享连接 [[note](https://share.note.sx/puq144ba#pTU87VIzX894kVhhfvys4j+vKKPXL0IdSXioYDHl8iU)]

> [!Tip] 
> 对于 `JSON` 部分的学习，了解熟悉其对应的语法规则即可。在实际项目中的使用
> - 如果是使用的华为云 IOT 等物联网云服务平台，需要去查找其平台所提供的官方文档给出的数据请求 `JSON` 格式标准来修改你的 `JSON` 对象。
> - 如果是通过 `Node-red` 自己搭建的物联网数据流转平台，可复可简，看自己发挥（我也没重头自己搭建过较为复杂的 `NR` 工程）

- **上下文（Context）**：存储临时或全局变量，支持跨流程数据共享
	- 这个较为复杂，放在后面

---
### 2. Node-red 安装部署：

1. 安装部署文章参考：（在这里感谢感谢感谢非常感谢!!!）
- [Linux 下安装 Node.js 超详细教程（小白友好版）_linux系统安装nodejsv18及以上,以及安装npm-CSDN博客](https://blog.csdn.net/i826056899/article/details/146500798)
- [Linux安装Node-RED并实现后台运行及开机启动-CSDN博客](https://blog.csdn.net/m0_46404447/article/details/140082721)
- [node-red-L1-如何设置让局域网可以访问？-CSDN博客](https://blog.csdn.net/weixin_45498306/article/details/142519841)

---

1. 环境要求：
- 说明：笔者是在 `VM` 中 `Ubuntu` 虚拟机环境进行安装部署的。（实际部署读者可以根据自己的需求来，也可以部署在 `Raspberry Pi`、`RDK` 等开发板中，`NR` 有提供相关 `GPIO` 插件允许用户通过 `NR` 控制 `GPIO`）
- `windows`、`Linux`、`MacOS` 都兼容，但最好不要按照官网的方法进行安装
	- 一方面 `Node-RED` 中文官网 [Node-RED : 安装](https://nodered.17coding.net/docs/getting-started/installation)感觉不知道多久没更新了（`node.js` 还在 `7.x` 版本......）
	- 其次 `Node-RED` 本身是基于 `node.js` 的，`node.js` 官网安装方法也不好装，需要魔法工具，但在虚拟机上配置魔法，笔者感觉好麻烦就没去做，但有查找过方法
		- 一个是通过魔法工具 `clash for window` 允许局域网代理来进行配置，但很麻烦
		- 一个是给你虚拟机装个魔法工具，但笔者 `VM` 经常爆掉，就懒得给它装了

---

3. 安装部署：
（1）使用 `NodeSource` 官方脚本安装 `node.js`：（适用于 `Ubuntu`、`Debian`、`CentOS` 等）

```bash
# 更新系统本地软件包列表
sudo apt update

# 安装 curl 工具
sudo apt install curl -y

# 运行 NodeSource 安装脚本，这里以 Node.js_20 版本为例
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# 安装 Node.js
sudo apt install -y nodejs

# 检验是否安装完成
node -v							# v20.11.1
npm -v							# 10.5.0
```

（2）通过 `Node.js` 包管理器 `npm` 安装 `node-red` (`npm` 是全球最大的开源库生态系统）

```bash
sudo npm install -g --unsafe-perm node-red # 指定安装版本可在后添加指定版本号

# 运行 node-red
node-red			
```

（3）配置 `node-red` 用户目录下的 `settings.js` 文件允许局域网进行访问 `node-red`
- `node-red` 用户目录一般位于 `$HOME/.node-red` （`$Home` 为主目录，即 `~` 目录）
	- 如果找不到 `.node-red` 目录，可以通过以下命令查找：

```bash
sudo find . -name ".node-red"
```

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250401165552444.png" width = "400px"/>
</div>

- 假设当前的 `.node-red` 路径为：`./.node-red`
- 通过以下命令进行 `settings.js` 文件修改：

```bash
sudo vim ./.node-red/settings.js

# 如果提示未安装 vim 工具的话，就输入下述命令下载 vim
# sudo apt-get install vim
```

- 进入到 `settings.js` 的编辑界面

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250401171011351.png" width = "600px"/>
</div>

按下 `ESC` 键，同时输入 `/uiHost` 可以找到 `uiHost` 属性进行配置

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250401171503858.png" width = "600px"/>
</div>

- 由注释说明，如果监听所有 `IPv6` 地址就将 `uiHost` 设置为 `"::"`，具体原理不用管，笔者不会，讲了读者也听不懂
- 再次按下 `ESC` 键（别管，笔者的习惯就是如此），按下 `i` 进入 `vim` 的 `input` 编辑模式，通过上下左右键移动光标到指定位置，`Backspace` 删除原设置，修改为 `"::"`
- 修改完成后，再次按下 `ESC` 键, 退出 `input` 输入编辑模式，再次输入 `:wq` 保存并退出 `vim` 文件编辑
- 重启 `Node-red`

（4）通过 `pm2` 进程管理工具实现 `NR` 系统开机自启动

```bash
sudo npm install -g pm2 		# 通过 npm 安装 pm2
which node-red					# 查找 node-red 命令文件路径
							   # 我的是在 /usr/bin/node-red

pm2 start path-NR -- -v			# 将 path-NR 替换成前面找到的文件路径
pm2 save						# 保存当前的 pm2 平台运行状态（node-red正在后台运行）
pm2 startup						# 配置启动脚本

# pm2 list						# 查看当前 pm2 控制的后台运行列表
# pm2 info node-red				# 查看 pm2 控制的 node-red 详细运行信息
# pm2 logs node-red 			# 查看当前 node-red 进程的日志文件
```

---
- 至此，`node-red` 的安装部署全部就完成了!!!，接下来就可以通过在相同局域网下的 `windows` 系统的浏览器输入 `http://IP_address: 1880` 就可以进入到 `NR` 了
	- 其中 `IP_address` 为部署了 `NR` 的设备的 `IP` 地址，`1880` 为`Node-RED`的默认端口号。
	- 你问我如何查找设备的 `IP` 地址??? 
		- `windows` 系统：
			- `win + R` 输入 `cmd` 进入系统命令窗口，输入 `ipconfig` 查找
		- `Linux` 系统：
			- `sudo apt-get install net-tools` 下载 `net-tools` 网络工具
			- 输入 `ifconfig` 查找
		- （图片上 `Linux` 下 `Window`）

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250401172916071.png" width = "600px"/>
</div>
<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250401173030833.png" width = "600px"/>
</div>

- 至此，就真的进来了!!!，完结撒花!

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250401173257808.png" width = "500px"/>
</div>


### 3.  Node-red 的学习：

- 快速上手视频教学：
	- [【1小时0基础速成NodeRed】物联网相关配置详解_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV15741177cS/?spm_id_from=333.337.search-card.all.click)
- 充满信心耐心视频教学：
	- [node-red教程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1pp4y1T7kh/?spm_id_from=333.337.search-card.all.click)

> [!Tip] 
> emm，笔者不是很想写这部分，因为一方面是操作上的东西，过一遍视频做一遍就大概懂得都懂。另一方面，`node-red` 作为系统链路层，涉及的面太广了，也不太好讲。文字固化反而显得有点繁文累赘。
> 但是在后面讲 MQTT 时，会以实例来讲解的，这里的大家就请先耐心观看视频。


### 4. 物联网 IOT 与 `Node-red` ：

##### 4.1 物联网 IOT：

> [!NOTE] 
> 每当提及物联网，基本上就离不开 **“云、网、边、端”** 这几个概念，在故事开始的开始，还是小小的老子的时候，第一次听到这个概念，还不懂分别代表着什么意思，就算是第一遍听懂了，后面也不太好意思去问别人，当时傻逼逼的，从而在当上物联网的队长之后，对于整个项目技术的构成并没有很好的把握，所以在这里笔者想再次这一体系进行重新梳理和详细阐释：

- 下面是 `ChatGPT` 给出的物联网框架的分析：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250401180903580.png" width = "450px"/>
</div>

- 在这里面，提及到 **“云、网、边、端”** 的概念分别是：
- **云：** 集中式数据中心和云计算平台，它提供强大的数据存储、计算及大数据分析能力
- **网：** 承载数据传输与互联的网络基础设施，支持各种协议和通信标准，实现设备、边缘与云端之间的高效连接
- **边：** 边缘计算节点，这些节点部署在离数据生成源更近的地方，可以在本地进行数据预处理、实时分析和决策，从而降低延迟和带宽消耗
- **端：** 物联网中的终端设备，如各种传感器、执行器和智能设备，它们负责直接采集环境数据和执行控制指令

在实际的物联网项目中：
- 云即**云服务**，在物联网项目中的体现主要为两个方面：**云数据库**、**云计算**
	- 云数据库一般用于存储和管理来自各类物联网设备（如传感器等）采集的数据。实现设备状态信息、用户交互数据、环境检测数据等信息的实时数据访问以及数据备份。
	- 云计算则为系统提供强大的计算资源，允许用户按需租用计算能力。​通过在云平台上部署大型模型，项目系统能够高效地处理和分析海量数据，支持智能决策和服务优化

- 网即**网络传输**，在物联网项目中，网络传输是实现各端（云端、边缘端、终端设备）之间**数据互通**的关键环节。
	- 在数据传输过程中，依赖于各种网络协议，例如 `MQTT`、`TCP`、`UDP` 等网络协议。设备和系统遵循特定的网络协议规则（例如 `TCP` 协议的三次握手建立连接、四次挥手断开连接；`MQTT` 的话题订阅通信模型），以实现数据的请求发送

- 端即**终端设备**，是**直接与物理环境或用户交互​**​的物联网节点，主要功能为数据感知与执行指令。
	- 终端设备一般通过以主控板连接**传感器与执行器**的形式，**实现环境数据感知**，依托本地有限算力完成**数据初步分析**并执行预警、控制指令等**临时决策任务**，以此快速响应环境变化，同时将处理结果及**原始数据同步上传**至上层系统。

- 边即**边缘计算设备**，边缘计算设备是部署在物联网架构边缘侧的智能硬件，​**​靠近数据源头​**​，主要承担一部分数据采集（例如摄像头）、预分析处理及部分本地计算任务，同时作为**网络节点连接云端与终端设备**。

##### 4.2 Node-red 在物联网中可发挥的作用：

- `Node-RED` 作为基于Node.js的可视化低代码编程工具，通过流程编排引擎实现不同设备间通信链路构建、数据预处理（格式转换/过滤/聚合）及规则触发的业务逻辑执行，其网络协议适配、轻量级数据处理与自动化响应能力使其天然适配边缘计算场景，可作为辅助模块集成至边缘设备，承担设备连接管理、实时数据流调度与简单决策执行等工作。

### 5. `Node-red` 实战小 demo：

- 基于 `MQTT` 协议的数据预警系统：
- 前置条件：
	- 学习了解 `MQTT` 的相关概念并完成 `MQTT` 服务器的搭建
	- `note` 笔记分享连接：[[MQTT|MQTT]]
	 https://share.note.sx/478o4lod#1GxZISrA+6CB2CNTlL4ZNh++BJNFPpinNPwNfuwEOQs
- 

