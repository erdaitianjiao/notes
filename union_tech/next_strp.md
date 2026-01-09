

* 更多测试维度：
  * 小文件*10000 
  * 空文件*10000 
  * 小文件 100 - 100
* 对比测试ext4
* 用内置nvme测试
* 抓数据：
  * 整个过程的diskstat-diff：写一个py,开始跑的时候抓，Ctrl-C停止抓第二次，输出diff
  * 同期抓iostat存档

````
ntfsplus sdag1

root@uos:~/tmp# cat /sys/block/sdag/stat && time bash -c "cp randomfile/* /mnt && sync" && cat /sys/block/sdag/stat
  117987       33  1110107    74629  2061268   749561 17704911  7020369        0  7065896  7094999        0        0        0        0        0        0

real    2m6.967s
user    0m0.081s
sys     0m0.394s
  117987       33  1110107    74629  2093305   749561 17931445  7145064        0  7190623  7219693        0        0        0        0        0        0
  

````

```
ntfs3 agag1
root@uos:~/tmp# cat /sys/block/sdag/stat && time bash -c "cp randomfile/* /mnt && sync" && cat /sys/block/sdag/stat
  119484       33  1123862    75293  2105496   862233 18163577  7180868        0  7227141  7256162        0        0        0        0        0        0
                                                
real    0m34.298s
user    0m0.070s
sys     0m0.352s
  122072       33  1144566    76008  2107793   876112 18372993  7213898        0  7260914  7289907        0        0        0        0        0        0

```

