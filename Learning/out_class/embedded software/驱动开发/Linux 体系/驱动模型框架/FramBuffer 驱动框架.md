### 1. 基础概念 

- 在 Linux 系统中一般通过 `Framebuffer` 设备模型来驱动程序控制 `LCD` 显示屏
- `Framebuffer` (中译：帧缓冲)，一块用于存储屏幕显示数据的内存区域，直接或间接映射到显存（通常 `Framebuffer` 是直接映射到显存的），其核心作用是存储屏幕上每个像素的颜色信息，并由 LCD 显示控制器读取并输出到屏幕。 

---
 
- 为了避免屏幕闪烁，`Framebuffer` 常使用**双缓冲**机制：
	- 前缓冲区（`Front Buffer`）：当前正在显示的数据
	- 后缓冲区（`Back Buffer`）：正在绘制的下一帧数据
- 绘制完成后，前后缓冲区进行交换（Swap），以提供平滑的显示效果。

---

- `Framebuffer` 实质上是一个线性数组，每个元素表示屏幕上的一个像素。存储方式主要受以下参数的影响：
	- `xres` （水平分辨率）：屏幕像素宽度，即每行像素数量
	- `yres` （垂直分辨率）：屏幕像素高度，即总行数
	- `bits_per_pixel`（每像素位数，`bpp`）：每个限速的存储单位，决定颜色深度
	- `stride` （行长度）：表示 `Framebuffer` 每行所占用的字节数
- `Framebuffer` 设备在 Linux 内核中的主要数据结构为 `struct fb_info`：

```c
struct fb_info {
	struct fb_var_screeninfo var;	// 可变参数（分辨率、色保等）
	struct fb_fix_screeninfo fix;	// 固定参数（帧缓冲地址等）
	void *screen_base;				// 显存虚拟地址指针
	struct fb_ops *fbops;			// 驱动提供的操作函数
}
```
- 其中 `Framebuffer` 的存储布局定义在 Linux 内核的 `fb_var_screeninfo` 与 `fb_fix_screeninfo` 结构体中
- `fb_var_screeninfo`（可变参数）：

```c
struct fb_var_screeninfo {
	__u32 xres; 			// 水平方向的分辨率（可见区域）
	__u32 yres; 			// 垂直方向的分辨率（可见区域）
	__u32 xres_virtual; 	// 水平方向的虚拟分辨率 
	__u32 yres_virtual; 	// 垂直方向的虚拟分辨率 
	__u32 bits_per_pixel; 	// 每个像素的位深度（bpp
}
```
- 参数解释：
	- 00 `xres` / `yres`：真实屏幕分辨率（即 LCD 设备的分辨率）。
	- `xres_virtual` / `yres_virtual`：`FrameBuffer` 的虚拟分辨率，可用于**双缓冲**等技术。
	- `bits_per_pixel`：定义像素存储格式. (`RGB` 三原色)
- 常见的像素存储格式（了解即可）：
	- `32bpp`：`RGB8888`
	- `16bpp`：`RGB565`、`RGB555`

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250303123904.png" width = "600 px"/>
</div>

- `fb_fix_screeninfo`（固定参数）：

```c
struct fb_fix_screeninfo {
	__u32 line_length;		// 每行所占字节数
	__u32 smem_start;		// Framebuffer 物理地址
	__u32 smem_len;			// Framebuffer 大小长度
}
```
- `Framebuffer` 的大小长度由其分辨率决定。 例如：假设 `LCD` 屏的分辨率为 `1024x768`，每一个像素的颜色用 32 位来表示，那么 `Framebuffer` 的大小为：

$$
1024*768*32/8 = 3145728  （字节）
$$
- 在 Linux 内核中，`Framebuffer` 由 `fbdev` 子系统管理，提供 `/dev/fbX` 设备接口来供用户态程序获取 `Framebuffer` 设备文件信息
	- 应用程序可以直接通过 `mmap()` 映射写入显存，也可以使用 `ioctl()` 等接口函数进行配置



### 2. `Framebuffer` 驱动 `LCD` 流程

- 在嵌入式系统中，LCD 显示实现流程如下：
	- 应用程序向 `/dev/fb0` 写入像素数据（或使用 `mmap()` 直接写入显存）
	- `FrameBuffer` 驱动程序将数据存入显存
	- `LCD` 显示控制器从显存读取数据，并进行扫描输出
	- `LCD` 面板接收数据，显示对应的图像
