---
share_link: https://share.note.sx/w7xbnwzb#W/6bs7MU4m7jx+w6orKvS5PDhUQtOfu3dwVcdb32orw
share_updated: 2025-02-13T16:46:34+08:00
---
## 1.  内存管理

- `kmalloc()`：分配内核动态内存，类似 `malloc`
```c
void *kmalloc(size_t size, gfp_t flags); 
```
- 参数解释：
	- `size`：需要分配的内存大小 (字节数)
	- `flags`：内存分配的标志，用于指定内存分配的方式，常见的标志有：-
		- `GFP_KERNEL`：正常的内存分配标志（用于内核代码）。
		- `GFP_ATOMIC`：在中断上下文或不可阻塞的情况下分配内存（不能被阻塞）。
		- `GFP_USER`：用于从用户空间分配内存（用户空间的内存分配）。
		- `GFP_DMA`：为 `DMA` 分配内存（用于设备和内存之间的数据传输）。
- 返回值：
	- 返回分配成功的内存块的指针。如果分配失败，返回 `NULL`
- 示例：
``` c
void *ptr;
ptr = kmalloc(1024, GFP_KERNEL);    // 分配 1024 字符内存
if(! ptr){
	pr_err("Memory allocation failed!\n");
	return -ENOMEN;
}
```

- `kfree()` ：释放由 `kmalloc` 分配的内存。
``` c
void kfree(const void *ptr);
```
- 参数解释：
	- `ptr`：指向要释放内存的指针
- 示例：
```c
kfree(ptr);		// 释放之前通过 kmalloc 分配的内存
```

- `copy_to_user()`：从内核空间拷贝数据到用户空间
``` c
long copy_to_user(void __user *to, const void *from, unsigned long len);
```
- 参数解释：
	- `to`：指向用户空间的目标地址
	- `from`：指向内核空间的源地址
	- `len` ：需要拷贝的数据字节长度
- 返回值：
	- 成功时，返回 0
	- 失败时，返回未能拷贝的字节数
- 示例：
```c
char *kernel_data = "Hello, user!";
char __user *user_data = buffer; 	// 用户空间缓冲区
int result = copy_to_user(user_data, kernel_data, strlen(kernel_data));
if (result){
	pr_err("Failed to copy data to user space\n");
}
```

- `copy_from_user()`：从用户空间拷贝数据到内核空间
``` c
long copy_from_user(void *to, const void __user *from, unsigned long len);
```
- 参数解释：
	- `to` ：指向内核空间的目标地址
	- `from`：指向用户空间的源地址
	- `len`：需要拷贝的数据字节长度
- 返回值：
	- 成功时，返回 0
	- 失败时，返回未能拷贝的字节数

- `get_user()`：从用户空间获取变量数据
```c
int get_user(var, ptr)
```
- 参数解释：
	- `var`：要从用户空间数据读取到的变量
	- `ptr`：指向用户空间变量的指针
- 返回值：
	- 成功时，返回 0
	- 失败时，返回非零值表示错误（如访问用户空间的非法地址）
- 示例：
```c
int value;
int result = get_user(value, user_ptr);
if(result){
	pr_err("Falied to get data from user space\n");
}
```

- `put_user()`：传输变量数据到用户空间
```c
int put_user(value, ptr)
```
- 参数解释：
	- `value`：要写入到用户空间的变量
	- `ptr`：指向用户空间目标变量的指针
- 返回值：
	- 成功时，返回 0
	- 失败时，返回非零值表示错误

## 2. 进程调度：

- `schedule()`: 控制执行进程主动让出 CPU，触发调度
```c
void schedule(void);
```
- 使用场景：
	- 通常在需要主动让出 CPU 的地方使用，例如，当一个进程没有更多可执行的工作，或者它正在等待某些资源时（如 `I/O` 操作完成）。
		- 设备驱动开发过程中，等待 `I/O` 操作完成时，驱动会让出 CPU 等待操作完成
- 示例：在
```c
if (condiction_to_wait){
	schedule();				// 当前进程主动让出 CPU，阻塞挂起
}
```

- `msleep()`：让进程在指定的毫秒数内休眠，这段时间进程会挂起 （低精度）
```c
void msleep(unsigned int millisecs);
```
- 参数解释：
	- `millisecs`：休眠时间（以毫秒为单位）
- 使用场景：
	- 用于设备驱动中，当需要延迟一定时间后进行操作时
		- 等待硬件设备的状态变化
- 示例：
```c
msleep(100);
```

- `usleep_range()`：允许进程休眠一个微小的时间范围 (以微秒为单位)
```c
void usleep_range(unsigned long min, unsigned long max);
```
- 参数解释：
	- `min`：最短休眠时间（以微秒为单位）
	- `max`：最长休眠时间（以微秒为单位）
- 说明：
	- 该函数将使进程在一个特定的时间范围内休眠。在 `min` 到 `max` 之间的随机时间内被唤醒
- 使用场景：
	- 高精度定时操作
- 示例：
```c
usleep_range(500, 1000);		// 在500到1000微秒之间随机休眠 
```

- `mdelay()/udelay()`：忙等待方式延迟，不建议长时间阻塞情况
```c
void mdelay(unsigned long msecs);		// 毫秒级延迟
void udelay(unsigned long usecs);		// 微妙级延迟
```
- 参数解释：
	- `msecs`：延迟时间（单位：毫秒）
	- `usecs`：延迟时间（单位：微妙)
- 说明：
	- `mdelay()` 和 `udelay()` 都是忙等待延迟函数，它们会阻塞当前进程（即在该延迟函数下，进程不会进入休眠挂起，而是会继续占用 CPU，调度器无法进行下一步调度）

