### linux io子模块

#### 一、VFS层

##### 1) 结构体

###### 1. strcut file

~~~~c
struct file {
	// 三者互斥使用，节省空间
	// f_llist      : lock-free 链表节点，用于文件关闭队列
	// f_rcuhead    : RCU 延迟释放文件对象
	// f_iocb_flags : 异步 I/O (AIO) 时使用的标志
	union {
		struct llist_node	f_llist;
		struct rcu_head 		f_rcuhead;
		unsigned int 		f_iocb_flags;
	};

	// 文件锁，保护 f_flags 和 f_ep
	// 注意：不能在 IRQ 上下文获取
	spinlock_t		f_lock;

	// 文件模式，例如 O_RDONLY、O_WRONLY、O_RDWR
	fmode_t			f_mode;

	// 文件引用计数，多线程安全
	atomic_long_t		f_count;

	// 保护文件偏移 f_pos 的互斥锁
	struct mutex		f_pos_lock;

	// 文件当前读写偏移
	loff_t			f_pos;

	// 文件状态标志，例如 O_APPEND、O_NONBLOCK
	unsigned int		f_flags;

	// 文件拥有者信息，用于 F_SETOWN/F_GETOWN 信号机制
	struct fown_struct	f_owner;

	// 文件创建时的凭证信息（用户、组等）
	const struct cred	*f_cred;

	// read-ahead 状态，用于顺序读取优化
	struct file_ra_state	f_ra;

	// 文件对应的路径
	struct path		f_path;

	// 缓存的 inode 指针，加快文件访问
	struct inode		*f_inode;

	// 指向文件操作表（read/write/ioctl 等函数指针集合）
	const struct file_operations	*f_op;

	// 文件版本号，NFS 等使用
	u64			f_version;

#ifdef CONFIG_SECURITY
	// 安全模块使用，例如 SELinux
	void			*f_security;
#endif

	// 私有数据指针，用于驱动或文件系统自定义信息
	void			*private_data;

#ifdef CONFIG_EPOLL
	// epoll 相关：链接到事件通知列表
	struct hlist_head	*f_ep;
#endif

	// 文件对应的内存映射（page cache）
	struct address_space	*f_mapping;

	// 写回错误状态
	errseq_t		f_wb_err;

	// 同步文件系统错误，供 syncfs 使用
	errseq_t		f_sb_err;

} __randomize_layout
  __attribute__((aligned(4))); // 保证 4 字节对齐，防止异常对齐

~~~~

###### 2. struct file_operations

~~~c
struct file_operations {
	// 所属模块指针，用于防止模块卸载时文件还在使用
	struct module *owner;

	// 文件定位接口（lseek）
	// 调整文件偏移量 f_pos，受 f_pos_lock 保护
	loff_t (*llseek) (struct file *, loff_t, int);

	// 同步读写接口
	// read/write 会修改 f_pos，需要 f_pos_lock
	ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
	ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);

	// 高级异步/分散聚集读写接口
	ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
	ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);

	// 异步 I/O 完成轮询接口
	int (*iopoll)(struct kiocb *kiocb, struct io_comp_batch *, unsigned int flags);

	// 目录迭代接口 (readdir)
	// iterate_shared 遍历目录项
	int (*iterate_shared) (struct file *, struct dir_context *);

	// poll/select/epoll 支持
	__poll_t (*poll) (struct file *, struct poll_table_struct *);

	// ioctl 接口
	// unlocked_ioctl 是普通 ioctl，compat_ioctl 用于 32-bit 兼容
	long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
	long (*compat_ioctl) (struct file *, unsigned int, unsigned long);

	// 内存映射接口
	// mmap 申请虚拟内存区域，mmap_supported_flags 表示可支持的标志
	int (*mmap) (struct file *, struct vm_area_struct *);
	unsigned long mmap_supported_flags;

	// 打开 / 刷新 / 关闭
	// open/release 对 inode 和 file 做初始化和清理
	int (*open) (struct inode *, struct file *);
	int (*flush) (struct file *, fl_owner_t id);
	int (*release) (struct inode *, struct file *);

	// 文件同步
	// fsync 将文件数据或元数据写回磁盘
	int (*fsync) (struct file *, loff_t, loff_t, int datasync);

	// 异步通知（SIGIO）
	int (*fasync) (int, struct file *, int);

	// 文件锁接口
	// lock/flock/setlease 实现 POSIX 文件锁 / advisory lock / lease
	int (*lock) (struct file *, int, struct file_lock *);
	int (*flock) (struct file *, int, struct file_lock *);
	int (*setlease)(struct file *, int, struct file_lock **, void **);

	// 零拷贝 splice 接口
	ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
	ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
	void (*splice_eof)(struct file *);

	// 预分配磁盘空间
	long (*fallocate)(struct file *file, int mode, loff_t offset, loff_t len);

	// 调试接口
	void (*show_fdinfo)(struct seq_file *m, struct file *f);

#ifndef CONFIG_MMU
	// 非 MMU 系统 mmap 支持
	unsigned (*mmap_capabilities)(struct file *);
#endif

	// 高性能文件复制 / remap
	ssize_t (*copy_file_range)(struct file *, loff_t, struct file *, loff_t, size_t, unsigned int);
	loff_t (*remap_file_range)(struct file *file_in, loff_t pos_in,
				   struct file *file_out, loff_t pos_out,
				   loff_t len, unsigned int remap_flags);

	// 文件访问模式提示
	int (*fadvise)(struct file *, loff_t, loff_t, int);

	// io_uring 异步 I/O 接口
	int (*uring_cmd)(struct io_uring_cmd *ioucmd, unsigned int issue_flags);
	int (*uring_cmd_iopoll)(struct io_uring_cmd *, struct io_comp_batch *, unsigned int poll_flags);

} __randomize_layout; // 布局随机化，提高安全性

~~~

###### 3. struct inode

~~~c
struct inode {
	// 文件权限和类型 (S_IFREG/S_IFDIR/S_IRUSR 等)
	umode_t			i_mode;

	// 文件操作标志，如 iopflags
	unsigned short		i_opflags;

	// 文件拥有者 uid / gid
	kuid_t			i_uid;
	kgid_t			i_gid;

	// 文件状态标志 (immutable, append-only, etc.)
	unsigned int		i_flags;

#ifdef CONFIG_FS_POSIX_ACL
	// POSIX ACL 权限
	struct posix_acl	*i_acl;
	struct posix_acl	*i_default_acl;
#endif

