## 字符设备驱动(GPIO)

### 一、概述

​	字符设备驱动主要是通过实现对结构体`file_operations`成员函数`open write read release `等函数的实现，使得用户可以通过系统调用的`open write read release `来操作外设，初始化字符设备驱动中`cdev_init(struct cdev *, const struct file_operations *);`就需要用到`file_operations`结构体，并且还要加载`cdev`具体操作在第二部分呈现

​	并且需要在 `/dev/ `下创建对应设备节点 `/dev/test`，系统调用的目标就是 `/dev/test` 对文件操作就是调用上文提到的`open write read`等函数，创建设备需要用到 `device_create`函数，调用`device_create`需要参数` struct class` 所以要先初始化一个 `class` ，需要用到`class_create` 函数

​	要实现具体功能的情况下需要对外设进行初始化，以`gpio`为例，需要对`gpio`配置输入输出模式，可以通过写寄存器的方式初始化，也可以使用`pincrtl`和`gpio`子系统来初始化引脚，但是使用`pincrtl`和`gpio`子系统需要在设备树文件下添加相应的节点

### 二、设备树中添加led信息

1. 在根节点下添加`gpio`信息

   ~~~dts
   / {
   gpio {
   	#address-cells = <1>;
   	#size-cells = <1>;
   		compatible = "atkalpha-gpio";
   		pinctrl-names = "default";
   		pinctrl-0 = <&pinctrl_gpio>;
   		gpiopin = <&gpio5 1 GPIO_ACTIVE_HIGH>;
   		status = "okay";
   	};
   };
   ~~~

2. 在 `iomuxc`下创建`pinctrl_gpio`子节点

   ~~~
   pinctrl_gpio: gpiogrp {
   	fsl,pins = <
   		XXX_PAD_SNVS_TAMPER1__GPIOX_IO0X 0x10B0 
   	>;
   };
   ~~~

   

### 三、初始化流程

定义信息，建立结构体，一些变量和返回值进行整合处理

~~~c
#define GPIO_CNT    1
#define GPIO_NAME   "gpio"
#define GPIOPOFF     0
#define GPIOPON      1

struct gpio_dev{

    dev_t devid;
    struct cdev cdev;
    struct class *class;
    struct device *device;
    int major;
    int minor;
    struct device_node *nd;
    int gpiopin;
};
~~~



1.  从设备树中获得引脚信息，并通过gpio子系统初始化gpio引脚

   - 使用pincrtl和gpio子系统

 ~~~c
struct gpio_dev gpio;

// 获取gpio设备节点
gpio.nd = of_find_node_by_path("/gpio");	
// 错误处理
if (gpio.nd == NULL) {						

	printk(gpio node find failed\r\n");

} else {

	printk("gpio node find success\r\n");

}											
// 获取设备树中gpio属性，得到gpio引脚
gpio.gpiopin = of_get_named_gpio(gpio.nd, "gpio-pin", 0);
// 错误处理           
if (gpio.gpiopin < 0) {										

	printk("cant get gpiopin\r\n");
	return -EINVAL;

} else {

    	printk("gpio num = %d\r\n", gpio.gpiopin);

}           

// 设置引脚为输出，并且输出高电平           
ret = gpio_direction_output(gpio.gpiopin, 1); 
// 错误处理         
if (ret < 0) {									

	printk("cant set gpio\r\n");

}

 ~~~

2. 初始化字符设备

~~~c
// 创建设备号 如果给出设备号就直接使用 若未给出 就向提出申请设备号
if (gpio.major) {

   gpio.devid = MKDEV(gpio.major, 0);
   register_chrdev_region(gpio.devid, GPIO_CNT, GPIO_NAME);

} else {

   alloc_chrdev_region(&gpio.devid, 0, GPIO_CNT, GPIO_NAME);
   gpio.major = MAJOR(gpio.devid);
   gpio.minor = MINOR(gpio.devid);

}

printk("gpio major = %d, gpio = %d \r\n",gpio.major, gpio.minor);

// 初始化cdev
gpio.cdev.owner = THIS_MODULE;
cdev_init(&gpio.cdev, &gpio_fops);			

// 添加一个cdev
cdev_add(&gpio.cdev, gpio.devid, GPIO_CNT);
~~~

   

3. 添加字符设备到内核

~~~c
// 创建类
gpio.class  = class_create(THIS_MODULE, GPIO_NAME);
// 错误处理
if (IS_ERR(gpio.class)) {

	return -PTR_ERR(gpio.class);

}

// 创建设备 创建完设备就能在 /dev/ 下找到对应外设了
gpio.device = device_create(gpio.class, NULL, gpio.devid, NULL, GPIO_NAME);
// 错误处理
if (IS_ERR(gpio.device)) {

    return -PTR_ERR(gpio.device);

}
~~~

### 四、`file_operations`成员函数的实现

在初始化cdev的时候需要用到`&gpio_fops` 这个就是`file_operations`类型的结构体

~~~c
static struct file_operations gpio_fops = {

   .owner = THIS_MODULE,
   .open = gpio_open,
   .write = gpio_write,
   .release = gpio_release,

};

static int gpio_open(struct inode *inode, struct file *filp) {

   filp->private_data = &gpio;
   return 0;

}

static ssize_t gpio_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *offt) {

   int retvalue;
   unsigned char databuf[1];
   unsigned char gpiostat;
   struct gpio_dev *dev = filp->private_data;

   retvalue = copy_from_user(databuf, buf, cnt);
   if (retvalue < 0) {

       printk("kernel write failed\r\n");

   }

   gpiostat = databuf[0];

   if (gpiostat == GOIOON) gpio_set_value(dev->gpiopin, 0);
   else if (gpiostat == GPIOOFF) gpio_set_value(dev->gpiopin, 1);

   return 0;
}

