# python笔记

 #### 1.保留字

> 关键词，在python不能作为其他变量名

  ~~~python
   import keyword
   keyword.kwlist
  ['False', 'None', 'True', 'and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
  
  ~~~

 #### 2. 注释

   ~~~python
   #本行为注释     单行注释
   """
   一行注释
   二行注释
   三行注释
   ......
   多行注释        双引号多行注释
   """
   '''
   多行注释        单引号多行注释
   '''
   ~~~

#### 3. 同一行显示多条语句

>可以用 **; **分开(多用于命令行模式)


####  4. print输出
>print可以输出变量的内容
>输出默认换行，要实现不换行输出，则使用print("sth" end = "") 表示结尾没东西

#### 5. 引入其他模块
>在 python 用 import 或者 from...import 来导入相应的模块
>将整个模块(somemodule)导入，格式为： import somemodule
>从某个模块中导入某个函数,格式为： from somemodule import somefunction
>从某个模块中导入多个函数,格式为：<br> from somemodule import firstfunc, secondfunc, thirdfunc
>将某个模块中的全部函数导入，格式为： from somemodule import *

#### 6. 命令行参数
>很多程序可以执行一些操作来查看一些基本信息，Python可以使用-h参数查看各参数帮助信息：
```bash
$ python -h
usage: python [option] ... [-c cmd | -m mod | file | -] [arg] ...
Options and arguments (and corresponding environment variables):
-c cmd : program passed in as string (terminates option list)
-d     : debug output from parser (also PYTHONDEBUG=x)
-E     : ignore environment variables (such as PYTHONPATH)
-h     : print this help message and exit
[ etc. ]
```

### 二、数据类型

  #### 1. 数字数据类型


1. int **整数**  如1，2
2. bool **布尔数** 有True 和 False
3. float **浮点数** 如 1.23 3E-2
4. complex **复数** 如1 + 2j


   ~~~python
   a = 1
   b = True
   c = 1.2
   d = 1 + 2j
   ~~~

  #### 2. 字符串类型

> 单引号 '' 和双引号 " 在使用完全相同
>使用三引号("""or''')可以定义一个多行字符串
>反斜杠可以用来转义，使用 **r** 可以让反斜杠不发生转义<br>如 **r"this is a line with \n"** 则 **\n** 会显示，并不是换行
>    按字面意义级联字符串，如 **"this " "is " "string"** 会被自动转换为 **this is string**
>字符串可以用 **+** 运算符连接在一起，用 ***** 运算符重复
>Python 中的字符串有两种索引方式，从左往右以 **0** 开始，从右往左以 **-1** 开始
>Python 中的字符串不能改变
>Python 没有单独的字符类型，一个字符就是长度为 1 的字符串
>字符串切片 **str[start:end]**

> [!NOTE]
>
> 其中 start（包含）是切片开始的索引，end（不包含）是切片结束的索引
>  字符串的切片可以加上步长参数 step，语法格式如下：**str[start\:end\:step]**

 ```python
a = "it's my python"
b = 'it\'s my python'
c = """
this is a 
more than one raw
sentence
"""

str='123456789'
 
print(str)                 # 输出字符串
print(str[0:-1])           # 输出第一个到倒数第二个的所有字符
print(str[0])              # 输出字符串第一个字符
print(str[2:5])            # 输出从第三个开始到第六个的字符（不包含）
print(str[2:])             # 输出从第三个开始后的所有字符
print(str[1:5:2])          # 输出从第二个开始到第五个且每隔一个的字符（步长为2）
print(str * 2)             # 输出字符串两次
print(str + '你好')         # 连接字符串
 
print('hello\nrunoob')      # 使用反斜杠(\)+n转义特殊字符
print(r'hello\nrunoob')     # 在字符串前面添加一个 r，表示原始字符串，不会发生转义
 ```