	// 指向 inode 操作函数表（类似 file_operations）
	const struct inode_operations	*i_op;

	// 指向超级块
	struct super_block	*i_sb;

	// 文件数据的映射，page cache
	struct address_space	*i_mapping;

#ifdef CONFIG_SECURITY
	// 安全模块，如 SELinux、SMACK
	void			*i_security;
#endif

	// inode 编号
	unsigned long		i_ino;

	// 链接计数，直接读取 i_nlink，修改需用函数
	union {
		const unsigned int i_nlink;
		unsigned int __i_nlink;
	};

	// 特殊设备号（字符/块设备）
	dev_t			i_rdev;

	// 文件大小
	loff_t			i_size;

	// 访问、修改、状态更改时间
	struct timespec64	i_atime;
	struct timespec64	i_mtime;
	struct timespec64	__i_ctime;

	// inode 保护锁
	spinlock_t		i_lock; // 保护 i_blocks, i_bytes, 可能 i_size
	unsigned short          i_bytes;
	u8			i_blkbits;
	u8			i_write_hint;
	blkcnt_t		i_blocks;

#ifdef __NEED_I_SIZE_ORDERED
	seqcount_t		i_size_seqcount; // size 的序列计数
#endif

	// inode 状态
	unsigned long		i_state;

	// 文件系统读写信号量
	struct rw_semaphore	i_rwsem;

	// 脏数据时间记录
	unsigned long		dirtied_when;
	unsigned long		dirtied_time_when;

	// 哈希链表节点，用于 inode 缓存
	struct hlist_node	i_hash;

	// IO 队列
	struct list_head	i_io_list;

#ifdef CONFIG_CGROUP_WRITEBACK
	// 写回关联的 cgroup wb
	struct bdi_writeback	*i_wb;
	int			i_wb_frn_winner;
	u16			i_wb_frn_avg_time;
	u16			i_wb_frn_history;
#endif

	// inode LRU 链表
	struct list_head	i_lru;
	struct list_head	i_sb_list;
	struct list_head	i_wb_list;

	union {
		// dentry 哈希头，用于 dentry 链表
		struct hlist_head	i_dentry;
		struct rcu_head		i_rcu; // RCU 延迟回收
	};

	// 文件版本号 / 序列号（futex 等使用）
	atomic64_t		i_version;
	atomic64_t		i_sequence;

	// 引用计数
	atomic_t		i_count;      // 打开的 inode 计数
	atomic_t		i_dio_count;  // 直接 IO 计数
	atomic_t		i_writecount; // 写计数
#if defined(CONFIG_IMA) || defined(CONFIG_FILE_LOCKING)
	atomic_t		i_readcount;  // 文件只读打开计数
#endif

	union {
		const struct file_operations	*i_fop; // default_file_ops
		void (*free_inode)(struct inode *);
	};

	// 文件锁上下文
	struct file_lock_context	*i_flctx;

	// 文件数据地址空间（page cache）
	struct address_space	i_data;

	// 设备列表
	struct list_head	i_devices;

	union {
		struct pipe_inode_info	*i_pipe;  // 管道
		struct cdev		*i_cdev;   // 字符设备
		char			*i_link;   // 符号链接
		unsigned		i_dir_seq; // 目录序列号
	};

	// inode generation number
	__u32			i_generation;

#ifdef CONFIG_FSNOTIFY
	__u32			i_fsnotify_mask; // inode 关注事件
	struct fsnotify_mark_connector __rcu	*i_fsnotify_marks;
#endif

#ifdef CONFIG_FS_ENCRYPTION
	struct fscrypt_info	*i_crypt_info; // 文件加密信息
#endif

#ifdef CONFIG_FS_VERITY
	struct fsverity_info	*i_verity_info; // 文件完整性校验信息
#endif

	// 文件系统或设备私有指针
	void			*i_private;
} __randomize_layout;

~~~

###### 4. struct inode_oerations

~~~c
struct inode_operations {
	// 查找目录项（目录 inode 中）
	struct dentry * (*lookup) (struct inode *, struct dentry *, unsigned int);

	// 获取符号链接的目标路径
	const char * (*get_link) (struct dentry *, struct inode *, struct delayed_call *);

	// 权限检查
	int (*permission) (struct mnt_idmap *, struct inode *, int);

	// 获取 inode 的 ACL
	struct posix_acl * (*get_inode_acl)(struct inode *, int, bool);

	// 读取符号链接
	int (*readlink) (struct dentry *, char __user *, int);

	// 创建新文件
	int (*create) (struct mnt_idmap *, struct inode *, struct dentry *,
		       umode_t, bool);

	// 创建硬链接
	int (*link) (struct dentry *, struct inode *, struct dentry *);

	// 删除文件
	int (*unlink) (struct inode *, struct dentry *);

	// 创建符号链接
	int (*symlink) (struct mnt_idmap *, struct inode *, struct dentry *,
			const char *);

	// 创建目录
	int (*mkdir) (struct mnt_idmap *, struct inode *, struct dentry *,
		      umode_t);

	// 删除目录
	int (*rmdir) (struct inode *, struct dentry *);

	// 创建设备节点
	int (*mknod) (struct mnt_idmap *, struct inode *, struct dentry *,
		      umode_t, dev_t);

	// 重命名文件或目录
	int (*rename) (struct mnt_idmap *, struct inode *, struct dentry *,
			struct inode *, struct dentry *, unsigned int);

	// 修改 inode 属性（如权限、时间戳）
	int (*setattr) (struct mnt_idmap *, struct dentry *, struct iattr *);

	// 获取 inode 属性
	int (*getattr) (struct mnt_idmap *, const struct path *,
			struct kstat *, u32, unsigned int);

	// 列出扩展属性（xattr）
	ssize_t (*listxattr) (struct dentry *, char *, size_t);

	// 文件映射信息
	int (*fiemap)(struct inode *, struct fiemap_extent_info *, u64 start, u64 len);

	// 更新时间（如访问时间、修改时间）
	int (*update_time)(struct inode *, int);

	// 原子打开文件接口
	int (*atomic_open)(struct inode *, struct dentry *,
			   struct file *, unsigned open_flag,
			   umode_t create_mode);

	// 创建临时文件
	int (*tmpfile) (struct mnt_idmap *, struct inode *,
			struct file *, umode_t);