- `wake_up()`：唤醒等待队列中阻塞的任务
```c
void wake_up(wait_queue_head_t *q);
```
- 参数解释：
	- `q`：等待队列的头指针。（条件阻塞进程队列）
- 说明：
	- `wake_up()` 会将队列中一个或多个阻塞进程唤醒，使其进入就绪状态
- 使用场景：
	- 通常在内核一些事件完成时，需要进行下一步调用其他进程时使用
		- 在驱动程序中，当设备准备好数据时，可以唤醒请等待的进程：
- 示例：
```c
wake_up(&my_wait_queue);				// 唤醒等待队列中的任务		
```


- `wait_event()`：阻塞当前进程，当前进程加入等待队列，直到 `condition` 为真时，进程才会被唤醒
```c
void wait_event(wait_queue_head_t q, condition);
```
- 参数解释：
	- `q`：等待队列的头指针。
	- `condition`：条件表达式，表示唤醒的触发条件
- 说明：
	- 当前进程会被挂起，直到 `condition` 条件为真时，进程会从等待队列中被移除，并进入就绪队列进行调度
- 示例：
```c
wait_event(my_wait_queue, condition == TRUE);
```


- `wait_event_interruptible()`：允许进程在阻塞状态被中断进行信号处理
```c
wait_event_interruptible(wait_queue_head_t q, condition);
```
- 参数解释：
	- `q`：等待队列的头指针
	- `condition`：条件表达式，表示唤醒的触发条件】
- 返回值：
	- 条件成功唤醒时，返回 0
	- 进程被信号中断时，返回 `-ERESTARTSYS`
- 说明：
	- 当前进程会被挂起，直到 `condition` 条件为真时，进程会从等待队列中被移除，并进入就绪队列进行调度执行（与 `wait_event` 相同），但在进程阻塞阶段时，当进程接收到了信号，`wait_event_interruptible` 会中断进程，允许进程进行信号处理
- 示例：
```c
int ret = wait_event_interruptible(my_wait_queue, condition == TRUE);
if(ret == -ERESTARTSYS){
	// 进行信号中断处理
}
```
## 3. 设备文件操作：

- `cdev_init()`：初始化字符设备结构体
```c
void cdev_init(struct cdev *cdev, const struct file_operations *fops);
```
- 参数解释：
	- `cdev`：指向字符设备结构体 (`struct cdev`) 的指针
	- `fops`：指向设备操作结构体 (`file_operations`) 的指针
- 说明：
	- 该函数初始化 `cdev` 结构，主要是将设备的操作与 `file_operations` 结构关联起来
- 示例：
	- 注册设备之前，通常会使用 `cdev_init()` 初始化设备结构体
```c
struct cdev my_cdev;				// 设备信息结构体
cdev_init(&my_cdev, &my_fops);		// 将操作结构体与设备操作进行关联
```

- `register_chrdev()`：注册字符设备
```c
int register_chrdev(unsigned int major, const char *name, const struct file_operations *fops);
```
- 参数解释：
	- `major`：主设备号 （`major` 为 0 时，内核会自动分配一个主设备号）
	- `name`：设备名称，用于在内核中标识设备
	- `fops`：`file_operations` 结构指针，即设备支持文件操作接口
- 返回值：
	- 成功时，返回 0
	- 失败时，返回负值
	- 返回 `-EBUSY` 表示主设备号已经被占用
- 说明：
	- **`register_chrdev()`** 是一种较老的设备注册方法。它不仅注册字符设备，还隐式地为设备分配主设备号并初始化设备的文件操作集。
	- `register_chrdev()` 更简单，但已经过时，通常不推荐使用。而 `cdev_add()` 和 `alloc_chrdev_region()` 的组合则是更现代的做法，适用于需要动态分配设备号并初始化设备的情况。
- 注册设备后，内核会为该设备在 `/dev` 目录下创建一个设备文件
```c
int ret = register_chrdev(0, "mu_device", $my_fops);
if(ret < 0){
	pr_err("Failed to register character device\n");
	return ret;
}
```

- `unregister_chrdev()`：注销字符设备，将设备从内核中移除，并释放设备相关资源
```c
void unregister_chrdev(unsigned int major, const chat *name);
```
- 参数解释：
	- `major`：主设备号，表示要注销的设备
	- `name`：设备名称
- 使用场景：
	- 进行设备驱动卸载
```c
unregister_chrdev(major, "my_devic");
```

`alloc_chrdev_region()`：动态分配主次设备号
```c
int alloc_chrdev_region(dev_t *dev, unsigned baseminor, unsigned count, const char *name);
```
- 参数解释：
	- `dev`：指向 `dev_t` 变量的指针，用来存储分配的设备号
		- `dev_t` 是设备主次设备号组合
			- `MAJOR(dev)`：主设备号宏
			- `MINOR(dev)`：次设备号宏
	- `baseminor`：指定分配的次设备号范围的起始值，通常为 0
	- `count`：要分配的设备号的数量
	- `name`：设备名称
- 返回值：
	- 成功时，返回为 0
	- 失败时，返回负值
	- 返回 `ENOMEM`，表示内存不足
- 说明：
	- 用于内核动态分配主设备号以及对应次设备号
- 示例：
```c
dev_t dev;
int ret = alloc_chrdev_region(&dev, 0, 1, "my_device");
if(ret < 0){
	pr_err("Failed to allocate device number\n");
	return ret;
}
```

