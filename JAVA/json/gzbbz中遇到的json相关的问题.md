# java对象的序列化与反序列化

​		fastjson的主要工具类是JSON，以下代码实现Java对象的序列化与反序列化

​		

1. // 将Java对象序列化为Json字符串  
2. ​    String objectToJson = JSON.toJSONString(initUser());  
3. ​    System.out.println(objectToJson);  
4. ​    // 将Json字符串反序列化为Java对象  
5. ​    User user = JSON.parseObject(objectToJson, User.class);  
6. ​    System.out.println(user); 

# 两个核心概念

## 序列化

​	把对象变成字节流（字节流中的内容是对象的成员变量的值）

## 反序列化	

​	从字节流中恢复对象的状态

​	