# Linux 基础指令 
@[TOC](文章目录）


### Linux命令格式

```bash
command [-options] [parameter]
```

- command : 命令本身

- -options :[可选，非必填写]命令的一些选项，可以通过选项控制命令的行为细节
- parameter:[可选，非必填] 命令的参数，多用于命令的指向目标等

eg：ls -l  /home ,ls 是命令本身，-l是选项，/home是参数



### 一、目录及文件指令

##### 1.  cd命令



- cd : change directory 用于转换工作目录到是定文件夹

  常用选项有：

 ~~~ bash
 cd ..    返回上一级
 cd ../.. 返回上两级目录
 cd ~     进入home目录
 cd -     回到上次工作目录
 ~~~

##### 2. pwd命令

- pwd: print work directory 显示当前工作目录的完整路径

 ~~~ bash
 pwd
 ~~~

  

##### 3. ls命令
- ls : list 用于显示指定目录下的文件夹和目录列表(若未指定，则直接显示工作目录下列表
~~~ bash
ls        查看当前目录的文件
ls -l     查看文件和目录的详细资料
ls -a     查看文件内的所有文件，包括隐藏文件
ls -R	  连同子目录的内容一起列出(递归列出文件)
~~~



##### 4. mkdir命令

- mkdir: make directory 用创建一个新的目录
~~~ bash
mkdir test 				创建一个名字叫test的目录
mkdir test1 test2   	同时创建两个目录
mkdir -p /tmp/test3     -p表示创建目录树，如果中间没有某一个文件夹会自动创建
~~~



##### 5. rmdir 命令

- rmdir: remove directory 移除一个空目录(只能删除空目录，不然会报错)

~~~ bash
rmdir test		删除一个名字叫test的目录
~~~



##### 6. rm 命令

- rm: remove  删除一个文件

~~~ bash
rm test.txt		删除一个名字叫test的文本文件
参数：
-r              删除目录
-f              强制执行
~~~



##### 7. cp命令

- cp: copy 复制文件或目录到指定位置

~~~ bash
cp -rp test/test1.txt test/test2		将目录下的test1.txt复制到test2文件夹中
参数：
-r		复制目录
-p		保留文件属性
~~~



##### 8. mv命令

- mv: move 移动文件或目录到指定位置

~~~ bash
mv  test/test1.txt test/test2		将目录下的test1.txt移动到test2文件夹中
参数：
-f		若目标已经存在，不会询问
-i		若目标文件已经存在，则询问是否覆盖
-u		若目标文件已经存在。且比目标文件新，才会更新
~~~



### 二、文件指令



##### 1. touch命令

- touch: touch 移动文件或目录到指定位置

~~~ bash
touch  test1.txt 		在当前目录路下创建test1.txt
~~~



##### 2. cat命令

- cat: concatenate 显示文件内容
~~~ bash
cat  test1.txt 		查看test1.txt
~~~



##### 3. more命令

- more:  显示文件内容
~~~ bash
more test1.txt 		查看test1.txt
操作：
(空格) 或 f 	向后翻页
b          	  向前翻页
(Enter)       换行
q或Q          退出
~~~



##### 4. vim命令

- vim:  用vim编辑指定文件
~~~ bash
vim test1.txt 		编辑test1.txt

操作：
三种模式 命令模式、编辑模式、末行模式

模式间切换方法：
（1）命令模式下，输入:后，进入末行模式
（2）末行模式下，按esc慢退、按两次esc快退、或者删除所有命令，可以回到命令模式
（3）命令模式下，按下i、a等键，可以计入编辑模式
（4）编辑模式下，按下esc，可以回到命令模式
~~~



### 三、文件搜索指令



##### 1. find命令

- find: 在指定范围内找到符合条件的文件或目录


~~~bash
find [搜索范围] [匹配条件]
find /home -name test		在home目录里查找文件test
参数:
-name		以名称
-size		以文件大小
-a   		两个条件同时满足
-o   		两个条件满足一个几以上
~~~



### 四、帮助指令



##### 1.man命令

- man: 查看帮助文档

~~~bash
man [指令或者配置文件]
操作:
(空格) 或f           翻页               
 (Enter)            换行                 
 q或Q               退出
~~~



##### 2.help命令

- help: 获取帮助信息

~~~bash
help [指令]
~~~



###  五、管理命令



##### 1.su命令

- su: 切换用户

~~~bash
su        		 切换到root用户
su root	    	 切换到root用户
su -			 切换到root用户，同时目录切换到/root
su -root  		 切换到root用户，同时目录切换到/root
su 普通用户	  	   切换到普通用户
su - 普通用户	   切换到普通用户，同时目录切换到普通用户所在目录
~~~

###  六、压缩命令



##### 1.tar命令

- tar: 归档管理
~~~bash
tar 选项[-zcf] [压缩后文件名] [目录]
参数:
-c     打包                 
-v     显示详细信息                 
-f     指定文件名            
-z     打包同时压缩 

tar [选项] [压缩包名称]
-x     解包           
-v    显示详细信息           
-f     指定解压文件           
-z     解压缩
~~~



//待更新中



参考文献：

[Linux 命令大全（看这一篇就足够）](
https://blog.csdn.net/weixin_43424363/article/details/124237304?ops_request_misc=%7B%22request%5Fid%22%3A%22171505711416800182727583%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=171505711416800182727583&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-4-124237304-null-null.142^v100^pc_search_result_base8&utm_term=linux常用命令大全&spm=1018.2226.3001.4187)
