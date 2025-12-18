  
## 0.  学习资料（感谢感谢!!!!）

- 学习视频：
1. 2024 年：
	- [01-操作系统概述 (操作系统的历史、学习操作系统的方法) [南京大学2024操作系统]_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Xm411f7CM/?spm_id_from=333.337.top_right_bar_window_history.content.click&vd_source=e71be9ebbb70e681992d6bfd2b2fb088)
2. 2025 年：
	- [01 - AI 时代的操作系统课 [2025 南京大学操作系统原理]_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1XZAbeqEyt/?spm_id_from=333.337.search-card.all.click&vd_source=e71be9ebbb70e681992d6bfd2b2fb088)

## 1. 绪论：

- 操作系统是管理软/硬件资源、为程序提供服务的程序
	- 硬件和软件的中间层
		- 对单机（多处理器）作出抽象
		- 支撑多个程序执行
	- 推广到 “ Systems ”：
		- 对多台计算机抽象（分布式系统）、对存储设备的抽象 (存储系统)

### 1.1 应用视角下的操作系统

-  `ELF` 目标文件：(**可以深入了解学习一下**)
	- `ELF` 文件头包含了程序启动的很多信息：
		- `Entry point address` ：程序执行地址
- 


### 1.2 硬件视角下的操作系统

# 2. 虚拟化

## 2.1 程序与进程：

### 进程创建与调转：

#### fork() ：

`fork` 创建子进程：

- 当父进程调用 `fork()` 时，操作系统会创建一个新的子进程。这个子进程几乎完全复制父进程的进程映像（除了进程 ID 和部分资源等）
	- `fork()` 函数调用：
		- 子进程返回 `0`;
		- 父进程返回子进程 `PID`


##### UNIX —— 进程托孤机制：

- 进程可通过 `fork()` 创建子进程，子进程进程信息会保留父进程 `ID` 号（`ppid`，可通过 `getppid()` 函数获取）
- 而父进程与子进程的运行时间是可能不一致的，存在父进程比子进程更先一步退出的情况，有的操作系统的设计，会通过 `waitpid()` 让父进程等待子进程结束再进行退出 `exit` 操作，但有的不会这么设计，例如 `UNIX`
- 在 `UNIX` 中进程托孤机制为：当某一子进程其父进程进行 `exit` 操作时，会将该子进程的父进程挂到用户态 `1` 号进程（但 `1` 号进程不会对其进行任何操作）

> [!NOTE] Fork Bomb
   Linux 的 `OOM` 保护
> 
> ```bash
> :(){:|:&};: # bash: 允许冒号作为 identifier
> ``` 
> 
> 现代 `Linux` 系统无 `OOM` 保护，目前机制为当操作系统资源已被进程所使用完毕，再进行进程创建时，系统会自动查找合适进程进行杀死后将资源给到新创建进程


```c
int main() {
	pid_t x = fork();
	pid_t y = fork();
	printf("x: %d, y: %d\n", x, y);
}
```

- 当操作系统初始状态只有一个进程时，该进程调用执行该函数，该操作系统中含有多少个进程？
	- 含有 `4` 个进程：
		- 当前唯一 `1` 进程首先会调用 `fork()` 派生出其子进程 `2`
		- `1、2` 进程含有相同进程映像，所以其进程程序计数器 `PC` 都指向 `y = fork()`，执行两进程会再分别创造其对应子进程

```c
int main() {
	for(int i = 0; i < 2; i ++) {
		fork();
		printf("🌟");
	}
}
// 🌟🌟🌟🌟🌟🌟
```

- 同理对上述代码进行分析：
	- 当前唯一 `1` 号进程调用 `fork()` 派生出其子进程 `2` 时，`1、2` 进程含有相同进程映像，进程程序计数器都指向 `printf("🌟")`，所以第一次 `fork()` 会打印 `2` 个 `🌟`
	- 而当进入第二次 `fork()` 时，由上分析可得，此时操作系统含有 `4` 个进程，且同时各进程程序计数器都指向 `printf("🌟")` ，所以第二次 `fork()` 会打印剩余 `4` 个 `🌟`

```c
int main() {
	for(int i = 0; i < 2; i ++) {
		fork();
		printf("🌟\n");
	}
}
// 执行 ./a.out | cat
// 🌟
// 🌟
// 🌟
// 🌟
// 🌟
// 🌟
// 🌟
// 🌟
```

> [!NOTE] 输出缓冲
> 在 `UNIX` 系统中，默认情况下，当标准输出 `stdout` 为终端时，缓冲行为是行缓冲的，意味着每遇到换行符就刷新缓冲区。但当 `stdout` 为管道时，它可能是全缓冲的，意味着缓冲区只在满时或显式刷新时输出

- 综上所述，当第一次执行 `fork()` 时，`1、2` 进程缓冲区为 `🌟\n`，并不刷新输出；且在第二次 `fork()` 并 `4` 个进程分别执行 `printf("🌟\n")` 后，其各缓冲区都为 `🌟\n🌟\n`。最后当程序退出时，缓冲区会被刷新，输出 `8` 个 `🌟`。


####  execve()：

- 函数签名：

```c
int execve(const char *filename, char *const argv[], char *const envp[]);
```

-  函数作用：
	- 将当前进程**重置**成一个可执行文件描述状态机的**初始状态**
	- **操作系统维护的状态不变**：进程号、目录、打开的文件.......
- `execve` 是**唯一能够"执行程序"的系统调用**
	- 所以一般所有软件或者命令的调用，第一段系统调用都会是 `execve()`，将 `fork()` 派生出的子进程在保留当前状态（打开文件、状态参数等）情况下，转向去执行其他程序


####  操作系统进程模型：

```c
int pid = fork();
if(pid == -1) {
	perror("fork");
	goto fail;
} else if(pid == 0) {
	ececve(.....);
	perror("execve");
	exit(EXIT_FAILURE);
} else {
	....;
	int status;
	waitpid(pid, &status, 0);
}
```



## 2.2 mmap 和进程地址空间：

### 进程的初始状态：
#### 进程 `execve()` 后的状态：

- 手动探索：
	- 寄存器：
		- 可通过 `GDB` 调试器所提供的 `start i` 指令，在程序执行第一条指令前停止程序运行，再通过 `printf` 直接打印寄存器值查看
	- 内存 ：
		- 内存是一个 `byte array` 字节数组（取决于系统类型， `64` 位系统其 `char *` 为 `64` 字节），可通过 `char *p` 指针的方式去读取可读取内存段

- 当一个进程进行 `execve` 系统调用时，它实际上是在请求"请求当前正在执行的程序替换成一个全新的程序"，即 `execve` 并不是创建一个新进程，而是在**现有进程的“躯壳”里**，装入一个新的"灵魂" (新的程序代码和数据)，进程 ID（`PID`）保存不变，但程序本身被完全替代。
- 综上所述，所以在调用 `execve()` 后：
	- **销毁旧的地址空间**：
		- 当前进程的几乎所有内存状态（代码、数据、堆、栈等）都会被销毁，这包括
			- 程序的代码段（`.text`）
			- 已初始化数据段（`.data`）
			- 未初始化数据段（`.bss`）
			- 堆（`heap`）
			- 用户态栈（`user stack`）
			- 所有内存映射的区域（动态库等）
	- **构建新的地址空间**：
		- 从指定的可执行程序文件（`ELF` 文件）中，构建一个全新的、干净的地址空间
			- **代码段**和**数据段**从新程序对应部分映射进来
			- 创建一个全新的空堆
			- 创建一个全新的空栈
	- **寄存器重置**：
		- 用户态的寄存器会被清空或设定为确定的初始值


### 在状态机状态上增加/删除/修改一段可访问的内存：

- 函数库：

```c
#include <sys/mman.h>
```

- 基础函数签名：

```c
// 内存映射
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
```

- 参数解释：
	- `addr`：指定映射的起始地址，通常设为 `NULL`，由系统自动选择合适的位置
	- `length`：映射区域长度，单位为字节（`Bite`）
	- `prot`：指定内存保护方式，如 `PROT_READ` (可读)，`PROT_WRITE` (可写) 等
	- `flags`：控制映射行为，如 `MAP_SHARED` (共享映射)，`MAP_PRIVATE` (私有映射) 等
	- `fd`：要映射的文件描述符（默认 `-1`，无文件描述符）
	- `offset`：文件中的偏移量，表示从文件的哪个位置开始映射