- `cdev_add()`：注册设备，将字符设备添加到内核，即将设备与设备号关联起来，并使其可用
```c
int cdev_add(struct cdev *cdev, dev_t dev, unsigned count);
```
- 参数解释：
	- `cdev`：描述设备及其操作结构体
	- `dev`：设备号，通常由 `alloc_chrdev_region()` 进行分配
	- `count`：设备数量
- 返回值：
	- 成功时，返回 0
	- 失败时，返回负值
- 说明：
	- 注册设备后，内核会为该设备在 `/dev` 目录下创建一个设备文件，供用户应用程序进行操作
- 示例：
```c
cdev_add(&my_cdev, dev, 1); 		// 将 my_cdev 设备添加到内核
```


- `cdev_del()`：将字符设备移除内核，通常在设备卸载时调用
```c
void cdev_del(struct cdev *cdev);
```
- 参数解释：
	- `cdev`：指向要删除的 `cdev` 结构体
- 说明：
	- 该函数会从内核中移除指定的字符设备，并释放相关资源。此操作通常在设备卸载时进行
- 示例
```c
cdev_del(&my_cdev);			// 移除设备
```

## 4. 文件操作：

- 文件操作函数通常绑定定义在 `struct file_operations` 结构体中
- `open()`：用户进程打开设备文件
```c
int open(struct inode *inode, struct file *file);
```
- 参数解释：
	- `inode`：指向设备文件对应的 `inode` 结构，该结构体重要成员：`dev` 设备号
	- `file`：指向 `struct file` 结构。该结构体包含文件操作的相关信息（文件描述符、文件操作函数等）
- 返回值：
	- 成功时，返回 0
	- 失败时，返回错误
- 说明：
	- 调用该函数可打开设备，可以在此函数中初始化设备状态、分配资源或执行与设备相关的任务（例如，打开设备时，初始化硬件设备）
- 示例：
```c
int my_device_open(struct inode *inode, struct file *file){
	pr_info("Opening device\n");
	// 进行设备相关操作
	return 0;
}
```


- `release()`：用户进程关闭设备文件
```c
int release(struct inode *inode, struct file *file);
```
- 参数解释：
	- `inode`： 指向设备文件对应的 `inode` 结构。
	- `file`：指向 `struct file` 结构，表示要打开的设备文件
- 返回值：
	- 成功时，返回 0
	- 失败时，返回负值
- 说明：
	- 该函数在设备文件被关闭时被调用，通常会释放 `open()` 中分配的资源，如解除硬件锁定，关闭设备的 `I/O` 操作等
- 示例：
```c
int my_device_release(struct inode *inode, struct file *file) {
    pr_info("Releasing device\n");
    // 执行硬件清理操作
    return 0;
}
```

- `read()`: 从设备文件中读取数据到用户空间
```c
ssize_t read(struct file *file, struct char __user *buf, size_t count, loff_t *ppos);
```
- 参数解释：
	- `file`：指向 `struct file` 结构，表示要打开的设备文件
	- `buf`：指向用户空间的缓冲区，读取的数据将存储在这里
	- `count`：表示读取的字节长度
	- `ppos`：指向文件指针的位置（偏移量），表示从文件的哪个位置开始读取
- 返回值：
	- 返回读取的字节长度
	- 如果设备没有数据可供读取，通常返回 0，表示 `EOF`（文件结束）
- 说明：
	- 由于操作需要将设备文件数据读取到用户空间，设备文件在内核态，所以函数经常与 `copy_to_user()` 函数联用
- 示例：
```c
ssize_t my_device_read(struct file *file, char __user *buf, size_t count, loff_t *ppos) {
    copy_to_user();   // 从设备获取数据并复制到用户空间
    return count;  		  // 返回成功的字节数
}
```

- `write()`：向设备文件中写入数据
```c
ssize_t write(struct file *file, const char __user *buf, size_t count, loff_t *ppos);
```
- 参数解释：
	- `file`：指向 `struct file` 结构，表示要打开的设备文件
	- `buf`：指向用户空间的缓冲区，写入的数据将存储在这里
	- `count`：表示读取的字节长度
	- `ppos`：指向文件指针的位置（偏移量），表示从文件的哪个位置开始读取
- 返回值：
	- 返回写入的字节长度，成功时返回正值
	- 失败时返回负值
- 说明：
	- 同 `read()` 函数，该函数经常与 `copy_from_user()` 函数联用
- 示例：
```c
ssize_t my_device_write(struct file *file, const char __user *buf, size_t count, loff_t *ppos){
	copy_from_user(); 	// 向设备文件写入数据
	retrun count;		// 返回写入的字节长度
}
```

- `llseek()`：修改文件读写指针位置
```c
loff_t llseek(struct file *file, loff_t offset, int whence);
```
- 参数解释：
	- `file`： 指向 `struct file` 结构，表示要打开的设备文件
	- `offset`：指针偏移量，用于修改文件指针的位置
	- `whence`：指定如何计算新的文件指针位置，常见的值有：
		- `SEEK_SET`：从文件开头计算偏移
		- `SEEK_CUR`：从当前指针位置计算偏移
		- `SEEK_END`：从文件结尾计算偏移
- 返回值：
	- 返回新的文件指针位置
- 说明：
	- 对于字符设备，文件偏移量通常不具备意义
- 示例：
```c
loff_t my_device_llseek(struct file *file, loff_t offset, int whence) {
    // 根据 whence 和 offset 计算新的偏移位置
    return new_offset;
}
```

- `mmap()`：内存映射机制，允许将设备的内存区域映射到用户空间，从而通过指针直接访问设备内存，而不需要通过读写系统调用
```c
mmap(struct file *filp, struct vm_area_struct *vma);
```
- 参数解释：
	- `filp`：指向 `struct file` 结构，表示要映射的文件
	- `vma`：指向`struct vm_area_struct` 结构，描述了内存映射区域的属性