	// ACL 获取/设置
	struct posix_acl *(*get_acl)(struct mnt_idmap *, struct dentry *, int);
	int (*set_acl)(struct mnt_idmap *, struct dentry *, struct posix_acl *, int);

	// 文件属性设置/获取（generic fileattr 接口）
	int (*fileattr_set)(struct mnt_idmap *, struct dentry *, struct fileattr *fa);
	int (*fileattr_get)(struct dentry *, struct fileattr *fa);

	// 获取文件偏移上下文
	struct offset_ctx *(*get_offset_ctx)(struct inode *inode);
} ____cacheline_aligned;

~~~

###### 6. struct address_space

~~~c
struct address_space {
	// 指向所属 inode
	struct inode		*host;

	// 存储文件的所有缓存页（xarray 高性能数组）
	struct xarray		i_pages;

	// 页无效化锁（invalidate 页时保护）
	struct rw_semaphore	invalidate_lock;

	// 页分配标志
	gfp_t			gfp_mask;

	// 文件是否可写 mmap 的计数
	atomic_t		i_mmap_writable;

#ifdef CONFIG_READ_ONLY_THP_FOR_FS
	// 非共享内存文件的大页计数（Transparent Huge Page）
	atomic_t		nr_thps;
#endif

	// 内存映射树（文件映射区段）
	struct rb_root_cached	i_mmap;

	// 页缓存中页数量
	unsigned long		nrpages;

	// 写回索引，用于 writeback
	pgoff_t			writeback_index;

	// address_space 操作函数表
	const struct address_space_operations *a_ops;

	// 标志位（如 AS_SHARED、AS_MAPPED）
	unsigned long		flags;

	// mmap 锁（保护 i_mmap RB 树）
	struct rw_semaphore	i_mmap_rwsem;

	// 写回错误
	errseq_t		wb_err;

	// 私有数据锁
	spinlock_t		private_lock;

	// 私有数据链表
	struct list_head	private_list;

	// 文件系统或设备私有数据
	void			*private_data;

} __attribute__((aligned(sizeof(long)))) __randomize_layout;

~~~

###### 7. struct address_space_operations 

~~~c
struct address_space_operations {
	// 将单个页写回磁盘
	int (*writepage)(struct page *page, struct writeback_control *wbc);

	// 从文件读取 folio（新型页）
	int (*read_folio)(struct file *, struct folio *);

	// 将映射中多个脏页写回磁盘
	int (*writepages)(struct address_space *, struct writeback_control *);

	// 标记 folio 脏页，如果成功返回 true
	bool (*dirty_folio)(struct address_space *, struct folio *);

	// 预读页缓存
	void (*readahead)(struct readahead_control *);

	// 写入开始：获取/分配页
	int (*write_begin)(struct file *, struct address_space *mapping,
				loff_t pos, unsigned len,
				struct page **pagep, void **fsdata);

	// 写入结束：更新页、写回
	int (*write_end)(struct file *, struct address_space *mapping,
				loff_t pos, unsigned len, unsigned copied,
				struct page *page, void *fsdata);

	// FIBMAP：获取逻辑块映射到物理块
	sector_t (*bmap)(struct address_space *, sector_t);

	// 使 folio 的指定区域无效
	void (*invalidate_folio) (struct folio *, size_t offset, size_t len);

	// 释放 folio
	bool (*release_folio)(struct folio *, gfp_t);

	// 回收 folio
	void (*free_folio)(struct folio *folio);

	// 直接 IO（绕过页缓存）
	ssize_t (*direct_IO)(struct kiocb *, struct iov_iter *iter);

	// 将 folio 内容迁移到目标页
	int (*migrate_folio)(struct address_space *, struct folio *dst,
			struct folio *src, enum migrate_mode);

	// 回写 folio
	int (*launder_folio)(struct folio *);

	// 检查 folio 是否部分 uptodate
	bool (*is_partially_uptodate) (struct folio *, size_t from, size_t count);

	// 检查 folio 脏页与写回状态
	void (*is_dirty_writeback) (struct folio *, bool *dirty, bool *wb);

	// 处理页错误并移除页
	int (*error_remove_page)(struct address_space *, struct page *);

	/* swapfile 支持 */
	int (*swap_activate)(struct swap_info_struct *sis, struct file *file,
				sector_t *span);
	void (*swap_deactivate)(struct file *file);
	int (*swap_rw)(struct kiocb *iocb, struct iov_iter *iter);
};

~~~

###### 8.  struct page

~~~c
struct page {
	// 原子标志，表示页状态，如 PageDirty、PageLocked
	unsigned long flags;

	// 联合体：页的用途不同，使用不同的成员
	union {
		// 页缓存/匿名页
		struct {
			union {
				struct list_head lru;      // LRU 链表，用于 pageout
				struct { void *__filler; unsigned int mlock_count; }; // mlock 计数
				struct list_head buddy_list; // 空闲页链表
				struct list_head pcp_list;   // per-cpu 空闲链表
			};
			struct address_space *mapping; // 对应的 file/inode 地址空间
			union {
				pgoff_t index;     // 文件偏移
				unsigned long share; // fsdax share count
			};
			unsigned long private; // buffer_head/swap entry/page buddy order
		};

		// page_pool 网络栈使用
		struct {
			unsigned long pp_magic;       // 魔数，防止误用
			struct page_pool *pp;
			unsigned long _pp_mapping_pad;
			unsigned long dma_addr;
			union {
				unsigned long dma_addr_upper; // 64-bit DMA
				atomic_long_t pp_frag_count;  // frag page
			};
		};

		// compound page 尾页
		struct {
			unsigned long compound_head; // bit0=1 表示 tail
		};

		// ZONE_DEVICE 页
		struct {
			struct dev_pagemap *pgmap;
			void *zone_device_data;
		};

		// RCU 回收
		struct rcu_head rcu_head;
	};

	// 用户映射引用计数或页类型
	union {
		atomic_t _mapcount;   // 可映射到用户空间时计数
		unsigned int page_type; // 页类型
	};

	// 页引用计数
	atomic_t _refcount;

#ifdef CONFIG_MEMCG
	unsigned long memcg_data; // 内存控制组信息
#endif

#ifdef WANT_PAGE_VIRTUAL
	void *virtual; // 内核虚拟地址，高内存需 kmapping
#endif

#ifdef CONFIG_KMSAN
	struct page *kmsan_shadow; // KMSAN 内存检查 shadow page
	struct page *kmsan_origin; // KMSAN origin 信息
#endif

#ifdef LAST_CPUPID_NOT_IN_PAGE_FLAGS
	int _last_cpupid; // 上次访问页的 CPU/PID
#endif
};

