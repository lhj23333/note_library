
#### 1. 进程文件句柄

- 在 Linux 系统中，用户程序进行系统调用时，通过会借助 `glibc` 所提供的 `ABI` (`application binary interface`) 进行内核操作，例如：`open`、`write`、`read` 等等。在实际代码调用中，会通过 `fd` 来作为文件操作的实例化对象，该对象类型为文件句柄类型 （整型）。

```c
int fd = open("./xxx/xxx");
```

- 当我们进行编译运行代码程序后，内核会为该程序创建进程。同时我们可以依据 `PID` 进程号，在根文件系统 `./proc/(PID)/`（`proc` 为 `process` ）下找到在程序执行过程中所创建的进程文件描述符 `fd`（`file description`）

```shell
./test	&			# 在后台运行 test 应用进程
__________
[1] xxxx		 	# 终端输出打印进程号 PID
```

- 通过 `cat fd` 命令，我们可以查看该文件描述符 `fd` 下所包含的文件句柄
	- 文件句柄：句柄所对应结构体事实上为一个指针，指向资源文件（换句话说，就是句柄是资源文件的抽象实例）

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250228090055475.png" width = "800px"/>
</div>

- 在上图可以看到，除了我们执行程序运行所调用到的 `/home/book/fileio/....`（句柄号为 `fd = 3`）, 在前面存在有 `fd = 1、2、3` 三个句柄，这三个句柄是进程创建会默认创建的。分别是
	- `fd = 0`：标准输入 `stdin`
	- `fd = 1`：标准输出 `stdout`
	- `fd = 2`：标准错误 `stderr`
	- `fd = 3`：`ABI` `open` 操作

- 当你通过调用 `open` 打开文件时，首先内核会获取一个未使用的句柄号 `fd = 3, 4 ...` 该句柄是一个指针会指向操作文件 `file` 结构体（在后面会详细进行讲解）

#### 2. 进程资源管理解析：

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250227214033.png" width = "700px"/>
</div>

- 在 Linux 内核中，`task_struct` 是用来描述一个进程的主要数据结构，每个进程都有一个对应的 `task_struct`, 其内包含了进程的各种信息。如进程 `ID` (`PID`)、调度信息、内存管理信息等。同时，在每个进程 `task_struct` 中都会有一个 `*file` 指针指向 `files_struct` 结构体

```c
struct files_struct *files;
```

- `files_struct` 是每个进程用来管理文件描述符的结构。它包含了进程打开的文件相关的各种信息。

```c
struct files_struct {
	atomic_t count;			// 引用次数
	struct fdtable *fdt;	// 文件描述符表
	.....
}
```

- `files_struct` 中的 `fdt` 字段是一个指向 `fdtable` 结构的指针。在 Linux 系统中，系统是通过 `fdtable` 结构来管理进程的文件描述符表 (`FD table`)

```c
struct fdtable {
	unsigned int max_fds;			// 最大文件描述符数
	struct file **fd;				// 文件描述符表，数组形式
	struct file **fdtable;			// 文件结构指针表
}
```

- 在 `fdtable` 结构体中存在元素 `fd`，该元素是一个指向 `file` 结构的指针，`file` 结构内描述具体打开文件的结构，就是我们前面讲述的进程运行所需调用的文件（`stdin`、`stdout`、`stderr` 等）。每当进程打开一个文件时，内核会创建一个 `file` 结构体实例。

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250228102829001.png" width = "700px"/>
</div>
<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250228103004392.png" width = "700px"/>
</div>

- 哪么?? 在 `file` 是如何去调用系统资源的呢 ？？
- 我们知道，在硬件上，文件资源以二进制形式存储在板载存储器中。当需要访问这些资源时，系统通过操作程序计数器（`pc`）指向相应的存储地址，从而实现对资源的调用和访问
- 在 `file` 结构体中，包含了文件的具体信息（如文件指针、文件操作函数等）

```c
struct file{
	struct path f_path;				// 文件路径
	struct file_operations *f_op;	// 文件操作指针
}
```

- 其中的 `f_ops` 指向文件资源在存储器中的地址，当我们调用 `read(fd, buff, size_of)` 函数去读取文件资源指定字节长度时，事实上是在 `f_op` 地址上读取 `sizeof` 长度字节并将信息存储到 `buff` 缓存中（此时 `f_op = f_op + sizeof`）

> [!NOTE]  `int main()` 传参
> 在标准 `int main()` 中，函数原型是 `int main(argc, *argv, env){}`
> 其中传参类型解释为：
> - `argc`：运行命令单词总数
> - `*argv`：运行命令单词 `char` 型列表 （首位序号为“0”）
> - `env`：运行命令所需环境（一般不传入，系统会自动调用系统环境）
> 