- 说明：
	- `mmap()` 允许用户程序将设备的内存映射到进程的虚拟地址空间，这样用户就可以通过内存访问设备数据，不需要 `read()` 和 `write()` 等系统调用
	- 内存映射是与硬件设备进行高速数据交换的一种常见方式，特别是在需要频繁数据交互的场景中
- 示例：
```c
int my_device_mmap(struct file *filp, struct vm_area_struct *vma) { 
	// 将设备内存映射到用户空间
	return 0; 
}
```

## 5. 同步机制：

### 互斥锁 (`Mutex`)：

- 参数 `lock`：指向 `struct mutex` 的指针。该结构体包含了互斥锁的状态。
- `mutex_init()`：初始化互斥锁
```c
void mutex_init(struct mutex *lock);
```

- `mutex_lock()`：请求获取互斥锁
```c
void mutex_lock(struct mutex *lock);
```
- 说明：请求获得互斥锁。如果该锁已经被其他进程/线程持有，则调用的进程/线程将被阻塞，直到锁被释放。

- `mutex_unlock()`：解锁互斥锁
```c
void mutex_lock(struct mutex *lock);
```

- `mutex_tryclock()`：尝试获取互斥锁
```c
int mutex_trylock(struct mutex *lock);
```
- 返回值：
	- 如果成功获取锁，则返回 0
	- 如果锁已经被其他进程/线程占用，则返回 `-EBUSY`
- 示例：
```c
struct mutex my_lock;
mutex_init(&my_lock);		// 互斥锁初始化

mutex_lock(&my_lock);		// 获取锁，如果锁被占用则阻塞等待
// 进程/线程访问资源进行操作
mutex_unlock(&my_lock);		// 释放锁
```

### 自旋锁 (`Spinlock`)：

- 参数 `lock`：指向 `spinlock_t` 的指针
- `spin_lock_init()`：初始化自旋锁
```c
void spin_lock_init(spinlock_t *lock);
```

- `spin_lock()`：获取自旋锁
```c
void spin_lock(spinlock_t *lock);
```
- 说明：如果自旋锁当前没有被占用，则会立即获得锁；如果锁已经被占用，当前进程/线程会一直自旋，直到锁被释放。

- `spin_unlock()`: 释放自旋锁
```c
void spin_unclock(spinlock_t *lock);
```
- 示例：
```c
spinlock_t my_spinlock;
spin_lock_init(&my_spinlock);		// 自旋锁初始化

spin_lock(&my_spinlock);			// 获取自旋锁，一直等待
// 访问资源执行操作
spin_unlock(&my_spinlock);			// 释放锁
```

### 读写锁 (`Read/Write Lock`)：

- 参数 `lock`：指向读写锁（`rwlock_t`）类型的指针
- `rwlock_init()`：初始化读写锁
```c
void rwlock_init(rwlock_t *lock);
```
- `read_lock()/read_unlock()`:：获取/释放读锁
```c
void read_lock(rwlock_t *lock);
void read_unlock(rwlock_t *lock);
```
- `write_lock()/write_unlock()`：获取/释放写锁
```c
void write_lock(rwlock_t *lock);
void write_unlock(rwlock_t *lock);
```

### 信号量（`Semaphore`）：

- `sema_init()`：初始化信号量
```c
void sema_init(struct semphore *sem, int val);
```
-  参数解释：
	- `sem`：指向信号量的指针
	- `val`：初始值，表示当前信号量的可用资源数量，通常为非负整数

- `down()`：获取信号量，通常也叫做 `P` 操作
```c
void down(struct semphore *sem);
```
- 参数解释：
	- `sem`：指向信号量的指针
- 说明：
	- 如果信号量的值大于 0，则将其减 1，并成功获取资源；如果信号量的值为 0，当前进程会被阻塞，直到信号量的值变为正。

- `up()`：释放信号量，通常也叫 `V` 操作
```c
void up(struct semaphore *sem);
```
- 参数解释：
	- `sem`：指向信号量的指针
- 说明：
	- 信号量值加 1，释放一个资源并唤醒可能等待该信号量的进程
- 示例：
```c
struct semaphore my_sem;
sema_init(&my_sem, 1);

down(&my_sem);  // 获取资源
// 执行操作
up(&my_sem);    // 释放资源
```
## 6. 中断处理：

- `request_irq()`：注册中断号，并为该中断号指定一个中断处理函数
```c
int request_irq(unsigned int irq, irq_handler_t handler, unsigned long irqflags, const char *devname, void *dev_id);
```
- 参数解释：
	- `irq`：中断号。设备中断唯一标识 (`IRQ`)
	- `handler`：中断处理函数，内核会调用该函数处理中断
	- `irqflags`：中断标志位，用于设置中断行为
		- `IRQF_SHARED`：中断号可共享
	- `devname`：设备名称，通常为设备描述符
	- `dev_id`：设备标识符，通常指向设备驱动的数据结构
```c
// 中断处理函数
irqreturn_t handler(int ire, void *dev_id);
```
- 返回值：
	- 成功返回 0，失败返回负值
	- `-EINVAL` 表示无效中断号
- 示例：
```c
int my_irq_handler(int irq, void *dev_id){
	pr_info("IRQ %d triggered\n", irq);
	// 中断处理逻辑
	return IRQ_HANDLED;
}

// 请求中断
request_irq(IRQ_NUM, my_irq_handler, IRQF_SHARED, "my_device", NULL);
```

