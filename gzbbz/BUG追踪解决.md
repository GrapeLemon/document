# BUG追踪解决

我佛了，BUG产生的原因是 String 和 JSONString之间是有差别的。。。而且JSONArray的String也是不同的，这个就真的难受了

​	

# 2019/07/12

​	

```java
//抛出网点费用明细汇总事件
eventBus.asyncPost(new PointFeePersistenceEvent(event.getAccountPeriod()));
//抛出分机费用汇总事件
eventBus.asyncPost(new ExtensionStatisticsEvent(event.getAccountPeriod()));

```

现在还差着两个事件要追踪下去

```java
ExtensionStatisticsHandler	//分机汇总统计执行类 
    这个出bug了，原因是报账全局配置中没有配好延迟统计时间
PointFeeBusinessService		//网点费用汇总
```



# 事件抛出链条分析

SMSRepository ->  BSSRepository  ->  EIMRepository  ->PointExtensionBusinessService -> 

(  PointFeeBusinessService -> PointFeeStatisticsHandler  &&  ExtensionStatisticsHandler )

现在还差这步

```
uniteReportService.getPointFeeDetailService().save(pointFeeDetails);
//在这个类里面
PointFeeBusinessService
```

```java
//这个方法有bug
// sumLowOutbandMinCount
//这行代码的取的值为空了 很显然，在前面并没有统计执行低消的数量
// lowConsumpCount += value.getLowconsump_count();

```



# 2019/07/15

```java
ExtensionExportTask
	init
		JSONArray.parseArray
//现在假定是输入有误，解决方案是把修改输入

这个类型就是源头了 处理一下
PointExtensionDetailService
	export
```



# 2019/07/16

​	主要是计算的验证问题，然而今天狗了一天，好像什么都没干到哈哈