### 内核实验

#### 一、Kbuild和Makefile

##### Kbuild

~~~makefile
EXTRA_CFLAGS = -Wall -g

obj-m        = supermodule.o
supermodule-y = module-a.o module-b.o

# -Wall 启用所有常见的警告信息（Warn all），帮助发现代码中的潜在问题
# -g 在可执行文件中包含调试信息，方便使用 gdb 等调试工具

# 目标（target）的后缀决定了它们的用途
# -m（modules）标示目标为可加载内核模块
# -y（yes）标示目标是编译对象文件然后将其链接到模块（`$(模块名称)-y`）或内核（`obj-y`）
#  其他任何目标后缀都将被 `Kbuild` 忽略，不会被编译
~~~

##### Makefile

~~~
KDIR = /linux/
kbuild:
	make -C $(KDIR) M=`pwd`
clean:
	make -C $(KDIR) M=`pwd`
~~~

#### 二、内核数据结构

#####  1. list 链表

###### 定义

```c
struct list_head {
    struct list_head *next, *prev;
};
```

###### 操作函数及宏定义

```c
list
LIST_HEAD(name)  		// 用于声明链表的标记（sentinel）
INIT_LIST_HEAD(struct list_head *list) 	// 用于在进行动态分配时，通过设置链表字段 next 和 prev，来初始化链表的标记
list_add(struct list_head *new, struct list_head *head) 	// 将 new 指针所引用的元素添加到 head 指针所引用的元素之后
list_del(struct list_head *entry)  	// 删除属于列表的 entry 地址处的项目
list_entry(ptr, type, member) 	 	// 返回列表中包含元素 ptr 的类型为 type 的结构，该结构中具有名为 member 的成员。
list_for_each(pos, head) 使用 pos    // 作为游标来迭代列表
list_for_each_safe(pos, n, head)	// 使用 pos 作为游标，n 作为临时游标来迭代列表。此宏用于从列表中删除项目
```

`list_entry`实现原理

~~~bash
#define list_entry(ptr, type, member) container_of(ptr, type, member)

#define container_of(ptr, type, member) ({                      \
        const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
        (type *)( (char *)__mptr - offsetof(type,member) );})
# 实际上就是减去偏移地址
~~~

##### 2. spinlock_t 自旋锁

###### 定义

```c
typedef struct spinlock {
     struct rt_mutex_base    lock;
#ifdef CONFIG_DEBUG_LOCK_ALLOC
     struct lockdep_map  dep_map;
#endif
} spinlock_t;
```

###### 操作方法

~~~c
#include <linux/spinlock.h>

DEFINE_SPINLOCK(lock1);
spinlock_t lock2;

spin_lock_init(&lock2);

spin_lock(&lock1);
/* 临界区（critical region） */
spin_unlock(&lock1);

spin_lock(&lock2);
/* 临界区 */
spin_unlock(&lock2);
~~~

##### 3. mutex 互斥锁

###### 定义

```c
struct mutex {
        atomic_long_t           owner;
        spinlock_t              wait_lock;
#ifdef CONFIG_MUTEX_SPIN_ON_OWNER
        struct optimistic_spin_queue osq; /* Spinner MCS lock */
#endif
        struct list_head        wait_list;		// 有一个等待队列
#ifdef CONFIG_DEBUG_MUTEXES
        void                    *magic;
#endif
#ifdef CONFIG_DEBUG_LOCK_ALLOC
        struct lockdep_map      dep_map;
#endif
};
// 其实是用spinlock实现的
```

###### 操作方法

```c
#include <linux/mutex.h>

/* 互斥锁初始化函数 */
void mutex_init(struct mutex *mutex);
DEFINE_MUTEX(name);

/* 互斥锁获取函数 */
void mutex_lock(struct mutex *mutex);

/* 互斥锁释放函数 */
void mutex_unlock(struct mutex *mutex);
```

##### 4. acomic_t 原子变量

###### 定义

```c
typedef struct {
        int counter;
} atomic_t;
// 其实就包含了一个int
```

###### 操作函数

```c
#include <asm/atomic.h>

void atomic_set(atomic_t *v, int i);
int atomic_read(atomic_t *v);
void atomic_add(int i, atomic_t *v);
void atomic_sub(int i, atomic_t *v);
void atomic_inc(atomic_t *v);
void atomic_dec(atomic_t *v);
int atomic_inc_and_test(atomic_t *v);
int atomic_dec_and_test(atomic_t *v);
int atomic_cmpxchg(atomic_t *v, int old, int new);
```

原子变量的实现和架构有关，比如在x86架构上，实现的原理就是在汇编上使用lock

###### 原子位操作函数

```c
#include <asm/bitops.h>

void set_bit(int nr, void *addr);
void clear_bit(int nr, void *addr);
void change_bit(int nr, void *addr);
int test_and_set_bit(int nr, void *addr);
int test_and_clear_bit(int nr, void *addr);
int test_and_change_bit(int nr, void *addr);
```



#### 三、进程

##### 1. 结构体

###### task_struct

描述进程的描述符

```c
// 主要的成员变量
// 进程标识
pid_t pid;                      // 进程ID
pid_t tgid;                     // 线程组ID（主线程PID）
char comm[TASK_COMM_LEN];       // 进程名

// 描述进程关系
struct task_struct *parent;     // 父进程（接收SIGCHLD）
struct task_struct *real_parent;// 真正的父进程
struct list_head children;      // 子进程链表
struct list_head sibling;       // 兄弟进程链表
struct task_struct *group_leader;// 线程组领导者

// 进程状态
unsigned int __state;           // 进程状态（TASK_RUNNING等）
int exit_state;                 // 退出状态
int exit_code;                  // 退出代码
unsigned long flags;            // 进程标志（PF_*）

// 进程调度
unsigned int __state;           // 进程状态（TASK_RUNNING等）
int exit_state;                 // 退出状态
int exit_code;                  // 退出代码
unsigned long flags;            // 进程标志（PF_*）

// 内存管理
struct mm_struct *mm;           // 内存描述符（用户空间）
struct mm_struct *active_mm;    // 活动内存描述符

// 资源管理
struct files_struct *files;     // 打开文件表
```

##### 2. 函数宏定义等

###### current

获取当前进程的pid

~~~c
#define current get_current()

static __always_inline struct task_struct *get_current(void)
{
	if (IS_ENABLED(CONFIG_USE_X86_SEG_SUPPORT))
		return this_cpu_read_const(const_current_task);

	return this_cpu_read_stable(current_task);
}
~~~

#### 四、内存管理

##### 1. 结构体

##### 2. 函数宏定义等

###### kmalloc(size, flag) **申请内存**

flag 参数

- GFP_KERNEL ——使用此值可能导致当前进程被挂起。因此，它不能在中断上下文中使用。` 

- GFP_ATOMIC ——使用此值确保 kmalloc() 函数不会挂起当前进程。它可以随时使用。

###### kfree() 释放内存



#### 五、字符设备驱动









