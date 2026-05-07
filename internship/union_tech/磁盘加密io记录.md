### 磁盘加密io

#### 测试环境



#### 一、加密过程

```bash
# 对磁盘加密过程 使用cryptsetup

# aes加密
sudo cryptsetup luksFormat --cipher aes-xts-plain64 --key-size 512 /dev/nvme0n1p5

# sm4加密 
sudo cryptsetup luksFormat --cipher sm4-xts-plain64 --key-size 256 /dev/nvme0n1p6

# 打开磁盘
sudo cryptsetup open /dev/nvme0n1p6 aes_disk
sudo cryptsetup open /dev/nvme0n1p7 sm4_disk

# 关闭磁盘
sudo cryptsetup close aes_disk


# 创建一个dm-liner设备
sudo blockdev --getsz /dev/nvme0n1p6
echo "0 161226752 linear /dev/nvme0n1p6 0" | sudo dmsetup create linear_disk

sudo dmsetup remo linear_disk 
```

#### 二、更改内核后fio没有io_uring

使用编译安装新版fio

```bash
wget https://github.com/axboe/fio/archive/refs/tags/fio-3.36.tar.gz
tar -xzf fio-3.36.tar.gz
cd fio-fio-3.36

./configure
make
sudo make install

/usr/local/bin/fio --version
/usr/local/bin/fio --enghelp | grep uring
```

#### 三、BLK追踪指令

```
sudo blktrace -d /dev/nvme0n1 -d /dev/nvme0n1 -d /dev/dm-0 -d /dev/dm-1
sudo blkparse -i dm-1 > blktrace-aes-disk-async-dw-dm-1

```



~~~sh
#!/bin/bash

# write/read size
FILE_SIZE=4096

# aes or nor or sm4 or lin
TEST_NAME=aes

SLEEP_TIME=5

# fs or pure
TEST_TYPE=fs

IO_ENGINE="io_uring"

# DEVICE=/dev/nvme0n1p7
# DEVICE=/dev/nvme0n1p6
DEVICE=/dev/nvme0n1p5

# MAP_DEVICE=/dev/mapper/linear_disk
# cache
CACHE=""

# TMP_FILE=/home/test/workspace/NOR_test/tmpfile
# TMP_FILE=""