- `free_irq()`：释放通过 `request_irq()` 注册的中断号和对应的中断处理程序
```c
void free_irq(unsigned int irq, void *dev_id);
```
- 参数解释：
	- `irq`：中断号
	- `dev_id`：设别标识符，必须与 `request_irq()` 时使用的 `dev_id` 参数相同
- 说明：
	- 在设备驱动的退出或清理阶段，如果不再需要处理中断，需要调用 `free_irq()` 来释放中断资源，避免资源泄漏。

- `enable_irq()/disable_irq()`：启用/禁用指定中断
```c
void enable_irq(unsigned int irq);
void disable_irq(unsigned int irq); 
```
- 参数解释：
	- `irq`：中断号
- 说明：
	- 该函数会启用/禁用指定中断，使得该中断号对应的硬件中断信号会/不会被处理
- 使用场景：
	- 这些函数通常用于在中断处理期间禁用中断，避免中断嵌套导致的不确定性或死锁。也可用于在某些关键区域内禁止特定的硬件中断

- `local_irq_enable()/local_irq_disable()`：启用/禁用本地中断
```c
void local_irq_enable(void);
void local_irq_disable(void);
```
- 说明：
	- `local_irq_disable()`：禁用当前处理器的所有本地中断，这意味者当前处理器所有中断请求都会被屏蔽，直到中断被重新启用
	- `local_irq_enable()`：启用当前处理器的所有本地中断
- 使用唱场景：
	- 通常用于保护临界区，确保在关键操作期间不会被中断打断。禁用中断防止中断处理程序抢占当前正在执行的任务
- 示例：
```c
local_irq_disable();
// 进行不希望被中断打断的逻辑
local_irq_enable();
```

- `tasklet_schedule()`：调度 `tasklet` 进行下半部处理
```c
void tasklet_schedule(struct tasklet_struct *t);
```
- 参数解释：
	- `t`：指向 `tasklet_struct` 结构体的指针。这个结构体表示一个 `tasklet`，包含任务的执行代码
- 说明：
	- `tasklet_schedule()` 会将指定的 `tasklet` 加入到待处理任务队列，执行完成中断处理的下半部。
- 使用场景：
	- `tasklet` 是用于延迟处理的机制，它会在当前中断处理程序结束后或软中断上下文中被执行
- 示例：
```c
struct tasklet_struct my_tsaklet;
tasklet_init(&my_tasklet, my_tasklet_function, 0);

// 在中断处理程序中调度 tasklet
tasklet_schedule($my_tasklet);

// tasklet 执行函数
void my_tasklet_function(unsigned long data){
	// 执行下半部任务
}
```

## 7. 定时器：

- `struct timer_list` 结构体：
```c
struct timer_lsit{
	struct hlist_node entry;	// 链表节点，用于定时器队列管理
	unsigned long expires;		// 定时器超时时间(以 jiffies 为单位)
	void (*function)(struct timer_list *)	// 回调函数
	unsigned long data			// 用于传递给回调函数的用户数据
	const struct timer_info *timer_info;	// 定时器信息
}
```

- `init_timer()`：初始化定时器结构体
```c
void init_timer(struct timer_list *timer);
```
- 参数解释：
	- `timer`：指向 `struct timer_list` 结构体的指针，该结构体包含定时器的所有相关信息，包括回调函数 `function`、超时时间 `expires` 等
- 说明：
	- `init_timer()` 会将定时器结构体中所有字段初始化为零
- 示例：
``` c
struct timer_list my_timer;				// 创建定时器实例
init_timer(&my_timer);					// 初始化定时器
// 手动配置超时时间和回调函数
my_timer.expires = jiffies + mesecs_to_jiffies(1000);
my_timer.function = my_timer_callback;	// 链接定时器回调函数
```

- `timer_setup()`：初始化定时器
```c
void timer_setup(struct timer_list *timer, void (*callback)(struct time_list *), unsigned long flags);
```
- 参数解释：
	- `timer`：指向定时器结构体的指针
	- `callback`：定时器超时回调函数。
	- `flags`：标志位，用于指定定时器的一些特性
		- `TIMER_IRQSAFE`：标记定时器为中断安全类型，表示该定时器可以在中断上下文中执行
- 说明：
	- `timer_setup()` 和 `timer_init()` 函数都是初始化定时器的方法，但 `timer_setup()` 函数更为现代，比较常用

- `add_timer()`：启动定时器
```c
int add_timer(struct timer_list *timer);
```
- 参数解释：
	- `timer`：定时器结构体指针
- 返回值：
	- 成功时返回 0，失败时返回负值
- 说明：
	- `add_timer()` 会将已初始化的定时器添加到内核定时器队列，并启动定时器。当定时器的超时时间达到时，会调用定时器的回调函数
- 示例：
```c
struct timer_list my_timer;				// 创建定时器实例
init_timer(&my_timer);					// 初始化定时器
// 设置超时时间，msecs_to_jiffies：将毫秒转换为jiffies
my_timer.expires = jiffies + mesecs_to_jiffies(1000);
my_timer.function = my_timer_callback;	// 链接定时器回调函数

add_timer(&my_timer);					// 启动定时器
```

- `mod_timer()`：修改已添加定时器的超时时间
```c
int mod_timer(struct timer_list *timer, unsigned long expires);
```
- 参数解释：
	- `timer`：指向已初始化的定时器结构体
	- `expires`：超时时间，默认单位 `jiffies`

> [!Note] `jiffies`
> `jiffies` 是一个内核全局变量，表示系统自启动以来的时钟节拍数（单位是时钟滴答），取决于系统频率。
> 通过 `msecs_to_jiffies()` 可以将毫秒转换为时钟节拍