~~~

###### 9. struct super_block

~~~c
struct super_block {
	struct list_head	s_list;   // 全局 super_block 链表
	dev_t			s_dev;    // 设备号，用于索引
	unsigned char		s_blocksize_bits; // 块大小位数
	unsigned long		s_blocksize;      // 块大小
	loff_t			s_maxbytes;      // 文件最大大小

	struct file_system_type	*s_type;   // 文件系统类型
	const struct super_operations	*s_op;   // superblock 操作函数
	const struct dquot_operations	*dq_op;  // 配额操作
	const struct quotactl_ops	*s_qcop; // 配额控制操作
	const struct export_operations *s_export_op; // NFS 导出操作

	unsigned long		s_flags; // 文件系统标志
	unsigned long		s_iflags; // 内部标志
	unsigned long		s_magic;  // 魔数（文件系统标识）
	struct dentry		*s_root; // 根目录 dentry

	struct rw_semaphore	s_umount; // 卸载锁
	int			s_count;  // 挂载计数
	atomic_t		s_active; // 活跃引用计数

	void			*s_fs_info; // 文件系统私有信息
	u32			s_time_gran; // 时间粒度
	time64_t		s_time_min, s_time_max; // 时间限制

	char			s_id[32]; // 文件系统标识（可读信息）
	uuid_t			s_uuid;   // 文件系统 UUID

	const char *s_subtype; // 文件系统子类型（如 ext4 / ext4dev）
	const struct dentry_operations *s_d_op; // dentry 默认操作

	struct shrinker s_shrink; // 回收器，用于 inode/dentry 缓存回收

	atomic_long_t s_remove_count; // 链接为0但还在使用的 inode 数
	atomic_long_t s_fsnotify_connectors; // fsnotify 连接数

	int s_readonly_remount; // 只读挂载状态
	errseq_t s_wb_err; // 写回错误

	struct user_namespace *s_user_ns; // 用户命名空间（UID/GID映射）

	struct list_lru s_dentry_lru; // dentry LRU cache
	struct list_lru s_inode_lru;  // inode LRU cache
	struct rcu_head rcu;           // RCU 回收
	struct work_struct destroy_work; // 销毁工作

	struct mutex s_sync_lock; // 文件系统同步锁
	int s_stack_depth;        // 文件系统栈深度（层叠挂载）
	
	spinlock_t s_inode_list_lock; // inode 链表锁
	struct list_head s_inodes;    // 挂载文件系统的所有 inode

	spinlock_t s_inode_wblist_lock; // 写回 inode 链表锁
	struct list_head s_inodes_wb;   // 待写回 inode 链表
};

~~~

##### 2) 结构体之间的关系

~~~scss
super_block (文件系统挂载实例)
 ├─ s_root -> dentry (根目录)
 ├─ s_inodes -> inode list (文件系统的所有 inode)
 └─ s_bdi / s_fs_info -> backing device 或 FS 私有数据

inode (文件节点)
 ├─ i_sb -> super_block (所属文件系统)
 ├─ i_mapping -> address_space (页缓存)
 ├─ i_fop -> file_operations (默认文件操作)
 ├─ i_op -> inode_operations (元数据操作)
 └─ i_dentry -> 链接到 dentry

dentry (目录项 / 路径组件)
 ├─ d_inode -> inode (对应文件节点)
 ├─ d_parent -> 父目录 dentry
 ├─ d_child / d_subdirs -> 子目录树
 └─ d_op -> dentry_operations (操作函数)

file (打开的文件实例)
 ├─ f_path -> path { dentry + vfsmount }
 ├─ f_inode -> inode (文件节点)
 ├─ f_op -> file_operations (打开时可能覆盖默认 i_fop)
 └─ f_mapping -> address_space (页缓存)

address_space (页缓存映射)
 ├─ host -> inode (属于哪个 inode)
 ├─ i_pages -> 存储 struct page
 └─ a_ops -> address_space_operations (页缓存操作)

page (内存页)
 ├─ mapping -> address_space (归属的页缓存)
 ├─ index -> 在文件内的页偏移
 └─ private -> 私有数据或 buffer_head

~~~

###### **super_block**

- 代表挂载的整个文件系统
- 指向根目录 `s_root`，指向所有 inode 的链表 `s_inodes`
- inode 通过 `i_sb` 回指 `super_block`

###### **inode**

- 核心文件元数据（权限、大小、时间、块映射）
- 通过 `i_mapping` 指向文件页缓存（`address_space`）
- 通过 `i_op` 指向 inode 操作函数
- 通过 `i_fop` 指向文件操作函数
- 可以被多个 dentry 引用（硬链接）

###### **dentry**

- VFS 路径缓存节点
- `d_inode` 指向对应的 inode
- 构建目录树结构，用于快速路径查找
- `d_op` 提供操作函数

**file**

- 打开的文件实例（进程级）
- 每次 `open()` 会创建一个 `struct file`
- `f_inode` 指向 inode
- `f_op` 指向文件操作函数（可能覆盖 i_fop）
- `f_path` 保存文件路径信息
- `f_mapping` 与 inode 的 `i_mapping` 一致，用于页缓存访问

###### **address_space**

- 页缓存映射表
- 每个 inode 可能有一个 address_space
- 保存文件内容的 page 缓存
- 通过 `a_ops` 提供读写页的操作

###### **page**

- 内存页
- 通过 `mapping` 回指 address_space
- 保存文件内容或私有数据
- 可通过 writeback 写入磁盘

##### 3) 函数

###### 1. vfs_write()

函数原型

