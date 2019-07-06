# gzip/gunzip

​	gzip用于压缩文件，gunzip用于解压文件

## 	基本语法:	

​	 	gzip文件（功能描述：压缩文件，只能将文件压缩为*.gz文件）

​		gunzip 文件.gz  (功能描述：解压缩文件命令)

## 	应用实例：

​	案例1:	gzip压缩，将/home下的hello.txt文件进行压缩

​		gzip hello.txt #压缩之后源将会不存在，这就像把文件放进了压缩袋里面，而不是复制再压缩

​	案例2:	gunzip压缩，将/home下的hello.txt.gz 文件进行解压缩

​		gunzip hello.txt

# zip/unzip

​	zip用于压缩文件，unzip用于解压的。

​	**这个在项目打包发布中很有用**

## 基本语法：

​	zip 	[选项] XXX.zip 将要压缩的内容	(功能描述：压缩文件和目录的命令) 

unzip	[选项] XXX.zip  (功能描述：解压缩文件)

## 常用选项：

### zip常用

​	-r :	递归压缩，即压缩目录

### unzip常用

​	-d<目录> :  指定解压后文件的存放目录

## 应用实例：

​	案例1：将/home下的所有文件压缩成 mypackage.zip

​	案例2： 将mypackge.zip 解压到 /opt/tmp 目录下

### 	问题：

​		这个指令也是需要安装的 去网上找下载方法吧

​	iunx服务器上默认没有安装zip命令，所以使用时需安装：apt-get install zip 或  **yum install zip**

​	linux安装unzip命令：apt-get install unzip 或  **yum install unzip**

# tar	最好用的应该是这个

## 	含义：

​	tar指令是打包指令，最后打包后的文件是 .tar.gz的文件

## 	基本语法：

​	tar [选项] XXX.tar.gz 打包的内容 (功能描述：打包目录，压缩后的文件格式 tar.gz)

## 	选项说明：

​	选项	功能

​	-c		产生.tar打包文件

​	-v		显示详细信息

​	-f		指定压缩后的文件名

​	-z		打包同时压缩	#所以说直接用这个指令仅仅只是打包，还没有到压缩？

​	-x		解压.tar文件

所以之前智意的那招 tar -zvxf  包名 好像基本把上面的指令都用了。。

## 组合拳：

​	打包压缩:	-**zcvf**

​	解压：		-**zxvf**



## 	应用实例

​	案例1：压缩多个文件，将/home/a1.txt 和 /home/a2.txt压缩成 a.tar.gz

​		tar -zcvf  a.tar.gz  a1.txt  a2.txt

​	案例2：将/home的文件夹压缩成 myhome.tar.gz

​		tar  -zcvf   myhome.tar.gz   /home/

​	案例3：将a.tar.gz解压到当前目录

​		tar -zxvf   a.tar.gz

​	案例4：将myhome.tar.gz 解压到 /opt/tmp2目录下

​	tar -zxvf 	myhome.tar.gz  -C /opt/tmp2/		#指定的目录必须存在，这个命令不会为你创建目录；注意 -C 这个参数是大写C