- 其中涉及到几个名词解释：
	- 显存（`Video Memory`）：存储像素数据，可由 `CPU` 或 `GPU` 访问
	- 显示控制器 (`Display Conctroller`）：负责读取 `Framebuffer` 数据，并按照设定的刷新率输出到屏幕
	- 驱动程序：负责初始化 `Framebuffer`，设置分辨率、色深灯参数，并提供用户态 `API` 访问

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250303111413.png" width = "600 px"/>
</div>

---
- 描点绘制：
	- `LCD` 屏的显示机制，事实上是通过利用肉眼视觉延迟，高速刷新像素点来实现平滑的屏幕显示。对于单点像素，`LCD` 控制器是怎样的获取流程？
	- `LCD` 屏上每个像素点对应一个 `Framebuffer` 字节地址空间。对于 `LCD` 屏上某点 `(x,y)`, 其地址空间位置($P_{address}$)为：
		- 假设该 Linux 系统 中 `Framebuffer` 地址空间的基地址为 `fb_base`，其分辨率为 `(xres*yres)`
$$
P_{address} = fb\_ase + (y * xres + x) * bpp / 8
$$
	- 查询到点位地址空间后，我们只需要依据 `LCD` 屏规定的像素存储格式对地址空间内容进行修改设置即可

- 描点函数实现：
	- 具体不会依靠自己去从底层实现，但理解这部分代码有助于理解 `LCD` 屏显示的原理

```c
void lcd_put_pixel(int x, int y, unsigned int color) {
	unsigned char *pen_8 = fb_base + y*line_width + x*pixel_width;
	unsigned short *pen_16;
	unsigned int *pen_32;
	
	unsigned int red, green, blue;
	
	pen_16 = (unsigned short *)pen_8;
	pen_32 = (unsigned int *)pen_8
	
	switch(var.bits_per_pixel){
		case 8:
			*pen_8 = color;
			break;
		case 16:
			red = (color >> 16) & 0xff
			green = (color >> 8) & 0xff;
			blue = (color >> 0) & 0xff;
			color = ((red >> 3) << 11) | ((green >> 2) << 5) | (blue >> 3); 
			*pen_16 = color;
			break;
		case 32:
			*pen_32 = color;
			break;
		default:
			printf("can't support %dbpp", var.bits_per_pixel);
			break;
	}
}
```
---

#### 2.2 `FrameBuffer` 驱动基本框架：

- 由前面的基础概念讲解，经过分析可以得出在驱动层，我们需要实现的内容有：
	- 分配 `fb_info` 结构体
	- 初始化 `fb_var_screeninfo` 以及 `fb_fix_screeninfo` 结构体
	- 映射显存
	- 编写注册 `fb_ops` 操作
	- 注册 `Framebuffer` 设备
- 示例代码：

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/fb.h>
#include <linux/init.h>
#include <linux/platform_device.h>
#include <linux/uaccess.h>
#include <linux/io.h>

#define FB_WIDTH    800             // 屏幕宽度
#define FB_HEIGHT   480             // 屏幕高度
#define BPP         32              // 每像素 32 位

static struct fb_info *fb_info;
static u32 *fb_mem;                 // Framebuffer 内存

// 矩阵像素点填充
static void fbdev_fillret(struct fb_info *info, const struct fb_fillrect *rect){
    int x, y;
    u32 color = rect->color;
  
    for (y = rect->dy; y < rect->dy + rect->height; y++){
        for (x = rect->dx; x < rect->dx + rect->width; x++){
            if(x < FB_WIDTH && y < FB_HEIGHT){
                fb_mem[y * FB_WIDTH + x] = color;
            }
        }
    }
}

// 设备操作接口
static struct fb_ops fbdev_ops {
    .owner = FRAME_BUFFER_MODULE;
    .fb_fillrect = fbdev_fillret;
}

// 设备初始化
static int fbdev_probe(struct platform_device *pdev){
    fb_info = framebuffer_alloc(0, &pdev->dev);
    if(!fb_mem){
        framebuffer_release(fb_info);
        return -ENOMEM;
    }

    // 配置 var 结构体
    fb_info->var.xres = FB_WIDTH;
    fb_info->var.yres = FB_HEIGHT;
    fb_info->var.bits_per_pixel = BPP;
  
    // 配置 fix 结构体（固定参数）
    fb_info->fix.line_length = FB_WIDTH * (BPP / 8);
    fb_info->fix.smem_start = (unsigned long)fb_mem;
    fb_info->fix.smem_len = FB_WIDTH * FB_HEIGHT * (BPP / 8);
    fb_info->screen_base = (char __iomem *)fb_mem;
    fb_info->fbops = &fbdev_ops;

    // 注册 Framebuffer 设备
    if (register_framebuffer(fb_info) < 0){
        kfree(fb_mem);
        framebuffer_release(fb_info);
        return -EINVAL;
    }

    printk(KERN_INFO "Framebuffer device registered successfully!\n");
    return 0;
}

// 设备初始化
static int fbdev_remove(struct platform_device *pdev){
    unregister_framebuffer(fb_info);
    kfree(fb_mem);
    framebuffer_release(fb_info);
    printk(KERN_INFO, "ramebuffer device unregistered.\n");
    return 0;
}

static struct platform_driver fbdev_device = {
    .driver = {
        .name = "fdev_demo",
    },
    .prob = fbdev_probe,
    .remove = fbdev_remove,
}
  
static int __init fbdev_init(void){
    return platform_driver_register(&fbdev_driver);
}
  
static void __exit fbdev_exit(void){
    return platform_driver_unregister(&fbdev_driver);
}
  
module_init(fbdev_init);
module_exit(fbdev_exit);
  
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Demo");
MODULE_DESCRIPTION("Simple Framebuffer Driver");
```

#### 2.3 `FrameBuffer` 应用开发基本框架：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <linux/fb.h>
#include <sys/ioctl.h>
#include <sys/mman.h>

// 宏定义
#define FBDEVICE    "/dev/fb0"
#define WIDTH       1024
#define HEIGHT      600

// 32位bpp
#define WHITE       0xffffffff
#define BLACK       0x00000000
#define RED         0xffff0000
#define GREEN       0xff00ff00
#define BLUE        0xff0000ff
#define GREENP      0x0000ff00

// framebuffer 基地址
unsigned int *pfb = NULL;
  
// 函数声明
void draw_background(unsigned int width, unsigned int height, unsigned int colo);
void draw_line(unsigned int color);

int main(void) {
    int fd = -1, ret = -1;
  
    struct fb_fix_screeninfo fix_info = {0};
    struct fb_var_screeninfo var_info = {0};

    fd = open(FBDEVICE, O_RDWR);
    if (fd < 0){
        perror("open");
        return -1;
    }
    printf("open %s success \n", FBDEVICE);

    ret = ioctl(fd, FBIOGET_FSCREENINFO, &fix_info);
    if (ret < 0){
        perror("ioctl");
        return -1;
    }
    printf("smem_start = 0x%x, smem_len = %u.\n", fix_info.smem_start, fix_info.smem_len);

    ret = ioctl(fd, FBIOGET_VSCREENINFO, &var_info);
    if (ret < 0){
        perror("ioctl");
        return -1;
    }
    printf("xres = %u, yres = %u.\n", var_info.xres, var_info.yres);
    printf("xres_virtual = %u, yres_virtual = %u.\n", var_info.xres_virtual, var_info.yres_virtual);
    printf("bpp = %u.\n", var_info.bits_per_pixel);

    // 获取 framebuffer 地址长度
    unsigned long len = var_info.xres_virtual * var_info.yres_virtual + var_info.bits_per_pixel / 8;
    printf("len = %ld\n", len);
    
    // 内存映射直接操作地址字节
    pfb = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (pfb = NULL){
        perror('mmap');
        return -1;
    }

    printf("pfb = %p.\n", pfb);

    draw_back(WIDTH, HEIGHT, WHITE);
    draw_line(RED);
  
    close(fd);
  
    return 0;
}

// 刷背景函数
void draw_background(unsigned int width, unsigned int height, unsigned int color)
{
    unsigned int x, y;
    
    for (y = 0; y < height; y ++){
        for (x = 0; x < width; x ++){
            *(pfb + y * WIDTH + x) = color;
        }
    }
}

// 画线函数
void draw_line(unsigned int color){
    unsigned int x, y;

    for (x = 50; x < 600; x ++){
        *(pfb + y * WIDTH + x) = color;
    }
}
```