~~~c
ssize_t vfs_write(struct file *file, const char __user *buf, size_t count, loff_t *pos)
{
    ssize_t ret;

    // 如果文件没有写权限，直接报错（FMODE_WRITE 是打开文件时决定的）
    if (!(file->f_mode & FMODE_WRITE))
        return -EBADF;

    // FMODE_CAN_WRITE 检查文件系统是否允许写，比如只读挂载
    if (!(file->f_mode & FMODE_CAN_WRITE))
        return -EINVAL;

    // 检查用户态 buf 地址是否合法，防止非法指针 / 越界
    if (unlikely(!access_ok(buf, count)))
        return -EFAULT;

    // 核心：检查写入范围（是否超出文件大小限制等）
    // 会更新 *pos，用于 append 模式等
    ret = rw_verify_area(WRITE, file, pos, count);
    if (ret)
        return ret;

    // 限制单次写最大字节数（内核安全限制）
    if (count > MAX_RW_COUNT)
        count = MAX_RW_COUNT;

    // 文件系统层：通知进入写操作（可能需要 journal 锁，ext4 需要）
    file_start_write(file);

    // 老式写接口（很少使用，ext4 不用）
    if (file->f_op->write)
        ret = file->f_op->write(file, buf, count, pos);

    // 主路径：使用 write_iter 回调（ext4、xfs、btrfs 都走这一条）
    else if (file->f_op->write_iter)
        ret = new_sync_write(file, buf, count, pos);

    // 没有 write 回调则不支持写
    else
        ret = -EINVAL;

    // 写成功后：上报 fsnotify（inotify/fanotify）
    if (ret > 0) {
        fsnotify_modify(file);   // 通知文件内容改变
        add_wchar(current, ret); // 统计写的字符数
    }

    inc_syscw(current);          // 统计系统调用写次数

    file_end_write(file);        // write 对应的结束动作（释放 journal lock）

    return ret;
}

~~~

###### 2. vfs_read()

~~~c
ssize_t vfs_read(struct file *file, char __user *buf, size_t count, loff_t *pos)
{
    ssize_t ret;

    // 检查文件是否允许读（open 时 O_RDONLY / O_RDWR）
    if (!(file->f_mode & FMODE_READ))
        return -EBADF;

    // 检查是否有读能力（文件系统层做的额外限制）
    if (!(file->f_mode & FMODE_CAN_READ))
        return -EINVAL;

    // 检查用户态地址是否合法（防止 copy_to_user 出错）
    if (unlikely(!access_ok(buf, count)))
        return -EFAULT;

    // 调用 rw_verify_area 检查偏移 + 权限等是否正常
    // 会检查 *pos 是否越界、文件是否可读等
    ret = rw_verify_area(READ, file, pos, count);
    if (ret)
        return ret;

    // 内核限制单次读写最大大小（通常是 ~2GB）
    if (count > MAX_RW_COUNT)
        count = MAX_RW_COUNT;

    // 调用具体文件操作的 read 或 read_iter
    // 根据底层文件系统 / 设备来决定具体实现

    if (file->f_op->read)
        // 老式 read 回调（很少使用了）
        ret = file->f_op->read(file, buf, count, pos);
    else if (file->f_op->read_iter)
        // 新式的基于 iov_iter 的 read 接口（现代文件系统使用）
        ret = new_sync_read(file, buf, count, pos);
    else
        // 文件系统没有实现读 => 错误
        ret = -EINVAL;

    // 如果 ret > 0（成功读取了数据）
    if (ret > 0) {
        // 通知 fsnotify：有文件读取行为
        fsnotify_access(file);

        // 增加进程的“读字符统计”
        add_rchar(current, ret);
    }

    // 增加系统调用计数（sys_read 计数）
    inc_syscr(current);

    // 返回读取的字节数 / 错误码
    return ret;
}

~~~

###### 3. do_iter_readv_writev()

~~~c
static ssize_t do_iter_readv_writev(struct file *filp, struct iov_iter *iter,
		loff_t *ppos, int type, rwf_t flags)
{
	struct kiocb kiocb;  // 内核异步 I/O 控制块，用于封装一次 I/O 请求
	ssize_t ret;         // 返回值，用于存放读写结果

	// 初始化同步 kiocb，设置文件指针 filp 并标记为同步 I/O
	init_sync_kiocb(&kiocb, filp);

	// 设置 kiocb 的读写标志，比如 RWF_NOWAIT / RWF_HIPRI
	ret = kiocb_set_rw_flags(&kiocb, flags);
	if (ret)
		return ret;   // 如果设置标志失败，直接返回错误

	// 设置 kiocb 的文件偏移位置
	kiocb.ki_pos = (ppos ? *ppos : 0);

	// 根据类型执行读或写
	if (type == READ)
		ret = call_read_iter(filp, &kiocb, iter);   // 调用 VFS 层 read_iter
	else
		ret = call_write_iter(filp, &kiocb, iter);  // 调用 VFS 层 write_iter

	// 同步 I/O 不应该返回 -EIOCBQUEUED，如果返回就 BUG
	BUG_ON(ret == -EIOCBQUEUED);

	// 如果用户传入文件偏移指针，更新它
	if (ppos)
		*ppos = kiocb.ki_pos;

	return ret;  // 返回实际读写的字节数或错误码
}

~~~



#### 二、block层

##### 2） 结构体

###### 1. struct block_device

~~~c
// include/linux/blk_types.h

struct block_device {
    sector_t        bd_start_sect;      // 分区起始扇区（整盘为 0），用于分区映射
    sector_t        bd_nr_sectors;      // 该 block_device 的总扇区数（决定设备大小）

    struct gendisk *    bd_disk;        // 指向所属 gendisk（整盘），提供驱动访问、名字、队列等

    struct request_queue *  bd_queue;   // I/O 请求队列（仅整盘设备拥有），提供IO调度/合并/提交入口

    struct disk_stats __percpu *bd_stats; // 每 CPU 的读写统计，用于 /proc/diskstats

    unsigned long       bd_stamp;       // 元数据更新时间戳，用于内部刷新判断

    bool            bd_read_only;       // 逻辑只读标记（光盘、dm-verity、blockdev --setro）

    u8              bd_partno;          // 分区号：0=整盘，1=sda1，2=sda2...

    bool            bd_write_holder;    // 是否有写持有者（文件系统 mount 时设置防止多重 mount）

    bool            bd_has_submit_bio;  // 设备是否有 submit_bio 钩子（优化路径用）

    dev_t           bd_dev;             // 设备号（major:minor），如 259:0 对应 nvme0n1

    atomic_t        bd_openers;         // 打开计数（文件系统、LVM、dd 等都会增加）

    spinlock_t      bd_size_lock;       // 保护 bd_inode->i_size 更新（设备容量变更时）

    struct inode *      bd_inode;       // 块设备对应的 VFS inode（/dev/sda 的 inode）

    void *          bd_claiming;        // 正在声明占有此设备的模块（例如 dm-crypt/LVM）

    void *          bd_holder;          // 当前真正持有设备的实体（文件系统、LVM、mdraid）

    const struct blk_holder_ops *bd_holder_ops; // 持有者的操作函数，用于验证/释放逻辑

