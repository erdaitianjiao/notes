## linux驱动



### 一、驱动简介

#### 1. linux对驱动程序的调用



#### 2. 驱动模块加载和卸载

Linux驱动有两种运行方式：

1. 编译进内核，随内核启动自动运行
2. 编译为.ko模块，通过insmod命令加载

调试时通常选择模块方式，优点：

- 只需单独编译驱动代码，无需编译整个内核
- 可动态加载/卸载模块，避免重启系统
- 开发完成后，根据需求决定是否编入内核

模块加载 卸载函数

~~~c
moudule_init(xxx_init);
moudule_exit(xxx_exit);
~~~

linux加载 卸载 模块指令

~~~bash
insmod xx.ko	# 加载 xx.ko
rmmod  xx.ko	# 卸载 xx.ko
~~~

内核模块示例代码如下

~~~c
static int __init xxx_init(void) {
    
    /*入口函数具体内容*/
    
    return 0;
}

static void __exit xxx_exit(void) {
    
   
}

module_init(xxx_init);
module_exit(xxx_exit);

// 添加license和作者信息
MODULE_LICENSE()
MODULE_AUTHOR()
~~~



### 二、字符设备驱动开发

#### 1. 简介

字符设备是 Linux 驱动中最基本的一类设备驱动，字符设备就是一个一个字节，按照字节
流进行读写操作的设备，读写数据是分先后顺序的。比如我们最常见的点灯、按键、IIC、SPI，
LCD 等等都是字符设备，这些设备的驱动就叫做字符设备驱动



