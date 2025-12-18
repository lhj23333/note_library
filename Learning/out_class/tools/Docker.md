---
share_link: https://share.note.sx/ys9kaaa7#vAe6EUv4x4OGbmc49nKRrHkacJg37IWSol28Lcm0lPY
share_updated: 2025-01-14T18:47:47+08:00
---
### 1. Docker 概述

> [!NOTE] Docker 的一些术语流程理解：
> 
> - iso (镜像) = jar (项目工程文件) + environment (环境)
> - java 应用开发流程：
> 	- java  ---  .apk  ---  发布 (应用商店)  ---  用户使用（安装. apk 即可）
> - docker 应用开发流程：（开发 + 运维）
> 	- java  ---  .jar  + environment  ---  打包成 iso --- Docker 仓库  --- 下载镜像直接运行即可
> 
#### 1.1 Docker 核心思想：

1. **工程环境打包** ：
   Docker 的核心在于通过容器技术将应用程序及其所有依赖项（包括库、环境变量、配置文件等）打包成一个可移植的镜像。镜像是轻量级的、独立的，确保应用程序在任何支持 Docker 的环境中都能够一致运行。
   
2. 环境隔离：
   Docker 使用容器提供了强大的环境隔离能力。每个容器都有自己的文件系统、网络栈和进程空间，从而确保不同的应用程序在各自的容器中运行而互不干扰。
    - **防止冲突**：解决传统环境下的端口冲突、依赖版本不兼容等问题。
    - **安全性**：容器化运行的应用与主机系统及其他容器之间相对独立，提高安全性。

3. 资源利用最大化：
   Docker 容器的启动速度快、占用资源少，并且支持按需分配计算资源（如 CPU、内存和存储）。这使得多个容器可以在同一主机上高效运行，从而充分利用服务器资源，避免资源浪费。


#### 1.2 Docker 资料网址：