    struct mutex        bd_holder_lock; // 保护 bd_holder 访问的锁

    int             bd_fsfreeze_count;  // 文件系统冻结计数（fsfreeze -f）

    int             bd_holders;         // 持有者数量（引用计数）

    struct kobject      *bd_holder_dir; // sysfs 中展示 holder 信息的目录

    struct mutex        bd_fsfreeze_mutex; // 冻结操作 mutex

    struct super_block  *bd_fsfreeze_sb; // 被冻结的 superblock（fsfreeze 时挂载的文件系统）

    struct partition_meta_info *bd_meta_info; // GPT/MBR 分区元信息（UUID 等）

#ifdef CONFIG_FAIL_MAKE_REQUEST
    bool            bd_make_it_fail;    // 让 IO 提交失败（测试用）
#endif

    struct device       bd_device;      // 嵌入的 struct device，连接 sysfs/udev/drv model
} __randomize_layout;
~~~

###### 2. struct bio

~~~c
// include/linux/blk_types.h

struct bio {
	struct bio		*bi_next;	/* request queue link */
	struct block_device	*bi_bdev;
	blk_opf_t		bi_opf;		/* bottom bits REQ_OP, top bits
						 * req_flags.
						 */
	unsigned short		bi_flags;	/* BIO_* below */
	unsigned short		bi_ioprio;
	blk_status_t		bi_status;
	atomic_t		__bi_remaining;

	struct bvec_iter	bi_iter;

	blk_qc_t		bi_cookie;
	bio_end_io_t		*bi_end_io;
	void			*bi_private;
#ifdef CONFIG_BLK_CGROUP
	/*
	 * Represents the association of the css and request_queue for the bio.
	 * If a bio goes direct to device, it will not have a blkg as it will
	 * not have a request_queue associated with it.  The reference is put
	 * on release of the bio.
	 */
	struct blkcg_gq		*bi_blkg;
	struct bio_issue	bi_issue;
#ifdef CONFIG_BLK_CGROUP_IOCOST
	u64			bi_iocost_cost;
#endif
#endif

#ifdef CONFIG_BLK_INLINE_ENCRYPTION
	struct bio_crypt_ctx	*bi_crypt_context;
#endif

	union {
#if defined(CONFIG_BLK_DEV_INTEGRITY)
		struct bio_integrity_payload *bi_integrity; /* data integrity */
#endif
	};

	unsigned short		bi_vcnt;	/* how many bio_vec's */

	/*
	 * Everything starting with bi_max_vecs will be preserved by bio_reset()
	 */

	unsigned short		bi_max_vecs;	/* max bvl_vecs we can hold */

	atomic_t		__bi_cnt;	/* pin count */

	struct bio_vec		*bi_io_vec;	/* the actual vec list */

	struct bio_set		*bi_pool;

	/*
	 * We can inline a number of vecs at the end of the bio, to avoid
	 * double allocations for a small number of bio_vecs. This member
	 * MUST obviously be kept at the very end of the bio.
	 */
	struct bio_vec		bi_inline_vecs[];
};
~~~

###### 4. struct gendisk

~~~c
// include/linux/blkdev.h

struct gendisk {
	/*
	 * major/first_minor/minors should not be set by any new driver, the
	 * block core will take care of allocating them automatically.
	 */
	int major;
	int first_minor;
	int minors;

	char disk_name[DISK_NAME_LEN];	/* name of major driver */

	unsigned short events;		/* supported events */
	unsigned short event_flags;	/* flags related to event processing */

	struct xarray part_tbl;
	struct block_device *part0;

	const struct block_device_operations *fops;
	struct request_queue *queue;
	void *private_data;

	struct bio_set bio_split;

	int flags;
	unsigned long state;
#define GD_NEED_PART_SCAN		0
#define GD_READ_ONLY			1
#define GD_DEAD				2
#define GD_NATIVE_CAPACITY		3
#define GD_ADDED			4
#define GD_SUPPRESS_PART_SCAN		5
#define GD_OWNS_QUEUE			6

	struct mutex open_mutex;	/* open/close mutex */
	unsigned open_partitions;	/* number of open partitions */

	struct backing_dev_info	*bdi;
	struct kobject queue_kobj;	/* the queue/ directory */
	struct kobject *slave_dir;
#ifdef CONFIG_BLOCK_HOLDER_DEPRECATED
	struct list_head slave_bdevs;
#endif
	struct timer_rand_state *random;
	atomic_t sync_io;		/* RAID */
	struct disk_events *ev;

#ifdef CONFIG_BLK_DEV_ZONED
	/*
	 * Zoned block device information for request dispatch control.
	 * nr_zones is the total number of zones of the device. This is always
	 * 0 for regular block devices. conv_zones_bitmap is a bitmap of nr_zones
	 * bits which indicates if a zone is conventional (bit set) or
	 * sequential (bit clear). seq_zones_wlock is a bitmap of nr_zones
	 * bits which indicates if a zone is write locked, that is, if a write
	 * request targeting the zone was dispatched.
	 *
	 * Reads of this information must be protected with blk_queue_enter() /
	 * blk_queue_exit(). Modifying this information is only allowed while
	 * no requests are being processed. See also blk_mq_freeze_queue() and
	 * blk_mq_unfreeze_queue().
	 */
	unsigned int		nr_zones;
	unsigned int		max_open_zones;
	unsigned int		max_active_zones;
	unsigned long		*conv_zones_bitmap;
	unsigned long		*seq_zones_wlock;
#endif /* CONFIG_BLK_DEV_ZONED */

#if IS_ENABLED(CONFIG_CDROM)
	struct cdrom_device_info *cdi;
#endif
	int node_id;
	struct badblocks *bb;
	struct lockdep_map lockdep_map;
	u64 diskseq;
	blk_mode_t open_mode;

	/*
	 * Independent sector access ranges. This is always NULL for
	 * devices that do not have multiple independent access ranges.
	 */
	struct blk_independent_access_ranges *ia_ranges;
};
~~~

###### 5. struct request_queue

~~~c

struct request_queue {
	struct request		*last_merge;
	struct elevator_queue	*elevator;

	struct percpu_ref	q_usage_counter;

	struct blk_queue_stats	*stats;
	struct rq_qos		*rq_qos;
	struct mutex		rq_qos_mutex;

	const struct blk_mq_ops	*mq_ops;

	/* sw queues */
	struct blk_mq_ctx __percpu	*queue_ctx;

	unsigned int		queue_depth;

