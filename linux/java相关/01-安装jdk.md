# centos下使用yum源下载java



yum search java|grep jdk	#查看yum库中都有哪些jdk版本

yum install java-1.8.0-openjdk*	#悬着指定的版本安装，注意最后的*以及yum源安装的是openjdk,注意openjdk的区别

 java -version	#安装完成后查看版本信息

神奇是竟然不用配环境变量？？直接就可以用了？别说那么多写的HelloWorld编译一下。

# 

# 2019/07/20

本质上是要对操作系统中环境变量这个概念很熟悉才行。