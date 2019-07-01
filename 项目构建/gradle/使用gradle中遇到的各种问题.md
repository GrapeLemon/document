# 1- xxx.jar中没有主清单属性

​	原因：没有在manifest文件中配置Jar文件的主类

​	原理：Java插件在我们的项目中加入了一个Jar任务，没一个Jar对象(jar)都有一个manifest属性，这个属性是Mainfest的一个实例。可以对生成的Jar文件的主类进行配置。使用Manifest接口的attributes()方法。换句话说，我们可以使用一个包含键值对的map结构指定加入到manifest文件的属性集。

​	我们能够通过设置Main-Class属性的值，指定我们程序的入口点。只需要在build.gradle文件进行必要的改动即可：

apply plugin: 'java'

jar {
​    manifest {
​        attributes 'Main-Class': 'com.zero.HelloWorld'
​    }
}



# 2-  怎样指定jar包的名称

```groovy
jar {
    baseName = 'first-gradle'
    version =  '0.1.0'
}
```