	/* hw dispatch queues */
	struct xarray		hctx_table;
	unsigned int		nr_hw_queues;

	/*
	 * The queue owner gets to use this for whatever they like.
	 * ll_rw_blk doesn't touch it.
	 */
	void			*queuedata;

	/*
	 * various queue flags, see QUEUE_* below
	 */
	unsigned long		queue_flags;
	/*
	 * Number of contexts that have called blk_set_pm_only(). If this
	 * counter is above zero then only RQF_PM requests are processed.
	 */
	atomic_t		pm_only;

	/*
	 * ida allocated id for this queue.  Used to index queues from
	 * ioctx.
	 */
	int			id;

	spinlock_t		queue_lock;

	struct gendisk		*disk;

	refcount_t		refs;

	/*
	 * mq queue kobject
	 */
	struct kobject *mq_kobj;

#ifdef  CONFIG_BLK_DEV_INTEGRITY
	struct blk_integrity integrity;
#endif	/* CONFIG_BLK_DEV_INTEGRITY */

#ifdef CONFIG_PM
	struct device		*dev;
	enum rpm_status		rpm_status;
#endif

	/*
	 * queue settings
	 */
	unsigned long		nr_requests;	/* Max # of requests */

	unsigned int		dma_pad_mask;

#ifdef CONFIG_BLK_INLINE_ENCRYPTION
	struct blk_crypto_profile *crypto_profile;
	struct kobject *crypto_kobject;
#endif

	unsigned int		rq_timeout;

	struct timer_list	timeout;
	struct work_struct	timeout_work;

	atomic_t		nr_active_requests_shared_tags;

	struct blk_mq_tags	*sched_shared_tags;

	struct list_head	icq_list;
#ifdef CONFIG_BLK_CGROUP
	DECLARE_BITMAP		(blkcg_pols, BLKCG_MAX_POLS);
	struct blkcg_gq		*root_blkg;
	struct list_head	blkg_list;
	struct mutex		blkcg_mutex;
#endif

	struct queue_limits	limits;

	unsigned int		required_elevator_features;

	int			node;
#ifdef CONFIG_BLK_DEV_IO_TRACE
	struct blk_trace __rcu	*blk_trace;
#endif
	/*
	 * for flush operations
	 */
	struct blk_flush_queue	*fq;
	struct list_head	flush_list;

	struct list_head	requeue_list;
	spinlock_t		requeue_lock;
	struct delayed_work	requeue_work;

	struct mutex		sysfs_lock;
	struct mutex		sysfs_dir_lock;

	/*
	 * for reusing dead hctx instance in case of updating
	 * nr_hw_queues
	 */
	struct list_head	unused_hctx_list;
	spinlock_t		unused_hctx_lock;

	int			mq_freeze_depth;

#ifdef CONFIG_BLK_DEV_THROTTLING
	/* Throttle data */
	struct throtl_data *td;
#endif
	struct rcu_head		rcu_head;
	wait_queue_head_t	mq_freeze_wq;
	/*
	 * Protect concurrent access to q_usage_counter by
	 * percpu_ref_kill() and percpu_ref_reinit().
	 */
	struct mutex		mq_freeze_lock;

	int			quiesce_depth;

	struct blk_mq_tag_set	*tag_set;
	struct list_head	tag_set_list;

	struct dentry		*debugfs_dir;
	struct dentry		*sched_debugfs_dir;
	struct dentry		*rqos_debugfs_dir;
	/*
	 * Serializes all debugfs metadata operations using the above dentries.
	 */
	struct mutex		debugfs_mutex;

	bool			mq_sysfs_init_done;
};
~~~

###### 6. struct request

~~~c
/*
 * Try to put the fields that are referenced together in the same cacheline.
 *
 * If you modify this structure, make sure to update blk_rq_init() and
 * especially blk_mq_rq_ctx_init() to take care of the added fields.
 */
struct request {
	struct request_queue *q;
	struct blk_mq_ctx *mq_ctx;
	struct blk_mq_hw_ctx *mq_hctx;

	blk_opf_t cmd_flags;		/* op and common flags */
	req_flags_t rq_flags;

	int tag;
	int internal_tag;

	unsigned int timeout;

	/* the following two fields are internal, NEVER access directly */
	unsigned int __data_len;	/* total data len */
	sector_t __sector;		/* sector cursor */

	struct bio *bio;
	struct bio *biotail;

	union {
		struct list_head queuelist;
		struct request *rq_next;
	};

	struct block_device *part;
#ifdef CONFIG_BLK_RQ_ALLOC_TIME
	/* Time that the first bio started allocating this request. */
	u64 alloc_time_ns;
#endif
	/* Time that this request was allocated for this IO. */
	u64 start_time_ns;
	/* Time that I/O was submitted to the device. */
	u64 io_start_time_ns;

#ifdef CONFIG_BLK_WBT
	unsigned short wbt_flags;
#endif
	/*
	 * rq sectors used for blk stats. It has the same value
	 * with blk_rq_sectors(rq), except that it never be zeroed
	 * by completion.
	 */
	unsigned short stats_sectors;

	/*
	 * Number of scatter-gather DMA addr+len pairs after
	 * physical address coalescing is performed.
	 */
	unsigned short nr_phys_segments;

#ifdef CONFIG_BLK_DEV_INTEGRITY
	unsigned short nr_integrity_segments;
#endif

#ifdef CONFIG_BLK_INLINE_ENCRYPTION
	struct bio_crypt_ctx *crypt_ctx;
	struct blk_crypto_keyslot *crypt_keyslot;
#endif

	unsigned short ioprio;

	enum mq_rq_state state;
	atomic_t ref;

	unsigned long deadline;

	/*
	 * The hash is used inside the scheduler, and killed once the
	 * request reaches the dispatch list. The ipi_list is only used
	 * to queue the request for softirq completion, which is long
	 * after the request has been unhashed (and even removed from
	 * the dispatch list).
	 */
	union {
		struct hlist_node hash;	/* merge hash */
		struct llist_node ipi_list;
	};

	/*
	 * The rb_node is only used inside the io scheduler, requests
	 * are pruned when moved to the dispatch queue. special_vec must
	 * only be used if RQF_SPECIAL_PAYLOAD is set, and those cannot be
	 * insert into an IO scheduler.
	 */
	union {
		struct rb_node rb_node;	/* sort/lookup */
		struct bio_vec special_vec;
	};

