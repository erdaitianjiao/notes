#### 测试结果汇总

##### 一、概述

**数据位置在10.20.12.187:/home/uos/sumrize_log/下**

数据包含

1. 不同机型上的 aes sm4全盘加密和未加密的磁盘测试结果，默认采用ext4文件系统，使用fio进行磁盘速率测试 
2. ntfsplus 的性能测试速率，包括普通的速率测试，还有对ntfsplus写小文件阻塞的问题的分析日志，使用fs_mark测试小文件速率，和trace-cmd工具进行分析，包括对比ext4，ntfs3，ntfs-3g的对比测试
3. 6.1和6.18的aes sm4全盘加密和未加密的磁盘测试结果，主要是测试large-folio对加密io的影响，包括ext4和xfs不同文件系统数据

##### 二、目录结构

```c
// 总体目录结构
├── crypt_test_log				// 加密性能测试结果
│   ├── D3000					// 各个机器的测试结果
│   ├── hg4
│   ├── huaweitest
│   ├── hygon_3350
│   ├── i3
│   └── zx7000
├── io_test.sum.et				// 加密测试execl表格
├── large-folio.et				// large-folio测试execl表格
├── ntfslog.xlsx				// ntfslog测试execl表格		
├── ntfsplus_test_log			// ntfsplus性能测试结果
│   ├── io_test					// io速率测试
│   ├── ntfsplus_cp_test		// cp的一些数据测试	  
│   └── ntfsplus_fsmark_test	// 使用fs_mark工具测试小文件数据报告
└── README.md					
    
// 加密性能测试目录(以i3为例)
i3
├── crypt_test_tool.sh			// 挂盘格式化一键脚本
├── disk						// 裸盘测试数据
├── fio_test.sh					
├── fs							// 文件系统测试数据
├── iotest.fio					// fio测试的配置文件
├── iotest_rand.fio				// fio测试的随机读写的测试文件
├── large-folio					// large-folio测试结果
│   ├── ext4					// 临时文件夹
│   │   ├── rand
│   │   └── seq
│   ├── linux-6.1		
│   │   ├── ext4
│   │   └── xfs
│   ├── linux-6.18				// large-folio测试数据
│   │   ├── ext4
│   │   └── xfs
│   ├── linux-6.19
│   │   ├── ext4
│   │   └── xfs
│   └── xfs						// 临时文件夹
│       ├── rand
│       └── seq
└── win_test

// ntfsplus_test_log详细目录
ntfsplus_test_log  
├── io_test
│   ├── ext4_fs_log
│   ├── ntfs3
│   ├── ntfs-3g
│   └── ntfsplus
└── ntfsplus_famark_test				// 使用fs_mark测试输出的结果，包含时间，iostat等数据，sys/block/xxx/stat等数据
    ├── fsmark_test.sh										// 所用的测试脚本
    ├── perf_test_ext4_nvme0n1_20251215_163212.log			// nvme为自带的固态盘
    ├── perf_test_ext4_nvme0n1_iostat_20251215_163212.log	
    ├── perf_test_ext4_sdag_20251215_162903.log				// sdag为u盘
    ├── perf_test_ext4_sdag_iostat_20251215_162903.log
    ├── perf_test_ntfs3_nvme0n1_20251215_163839.log
    ├── perf_test_ntfs3_nvme0n1_iostat_20251215_163839.log
    ├── perf_test_ntfs3_sdag_20251215_162440.log
    ├── perf_test_ntfs3_sdag_iostat_20251215_162440.log
    ├── perf_test_ntfsplus_nvme0n1_20251215_164025.log
    ├── perf_test_ntfsplus_nvme0n1_iostat_20251215_164025.log
    ├── perf_test_ntfsplus_sdag_20251215_162042.log
    └── perf_test_ntfsplus_sdag_iostat_20251215_162042.log

```



