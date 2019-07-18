# find

## 含义

​	find指令从指定目录向下递归地遍历其各个子目录，将满足条件的文件或者目录显示在终端

## 基本语法

​	find [搜索范围] [选项]

选项说明

​	-name<查询方式>	 按照指定的文件名查找模式查找文件

​	-user<用户名> 		 查找属于指定用户名所有文件

​	-size<文件大小>		按照指定的文件大小查找文件

## 应用实例

 1. 按文件名，根据名称查找/home目录下的hello.txt文件

    ​	find /home -name hello.txt

 2. 按拥有者，查找/opt目录下,用户名称为nobody的文件

    ​	find /home -user root

 3. 查找整个linux系统下大于20m的文件(+n 大于 -n 小于 n 等于)

    ​	find  / -size +20M
    
    ​	find  / -size -20M
    
    ​	find  / -size 20M
    
    单位的话大写不行用小写 例 如M k
    
 4. 查找整个linux系统下文件后缀名为 .txt的文件

 5. 

    ​	find / -name *.txt

# locate

## 含义

​	locate指令可快速定位文件路径。

locate指令利用事先建立的系统中所有文件名称及路径的locate数据库实现快速定位给定文件。

locate指令无需遍历整个文件系统，查询速度较快。

为了保证查询结果的准确度，管理员必须定期更新locate时刻。

## 基本语法

​	locate 搜索文件

## 特别说明

​	由于locate指令基于数据库进行查询，所以第一次运行前，必须使用updatedb指令创建locate数据库

应用实例：

​	请使用locate指令快速定位 hello.txt 文件所在目录

updatedb

locate hello.txt	#这个指令真的好用啊

## 	遇到的问题

​		updatedb 找不到

​		你的系统是最小化安装，输入locate提示可能会提示`-bash: locate: command not found`可以运行

​		`yum -y install mlocate`安装locate，安装后在运行locate，又提示`locate: unexpected EOF reading `/var/lib/mlocate/mlocate.db'

​	`输入`updatedb`更新一下数据库

​	

# grep 和管道符号 |

## 	含义

​		网上查grep得到的意思是**] UNIX工具程序；可做文件内的字符串查找**

​		grep 过滤查找

​		"|" 管道符 ，表示将前一个命令的处理结果输出传递给后面的命令处理。

## 	基本语法

​		grep [选项] 查找内容 源文件

### 		常用选项

​		-n  显示匹配行及行号

​		-i	忽略字母大小写

## 应用实例：

​	案例1：请在hello.txt 文件中，查找"yes"所在行，并且显示行号

​	grep -n yes hello.txt

​	如果是配合管道玩的话，那就是不需要写源文件了，内容由管道前面的输入提供

​	cat hello.txt | grep -n yes