	/*
	 * Three pointers are available for the IO schedulers, if they need
	 * more they have to dynamically allocate it.
	 */
	struct {
		struct io_cq		*icq;
		void			*priv[2];
	} elv;

	struct {
		unsigned int		seq;
		rq_end_io_fn		*saved_end_io;
	} flush;

	u64 fifo_time;

	/*
	 * completion callback.
	 */
	rq_end_io_fn *end_io;
	void *end_io_data;
};

~~~

###### 7. struct bio

~~~c
// include/linux/blk_types.h

/*
 * main unit of I/O for the block layer and lower layers (ie drivers and
 * stacking drivers)
 */
struct bio {
	struct bio		*bi_next;	/* request queue link */
	struct block_device	*bi_bdev;
	blk_opf_t		bi_opf;		/* bottom bits REQ_OP, top bits
						 * req_flags.
						 */
	unsigned short		bi_flags;	/* BIO_* below */
	unsigned short		bi_ioprio;
	blk_status_t		bi_status;
	atomic_t		__bi_remaining;

	struct bvec_iter	bi_iter;

	blk_qc_t		bi_cookie;
	bio_end_io_t		*bi_end_io;
	void			*bi_private;
#ifdef CONFIG_BLK_CGROUP
	/*
	 * Represents the association of the css and request_queue for the bio.
	 * If a bio goes direct to device, it will not have a blkg as it will
	 * not have a request_queue associated with it.  The reference is put
	 * on release of the bio.
	 */
	struct blkcg_gq		*bi_blkg;
	struct bio_issue	bi_issue;
#ifdef CONFIG_BLK_CGROUP_IOCOST
	u64			bi_iocost_cost;
#endif
#endif

#ifdef CONFIG_BLK_INLINE_ENCRYPTION
	struct bio_crypt_ctx	*bi_crypt_context;
#endif

	union {
#if defined(CONFIG_BLK_DEV_INTEGRITY)
		struct bio_integrity_payload *bi_integrity; /* data integrity */
#endif
	};

	unsigned short		bi_vcnt;	/* how many bio_vec's */

	/*
	 * Everything starting with bi_max_vecs will be preserved by bio_reset()
	 */

	unsigned short		bi_max_vecs;	/* max bvl_vecs we can hold */

	atomic_t		__bi_cnt;	/* pin count */

	struct bio_vec		*bi_io_vec;	/* the actual vec list */

	struct bio_set		*bi_pool;

	/*
	 * We can inline a number of vecs at the end of the bio, to avoid
	 * double allocations for a small number of bio_vecs. This member
	 * MUST obviously be kept at the very end of the bio.
	 */
	struct bio_vec		bi_inline_vecs[];
};
~~~



###### 3. 块设备的 file_operations

~~~c
// block/fops.c
const struct file_operations def_blk_fops = {
	.open		= blkdev_open,
	.release	= blkdev_release,
	.llseek		= blkdev_llseek,
	.read_iter	= blkdev_read_iter,
	.write_iter	= blkdev_write_iter,
	.iopoll		= iocb_bio_iopoll,
	.mmap		= blkdev_mmap,
	.fsync		= blkdev_fsync,
	.unlocked_ioctl	= blkdev_ioctl,
#ifdef CONFIG_COMPAT
	.compat_ioctl	= compat_blkdev_ioctl,
#endif
	.splice_read	= filemap_splice_read,
	.splice_write	= iter_file_splice_write,
	.fallocate	= blkdev_fallocate,
};
~~~

#### 三、 映射层

##### 1. 结构体

###### 1) struct mapped_device

~~~c
// drivers/dm-core.h
struct mapped_device {
	struct mutex suspend_lock;

	struct mutex table_devices_lock;
	struct list_head table_devices;

	/*
	 * The current mapping (struct dm_table *).
	 * Use dm_get_live_table{_fast} or take suspend_lock for
	 * dereference.
	 */
	void __rcu *map;

	unsigned long flags;

	/* Protect queue and type against concurrent access. */
	struct mutex type_lock;
	enum dm_queue_mode type;

	int numa_node_id;
	struct request_queue *queue;

	atomic_t holders;
	atomic_t open_count;

	struct dm_target *immutable_target;
	struct target_type *immutable_target_type;

	char name[16];
	struct gendisk *disk;
	struct dax_device *dax_dev;

	wait_queue_head_t wait;
	unsigned long __percpu *pending_io;

	/* forced geometry settings */
	struct hd_geometry geometry;

	/*
	 * Processing queue (flush)
	 */
	struct workqueue_struct *wq;

	/*
	 * A list of ios that arrived while we were suspended.
	 */
	struct work_struct work;
	spinlock_t deferred_lock;
	struct bio_list deferred;

	/*
	 * requeue work context is needed for cloning one new bio
	 * to represent the dm_io to be requeued, since each
	 * dm_io may point to the original bio from FS.
	 */
	struct work_struct requeue_work;
	struct dm_io *requeue_list;

	void *interface_ptr;

	/*
	 * Event handling.
	 */
	wait_queue_head_t eventq;
	atomic_t event_nr;
	atomic_t uevent_seq;
	struct list_head uevent_list;
	spinlock_t uevent_lock; /* Protect access to uevent_list */

	/* for blk-mq request-based DM support */
	bool init_tio_pdu:1;
	struct blk_mq_tag_set *tag_set;

	struct dm_stats stats;

	/* the number of internal suspends */
	unsigned int internal_suspend_count;

	int swap_bios;
	struct semaphore swap_bios_semaphore;
	struct mutex swap_bios_lock;

	/*
	 * io objects are allocated from here.
	 */
	struct dm_md_mempools *mempools;

	/* kobject and completion */
	struct dm_kobject_holder kobj_holder;

	struct srcu_struct io_barrier;

#ifdef CONFIG_BLK_DEV_ZONED
	unsigned int nr_zones;
	unsigned int *zwp_offset;
#endif

#ifdef CONFIG_IMA
	struct dm_ima_measurements ima;
#endif
};
~~~



#### 调用流程

~~~
用户态 write(fd, buf, count)
  ↓ syscall
CPU → 内核态
  ↓ 系统调用表 syscall_64.tbl (__NR_write)
sys_write(fd, buf, count)
  ↓
ksys_write(fd, buf, count)
  ↓
fdget_pos(fd) → struct file
vfs_write(file, buf, count, &pos)
  ↓
file->f_op->write_iter(...)   // ext4 / blkdev / procfs

~~~