static int gpio_release(struct inode *inode, struct file *filp) {

   return 0;

}
~~~

### 五、调用api进行实现

其实就是调用`open` 和` write `就可以实现对gpio的操控

~~~c
int fd, retvalue;
fd = open(filename, O_RDWR);

retvalue = write(fd, 1, 1); // 设置为1
retvalue = write(fd, 0, 1); // 设置为0
~~~



### 六、参考

~~~c
// file_operation结构体定义如下

struct file_operations {
	struct module *owner;
	loff_t (*llseek) (struct file *, loff_t, int);
	ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
	ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
	ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
	ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
	int (*iterate) (struct file *, struct dir_context *);
	unsigned int (*poll) (struct file *, struct poll_table_struct *);
	long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
	long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
	int (*mmap) (struct file *, struct vm_area_struct *);
	int (*mremap)(struct file *, struct vm_area_struct *);
	int (*open) (struct inode *, struct file *);
	int (*flush) (struct file *, fl_owner_t id);
	int (*release) (struct inode *, struct file *);
	int (*fsync) (struct file *, loff_t, loff_t, int datasync);
	int (*aio_fsync) (struct kiocb *, int datasync);
	int (*fasync) (int, struct file *, int);
	int (*lock) (struct file *, int, struct file_lock *);
	ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
	unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long, unsigned long, unsigned long);
	int (*check_flags)(int);
	int (*flock) (struct file *, int, struct file_lock *);
	ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
	ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
	int (*setlease)(struct file *, long, struct file_lock **, void **);
	long (*fallocate)(struct file *file, int mode, loff_t offset,
			  loff_t len);
	void (*show_fdinfo)(struct seq_file *m, struct file *f);
#ifndef CONFIG_MMU
	unsigned (*mmap_capabilities)(struct file *);
#endif
};
// 部分函数原型

static inline struct device_node *of_find_node_by_path(const char *path)
{
	return of_find_node_opts_by_path(path, NULL);
}

static inline int of_get_named_gpio(struct device_node *np,
                                   const char *propname, int index)
{
	return of_get_named_gpio_flags(np, propname, index, NULL);
}

static inline int gpio_direction_output(unsigned gpio, int value)
{
	return -ENOSYS;
}

extern int alloc_chrdev_region(dev_t *, unsigned, unsigned, const char *);

void cdev_init(struct cdev *, const struct file_operations *);

int cdev_add(struct cdev *, dev_t, unsigned);

#define class_create(owner, name)		\
({						\
	static struct lock_class_key __key;	\
	__class_create(owner, name, &__key);	\
})

struct device *device_create(struct class *cls, struct device *parent,
			     dev_t devt, void *drvdata,
			     const char *fmt, ...);
~~~



   