- 返回值：
	- 成功时返回 0，失败时返回负值
- 说明：
	- `mod_timer()` 只会修改已经添加的定时器的过期时间，不会重新添加定时器
- 示例：
```c
struct timer_list my_timer;				// 创建定时器实例
init_timer(&my_timer);					// 初始化定时器
// 设置超时时间，msecs_to_jiffies：将毫秒转换为jiffies
my_timer.expires = jiffies + mesecs_to_jiffies(1000);
my_timer.function = my_timer_callback;	// 链接定时器回调函数

add_timer(&my_timer);					// 启动定时器

mod_timer(&my_timer. jiffier + mesecs_to_jiffies(2000));	// 修改超时时间
```

- `del_timer()`: 删除定时器
``` c
int del_timer(struct timer_list *timer);
```
- 参数解释：
	- `timer`：指向已初始化的定时器结构体
- 返回值：
	- 成功时返回 0，失败时返回负值
- 说明：
	- `del_timer()` 会从内核定时器队列中删除指定定时器，如果定时器还没超时，它将不再被执行。如果定时器已经超时且正在执行回调函数，调用 `del_timer()` 不会影响定时器的执行

- `del_timer_sync()`：同步删除定时器
```c
int del_timer_sync(struct time_list *timer);
```
- 参数解释：
	- `timer`：指向已初始化的定时器结构体
- 返回值：
	- 成功时返回 0，失败时返回负值
- `del_timer_sync()` 用于删除定时器，并确保定时器被删除之前，回调函数已经被执行完毕，即调用会等待定时器的回调函数执行完成


## 8. 内核日志：

- `printk()`：输出内核日志信息
```c
int printk(const char *fmt, ...);
```
- 参数解释：
	- `fmt`：格式化字符串，类似于 `printf` 中的格式化参数，可以使用 `%d、%s、%p` 等格式符
	- `...`：与 `fmt` 对应的变量数据
- 返回值：
	- 成功时返回输出字符个数，失败时返回负值
- 说明：
	- `printk()` 用于打印内核日志信息。它不仅将信息输出到控制台，还将日志存储到内核缓冲区，可以通过 `dmesg` 命令或 `/var/log/kern.log` 文件查看内核日志
	- 内核日志的输出级别是由 `printk()` 的日志级别来控制的，内核会根据日志级别决定是否答应该日志，以及讲日志输出到哪里
		- `KERN_EMERG`：紧急情况，系统不可用
		- `KERN_ALERT`：需要立即处理的情况
		- `KERN_CRIT`：严重错误，可能导致系统崩溃
		- `KERN_ERR`：错误日志，表示操作失败
		- `KERN_WARNING`：警告信息，表示潜在问题
		- `KERN_INFO`：信息日志，显示正常操作
		- `KERN_DEBUG`：调试信息
- 示例：
```c
printk(KERN_INFO "This is an info message: %s\n", "Hello, World!");
printk(KERN_ERR "This is an error message: %d\n", -1);
```

- `dev_info()` ：输出设备驱动中的信息日志
```c
void dev_info(struct device *dev, cconst char *fmt, ...);
```
- 参数解释：
	- `dev`：指向 `struct device` 结构体的指针
	- `fmt`：格式化字符串
- 说明：
	- `dev_info()` 用于输出设备的信息日志，它会在输出日志前加上设备的名称
- 示例：
```c
struct device *my_device;
dev_info(my_device, "Device initialized successfully\n");
```

- `dev_err()`：输出设备错误信息
```c
void dev_err(struct device *dev, const char *fmt, ...);
```
- 参数解释：
	- `dev`：指向 `struct device` 结构体指针
	- `fmt`：格式化字符串
- 说明：
	- `dev_err()` 用于输出设备的错误信息日志。它在设备发生错误时被调用，输出的日志会包含设备的名称

- `pr_info()` / `pr_err()`：简化版 `printk`
```c
void pr_info(const char *fmt, ...);			// 打印日志信息
void pr_info(const char *fmt, ...);			// 打印错误信息
```
- 参数解释：
	- `fmt`：格式化字符串
- 说明：
	- `pr_info()`：用于输出信息日志，默认日志级别为 `KERN_INFO`。
	- `pr_err()`：用于输出错误日志，默认日志级别为 `KERN_ERR`。

## 9. `ioctl()`: 

-  `ioctl()` 在 Linux 驱动开发中用于实现设备与用户空间的通信。它允许用户空间程序通过文件描述符向设备驱动 (`file`) 发送特定的命令和控制请求 (`cmd`)。
	- `ioctl()` 是通过设备文件的 `file_operations` 结构体中的 `unlocked_ioctl` 方法来实现的
- 与标准的读写操作不同，`ioctl()` 可以执行更加复杂和灵活的操作，如获取设备状态、修改设备参数、执行硬件特定的命令等
### 9.1 驱动端

- 在驱动程序，`ioctl()` 函数有多个变种，最常见的是 `unlocked_ioctl()`，它通常作为 `file_operations` 中的成员
```c
long unioctl(struct file *file, unsigned int cmd, unsigned arg);
```
- 参数解释：
	- `file`：指向 `struct file` 的指针，表示设备文件的描述符
	- `cmd`：命令码，用来表示用户空间对设备执行的操作
		- `_IO()`：表示不需要参数的命令
		- `_IOR()`：表示从设备获取数据
		- `_IOW()`：表示向设备传递数据
		- `_IOWR()`：表述设备和用户空间双向数据传输
	- `arg`：参数值，通常是一个指针，整数值或者其他类型的数据，用于传递附加参数到设备驱动中