- 官网地址：[Docker: Accelerated Container Application Development](https://www.docker.com/)
- 文档地址：[Docker Docs](https://docs.docker.com/)  巨详细!！
- 仓库地址：[The World’s Largest Container Registry | Docker](https://www.docker.com/products/docker-hub/)


#### 1.3 Docker 功能帮助：

- DevOps：开发 (develop) + 运维 (operation)
	- Docker：通过容器化技术实现了应用的打包、分发和部署。一旦开发完成的应用被打包为镜像，可以一键运行，不再需要复杂的环境配置过程，简化了从开发到生产的交付流程。
	- 传统：在传统流程中，开发和运维需要维护大量帮助文档，说明如何安装依赖、配置环境等。环境差异往往导致不可预期的问题，而 Docker 的镜像可以彻底解决这些问题。（谁配谁开心）
- 更便捷的升级和扩缩容：
	- Docker 容器的启动速度极快，通常在秒级，因此可以快速响应流量高峰。
	- 使用容器编排工具（如 Kubernetes 或 Docker Swarm），可以轻松实现应用的动态扩展或缩减，提升资源利用率的同时降低成本。
    - 升级时，通过滚动更新的方式替换旧容器，避免服务中断。
- 更简单的系统运维：
	- **开发环境高度一致**
		- Docker 提供了一致的运行环境，开发者可以直接在本地构建和测试容器镜像，然后将其部署到生产环境，无需担心环境差异导致的运行问题。
	- 容器管理便捷
		- 运维人员可以通过简单的命令对容器进行启动、停止、重启等操作，极大地减少了系统管理的复杂性。
- 更高效的计算资源利用：
	- **内核级别的虚拟化**：
		- Docker 使用操作系统内核的 cgroups 和 namespaces 技术对资源进行隔离和管理，不需要像虚拟机那样为每个实例运行完整的操作系统，因此占用资源更少
	- **轻量级容器**：  
		- 单台物理服务器可以运行数百甚至上千个 Docker 容器，大幅提升硬件利用率。 
	- **按需分配资源**：
		- Docker 支持为容器设定 CPU、内存等资源配额，确保不同应用的资源需求得到满足


### 2. Docker Engine安装：

#### 2.1  一些引入概念理解：

- Docker 架构图：

<div style="text-align: center;" >
	<img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250114111203.png" />
</div>

- 镜像 (image)：
	- 镜像是 Docker 的核心组件之一，类似于 Java 中的类 (class)，是一个轻量级、独立的、可运行的模板。它包含了运行某个应用程序所需的文件系统及其依赖。
	- **特点**：
	    - **只读性**：镜像本身是静态的，只包含应用程序及其运行环境。
	    - **分层存储**：镜像是由多个层 (layer) 组成，每一层对应一个修改步骤，支持高效存储与共享。
	- **作用**：通过镜像可以创建多个容器实例，实现应用环境的快速复用。

- 容器 (container)：
	- 容器是镜像的运行实例，可以理解为一个简化版的独立 Linux 系统，它包含了应用程序及其运行所需的所有文件和环境。
	- **特点**：
	    - **轻量级**：相比虚拟机，容器共享宿主机的操作系统内核，启动速度快，占用资源少。
	    - **可管理性**：通过 Docker 提供的命令，可以对容器进行启动、停止、暂停、删除等操作。
	    - **隔离性**：容器之间相互独立，且与宿主机隔离，保证了应用的运行安全性和稳定性。
	- **作用**：用于运行实际的应用服务，一个镜像可以启动多个容器，每个容器是一个独立的服务实例。

- 仓库 (repository)：
	- 仓库是存放镜像的地方，可以理解为镜像的集中管理中心。用户可以从仓库中拉取所需镜像，也可以将本地镜像推送到仓库中以便共享和备份。
	- **分类**：
	    - **公共仓库**：如 Docker Hub（默认官方仓库）、阿里云镜像仓库、清华大学开源镜像站等。
	    - **私有仓库**：企业或个人可以搭建私有仓库，用于存储自定义镜像。
	- **镜像加速**：由于网络原因，国内用户需要配置镜像加速器以提高镜像拉取速度。常见的镜像加速服务包括阿里云、腾讯云等。

#### 2.2 安装命令：

- 详细文档内容(Debian 系统)： [Debian | Docker Docs](https://docs.docker.com/desktop/setup/install/linux/debian/)

> [!Tip] 
> 这里以 Debian 系统版本为例，如需在别的系统版本上安装 Docker 请查看官方文档版本教程
> - [Install | Docker Docs](https://docs.docker.com/engine/install/)
>   
![image.png](https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250114121113.png)

##### 2.2.1 卸载冲突软件包：

- Linux发行版可能提供非官方的Docker软件包，这可能与Docker提供的官方软件包冲突。在安装正式版Docker Engine之前，必须先卸载这些软件包。
	- 需要卸载非官方软件包
		- `docker.io`
		- `docker-compose`
		- `docker-doc`
		- `podman-docker`
- 运行以下命令卸载卸载所有冲突的包：
``` shell
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

##### 2.2.2 安装 Docker：

- 在新主机上首次安装 Docker Engine 之前，需要设置 Docker apt 存储库。后续可以从存储库安装和更新 Docker。

详细步骤：

> [!Tip] 注意 ！！
> 以下讲解是以官方文档版本进行讲解，实际使用会受限于国外源连接以及网速问题，推荐使用国内源（例如清华源、阿里源），以下为各个国内源详细安装 Docker 教程步骤网址，也是依据官方文档内容修改而来的：
> - 清华源：[docker-ce | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

1. 设置 `Docker` 的 `apt` 存储库：
``` shell
# 添加Docker官方GPG密钥:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo curl -fsSL 
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 将存储库添加到 apt 源:
echo \
	## 默认从国外源安装 Docker (网速受限严重或者挂梯子)
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

```

2. 安装 `Docker` 软件
- Laster：
``` shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- specific version：
- 要安装特定版本的 `Docker Engine`，请首先列出存储库中的可用版本：
``` shell
# 列出可用版本:
apt-cache madison docker-ce | awk '{ print $3 }'

# 可用版本列表:
5:27.4.0-1~debian.12~bookworm
5:27.3.1-1~debian.12~bookworm
...
```
   选择所需的版本并安装：
```shell
VERSION_STRING=5:27.4.0-1~debian.12~bookworm
sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

3. 验证 Docker 是否安装成功
- 通过 `docker version` 检查`Docker` 是否安装成功： 
```shell
docker version
```
- 通过运行 hello-world 镜像进一步验证安装是否成功：
``` shell
sudo docker run hello-world
```

##### 2.2.3 更新 Docker Engine：

- 要升级 Docker Engine，请按照安装说明的步骤 2 操作，选择要安装的新版本。


#### 2.3 卸载 Docker Engine 命令：

1. 卸载 `Docker Engine` 、`CLI `、` containerd ` 和 ` Docker Compose ` 软件包：
```shell
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```
2. 主机上的映像、容器、卷或自定义配置文件不会自动删除。要删除所有映像、容器和卷：
``` shell
# /var/lib/docker ->  docker 默认工作路径
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```
3. 删除源列表和密钥环：
``` shell
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm /etc/apt/keyrings/docker.asc
```

#### 2.4 配置国内镜像加速：

- 阿里云镜像加速：[7、配置阿里云镜像加速_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1og4y1q7M4?spm_id_from=333.788.player.switch&vd_source=e71be9ebbb70e681992d6bfd2b2fb088&p=7)

### 3. Docker 命令：

- Docker 命令总框架图：

<div style = "text-align: center;" >
<img src =  "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250114184955.png" />
</div>

| 命令                                      | 描述                           | 示例命令                                |
| --------------------------------------- | ---------------------------- | ----------------------------------- |
| `docker --version`                      | 查看 Docker 版本                 | `docker --version`                  |
| `docker info`                           | 查看 Docker 系统的详细信息            | `docker info`                       |
| `docker ps`                             | 查看当前运行的容器列表                  | `docker ps`                         |
| `docker ps -a`                          | 查看所有容器，包括停止的容器               | `docker ps -a`                      |
| `docker images`                         | 查看本地镜像列表                     | `docker images`                     |
| `docker pull <image>`                   | 从 Docker Hub 拉取镜像            | `docker pull ubuntu`                |
| `docker build -t <tag> <path>`          | 构建镜像                         | `docker build -t my_image .`        |
| `docker run <image>`                    | 创建并启动容器                      | `docker run -it ubuntu /bin/bash`   |
| `docker run -d <image>`                 | 创建并在后台运行容器                   | `docker run -d -p 80:80 nginx`      |
| `docker exec -it <container> <command>` | 在正在运行的容器中执行命令                | `docker exec -it my_container bash` |
| `docker stop <container>`               | 停止运行中的容器                     | `docker stop my_container`          |
| `docker start <container>`              | 启动已停止的容器                     | `docker start my_container`         |
| `docker restart <container>`            | 重启容器                         | `docker restart my_container`       |
| `docker rm <container>`                 | 删除容器（必须先停止容器）                | `docker rm my_container`            |
| `docker rmi <image>`                    | 删除镜像                         | `docker rmi my_image`               |
| `docker logs <container>`               | 查看容器的日志                      | `docker logs my_container`          |
| `docker inspect <container>`            | 查看容器或镜像的详细信息                 | `docker inspect my_container`       |
| `docker network ls`                     | 列出所有网络                       | `docker network ls`                 |
| `docker volume ls`                      | 列出所有卷                        | `docker volume ls`                  |
| `docker-compose up`                     | 启动 docker-compose 配置的所有服务    | `docker-compose up`                 |
| `docker-compose down`                   | 停止并移除 docker-compose 配置的所有服务 | `docker-compose down`               |
| `docker-compose logs`                   | 查看 docker-compose 配置的服务日志    | `docker-compose logs`               |

#### 3.0  run 流程以及 Docker 运行原理：

##### 3.0.1 docker run 容器获取流程 ：

<div style = "text-align: center;" >
<img src =  "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250114155728.png" />
</div>

##### 3.0.2 Docker 原理：

Docker 是一个采用 **Client-Server** 架构的系统，客户端 (Client) 和服务端 (Server) 通过 Socket 进行通信。

1. Client-Server 架构：
- Docker-Client：
	- Docker 的客户端是用户与 Docker 交互的接口，通常是命令行工具 (CLI)，如 `docker` 命令。
	- 客户端负责将用户的操作请求（如运行容器、构建镜像等）转换为 API 请求并发送给服务端。
	- 支持本地与远程方式的 Docker 守护进程交互。
- Docker-Server (Daemon)：
	- 守护进程 (Docker Daemon) 是运行在宿主机上的服务程序，负责接收客户端请求并执行相应操作。
	- 它可以管理容器、镜像、网络和数据卷等 Docker 资源。

2. 通信机制：Socket
- Docker-Client 和 Docker-Server 之间的通信是通过 Socket 完成的，支持以下几种方式：
	- UNIX Socket (默认)：
		- 一种高效的本地进程间通信 (IPC) 机制，能够避免网络协议的额外开销。
		- 在 Linux 系统中，Docker 默认通过 `/var/run/docker.sock` 文件进行本地通信
	- TCP Socket：
		- Docker 也支持通过 TCP Socket 与远程守护进程通信，通常用于跨主机或容器集群管理场景
		- 需要通过配置守护进程 (如 `/etc/docker/daemon.json`) 开启指定的 TCP 端口（默认端口通过为 `2375` 或 `2376`，后者启用 TLS 加密）
	- HTTP API：
		- Docker 提供了 RESTful 风格的 HTTP API，所有客户端命令最终会被转换为对应的 API 调用。
		- 客户端通过 HTTP 或 HTTPS 请求访问守护进程，从而实现跨语言的管理与集成。

##### 3.0.3 Docker run 命令执行流程：

<div style = "text-align: center;" >
<img src =  "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250114163224.png"  width = "450px"/>
</div>

1. **客户端发起请求**
- 用户在命令行输入 `docker run` 命令，客户端工具将该操作转换为 HTTP API 请求，例如 `POST /containers/create` 和 `POST /containers/start`。

2. **通信建立**：
- 客户端通过 Socket（如 `/var/run/docker.sock` 或 `localhost:2375`）将请求发送到 Docker Daemon。

3. **守护进程处理请求**：
- Docker Daemon 接收到请求后，解析并执行相应操作：
	- 如果镜像不存在，自动从远程仓库拉取。
	- 创建并启动容器实例。

4. **响应结果**：
- Docker Daemon 将执行结果通过 Socket 返回给客户端，用户在终端中看到运行日志或输出结果。


##### 3.0.4 Docker 为什么比 VM 快：
<div style = "text-align: center;" >
<img src =  "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250114165302.png"  width = "500px"/>
</div>

1. 虚拟化技术的区别

	**虚拟机 (VM)**
    - **工作方式**：
        - 虚拟机通过 **Hypervisor**（虚拟化管理程序）虚拟出一整套硬件资源（CPU、内存、磁盘等），并在其上运行完整的操作系统实例。
        - 每个虚拟机实例与其他实例之间完全独立，拥有自己的内核、系统进程和文件系统。
    - **缺点**：
        1. **资源占用多**：
            - 每个虚拟机运行一个完整的操作系统，导致额外的内存和存储开销。
        2. **冗余步骤多**：
            - 从应用到硬件需要经过多个抽象层（应用程序 → 虚拟机操作系统 → 虚拟硬件 → 宿主机操作系统 → 物理硬件）。
        3. **启动较慢**：
            - 每次启动虚拟机都需要加载整个操作系统，启动时间通常以分钟计算。
	**Docker 容器**
	- **工作方式**：
	    - Docker 容器直接运行在宿主机的操作系统内核上，使用宿主机的资源实现进程级虚拟化。
	    - 容器之间互相隔离，每个容器有自己的文件系统、网络栈和进程空间，但共享宿主机的操作系统内核。
	- **优点**：
	    1. **轻量化**：
	        - 容器无需运行独立的操作系统，消除了虚拟机的操作系统层，直接与宿主机共享内核，极大减少了资源消耗。
	    2. **启动快**：
	        - 容器本质上是宿主机上的一个进程，启动速度以秒计算，而不是分钟。
	    3. **高效的资源利用**：
	        - 容器共享宿主机的资源（如内存、CPU），避免了硬件资源的重复虚拟化。

2. Docker 容器与 VM 的核心对比：

| 特性         | 虚拟机 (VM)               | Docker 容器                 |
| ---------- | ---------------------- | ------------------------- |
| **虚拟化层级**  | 硬件级虚拟化                 | 操作系统级虚拟化                  |
| **依赖操作系统** | 每个虚拟机运行一个独立的完整操作系统     | 共享宿主机的操作系统内核              |
| **资源占用**   | 高：需要更多内存和存储空间          | 低：只需应用和其依赖的核心环境           |
| **启动速度**   | 慢：启动时间通常以分钟计算          | 快：启动时间通常以秒计算              |
| **性能**     | 有硬件虚拟化的开销              | 几乎与宿主机性能一致                |
| **隔离性**    | 高：通过 Hypervisor 实现完全隔离 | 高：通过内核命名空间 (namespace) 实现 |

#### 3.1 帮助命令：

``` shell
docker version		# 显示 docker 的版本信息
docker info 		# 显示 docker 的系统信息，包括镜像和容器信息
docker xxxx --help	# 帮助命令
```

帮助文档地址：
- 官网：[docker | Docker Docs](https://docs.docker.com/reference/cli/docker/)

#### 3.2 常用镜像命令：

1. `docker images` 查看本地主机上所有的镜像：

``` shell
# docker images
REPOSITORY		TAG			IMAGE ID			CREATED				SIZE
hello-world		laster		bf756fb1ae65		4 months age		13.3 kB

# 解释
REPOSITORY		镜像仓库源
TAG				镜像标签
IMAGE ID		镜像id
CREATED			镜像创建时间
SIZE			镜像的大小

# Options
-a, --all		显示所有镜像（默认隐藏中间镜像）
-q, --quiet		仅显示镜像ID
```

2.  `docker search` 搜索镜像：

```shell
# docker search xxx
NAME			DESCRIPYION			STARS			OFFICAL			AUTOMATED
mysql			...					....			....			....

# Options
--filter=xxx=xxx
# --filter=STARS=3000	## 搜索收藏量大于3000的镜像

```

3.  `docker pull` 拉取镜像：

```shell
# docker pull xxx[:tag]
## 默认最新版本下载
docker pull xxx						# 如果不写 tag，默认下载 latest
using default tag: latest
lastest: Pulling from xxxx/xxx
xxxxxxxx: pull complete				# docker核心：分层layout下载
xxxxxxxx: pull complete
xxxxxxxx: pull complete
....
Digest:xxxxxxxxxxxx					# 防伪签名
Status: Downloaded newer image for xxx: latest
docker.io/xxxxx/xxx:latest			# image 真实地址

## 指定版本下载
docker pull xxx:x.x
x.x: Pulling from xxxx/xxx
xxxxxxxx: Already exists			# docker 已有分层
xxxxxxxx: Already exists
xxxxxxxx: Already exists
xxxxxxxx: pull complete				# 更新区别分层
xxxxxxxx: pull complete
xxxxxxxx: pull complete
....
Digest:xxxxxxxxxxxx					# 防伪签名
Status: Downloaded newer image for xxx: latest
docker.io/xxxxx/xxx:x.x				# image 真实地址
```

4.  `docker rmi` 删除镜像：
``` shell
docker rmi -f 镜像id					# 删除指定镜像
docker rmi -f 镜像id 镜像id ...		# 删除多个镜像
docker rmi -f $(docker images -ag)	# 删除所有镜像
```

#### 3.3 常用容器命令：

> [!Tip] 
> 有了镜像才能创建容器，容器是镜像的实例

1. `docker run` 创建并运行新容器：

``` shell
docker run [option] image

# option
--name="xxx"	# 容器实例名称
-d				# 以后台方式运行
-it				# 使用交互方式进行，进入容器查看内容
-p				# 指定容器端口
## -p 主机端口:容器端口
```

2. `docker ps` 查看容器状态：

```shell
docker ps

# option
			# 列出当前正在运行的容器
-a			# 列出历史运行容器
-n=xx		# 显示最近创建的容器，x为显示个数
-q			# 只显示容器编号
```

3. `docker rm` 删除容器：

```shell
docker rm xxx					# 删除指定容器。xxx为容器id/name（不能删除正在运行容器）
docker rm -f xxx				# 强制删除指定容器
docker rm -f $(docker ps -aq)	# 强制删除所有容器
docker ps -aq |xargs docker rm	# 运用管道删除所有容器
```

4. `docker xxx` 启动和停止现有容器：

```shell
# xxxx 为容器id/name
docker start xxxx				# 启动xxx容器
docker stop xxxx				# 停止xxx容器
docker restart xxxx				# 重启xxx容器
docker kill	xxxx				# 杀死xxx容器进程
```

5. `exit` 退出容器：

```shell
exit							# 容器停止并退出
Ctrl + P + Q					# 容器不停止退出
```

#### 3.4 其他操作命令：

1. `docker run -d` 后台创建容器：

```shell
docker run -d xxxx				# 在后台创建新容器
```

> [!Tip] 
> 这里常见一个坑：当创建完后，调用 `docker ps`，有时候会发现容器已停止
> 解答：docker 容器在后台运行时，就必须要有一个前台进程运行，否则 docker 就会检测为无应用自动停止，直接杀死容器进程。

2. `docker logs` 查看日志：

``` shell
docker logs [option] xxxx		# xxx为容器id/name

docker logs -tf xxx				# 显示容器xxx的输出所有日志	
docker logs -tf --tail n xxx	# 显示容器xxx输出的最新的n条日志


# option
-t								# 显示时间戳
-f								# 显示日志 output
--tail nmber					# 限制显示日志 output 条目
```

3. `docker top` 查看容器进程信息：

``` shell
docker top xxx					# xxx为容器id/name
UID			PID			PPID		C 		STIME		TTY
root		xxxx		xxxx		0		xx:xx		?

# 解释
UID				# 用户ID
PID				# 进程ID
PPID			# 父进程ID
```

4. `docker inspect` 查看容器所有信息：

```shell
docker inspect xxx				# xxx为容器id/name

## 包含有容器的创建时间，镜像源，容器id，进程id，运行状态等等所有信息
```

5. `docker exec` `docker attach` 进入当前正在运行的容器：

> [!Tip] 
> 容器通常都是以后台方式运行的

```shell
## xxx为容器id/name
# 方式一
docker exec -it xxx bash		# 进入容器后一个新的终端

# 方式二
docker attach xxx bash			# 进入容器当前正在执行终端 
```

6. `docker cp` 手动从容器内拷贝文件到主机：

```shell
docker cp xxx:yyyy zzzz			# 拷贝容器xxx的yyy路径文件到本地主机的zzzz路径

# 解释：
xxx			# 容器id/name
yyy			# 容器文件路径
zzz			# 本地主机文件路径

## 主机拷贝文件到容器一般使用挂载实现
```

### 4. Docker 镜像：



### 5. 容器数据卷：



### 6. DockerFile：



### 7. Docker 网络原理：


### 8. IDEA 整合 Docker：



### 9. Docker Compose：



### 10.  Docker Swarm：



### 11. CI\CD Jenkins： 