# 被追踪的fio输出
TRACED_LOG_W=traced_logs/${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_w${CACHE}_${IO_ENGINE}.log
TRACED_LOG_R=traced_logs/${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_r${CACHE}_${IO_ENGINE}.log

# blktrace原始输出文件
BLOCK_TRACE_W=block_trace_log/block_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_w${CACHE}_${IO_ENGINE}.log
BLOCK_TRACE_R=block_trace_log/block_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_r${CACHE}_${IO_ENGINE}.log

# blktrace处理文件
BLOCK_TRACE_W_LOG=block_trace_time_log/block_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_w${CACHE}_${IO_ENGINE}.txt
BLOCK_TRACE_R_LOG=block_trace_time_log/block_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_r${CACHE}_${IO_ENGINE}.txt

# mapper blktrace输出文件
# MAPPER_BLOCK_TRACE_W=map_trace_log/block_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_w${CACHE}_${IO_ENGINE}.log
# MAPPER_BLOCK_TRACE_R=map_block_trace_log/block_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_r${CACHE}_${IO_ENGINE}.log

# perf原始输出文件
PERF_TRACE_W=perf_log/perf_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_w${CACHE}_${IO_ENGINE}.data
PERF_TRACE_R=perf_log/perf_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_r${CACHE}_${IO_ENGINE}.data

# perf处理后文件
PERF_TRACE_STD_W=perf_log/perf_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_w${CACHE}_${IO_ENGINE}.txt
PERF_TRACE_STD_R=perf_log/perf_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_r${CACHE}_${IO_ENGINE}.txt

# iostat输出文件
IOSTAT_LOG_W=iostat_log/iostat_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_w${CACHE}_${IO_ENGINE}.data
IOSTAT_LOG_R=iostat_log/iostat_${TEST_NAME}_${TEST_TYPE}_disk_${FILE_SIZE}m_r${CACHE}_${IO_ENGINE}.data

FIO_SCR_W=${TEST_NAME}_${TEST_TYPE}_disk_w.fio
FIO_SCR_R=${TEST_NAME}_${TEST_TYPE}_disk_r.fio

echo " --- Fio ${TEST_NAME} Write Start --- "

for ((i=1; i<=1; i++)) 
do  
    # 清除缓存
	sync
	echo 3 > /proc/sys/vm/drop_caches
	sleep $SLEEP_TIME

	echo "test time $i"
	echo "test time $i" >> $TRACED_LOG_W

    # 记录iostat
    while true; do
        echo "=== $(date) ===" >> $IOSTAT_LOG_W
        iostat -dx 1 1 >> $IOSTAT_LOG_W
        sleep 0.1
    done &
    IOSTAT_PID=$!

    # 追踪blktrace
    blktrace -d $DEVICE -o $BLOCK_TRACE_W &
    BLK_PID=$!

    # blktrace -d $MAP_DEVICE -o $MAPPER_BLOCK_TRACE_W -w 5 &
    # BLK_PID=$!

    # 追踪perf
    perf_4.19 record -o $PERF_TRACE_W -F 99 &
    PERF_PID=$!

    sleep 1

    # 执行fio
	sudo fio $FIO_SCR_W >> $TRACED_LOG_W 2>&1 &
	
	FIO_PID=$!

	wait $FIO_PID
	
    kill $IOSTAT_PID
    kill -INT $PERF_PID
    kill -INT $BLK_PID

    sleep 5

    perf_4.19 report -i $PERF_TRACE_W --stdio > $PERF_TRACE_STD_W

    # 生成带绝对时间戳的合并报告
    blkparse -i $BLOCK_TRACE_W -f "%T.%9t %5p %C %a %d %S %N\n" | \
    awk -v current_time="$(date '+%Y-%m-%d %H:%M:%S')" '{
        print current_time, $0;
    }' > $BLOCK_TRACE_W_LOG

	sudo rm $TMP_FILE

done

echo " --- Fio ${TEST_NAME} Read Start --- "

for ((i=1; i<=1; i++))
do
	
	sync
	echo 3 > /proc/sys/vm/drop_caches
	sleep $SLEEP_TIME
	
	echo "test time $i"
	echo "test time $i" >> $TRACED_LOG_R

    while true; do
        echo "=== $(date) ===" >> $IOSTAT_LOG_R
        iostat -dx 1 1 >> $IOSTAT_LOG_R
        sleep 0.1
    done &
    IOSTAT_PID=$!

    blktrace -d $DEVICE -o $BLOCK_TRACE_R &
    BLK_PID=$!

    # blktrace -d $MAP_DEVICE -o $MAPPER_BLOCK_TRACE_R -w 5 &
    # BLK_PID=$!

    perf_4.19 record -o $PERF_TRACE_R -F 99 &
    PERF_PID=$!

	sudo fio $FIO_SCR_R >> $TRACED_LOG_R 2>&1 &

	FIO_PID=$!

    wait $FIO_PID
    kill $IOSTAT_PID
    kill -INT $PERF_PID
    kill -INT $BLK_PID

    sleep 5

    perf_4.19 report -i $PERF_TRACE_R --stdio > $PERF_TRACE_STD_R


    # 生成带绝对时间戳的合并报告
    blkparse -i $BLOCK_TRACE_R -f "%T.%9t %5p %C %a %d %S %N\n" | \
    awk -v current_time="$(date '+%Y-%m-%d %H:%M:%S')" '{
        print current_time, $0;
    }' > $BLOCK_TRACE_R_LOG

    sudo rm $TMP_FILE

done



~~~