- 返回值：
	- 成功时返回 0 或对应值。
	- 失败时返回-1，同会设置 `errno` 变量，指示错误类型
- 机制流程：
	- `unlocked_ioctl` 方法实现
	- 在 `file_operations` 中注册
```c
// unlocked_ioctl 方法实现
static long my_device_ioctl(struct file *file, unsigned int cmd, unsigned long arg) {
    switch (cmd) {
        case MY_IOCTL_CMD_1:
            // 处理命令 1
            break;
        case MY_IOCTL_CMD_2:
            // 处理命令 2
            break;
        default:
            return -EINVAL;  // 未知命令
    }
    return 0;
}

// 注册 unlocked_ioctl
static const struct file_operations my_device_fops = {
	.unlocked_ioctl = my_device_ioctl,
	// 其他操作
}
```
### 9.2 用户端：

- 在用户态程序中，`ioctl()` 是一个常用的系统调用接口，通常用于与设备驱动进行交互。它能够让用户程序通过文件描述符控制硬件设备或获取硬件设备的状态。
```c
int ioctl(int fd, unsigned long request, ...);
```
- 参数解释：
	- `fd`：设备的文件描述符
		- 设备文件可以通过 `open()` 系统调用打开，返回一个文件描述符，后续的 `ioctl()` 调用需要传递该文件描述符，设备文件一般位于 `/dev` 目录下
	- `request`：命令请求码，指定要执行的控制操作，不同的设备驱动会定义不同的请求码。
	- `...`：变长参数，根据 `request` 具体命令，可能需要提供额外的参数
- 返回值：
	- 成功时返回大于等于 0，表示设备操作成功
	- 失败时返回 `-1`，并且会设置 `error` 变量，指示错误类型
		- `EINVAL`：请求的命令无效或不支持。
		- `EBADF`：无效的文件描述符。
		- `ENOTTY`：不支持该请求的设备类型。
- 机制流程：
	- 打开设备文件
	- 调用 `ioctl` 执行设备控制操作

```c
// 打开设备
int fd = open("/dev/mydevice", O_RDWR);
if(fd == -1){
	perror("Failed to open device");
	return -1;
}

// 调用 ioctl()
int result = ioctl(fd, MY_IOCTL_CMD, &data)
if(result == -1){
	perror("ioctl failed");
	close(fd);
	return -1;
}
```

## 10. 网络驱动：

### 10.1 网络设备管理：
#### 10.1.1 注册网络设备：

- `ether_setup()`：初始化网络设备（以太网设备）
```c
void ether_setup(struct net_device *dev);
```
- 参数解释：
	- `dev`：指向 `struct net_device` 网络设备结构体的指针
- 说明：
	- `ether_setup()` 函数主要作用是设置 `net_device` 结构体中的各个字段，为网络设备初始化一些基础设置，包括：
		- 设备名称：按照给定模板进行初始化（如 `eth0`、`eth1`）
		- 设备类型：为设备设置以太网类型（如 `ETH_P_ALL` 和 `ETH_P_IP`）
		- 硬件地址：随机生成 MAC 地址或或者传递给 `net_device` 结构体的地址来初始化硬件地址。
		- 传输函数：设置设备发送（`ndo_start_xmit`）函数和接收 (`ndo_poll_controller`) 操作函数
		- 队列初始化：如果设备支持多队列（如通过 `alloc_netdev_mqs()` 分配的设备），`ether_setup()`会进行队列的初始化工作。
		- 链路层协议、网络接口标注、初始化状态、缓冲区等等
- 内部实现：
```c
void ether_setup(struct net_device *dev) {
    /* 设置 MAC 地址 */
    ether_addr_copy(dev->dev_addr, dev->perm_addr);

    /* 初始化硬件地址 */
    dev->addr_len = ETH_ALEN;  // 以太网地址长度，通常是 6 字节
    dev->netdev_ops = &ethernet_ops;  // 设置设备操作函数，如发送数据包、接收数据包等
    dev->flags |= IFF_BROADCAST | IFF_MULTICAST;  // 启用广播和多播
    dev->mtu = ETH_DATA_LEN;  // 设置最大传输单元，通常为 1500 字节
    dev->type = ARPHRD_ETHER;  // 设置设备类型为以太网设备
    dev->hard_header_len = ETH_HLEN;  // 设置硬件头部长度（以太网头部长度）
    dev->tx_queue_len = 1000;  // 设置发送队列长度
    dev->flags |= IFF_NOARP;  // 默认关闭 ARP（ARP 是以太网层协议，用于自动映射 IP 到 MAC 地址）

    /* 设置一些设备的默认操作 */
    dev->watchdog_timeo = 60 * HZ;  // 设置网络设备的超时值（单位：Hz，表示网络设备如果超过 60 秒没有响应，就认为设备失败）
}
```

- `alloc_netdev()`：分配网络设备结构
```c
struct net_device *alloc_netdev(int sizeof_priv, const char *name, int ether_type, netdev_tx_t (*init)(struct net_device *));
```
- 参数解释：
	- `sizeof_priv`：(`net_device`) 网络设备私有数据结构的大小
		- 私有数据结构通常由驱动程序定义，用于存储设备特定的状态信息
	- `name`：设备名称的模板。
		- `alloc_netdev()` 会根据这个模板自动为每个设备分配一个名称
		- 通常是格式字符串（例如 `eth%d`）
	- `ether_type`：设备的以太网类型，通常用于表示该设备所处理的协议类型
		- 如 `ETH_P_IP` 和 `ETH_P_ARP`
	- `init`：设备初始化函数指针
		- 常见的初始化函数有 `ether_setup()`
