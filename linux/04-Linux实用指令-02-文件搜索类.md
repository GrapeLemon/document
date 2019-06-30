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
    
    单位的话大写不行用小写 例如M k
    
 4. 查找整个linux系统下文件后缀名为 .txt的文件

    ​	find / -name *.txt

# locate

# grep 和管道符号 |