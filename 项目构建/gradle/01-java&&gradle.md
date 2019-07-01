# plugins{}

​	gradle不单单可以用来构建java项目，也可以用来构建其他项目，那么它怎么知道你要构建什么呢？诀窍就是设置上面这个值

```groovy
plugins {
    id 'java'
}
apply plugin: 'java'	//这样写也行 是等价的 在gzbbz中就是这样写的
apply plugin: 'war'
apply plugin: 'spring-boot'
apply plugin: 'idea'
apply plugin: 'application'
apply from: 'build_dist.gradle'
```

# Gradle常用任务

## build

​	构建

## clean

​	删除build目录以及所有构建完成的文件

## assemble

​	编译并打包jar文件，但不会执行单元测试

## check

​	编译并测试代码。