- 返回值：
	- 返回一个 `net_device` 类型指针，指向所分配的网络设备
	- 失败时返回 `NULL`
- 说明：
	- `alloc_netdev()` 用于分配一个 `net_device` 结构体，并初始化其中的基本参数。为设备驱动提供了一个空白的网络设备接口，驱动程序可以通过它进行进一步设备配置 (如设置设备的硬件地址，初始化传输函数等)
	- `alloc_netdev()` 只是分配了 `net_device` 结构体，设备的完整初始化工作通常需要调用其他函数（例如 `ether_setup()`）来完成。
- 示例：
```c
struct net_device *dev = alloc_netdev(sizeof(struct my_priv_data), "eth%d", NET_NAME_UNKNOW, ether_setup);
if(!dev){
	printk(KERN_ERR, "Failed to allocate network device\n");
}
```

- `alloc_netdev_mqs()`：分配和初始化支持多队列的网络设备
```c
struct alloc_netdev_mqs(int sizeof_priv, const char *name, int ether_type, netdec_tx_t(*init)(struct net_device *), int tx_queues, int rx_queues);
```
- 参数解释：
	- `sizeof_priv`：设备私有数据结构的大小
	- `name`：设备名称模板
	- `ether_type`：设备的以太网类型
	- `init`：初始化网络设备的函数
	- `tx_queues`：设备的发送队列数
		- 网络设备通常支持多个发送队列，以便并行处理多个数据流
	- `rx_queues`：设备的接收队列数
- 返回值：
	- 返回一个 `net_device` 类型指针，指向所分配的网络设备
	- 失败时返回 `NULL`
- 说明：
	- `alloc_netdev_mqs()` 是一个扩展版的 `alloc_netdev()`, 支持分配多个发送队列和接收队列，适用于多核处理器环境
- 示例：
```c
struct net_device *dev;
dev = alloc_netdev_mqs(sizeof(struct my_priv_data), "eth%d", NET_NAME_UNKONE, ether_setup, 4, 4);
if (!dev){
	printk(KERN_ERR "Failed to allocate network device\n"); 
}
```

- `register_netdev()`：注册网络设备
```c
int register_netdev(struct net_device *dev);
```
- 参数解释：
	- `dev`：指向 `net_device` 结构体的指针
		- `net_device` 是每个网络接口的核心数据结构，包含了网络接口的各种配置和状态
- 返回值：
	- 返回 0 表示成功，返回负值表示失败
- 说明：
	- `register_netdev()` 会将一个网络设备注册到内核的网络子系统中，并使其能够和其他网络设备进行通信
	- 注册网络设备后，网络子系统就能够通过 `dev` 结构体管理该设备
- 示例：
```c
struct net_device *dev = alloc_netdev(0, "eth%d", NET_NAME_UNKNOW, ether_setup);
int ret = register_netdev(dev);
if (ret < 0){
	printk(KERN_ERR "Failed to register network device\n");
}
```

- `unregister_netdev()`：注销网络设备
```c
void unregister_netdev(struct net_device *dev);
```
- 参数解释：
	- `dev`：指向`net_device`网络设备的指针
- 说明：
	- 它会清理与网络设备相关的资源，并且停止设备的发送/接收操作。
	- 在设备的释放或者驱动卸载的过程中，通常会调用该函数注销网络设备，以避免内存泄漏或资源占用。
		- 网卡驱动的 `exit()` 函数中，会调用此函数注销网络设备

#### 10.1.2 设置设备参数：

- `netif_carrier_on()`：启动网络设备链接状态
```c
void netif_carrier_on(struct net_device *dev);
```
- `netif_carrier_off()`：禁用网络设备链接状态
```c
void netif_carrier_off(struct net_device *dev);
```
- `netif_start_queue()`：启动网络设备发送队列
```c
void netif_start_queue(struct net_device *dev);
```
- `netif_stop_queue()`：停止网络设备发送队列
```c
void netif_stop_queue(struct net_device *dev);
```
#### 10.1.3 网络设备状态：

- `netif_device_attach()`：将网络设备添加到内核的网络设备列表
```c
void netif_device_attach(struct net_device *dev);
```
- `netif_device_detach()` ：从内核网络设备列表中移除网络设备
```c
void netif_device_detach(struct net_device *dev);
```
### 10.2 网络数据处理：
#### 10.2.1 接收数据包：

- `netif_rx()`：将接收到的数据包传递到网络栈中进行处理
- `dev_alloc_skb()`：为数据包分配内存 (`sk_buff` 结构)
- `skb_put()`：为 `sk_buff` 增加数据区长度
#### 10.2.2 发送数据包：

- `dev_queue_xmit()`：将数据包发送到设备队列中
- `netdev_tx_t()`：用于发送数据包的传输函数
#### 10.2.3 设置和获取网络接口配置：

- `dev_set_mac_address()`：设置设备的 MAC 地址
- `dev_get_drvdata()`：获取与网络设备关联的驱动数据
- `dev_set_drvdata()`：设置与网络设备关联的驱动数据

### 10.3 网络协议栈交互：
#### 10.3.1 传输层与数据链路层：

- `skb_push()`：为数据包添加头部
- `skb_pull()`：从数据包中移除头部
- `skb_reset_network_header()`：重置网络层头部指针
- `skb_reset_mac_header()`：重置 MAC 层头部指针 
#### 10.3.2 IP 地址管理：

- `inet_addr_to_string()`：将 IP 地址转换为字符串
- `in_aton()`：将字符串格式的 IP 地址转换为 `in_addr_t`
- `ip_route_output_key`：查找路由表，返回目标 IP 的路由信息