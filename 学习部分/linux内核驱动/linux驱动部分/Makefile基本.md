## Makefile基本语法

#### 一、语法

- **all**: 编译出来的可执行文件
- **target** 1 2 3: all 编译中需要的依赖文件

~~~makefile
all: target1 target2 target3
target1:
	command 1 # 编译规则 1
target2:
	command 2 # 编译规则 2
target3
	command 3 # 编译规则 3
~~~