- 成果返回：
	- 函数成果时返回映射区域的起始地址，失败是返回 `MAP_FAILED`

> [!NOTE] 文件映射到内存地址空间
> 文件的本质：字节序列，与内存的本质相同，所以我们可以直接将一段可执行程序映射到内存地址空间的一段可用连续空间，当程序指针指向该段内存地址头时，即可实现程序执行
> （可执行文件与加载器）
> 

> [!NOTE] 缺页异常
> 事实上，你可以任意先操作系统申请任意空间大小的内存，都是会被允许通过的（因为只要你不访问这个部分内存空间时，操作系统都是没有帮你去申请这部分内存空间的）。而当你去访问该部分内存空间时，操作系统就会产生一个缺页异常中断（内存空间的写时复制机制  | [[进程与线程#^ad1727|进程映像与写时复制]]）；如果缺页地址不属于你，则会返回一个  `segment failed` 异常，如果缺页地址属于你，再进行分配内存空间大小
>

- 在 `testkit` 代码中，父子进程遵循下述逻辑：
	- 当父进程调用 `fork()` 函数后，父进程会进入 `waitpid()` 等待子进程结束，子进程开始执行 `testCode` 进行代码测试。
	- 但在这里面子进程是如何将测试的结果（`timeout` 等）传递给父进程的呢？
		- `testkit` 会在父进程 `fork()` 前执行 `mmap` 映射一段**共享内存**，同时将标准输出 `stdout` 和标准错误 `stderr` 指向该段内存，`fork()` 派生出子进程后，子进程的 `stdout` 和 `stderr` 都会输出到该段共享内存，父进程在 `waitpid` 结束后输出打印该部分共享内存信息即可  

```c
// 解除一段地址空间映射
int munmap(void *addr, size_t length);		// 与 mmap 配对，同时 length 需写入正确的大小
```

```c
// 修改映射权限
int mprotect(void *addr, size_t length, int prot);
```

```c
// 内存映射修改同步磁盘
int msync(void *addr, size_t length, int flags)
```

- 函数功能：
	- 当你使用 `mmap` 系统调用将一个文件映射到进程的内存地址空间时，你对这块内的读写操作会直接反应到内存中的文件副本上，但这些文件修改**并不会立即被写回**到磁盘文件中。
	- `msync` 就是用来控制**何时，如何**将这些内存中的修改立即写回磁盘的。
- 参数解析：
	- `addr`：映射内存区域的起始地址
	- `length`：需要同步的内存区域长度
	- `flags`：同步行为标志位
		- **`MS_SYNC`**：**同步写入**。请求写入磁盘，并**等待**所有写入操作完成，`msync` 函数才会返回。这是最安全的方式，保证了数据确实写入了物理硬盘。
		- **`MS_ASYNC`**：**异步写入**。只是**通知**操作系统应该将这些数据写回磁盘，但 `msync` 函数会立即返回，不等待实际的磁盘I/O完成。性能更好，但不保证数据已完全落盘。

---
```bash
pmap pid # 查看进程程序的地址空间映射
```

- 我们可以通过 `pmap` 命令行工具去显示进程的内存映射信息
- 但 `pmap` 是如何实现的呢？
	- `pmap` 是的实现主要依赖于 `/proc` 文件系统，其会通过读取 **`/proc/[PID]/maps`** 文件，该文件包含了**进程的内存映射信息，如地址范围、权限、偏移量、设备、`inode` 和路径名**等
	- `pmap` 读取完 `/proc/[PID]/maps` 中的内容，并对其进行解析计算，最后得到进程每个内存区域的大小、权限等详细信息，并将其转换成更易读的格式显示输出
	- 可以通过 `strace pmap [PID]` 进行验证


### 入侵进程的地址空间：

- 在 `Linux` 系统下，可通过修改 `/proc/[pid]/mem` 来实现对另一进程的地址空间修改
- 除此之外，还可通过：
	- `ptrace` 系统调用：
		- 通过 `PTRACE_PEEKDATA` 和 `PTRACE_POKEDATA` 请求读写目标进程内存（`gdb` 调试工具即依赖此机制，但需附加到目标进程）
	- `process_vm_writev` 系统调用：
		- 允许直接向目标进程写入数据，效率高于 `ptrace`，且无需暂停目标进程。需要 `CAP_SYS_PTRACE` 能力或权限配置
	- 共享内存：
		- 使用 `shm_open` 或 `mmap` 创建共享内存区域，但需目标进程主动映射并配合访问
	- ..........




## 2.3 访问操作系统中的对象：

文件和设备：

- 文件：有“名字”的数据对象
	- 字节流（终端， `random`）
	- 字节序列（普通文件）
- 文件描述符：
	- 指向操作系统对象的“指针”
		- `Everything is a file`
		- 通过指针可以访问"一切"
	- 对象的访问都需要指针
		- `open`、`close` 、`read/write`（解引用）、`lseek` (指针内赋值/运算)、`dup`（指针间赋值）

```
open:
	p = malloc(sizeof(fileDescriptor));
close:
	delete(p);
read/write:
	*(p.dara++);
lseek:
	p.data += offset;
dup:
	q = p;
```

> [!NOTE]  dup 和 dup2
> 库函数：`#include <unistd.h>`
> ```c
> int dup(int oldfd);
> int dup2(int oldfd, int newfd);
> ```
> `dup` 函数：
> - 作用：复制文件描述符 `oldfd`，返回一个新的文件描述符
> - 返回值：成功返回新的文件描述符，失败返回 `-1`
> `dup2` 函数：
> - 作用：复制文件描述符 `oldfd` 到指定的 `newfd`
> - 返回值：成功返回新的文件描述符，失败返回 `-1`
> 
> 常用场景：输入/输出重定向：
> ```c
> #include <unistd.h>
> #include <fcntl.h>
> #include <stdio.h>
> 
> int main() {
>     int fd;
>     
>     // 将标准输出重定向到文件
>     fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
>     if (fd == -1) {
>         perror("open");
>         return 1;
>     }
>     
>     // 将标准输出(1)重定向到文件
>     if (dup2(fd, STDOUT_FILENO) == -1) {
>         perror("dup2");
>         return 1;
>     }
>     
>     close(fd);  // 关闭原文件描述符
>     
>     // 现在printf会输出到文件而不是终端
>     printf("这行文字会被写入output.txt文件\n");
>     printf("标准输出已被重定向！\n");
>     
>     return 0;
> }
> ```


### 文件描述符：

- 文件描述符是"进程状态"的一部分
	- 保存在操作系统中；程序只能通过整数编号访问
	- 文件描述符自带一个 `offset` 偏移量
		- 当一个文件描述符通过 `dup()` 进行复制时，那么就会有两个文件描述符指向同一个文件对象，那么同时对这个文件进行操作（`write` 操作），为保证此时这两个文件操作符能够对该文件操作偏移位置同步，所以每个文件描述符自带一个 `offset` 的偏移量
- 通过 `fork()` 和 `dup()` 之后，文件描述符共享 `offset` 
	- 但这样子的设计其实并不便于权限管理（例如：在 `windows` 系统中，遵循"最小权限原则"，默认 `handle` 是不可继承）
	- 所以 `Linux` 也同样引入了 `O_CLOEXEC` (在程序退出时关闭) ，可通过 ```fcnt(fd, F_SETFD, FD_CLOEXEC); ``` 进行权限管理


### 管道：

- `UNIX` 管道（`pipe` ）是一种典型的进程间通信机制，允许数据在不同进程之间单向流通。
- 管道可视为一种特殊的文件，其中一个进程将数据写入管道的一端，而另一个进程从另一端读取数据

- 库函数：

```c
#include <sys/types.h>
#include <sys/stat.h>
```

#### 匿名管道 (pipe)：

```c
int pipe(int pipfd[2]);
```

- 返回两个文件描述符
	- `pipfd[0]` 管道读口对象
	- `pipfd[1]` 管道写口对象
- 管道特性：
	- 当对 `pipfd[1]` 进行 `write` 操作时，如果管道缓冲区还有空间写操作正常，不会抛出异常
	- 当对 `pipfd[0]` 进行 `read` 操作时，如果管道内没有数据就会一直阻塞读取，直到有数据可读取为止
		- 所以当一个进程向一个无数据管道进行 `read` 操作时，该进程会阻塞
- 应用场景：
	- **管道实现进程数据通信**：
		- 当该 `read` 特性与 `fork()` 一起使用时：在进程通过 `pipe()` 创建管道，然后 `fork()` 派生出其子进程，此时父子进程指向同一管道文件描述符，父进程进行 `close([pipfd[0]])` 操作，子进程进行 `close([pipfd[1]])` 操作，此时就可以实现，父子进程之间的数据管道通信！
	- **命令行管道 `|`**：
		- 当管道 `pipfd` 与 `dup()` 一起使用时：通过 `dup()` 将 `stdout` 标准输出 `fd` 复制为管道读口 `pipfd[0]`，`stdin` 标准输入 `fd` 复制为管道写口，再结合上述 `fork()` 的使用，即可实现命令行管道 `|` 的实现，将 `|` 管道左侧指令的输出传递到 `|` 管道右侧指令的输入
	- **重定向写入 `>`**:
		- 与上同理，但在子进程接收到数据后，会执行 `write` 向指定文件描述符写入数据
  

#### 命名管道 (fifo)：

```c
int mkfifo(const chare *pathname, mode_t mode)
```

- 函数功能：
	- 在文件系统中创建指定路径（命名）的管道对象（文件对象）
	- 不同进程之间可以通过 `open()` 该管道对象的文件的读/写口进行操作，以实现不同进程之间的数据通信
- 参数解析：
	- `pathname`：要创建的 `FIFO` 的路径
	- `mode`：权限模式，例如 `0666`
- 返回值：
	- 成功时返回 0，失败时返回 `-1` 并设置 `errno`


#### 文件描述符适合什么

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251020161130553.png" width="500px"/>
</div>

- 优点：
	- 优雅，文本接口
- 缺点：
	- 和各种 `API` 紧密耦合
	- 对高速设备不够友好
		- 额外的延迟和内存拷贝
		- 单线程 `I/O`




## 2.4 终端、进程组和 UNIX Shell

### 终端：
#### 伪终端：

`PTY`（伪终端）由一对主/从设备构成，工作流程如下：

**主从设备核心机制**：

- **双向传递**： `master` 和 `slave` 形成完整的双向通信通道
- **透明转发**：程序不知道自己在与伪终端交互，以为是在与真实终端交互

```text
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   终端模拟器     │    │    PTY 设备     │    │     程序        │
│   (父进程)      │◄──►│   (内核)        │◄──►│   (子进程)      │
│                 │    │                 │    │                 │
│  - 显示输出     │    │  master 端      │    │  - shell        │
│  - 接收输入     │    │  slave 端       │    │  - vim          │
│  - 界面渲染     │    │                 │    │  - 其他命令     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
       ▲                       ▲                       ▲
       │                       │                       │
       │ 读写 master           │ 内核驱动转发           │ 读写 slave
       │                       │                       │
       └───────────────────────┼───────────────────────┘
                               │
                    ┌─────────────────┐
                    │   伪终端驱动     │
                    │   (内核模块)    │
                    └─────────────────┘
```

```
1. 第一阶段：
用户输入 'l' 's' '回车' 
    ↓
终端模拟器捕获键盘事件
    ↓
终端模拟器将字符写入 master
    ↓
伪终端驱动将数据从 master 转发到 slave
    ↓
shell 从 slave 读取到输入 "ls\n"

2. 第二阶段：
shell 解析 "ls" 命令，执行 ls 程序
    ↓
ls 程序将结果写入 stdout（实际是 slave）
    ↓
伪终端驱动将数据从 slave 转发到 master
    ↓
终端模拟器从 master 读取输出数据
    ↓
终端模拟器将输出渲染到屏幕
```

- 主设备（`PTY Master`）:
	- **控制者**：向从设备发送数据（用户输入）
	- **监听者**：读取从设备发送的数据（程序输出）
	- **管理者**：设置终端属性、窗口大小等

- 从设备（`PTY Slave`）:
	- **接口提供者**：为程序提供标准的终端接口
	- **数据传输者**：接收程序的输出，发送给主设备
	- **输入接收者**：从主设备接收输入，传递给程序

#### 如何创建伪终端：

- 伪终端经常被创建（`ssh、tmux new-window`）

```c
#include <pty.h>

int openpty(int *amaster, int *aslave, char *name, 
           const struct termios *termp, const struct winsize *winp);
```

- 函数功能：
	- 自动创建并配置一个伪终端（`pseudo-terminal`）对
	- 内部通过打开 `/dev/ptmx` 创建主从设备对

- 参数解析：
	- **`amaster`**: 输出参数，返回主设备（master）的文件描述符
	- **`aslave`**: 输出参数，返回从设备（slave）的文件描述符
	- **`name`**: 输出参数，返回从设备路径名（可为 NULL）
	- **`termp`**: 输入参数，设置从设备的终端属性（可为 NULL，使用默认值）
	- **`winp`**: 输入参数，设置从设备的窗口大小（可为 NULL）
- 函数返回：
	- 成功返回 0
	- 失败返回 -1 并设置 errno

---
- 终端模拟器实现：
	- `openpty()` + `fork()`
		- 子进程：关闭 `master`，将 `stdin/stdout/stderr` 指向 `slave`
		- 父进程：关闭 `slaver`， 从 `master` 读取输出显示到屏幕；将键盘输入写入 `master`

```c
#include <pty.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main() {
    int master, slave;
    char buffer[1024];
    fd_set readfds;
    
    // 1. 创建伪终端
    if (openpty(&master, &slave, NULL, NULL, NULL) == -1) {
        perror("openpty");
        exit(1);
    }
    
    // 2. 创建子进程运行 shell
    pid_t pid = fork();
    if (pid == 0) {
        // 子进程 - 运行 shell
        close(master);
        
        // 将 slave 设置为子进程的标准输入、输出、错误
        dup2(slave, STDIN_FILENO);
        dup2(slave, STDOUT_FILENO);
        dup2(slave, STDERR_FILENO);
        close(slave);
        
        // 启动 shell
        execl("/bin/bash", "bash", NULL);
        exit(1);
    }
    
    // 3. 父进程 - 终端模拟器主循环
    close(slave);
    
    while (1) {
        FD_ZERO(&readfds);
        FD_SET(master, &readfds);    // 监听程序输出
        FD_SET(STDIN_FILENO, &readfds); // 监听用户输入
        
        // 等待数据可读
        if (select(master + 1, &readfds, NULL, NULL, NULL) > 0) {
            // 检查是否有程序输出需要显示
            if (FD_ISSET(master, &readfds)) {
                ssize_t bytes = read(master, buffer, sizeof(buffer) - 1);
                if (bytes > 0) {
                    buffer[bytes] = '\0';
                    // 将程序输出显示到用户屏幕
                    printf("%s", buffer);
                    fflush(stdout);
                }
            }
            
            // 检查是否有用户输入需要发送给程序
            if (FD_ISSET(STDIN_FILENO, &readfds)) {
                ssize_t bytes = read(STDIN_FILENO, buffer, sizeof(buffer) - 1);
                if (bytes > 0) {
                    // 将用户输入发送给程序（通过 master）
                    write(master, buffer, bytes);
                }
            }
        }
    }
    
    close(master);
    return 0;
}
```

#### 终端模式：

- `Canonical Mode` ：按行处理
	- 回车发送数据（终端提供编辑功能）
- `Non-canonical Mode` ：按字符处理
	- 每个字符立即发送给程序
	- 用于实现交互式程序：`vim、ssh sshtron.zachlatta.com`

#### 终端属性控制：

- `tcgetattr/tcsetattr` (`terminal control get/set attr` )
- 可以控制终端的各种行为：回显、信号处理、特殊字符等
	- （`Linux` 登录密码输入 -> 关闭了回显）

#### 终端的实际应用：

- 用户登录：
	- 系统启动（`kernel -> init -> getty`）
	- 远程登陆（`sshd -> fork -> opentty`）
		- `stdin、stdout、stderr` 都会指向分配的终端
	- `vscode`（`fork -> opentty`）

### 会话 (Session) 和进程组 (Process Group)  

- 在 `Linux` 系统中，可通过按下 `Ctrl + C` 终止当前进程，但这里就存在一个问题，操作系统中所有的进程都是由最开始的进程 `fork()` 派生出来，那么当你终端输入 `Ctrl + C` 时，操作系统是如何选择终止当前终端的（同时由于 `fork()` 是在 `openty()` 后调用，那么所有进程可能共用同一 `pty` 字符对象，那么当 `master` 接收到 `Ctrl + C` 时，所有进程程序都会接收到该信号，按理应该都会终止？这又是为什么呢）


<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251020222951666.png" width="600px"/>
</div>

#### 控制终端（Controlling Terminal）

- 每个会话最多只能有一个控制终端
- 控制终端是会话与用户交互的接口（通常是伪终端`pty`）
- 终端特殊字符（如`Ctrl+C`、`Ctrl+Z`等）通过控制终端处理

#### 会话（Session）：

```bash
# 查看进程的会话ID
ps -o pid,pgid,sid,comm

# 创建新会话的示例
setsid()  # 系统调用，创建新会话并成为会话首进程
```

- **会话特性**：
	- **会话首进程**：创建会话的进程，通常是 `shell`
	- **控制终端**：会话可以关联一个终端设备
	- **前台进程组**：当前正在接收终端输入的进程组
	- **后台进程组**：不在前台但属于同一会话的进程组


> [!NOTE] 会话关闭，所有该会话创建的进程都会终止 
> 会话创建的所有子进程都会继承父进程的 `Session ID`，一个 `Session` 关联一个控制终端 `controlling terminal`
> 当 `Session Leader` 退出时，全体进程收到 `Hang UP` (`SIGHUP`)退出程序 


#### 进程组：

```c
// 设置进程组ID
setpgid(pid_t pid, pid_t pgid);

// 获取进程组ID
getpgid(pid_t pid);
```


#### Ctrl+C 信号传递机制：

- 每一个会话都会有一个 `Controlling terminal`，其内会包含有  `Foreground PGID` (前台进程组 `ID`) 以及 `Controlling SID` (控制会话 `ID`)
- `Foreground PGID` 会告诉操作系统目前前台进程组 `ID` 号，当 `Controlling SID` 接收到输入 `Ctrl + X` 时，只会终止该进程组 `ID` 内的所有进程

 - 具体流程：
	1. **终端驱动检测**：终端驱动程序检测到`Ctrl+C`（`ASCII 0x03`）
	2. **查找前台进程组**：终端驱动查找当前控制终端的前台进程组ID
	3. **发送 SIGINT**：向前台进程组内的**所有进程**发送`SIGINT`信号
	4. **信号处理**：进程组内进程收到 `SIGINT`，会将当前的 `PC` 压入栈中，并强制 `jump` 到 `SIGINT` 信号所对应的函数中, `SIGINT` 在 `libc` 中默认的行为是终止程序


#### API：

```c
setsid/getsid 		// setsid 使进程脱离 controlling terminal

setpgid/getpgid		// 获取/设置进程 pgid

tcsetpgid/tcgetpgid	// 获取/设置终端控制器 pgid
```
 
####  相关论文：

- ![SetUid Demystified]()


### UNIX Shell：

- `UNIX Shell`：基于文本替换的极简编程语言
	- 只有一种类型：字符串
	- 但不支持算术运算（但可通过 `expr 1 + 2`）
- 语言机制：
	- 预处理：`$()`，`<()`
	- 重定向：```cmd > file <file 2> /dev/null```
	- 顺序结构：`cmd1; cmd2` 、`cmd1 && cmd2`、`cmd1 || cmd2`
	- 管道：`cmd1 || cmd2`

```bash
<()	# 预处理，重定向到文件描述符

&&	# 前语句执行成功则调用后语句

||  # 前语句执行不成功则调用后语句（前后唯一）
```


####   推荐阅读：

- `Freestanding Shell` 
	- `Zero-dependency UNIX Shell`
	- 这个 `Shell` 没有引用任何库文件 —— 它只通过系统调用访问操作系统中的对象
- `man sh`
	- `dash - command interpreter(shell)`
	- `UNIX Shell` 机制设计想法


#### 实际场景分析：

- 重定向系统权限文件报错：

```bash
# 错误指令
sudo echo hello /etc/a.txt

# 正确指令
sudo bash -c 'echo hello /etc/a.txt'
```

- 流程分析：
	- 在 `shell` 中，指令的执行是通过 `fork()` 派生出子进程，再调用 `execve()` 指令去重置执行指令功能的，而重定向的实现，需要父进程在 `fork()` 派生前去 `open` 去打开路径文件描述符，子进程继承。而此时由于父进程并没有 `sudo`，所以导致父进程 `open` 系统权限文件失败，程序报错显示 `Permission denied`
	- 所以需要 先给 `bash` 先添加权限



## 2.5 libc 原理和实现

### libc 的定位与作用

#### 操作系统的三层接口关系

| 层级                                         | 典型代表                                | 说明                    |
| ------------------------------------------ | ----------------------------------- | --------------------- |
| `ABI`（`Application Binary Interface`）      | 系统调用、寄存器约定                          | 用户进程与内核交互的 “二进制契约”    |
| `libc`（`C Standary Library`）               | `glibc`、`musl` 、`uClibc`            | 将底层系统调用封装为可移植的 C 函数接口 |
| `API`（`Application Programming Interface`） | `printf` 、`malloc`、`pthread_create` | 用户编程时直接使用的标准库接口       |
> 🔹 系统调用  ≈ ABI
> 🔹 libc ≈ 地基上的框架（语言运行时）
> 🔹 应用程序 ≈ 建筑

---
### libc 的核心设计哲学：

#### 机制与策略分离（Mechanism vs Policy）

- 机制（`Mechanism`）：内核提供的基础能力，如创建进程、内存管理、`I/O`
- 策略（`Policy`）：由应用或用户空间库决定如何使用这些机制

例如：

```c
// 内核机制：write() 系统调用
ssize_t write(int fd, const void *buf, size_t count);

// 策略（libc 封装）：printf()
int printf(const char *fmt, ....) {
	// 处理格式化字符串 -> 调用 write()
}
```

> 🔹操作系统提供 “怎么做”的能力
> 🔹 `libc` 决定 “具体怎么用”

#### 最小完备性原则（Minimal Completeness）

- `libc` 只提供应用程序无法自行实现的功能：
- 避免重复造轮子，保持最小化与高移植性
- **用户空间的事情由用户空间完成**

例如：

- `strcpy`、`strlen` 等字符串操作完全可以在用户态实现
- 只用 `read()` 、`write()` 、`fork()` 等才需要调用内核


---
### libc 的实现架构：

#### 三层架构：

```text
+--------------------------------+
|   高级封装层（I/O, printf）     |
+--------------------------------+
|   C 标准库函数层（malloc, atoi）|
+--------------------------------+
|   系统调用封装层（syscall）    |
+--------------------------------+
|   操作系统内核（ABI）          |
+--------------------------------+
```

#### 系统调用封装层：

- 以 `Linux x86_64` 为例，系统调用通过 `syscall` 指令触发：

```c
long syscall(long num, ....);
```

- 内部实现：

```c
static inline long sysclass0(ling n) {
	long ret;
	__asm__ volatile("syscall"
		: "=a" (ret)
		: "a" (n)
		: "rcx", "r11", "memory");
	return ret;
}
```

> `libc` 内部会维护一个系统调用号表（`syscall number table`）
> 每个函数（如 `write`）会调用相应的系统调用号

#### C 标准函数层：

- 封装常见功能，如：
	- 字符串处理：`strlen`、`strcmp`、`strcpy`
	- 内存管理：`malloc`、`free`
	- 数学库：`sqrt`、`sin`、`pow`
	- 时间与文件 `I\O` ：`fopen` 、`fread` 、`fwrite`

- 这些函数很多不直接调用系统调用，而是：
	- 在用户态维护状态（如 `FILE` 缓冲区）
	- 只在必要时调用系统调用（如 `write()`）

#### 高级封装层：（语言运行时支持）

主要负责：
- 程序入口 `_start`
- 调用 `main` 并传递 `argc`、`argv`、`envp`
- 调用 `atexit` 注册的清理函数
- 处理异常与信号封装
- 支撑 `pthread`、`errno`、`locale` 等机制

例如程序启动流程：

```text
_start() → __libc_start_main() → main() → exit()
```


---
### libc 的实现举例（musl vs glibc）

| 项目         | 特点             | 使用场景     |
| ---------- | -------------- | -------- |
| **glibc**  | 功能最全、体积大、兼容性强  | 桌面/服务器系统 |
| **musl**   | 简洁、可移植、适合静态链接  | 嵌入式 / 容器 |
| **uClibc** | 轻量化、适配无 MMU 系统 | 嵌入式设备    |

---

### libc 与语言运行时（runtime）的关系

| 组件     | 职责                                           |
| ------ | -------------------------------------------- |
| `libc` | 提供 C 语言运行所需的函数接口（`malloc` 、`printf` 、`exit`） |
| 语言运行时  | 负责程序生命周期管理（启动、异常、垃圾回收等）                      |
| 操纵系统   | 提供底层机制（系统调用）                                 |
例如：
- C 运行时中 `_start` 和 `exit` 由 `libc` 实现
- C++ 运行时钟还需构造/析构全局对象
- `Go/Rust` 则自己实现了独立 `runtime`，不依赖 `libc` 启动


---
### 推荐学习：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251021205408020.png" width="500px"/>
</div>
- 调试建议：
	- 编译时添加 `-g` 保留调试信息
	- 使用 `gdb` 单步跟踪函数调用
	- `objdump -d` 反汇编观察指令集实现
	- 结合标准文档（如 `POSIX` ) 对照源码理解设计逻辑




## 2.6 可执行程序、链接器和加载器

### 推荐学习：

- `Linux Kernel` 源码
- 理解编译系统

### 可执行程序（Executable）：

#### 定义和本质：

- **定义**：可执行程序是操作系统能够**直接加载并运行的二进制文件**
- **本质**：从系统层面看，它是**描述进程初始状态的数据结构**，包括：
	- 程序指令（代码段 `.text`）
	- 初始数据（`.data` 、`.rodata` 、`.bss` ）
	- 段与段之间的映射关系（`Program Header Table`）
	- 程序入口点（`e_entry`）
- 在 `Linux` 下，最常见的可执行格式是 `ELF`（`Executable and Linkable Format`），它不仅用于可执行文件，也用于动态库（`.so`）和目标文件（`.o`）

#### ELF 文件格式结构：

- `ELF` 文件结构实际上分为两种"视图"：

|视图|用途|典型结构|
|---|---|---|
|**链接视图（Linking View）**|编译/链接阶段使用，用于符号与节(section)组织|`.text`, `.data`, `.bss`, `.symtab`, `.strtab`|
|**加载视图（Execution View）**|程序运行时使用，用于段(segment)映射|`PT_LOAD`, `PT_DYNAMIC`, `PT_INTERP`|

```text
ELF Header (描述文件基本信息)
├── e_ident[16]    - 魔数、架构信息
├── e_type         - 文件类型(可执行、共享库等)
├── e_machine      - 目标架构
├── e_entry        - 程序入口点
├── e_phoff        - Program Header Table偏移
└── e_shoff        - Section Header Table偏移

Program Header Table (加载视图)
├── PT_LOAD        - 需要加载的段
├── PT_INTERP      - 解释器路径(动态链接器)
├── PT_DYNAMIC     - 动态链接信息
└── PT_PHDR        - Program Header自身位置

Section Header Table (链接视图)
├── .text          - 代码段
├── .data          - 已初始化数据
├── .bss           - 未初始化数据(BSS)
├── .rodata        - 只读数据
├── .symtab        - 符号表
└── .strtab        - 字符串表
```

- **Section Header Table**（节表）：提供链接器在编译阶段的逻辑组织（用于目标文件 `.o`）
- **Program Header Table**（程序头表）：提供加载器在运行是需要加载到内存的段信息（用于运行时映射）

---
### 链接器（Linker）：

#### 静态链接（Static Linking）：

##### 概念：

- 静态链接是在编译期将所有目标文件 (`.a`) 和依赖库 (`.a` ) 整合为一个完整可执行文件，其结果是运行时，无需外部库依赖（加载时不需要动态链接器）

##### 链接器工作阶段：

|阶段|内容|说明|
|---|---|---|
|**符号解析（Symbol Resolution）**|将不同目标文件间的函数、变量引用解析为具体定义|建立全局符号表|
|**地址分配（Address Binding）**|为每个段（`.text`, `.data`）分配运行时虚拟地址|形成最终内存布局|
|**重定位（Relocation）**|修正代码中对符号的引用，使其指向正确的地址|典型例子：函数调用、全局变量引用|
|**文件生成（Output Generation）**|生成最终的可执行文件（ELF）|具备可直接加载的格式|

##### 地址空间分布（静态链接程序）：

```text
虚拟地址空间布局 (x86_64示例):

0x40000: .text (代码段, RX)
0x60000: .data (数据段, RW)
0x61000: .bss  (BSS段, RW)
0x7fffffffe000: stack(栈, 向下增长)
        ↑
        └─ heap (动态分配, 向上增长)
```

> [!NOTE]
> - 在嵌入式系统中 (如 `ARM Soc` )，这些段的布局由链接脚本 (`.ld` 文件) 手动控制，例如：
> 
> ```ld
> SECTIONS {
> 	.text : {*(.text)} > FLASH
> 	.data : {*(.data)} > RAM
> 	.bss  : {*(.bss) } > RAM
> }
> ```
> - 这决定了程序的代码、数据分别烧录或加载到哪块物理内存（如 `NOR/NAND`或 `SDRAM`）
> 



#### 动态链接：

##### 为什么需要动态链接

1. 静态链接存在的问题：

- 静态链接虽然快，运行独立，但存在明显缺点：
	- **空间浪费**：所有可执行文件都包含相同的库代码
	- **维护困难**：更新库需重新编译所有程序
	- **工程复杂**：大型项目修改一个模块需重新链接整个系统

> [!example] 
> 多个程序都使用 `printf()` (来自 `libc`)，若是静态链接，则每个程序都会把 `printf` 代码复制一份，而动态链接只需内存中保留一份


2. 动态链接的目的：

- **共享运行库**：多个进程共用一份 `.so` 代码段，节省物理内存
- **模块化工程**：改动单个库文件即可，无需重新链接整个项目
- **延迟更新与按需更新**：动态库可以再运行时加载/替换

---
##### 动态链接的基本机制：

- 动态链接建立在虚拟内存管理（`Virtual Memory Management`）之上
	- 利用内核提供的 `mmap` 系统调用，将共享库文件直接映射到进程地址空间

###### ELF 文件中的 INTERP 段

   - 动态可执行文件中 `ELF Header` 中有一个特殊段：`PT_INTERP`
   - 其记录了**解释器**（`interpreter`）路径，即动态连接其的位置

```text
PT_INTERP -> /lib64/ld-linux-x86-64.so.2
```

 - 当执行 `execve()` 时，内核检测到此段，会先加载动态链接器（`ld.so`），再由它去加载目标程序依赖的动态库

> [!NOTE] 验证
> - 在 `GDB` 中可通过 `starti` 查看进程程序所调用的第一条指令位置
> 	- 通过 `starti` 可看到第一条指令位于 `ld.so` 内部，而不在主程序内存地址空间
> - 动态链接是基于**虚拟内存映射**实现的，所以可通过进程内存映射查看动态库加载地址
> 	- `info proc mappings` 可查看动态库的加载地址
> 	- `strace ./a.out` (动态链接) 可查看到 `mmap()` 调用加载 `.so` 文件

###### 动态连接器 ld.so 的运行流程：

```text
内核阶段（操作系统做的事）：
	execve("a.out") →
		解析 ELF Header →
		发现 PT_INTERP →
		加载 ld.so 到内存 →
		交由 ld.so 接管执行 →
		
用户阶段（ld.so 做的事情）：
	1. 读取 a.out 的 Program Header
	2. 解析 .dynamic 段（依赖库信息）
	3. 使用 mmap() 加载所需共享库
	4. 构建全局符号表（查找 printf 等符号）
	5. 执行重定位（修正地址引用）
	6. 调用程序入口函数（_start → main）

```

- `ld.so` 实际上就是“用户态加载器”，负责在运行时修正和绑定符号

###### 动态链接与虚拟内存的关系：

- 动态链接能够实现“只加载一次、进程间共享”，是依赖于虚拟内存机制的：
	- 动态库是把文本“映射入进程的虚拟地址空间”，进程的虚拟页被建立为指向该文件所在的物理页（通过内核页缓存）

> [!Tip] 
> 当动态库被加载时，内核使用 `mmap()` 将库文件的内容**映射到进程的虚拟地址空间**，使得该进程虚拟页指向文件内核页缓存对应的物理页（即进程的虚拟地址被设置为“指向”真实的库文件内容），多个进程映射同一文件的相同偏移时，内核可以使它们 **指向同一份物理页**（实现共享），只有在可写且写入时，内核才为触发写操作的进程给出独立的物理页（[[进程与线程#^ad1727 | 写时复制]]）
> ```ruby
> 磁盘文件: /lib/libc.so  (文件偏移)
>         │
>         ├── mmap(offset=0x1000) in proc A → 虚拟地址 0x7f00_1000
>         │           ↳ 对应 page cache（页缓存） 的 物理帧 PFN_X
>         │
>         ├── mmap(offset=0x1000) in proc B → 虚拟地址 0x7f20_1000
>                     ↳ 也指向相同的物理帧 PFN_X  （只读代码段）
> ```


1. **内存映射（`mmap`）：**

- 动态库通过 `mmap` 映射进地址空间
- 文件的 `.text`（只读 diamagnetic）段可悲多个进程共享同一份物理页
- `.data` 段可被标记为写时复制 (`Copy on Write`)
	-  实际物理内存中 `.text` 只会有一份，`.data` 会进行写时复制

---
2. **延迟加载（`Lazy Allocation`）：**

- 系统不会在映射时立即分配物理内存
- 当 `fork()` 创建子进程时，父子进程共享所有内存页（只读方式）；
- 当访问到对应的虚拟页，有写操作时，会触发 `page fault`（缺页异常），内核才真正复该页

---
3.  **页共享与回收：**

- 系统可扫描内存页：
    - 发现重复的只读页：合并
    - 发现“冷”页：可压缩或交换（`swap`）
	    - `Windows` 系统磁盘管理中会存在几个 `G` 左右的分区，该分区即系统压缩不常用 `cold` 页，进行压缩存放的分区
- 硬件辅助：`Access/Dirty` 位跟踪页面访问状态。


---
##### 动态链接的实现机制：

###### 位置无关代码（PIC：Postion Independent Code）

- 动态链接库需要在任意地址加载，因此其代码必须是位置无关的
	- 代码段中不会直接使用绝对地址
	- 而是通过 “相对寻址 + 查表” 方式访问函数与变量
	- 编译时使用 `-fPIC` 选项生成

```asm
call *TABLE[print@systab]
```
- 实际运行时，由 `ld.so` 修改该表项，使其指向真正的函数地址

###### GOT/PLT 表机制（全局偏移表和过程链接表）

|表结构|功能|
|---|---|
|**GOT (Global Offset Table)**|存放全局变量与函数的实际地址|
|**PLT (Procedure Linkage Table)**|存放函数调用的跳转入口|

**调用过程：**

1. 编译时函数调用变为 `call PLT[printf]`
2. 程序首次执行时，`PLT` 跳转到 `ld.so`
3. `ld.so` 查找 `printf` 实际地址，并填入 `GOT`
4. 之后再调用时，直接命中 `GOT`

> [!error] 动态链接历史遗留问题
> 对于 ```extern int x```，我们不能“直接跳转”
> - x = 1, 同一个 `.so` (或 `executable`)
> ```asm
> 	mov $1, offset_of_x(%rip)
> ```
> - x = 1, 另一个 `.so`
> ```asm
> 	mov GOT[x], %rdi  ,取全局变量地址
> 	mov $1, (%rdi)
> ```
> - 处理的不是很好：
> 	- `-fPIC` 默认会为所有 `extern` 数据增加一层间接访问


---
##### 动态库加载与符号解析流程

1. **程序启动** → ELF Header 指向解释器（`ld.so`）；
2. **ld.so 初始化** → 读取依赖 `.so` 文件；
3. **mmap 加载** → 映射共享库；
4. **构建符号表** → 解析 `.dynsym` 和 `.dynstr`；
5. **重定位** → 修正函数与全局变量地址；
6. **运行初始化函数**（如 C 库的 `_init()`）；
7. **跳转入口点** → 执行主程序。


##### LD__PRELOAD：动态链接库一个神奇的机制：

- 在 `Linux` 系统中，`LD_PRELOAD` 是一个环境变量，可以允许在程序运行时，***强制先加载指定的共享库（`.so` 文件）***，而不是默认系统库

> [!NOTE] 简单理解
> `Linux` 动态链接器（`ld_linux.so`）在启动程序时，会自动取加载所需的动态库，而 `LD_PRELOAD` 可以插队，让我们自己所指定的库"先一步"被加载

###### 工作机制：

- 正常运行一个程序：

```bash
./a.out
```

- 系统加载流程：
	- 内核加载可执行文件
	- 交给动态链接器（`ld_linux.so`）负责解析依赖库
	- 动态链接器：
		- 读取 `ELF` 文件中的依赖信息（`.dynamic` 段）
		- 读取系统默认库路径（如 `/lib`）
	- 加载对应的共享库 `.so` 文件到内存

- 设置 `LD_PRELOAD`：

```bash
LD_PRELOAD=./mylib.so ./a.out
```

- 动态链接器在加载其他库前，会**优先加载**你指定的 `mylib.so`
- 即实现**函数劫持（hook）:**
	- 如果 `mylib.so` 中定义了某些与系统库同名的函数（比如 `malloc()`、`open()` 等），
	- 动态链接器会**优先使用你定义的版本**，覆盖掉原本系统库中的实现。

###### 应用场景：

1.  **监控与分析程序行为**

- 无需修改源码，即可动态收集日志；
- 可统计哪些库函数被调用、调用次数、时间等；
- 可用于性能分析或安全检测。

2.  **函数替换与调试**

- 可以让你替换某些系统函数的行为，而不用修改程序源码。
	- 例如：
		- 替换 `malloc()`：监控程序内存分配情况；
		- 替换 `open()`：记录程序打开了哪些文件；
		- 替换 `read()`、`write()`：调试 I/O 操作。



---
### 加载器（Loader）：

#### 进程加载原理：

- 操作系统执行一个可执行程序时，大致流程如下：

```text
fork() → 创建子进程
execve() → 在子进程中加载可执行文件（替换原来的进程映像）
  ↓
  1. 解析 ELF Header
  2. 按 Program Header 映射各个 PT_LOAD 段到虚拟内存（mmap）
  3. 建立栈、堆、bss 区域
  4. 把控制权交给 e_entry(入口地址) 
```

- 这一步由内核函数 `/fs/binfmt_elf.c::load_elf_binary()` 实现，它会读取 `ELF` 文件，将对应段映射到进程的虚拟内存空间

> [!NOTE] `./a.py` 与 `python3 ./a.py`
> 对于 `python3 ./a.py` 大家会比较熟悉，就是通过 `python3` 解释器去解析 `a.py` 程序并执行。在 `bash` 视角中，`a.py` 实际上是一个文本 `text` 文件，但我们实际还可以通过 `./a.py` 去直接完成该 `python` 文件的解析执行，而这个文件并不是 `.exe` 文件，这又是为什么捏？
> 
> 实际上对于 `Python` 以及 `Shell` 等脚本文件，在文件顶部都会编写一段 `#! /usr/bin/xxxx` 这样的注释代码段。当直接执行脚本文件（即 `./a.py` ）时，在 ` Linux ` 内核中的 `/fs/binfmt_script.c` 中，会解析文件头的 `#!` 并间接执行解释器程序
> - 即当你运行 `./a.py` 时，操作系统接收到指令
> - 操作系统会依照 `execve(A, ["A", "B, C", "any_file"], envp)` 的规则对指令进行转换并传入环境参数
> - 进而转换执行 `/user/bin/env python3 a.py`

#### 加载器主要任务：

- 解析可执行文件头（`ELF Header`）
- 创建进程地址空间
- 加载各个 `PT_LOAD` 段（使用 `mmap`）
- 设置堆栈、参数、环境变量
- 跳转到程序入口（`e_entry`）



# 操作系统上的应用生态：

- 从计算机启动到应用程序运行，整个生态大致分为以下层级：

> **硬件 → 内核（Kernel） → 初始用户空间（Initramfs） → 根文件系统（RootFS） → 系统服务（systemd等） → 应用层（用户应用）**

- 操作系统的核心任务：
	- 为上层应用提供统一的**硬件抽象接口（系统调用层）** 和**运行时环境**

---
### 内核：

#### 内核的作用与启动：

- 内核是操作系统的核心部分（`Core Component`），负责管理硬件资源，并向用户空间提供抽象接口
- 当引导加载程序（如 `U-Boot` 或 `GRUB`）将内核镜像加载到内存后，内核执行以下关键任务：
	- 建立分页机制与内存映射（启用 `MMU`）
	- 初始化中断系统（`IDT/GIC`）
	- 初始化各类驱动（时钟、串口、块设备、网络等）
	- 挂载临时根文件系统（`Initramfs`）
	- 创建第一个用户进程：`init`

> 这一过程意味着：内核将计算机从“裸机执行环境”推进到“多任务、多进程系统”

#### 内核中的"对象"和系统调用：

- 内核中存在许多可被用户态进程操作的对象（`resources/objects`）：
	- 文件（设备 `/dev` 文件、`socket`、管道 `pipe` 等）
	- 进程（`task_struct`）
	- 内存页与虚拟地址空间
	- 信号（`signal`）
	- 中断（`IRQ`）
- 用户程序可通过系统调用（`syscall`）接口与内核交互：
	- 常见接口：`open()` 、`read()` 、`write()`、`mmap()`、`fork()`、`execve()`、`waitpid()`
	- 系统调用是内核与应用交互的唯一正规通道

---
### Linux Kernel 与发行版生态

- `Linux` 只是内核本身，而日常我们所说的 `Ubuntu`、`Fedora` 、`Android` 等其实是 `Linux` 发行版 `Distributions`，在内核之上打包了各种组件和应用

1. 系统工具层：
	-  提供最基础的命令和二进制工具
		- `coreutils`：基本命令（`ls`、`cp`、`rm` 、`cat` 等）
		- `binutils`：二进制构建与链接工具（`as`、`ld` 、`objdump` ）
		- `systemd`：现代 `Linux` 初始化系统和服务管理器（取代旧的 `SysVinit`）

2. 桌面系统层：
	- 基于 `X11` 或 `Wayland` 图形栈：
		- 桌面环境：`Gnome` 、`KDE Plasma`、`XFCE`
		- 移动系统：`Android`（基于 `Linux` 内核 + 自有用户空间架构）

3. 应用层：
	- 面向最终用户的程序，如：
		- 文件管理器、浏览器、`VSCode`、编辑器、播放器等
	- 这些应用运行在用户空间（`User Space`），依靠系统调用与内核交互

> **系统工具与桌面系统的界限模糊** , 但它们的共同特征是: 运行在用户空间, 通过系统调用操作内核资源

---
### Initramfs (初始用户空间)

1. 定义与作用：
	- `Initramfs` 是一个临时的根文件系统（`Root FileSystem`）、由内核在启动时解压到内存中
	- 用于在真正的磁盘根文件系统（`/rootfs`）准备好之前，**提供最小可用环境**

2. 内容和特征：
	- 可以极简（仅包含一个可执行 `bin` 文件）
	- 包含：
		- 基本命令工具集（`busybox`、`ash`）
		- 设备文件（`/dev/console` 、`/dev/null`）
		- 驱动模块和加载脚本
		- 挂载工具（`mount`、`modprobe`、`fsck`）

3. `Initramfs` 的职责
	- 提供 `/dev/console` 供内核日志与输入输出
	- 加载磁盘与网络驱动，使系统能访问真正的根文件系统
	- 挂载并切换到最终的 `rootfs`（通过 `switch_root` 或 `pivot_root`）
	- 启动主初始化程序（`sbin/init` 或 `systemd`）

---
### Linux 系统的启动流程

- [[Linux系统启动流程.canvas|Linux系统启动流程]]

#### 阶段 1：内核引导阶段

1. `Bootloader` (`U-Boot/GRUB`) 加载内核与 `Initramfs` 到内存
2. 内核解压并初始化硬件
3. 内核挂载临时 `Initramfs`
4. 执行 `init` (`Initramfs` 中的启动脚本)

#### 阶段 2：根文件系统接管

1. 解析 `/proc/cmdline` 中的启动参数（如 `root=/dev/mmcblk0p2`）
2. 加载磁盘/网络驱动（真正的根文件系统 `/rootfs` 所在存储介质）
3. 挂载真正的根文件系统 `/`
4. 调用 `switch_root` 或 `pivot_root`，切换到真正根文件系统

#### 阶段 3：用户空间初始化

1. 启动 `/sbin/init` 或 `/lib/systemd/systemd`
2. `systemd` 解析 `.service` 配置文件，启动系统服务（如网络、登录、图形界面）
3. 启动用户应用环境

---
### 操作系统的抽象层次（自底向上）

| 层级     | 主要职责                           | 实例                     |
| ------ | ------------------------------ | ---------------------- |
| 硬件层    | 提供指令集架构（`ISA`）、中断、内存控制器、外设寄存器等 | `ARM`、`x86`、`RISC-V`   |
| 内核层    | 提供抽象：进程、内存、文件系统、网络协议栈、系统调用接口   | `Linux Kernel`         |
| 初始用户空间 | 提供最小命令环境，负责根文件系统切换             | `Initramfs`、`BusyBox`  |
| 系统工具层  | 构建运行时环境                        | `systemd` 、`coreutils` |
| 应用层    | 普通用户程序                         | `shell`、浏览器、编辑器、服务器    |


# 3. 并发控制

## 3.1 多处理器编程 —— 从入门到放弃

### 为什么需要多处理器编程：

- 单核时代的极限：
	- 早期的计算机系统大多只有一个 `CPU`，所有程序指令都在单个处理器上顺序执行，随着硬件发展，单核 `CPU` 的性能受到功耗与频率的物理极限制约（`Power Wall`），于是现代计算机普遍采用**多核（`multi-core`）或多处理器（`multi-processor`）结构**
	- 这意味着：

>  同一个程序现在可能同时被多个处理器执行

 - 于是，问题就来了：
	 - 如何让程序**正确地利用多个 `CPU`**
	 - 该如何处理并发带来的不确定性

---
### 并发与并行

| 概念                | 含义                 | 关键特征              | 关键区别      |
| ----------------- | ------------------ | ----------------- | --------- |
| 并发（`Concurrency`） | 逻辑上的“同时执行”，可能是交替执行 | 操作系统层面（调度）模拟“同时”  | 关注结构与调度   |
| 并行（`Parallelism`） | 物理上的“同时执行”         | 依赖多个 `CPU` 真正同时运行 | 关注性能于硬件能力 |

>  **并发**是**软件设计问题**，**并行**是**硬件执行问题**
>  
>  - 并发问题是操作系统模拟出的“幻觉”，而并行问题则为真实物理世界逻辑，所以当并行问题解决时，并发问题一定解决
>  - 并发是“看起来同时”，并行是“真的同时”


---
### 并发模型的演化：

- 从进程到线程：并发的实现方式

1. **进程**（`Process`）
	- 每个进程拥有独立的地址空间
	- 资源隔离、安全性强
	- **代价大**：上下文切换昂贵，内存占用高、且共享困难


2. **线程**（`Thread`）：
	- 线程属于进程
	- 多个线程共享同一地址空间（代码段、堆、全局变量）
	- 但各自拥有自己的**栈**和**寄存器上下文**
	- 轻量级上下文切换，便于数据共享

>  引入线程之后，程序能以较低代价实现真正的并发执行，但线程间共享进程内存也带来同步与一致性问题（资源抢占）

---
### 程序的状态机模型：

- 我们可以将一个程序抽象为一个「状态机」

1. **单线程程序**：
	- 初始状态：```main(argc, argv, envp)```
	- 状态迁移：执行一条语句（或一条 `CPU` 指令）

2. **多线程程序**：
	- 通过调用 `spawn(fn)` 创建新的“状态机”（线程）
	- 每个线程独立运行自己的指令流
	- 共享全局变量和堆
	- 每次执行时，系统从所有线程中选择一个，执行一条指令

> 多线程程序的执行路径由"线程调度顺序"决定, 这种调度是**非确定性**的

---
### 确定性与非确定性: 并发的根本挑战

1. 单线程的确定性：
	- 单线程程序的执行是确定性的 (`deterministic`)
		- 输入相同
		- 系统调用序列相同
		- 每次运行结果相同

> 操作系统的设计让每个程序都认为自己独享整个计算机

2. 多线程的非确定性：
	- 多线程程序中，每次运行的线程交织顺序可能不一样
		- 这导致结果不唯一
		- 成为**非确定性**（`non-determinism`）
		- 进一步会引发**竞态条件**（`race condition`）问题

- 实例：

```c
iny cnt = 0;

void thread1() {cnt ++;}
void thread2() {cnt ++;}
```

- 其底层执行大致如下：

```pgsql
load cnt
add 1
store cnt
```

- 如果两个线程交叉执行，可能产生：

```
Thread1 load cnt = 0
Thread2 load cnt = 0
Thread1 store cnt = 1
Thread2 store cnt = 1
```

- 最终 `cnt == 1` 而不是 2

---
### 编译器优化带来的陷阱

- 在单线程环境下，编译器的优化是安全的
	- 删除死代码
	- 可以重排指令
	- 只要最终堆系统调用的可观察结果一致即可
- 但在多线程下，这种假设不成立

- 例如：

```c
while(!flag);
```

- 在多线程环境中，编译器可能会优化掉循环中的 `flag` 重新读取（认为它不会被修改），导致循环永远无法退出

> 编译器认为程序是单线程的，不会预期另一个线程修改变量

- 对策：

1. 使用 `volatile` 声明共享变量，禁止编译器优化：

```c
volatile int flag;
while(!flag);
```

2. 插入空汇编语句，告诉编译器该处有"内存副作用"：

```c
asm volatile("" ::: "memory");
```

> 不过，这些仅仅防止编译器优化，但并不能防止 `CPU` 内部乱序处理

---
### CPU 的乱序执行和宽松内存模型

#### 乱序执行

- 现代 `CPU` 非常聪明：他不会老老实实“按顺序”执行你的指令
- 为了性能，它会乱序执行（`Out-of-Order Execution`）
	- 允许不同地址的 `load/store` 重排
	- 提前执行可以确定结果的指令
	- 将结果暂存在本地缓存中延迟写回
- 这种行为将导致：
	- 一个线程的写操作对另一个线程**可能暂时不可见**
	- 出现“读到旧值”的情况

#### 宽松内存模型（Relaxed Memory Model）：

- 在理想模型中（顺序一致性 `Sequential Consistency`）：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251111212930229.png" width="500px"/>
</div>

> 所有线程看到的内存修改顺序是一致的

- 但在实际中：
	- 每个 `CPU` 都有自己的缓存
	- `store` 先写入缓存
	- 缓存异步同步到主内存
	- 其他 `CPU` 可能仍在读旧内存

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251111213906893.png" width="300px"/>
</div>

> 不同线程所看到的世界可能是不一致的

---
### 内存屏障（Memory Barrier）：

为控制乱序和缓存延迟，`CPU` 提供了**内存屏障指令**（`fence/barrier`）:

| 指令       | 含义                   |
| -------- | -------------------- |
| `mfence` | 禁止所有 `load/store` 重排 |
| `sfence` | 禁止 `store` 重排        |
| `lfence` | 禁止 `load` 重排         |
- 在高级语言中，对应为：

```c
atomic_thread_fence(memory_order_seq_cst);
```

- 通过这些屏障, 我们可以强制保证：某些关键内存操作的执行顺序

---
### 并发同步的正确处理

- 操作系统与语言层面提供了更高层的抽象，用于安全地控制线程间访问：

| 同步机制                       | 功能       | 底层实现              |
| -------------------------- | -------- | ----------------- |
| 互斥锁（`Mutex`）               | 互斥访问临界区  | 原子指令 + 内存等待队列     |
| 信号量（`Semaphore`）           | 控制资源数量   | 计数机制              |
| 条件变量（`Condition Variable`） | 等待某一条件发生 | 结合互斥锁使用           |
| 原子操作（`Atomic`）             | 硬件级原子读改写 | `lock cmpxhg` 等指令 |

---
### TLB 与多核内存访问的一致性：

- 除了缓存，还有另一个关键部件：`TLB`（`Translation Lookaside Buffer`）
	- 它缓存虚拟地址到物理地址的映射
	- 每个 `CPU` 核都有独立的 `TLB`
	- 若内存映射（页表）被修改，例如 `munmap()` 、`mprotect()`:
		- 其他 `CPU` 核的 `TLB` 可能仍保存旧的映射
		- 访问“已经释放”的内存
- 为保持一致，操作系统必须：
	- 通过 **TLB Shutdown**机制，让所有 `CPU` 的 `TLB` 失效并重新加载
	- 虽然代价昂贵，但这是多核系统中**保证内存一致性**的必要手段

---
### 硬件与语言的多层解决方案：

| 层级    | 主要机制                                             | 目标          |
| ----- | ------------------------------------------------ | ----------- |
| 硬件层   | `Cache` 一致性协议（如 `MESI`）、内存屏障                     | 保证缓存一致性与可见性 |
| 编译器层  | 禁止重排、`volatile` 语义                               | 保证程序执行语义    |
| 操作系统层 | 调度、锁、`TLB` 同步                                    | 保证线程/进程正确执行 |
| 语言层   | C++11 `std::atomic`、Java `volatile/synchronized` | 提供统一的内存模型抽象 |

---

- 多处理器编程的核心问题：

> 如何在性能与正确性之间取得平衡

- 并发使得程序非确定
- 编译器与 `CPU` 的优化进一步破环顺序
- 共享内存与缓存机制引入新的复杂性
- 解决之道：
	- 使用同步机制
	- 理解内存模型
	- 避免“玩弄共享内存”
	- 让系统替你维护“顺序幻觉”




## 3.2 互斥




# 命令：

### `bash`:


<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251020154404021.png" width="500px"/>
</div>

```bash
strace  # 输出程序系统调用函数
```

```bash
pmap 进程PID # 查看进程程序的地址空间映射
```

```bash
make -n   # 编译时，查看实时执行（阅读Markfile文件可用）
```

```bash
stty - a	# 查看按键绑定
```

```bash
elfcat		# 查看 ELF 文件信息
```

```bash
ldd [option] iflle	# 打印 file 所依赖的动态链接库
```


### `gdb`:

```bash
info proc mappings 		# 查看当前进程内存映射
```

#### 图形化

```bash
# 布局控制：
(gdb) layout src     # 显示源代码窗口
(gdb) layout asm     # 显示汇编窗口  
(gdb) layout split   # 同时显示源代码和汇编
(gdb) layout regs    # 显示寄存器窗口

# 焦点切换：
(gdb) focus cmd      # 焦点切换到命令窗口
(gdb) focus src      # 焦点切换到源代码窗口
(gdb) focus asm      # 焦点切换到汇编窗口

# 刷新屏幕：
(gdb) refresh        # 刷新 TUI 显示
```

