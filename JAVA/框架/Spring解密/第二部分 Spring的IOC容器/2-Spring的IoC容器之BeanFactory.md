# 前言

​	我们前面说过，Spring的IoC容器是一个IoC Service Provider，但是，这只是它被冠以IoC之名的部分原因，我们不能忽略的是“容器”。Spring的IoC容器是一个提供IoC支持的轻量级容器，除了基本的IoC支持，它作为轻量级容器来提供了IoC之外的支持。如再Spring的IoC容器之上，Spring还提供了相应的AOP框架支持、企业级服务集成等服务。Spring的IoC容器和IoC Service Provider所提供的服务之间存在一定的交集。

![1564812400818](D:\我的文档\JAVA\框架\Spring解密\images\Ioc容器与ISP之间的关系)

这里先主要关注**Spring的IoC容器提供的IoC相关支持以及衍生的部分高级特性**，而IoC容器提供的其他服务将在后续章节中陆续阐述。

# Spring提供的两种容器类型

## BeanFactory

​	基础类型IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用 **延迟初始化策略(lazy-load)**。只有当客户端对象需要访问容器中的某个受管对象的时候，才对该对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景，BeanFactory是比较合适的类型.

## ApplicationContext

​	ApplicationContext在BeanFactory的基础上构建，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持，、ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等，这些会在后面详述。ApplicationContext所管理的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，相对于BeanFactory来说，ApplicationContext要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器启动事件较之BeanFactory也会长一些。在那些资源充足，并且要求更多功能的场景中，ApplicationContext类型的容器是比较合适的选择。



![1564813789974](D:\我的文档\JAVA\框架\Spring解密\images\BeanFactory和ApplicationConext继承关系)

@@注意@@

​	ApplicationContext间接继承自BeanFactory，所以说它是构建于BeanFactory之上的IoC容器。此外，你应该注意到了，ApplicationContext还继承了其他三个接口，它们之间的关系，在后面说明。

# 解释

​	BeanFactory，顾名思义，也就是生产Bean的工厂，当然，严格来说，这个“生产过程”可能不像说起来那么简单。既然Spring框架提倡使用POJO，那么把每个业务对象看作一个JavaBean对象，或许更容易理解为什么Spring的IoC容器会起这样一个名字。作为spring提供的基本IoC容器，**BeanFactory可以完成作为 IoC Service Provider**的所有职责，包括业务对象的注册和对象间依赖关系的绑定。

BeanFactory就像一个汽车生产厂。你从其他汽车零件厂商或者自己的零件生产部门取得汽车零件送入这个汽车生产厂，最后，只需要从生产线的终点取得成品汽车就可以了。

相似的，将应用所需的所有业务对象交给BeanFactory之后，剩下要做的，就是直接从BeanFactory取得最终组装完成并且可用的对象。至于这个最终业务对象如何组装，你不需要关心，BeanFactory会帮你搞定。

所以，对于客户端来说，与BeanFactory打交道其实很简单。最基本地，BeanFactory肯定会公开一个取得组装完成的对象的方法接口，就像下面代码中真正的BeanFactory的定义所展示的那样。

```java
代码清单4-1 BeanFactory的定义 16
public interface BeanFactory {
String FACTORY_BEAN_PREFIX = "&";
Object getBean(String name) throws BeansException; 1722 Spring 的 IoC 容器
Object getBean(String name, Class requiredType) throws BeansException;
/**
* @since 2.5
*/
Object getBean(String name, Object[] args) throws BeansException;
boolean containsBean(String name);
boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
/**
* @since 2.0.3
*/
boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
/**
* @since 2.0.1
*/
boolean isTypeMatch(String name, Class targetType) throws NoSuchBeanDefinitionException;
Class getType(String name) throws NoSuchBeanDefinitionException;
String[] getAliases(String name);
}
```

上面的代码中的方法基本都是查询相关的方法，例如，取得某个对象的方法（getBean），查询某个对象是否存在于容器中的方法（containBean），或者取得某个bean的状态或者类型的方法等。因为通常情况下，对于独立的应用程序，只有主入口类才会跟容器的API直接耦合。

# 拥有BeanFactory之后的生活

![1564816409149](D:\我的文档\JAVA\框架\Spring解密\images\BeanFactory用的配置文件)

```java
拉响启航的汽笛。在BeanFactory出现之前，我们通常会直接在应用程序的入口类的main方法中，
自己实例化相应的对象并调用之，如以下代码所示： 11
FXNewsProvider newsProvider = new FXNewsProvider();
newsProvider.getAndPersistNews(); 12
不过，现在既然有了BeanFactory，我们通常只需将“生产线图纸”交给BeanFactory，让
BeanFactory为我们生产一个FXNewsProvider，如以下代码所示： 13
14
BeanFactory container = ➥
new XmlBeanFactory(new ClassPathResource("配置文件路径"));
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider");
newsProvider.getAndPersistNews();
或者如以下代码所示：
15
ApplicationContext container = ➥
new ClassPathXmlApplicationContext("配置文件路径");
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider");
newsProvider.getAndPersistNews(); 16
亦或如以下代码所示：
ApplicationContext container = ➥ 1724 Spring 的 IoC 容器
new FileSystemXmlApplicationContext("配置文件路径");
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider");
newsProvider.getAndPersistNews();
这就是拥有BeanFactory后的生活。当然，这只是使用BeanFactory后开发流程的一个概览而已，
具体细节请容我慢慢道来。
```



# BeanFactory的对象注册与依赖绑定方式

## 	直接编码方式

​	其实，把编码方式单独提出来称作一种方式并不十分恰当。因为不管什么方式，最终都需要编码才能	“落实” 所有信息并付诸使用。不过，通过这些代码，起码可以让我们更加清楚BeanFactory在底层是如何运作的。

BeanFactory只是一个接口。我们最终需要一个该接口的实现来进行实际的Bean管理，DefaultListableBeanFactory 就是这么一个比较通用的BeanFactory实现类。DefaultListableBeanFactory 除了间接地实现了BeanFactory接口，还实现了BeanDefinitionRegistry 接口，该接口才是在BeanFactory的实现中 **担当Bean注册管理的角色**。 基本上，BeanFactory接口只定义如何访问容器内管理的Bean的方法，各个BeanFactory的具体实现类负责具体Bean的注册以及管理工作。BeanDefinitionRegistry 接口定义抽象了Bean的注册逻辑。通常情况下，具体的BeanFactory实现类会实现这个接口来管理Bean的注册，下面是它们的关系图。

![1564817940298](D:\我的文档\JAVA\框架\Spring解密\images\BeanFactory类图)

打个比方，BeanDefinitionRegistry 就像图书馆的书架，所有的书是放在书架上的。虽然你还书或者借书都是跟图书馆（BeanFactory）打交道，但书架才是图书馆存放各类图书的地方。所以，书架相对于图书馆来说，就是它的BeanDefinitionRegistry 。

**每一个受管的对象，在容器中都会有一个BeanDefinition的实例(instance)相对应，该BeanDefinition实例负责保存对象的所有必要信息，包括其对应的对象的class类型、是否是抽象类、构造方法参数以及其他属性等，当客户端向BeanFactory请求相应对象的时候，BeanFactory会通过这些信息为客户端返回一个完备可用的对象实例**RootBeanDefinition和 ChildBeanDefinition是BeanDefinition的两个主要实现类。 

```java
代码清单4-4 通过编码方式使用BeanFactory实现FX新闻相关类的注册及绑定
public static void main(String[] args)
{
DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
BeanFactory container = (BeanFactory)bindViaCode(beanRegistry);
FXNewsProvider newsProvider = ➥
(FXNewsProvider)container.getBean("djNewsProvider");
newsProvider.getAndPersistNews();
}
public static BeanFactory bindViaCode(BeanDefinitionRegistry registry)
{
AbstractBeanDefinition newsProvider = ➥
new RootBeanDefinition(FXNewsProvider.class,true);
AbstractBeanDefinition newsListener = ➥
new RootBeanDefinition(DowJonesNewsListener.class,true);
AbstractBeanDefinition newsPersister = ➥
new RootBeanDefinition(DowJonesNewsPersister.class,true);
// 将bean定义注册到容器中
registry.registerBeanDefinition("djNewsProvider", newsProvider);
registry.registerBeanDefinition("djListener", newsListener);
registry.registerBeanDefinition("djPersister", newsPersister);
// 指定依赖关系
// 1. 可以通过构造方法注入方式
ConstructorArgumentValues argValues = new ConstructorArgumentValues();
argValues.addIndexedArgumentValue(0, newsListener);
argValues.addIndexedArgumentValue(1, newsPersister);
newsProvider.setConstructorArgumentValues(argValues);
// 2. 或者通过setter方法注入方式
① 在Spring的术语中，把BeanFactory需要使用的对象注册和依赖绑定信息称为Configuration Metadata。我们这里所
展示的，实际上就是组织这些Configuration Metadata的各种方式。因此这个标题才这么长。4.2 TBeanFactoryT 的对象注册与依赖绑定方式 F F 25
MutablePropertyValues propertyValues = new MutablePropertyValues();
propertyValues.addPropertyValue(new ropertyValue("newsListener",newsListener));
propertyValues.addPropertyValue(new PropertyValue("newPersistener",newsPersister));
newsProvider.setPropertyValues(propertyValues);
// 绑定完成 2
return (BeanFactory)registry;
```

关于代码的解释很精彩 信手拈来。

## 外部配置文件方式

意也可以引入自己的文件格式，前提是真的需要。 Spring的IoC容器支持两种配置文件格式：Properties文件格式和XML文件格式。当然，也可以引入自己的文件格式，前提是真的需要。

采用外部配置文件时，Spring的IoC容易有一个统一的处理方式。通常情况下，需要根据不同的外部配置文件格式，给出相应的BeanDefinitionReader的实现类，由BeanDefinitionReader的相应实现类负责将相应的配置文件内容读取并映射到BeanDefinition，然后将映射后的BeanDefinition注册到一个BeanDefinitionRegistry中，之后，BeanDefinitionRegistry即完成Bean的注册和加载。

​	

![1564831764058](D:\我的文档\JAVA\框架\Spring解密\images\配置文件加载)

后面讲的是XML的方式，现在发现认真看（也就是一个字一个字的看并没有什么难懂的）关键是写了配置文件还是要在启动类中加载才回去使用

## 注解方式

如果需要注解标注的方式为FXNewsProvider注入所需要的依赖，现在可以使用 @Autowired 以及 @Componet对相关类进行标记。下面是使用了注解之后的代码

![1564832664418](C:\Users\lwx\AppData\Roaming\Typora\typora-user-images\1564832664418.png)

@AutoWired是这里的主角，它的存在将告知Sring容器需要为当前对象注入哪些依赖对象

@Component说是要配合的 classpath-scanning来做（也就是在XML中加一些东西），但是在SpringBoot中应该是不需要了，需要注意的是单单使用spring还是要写XML的。

# BeanFactory的XML之旅

## < beans >  之唯我独尊

<beans>是XML配置文件中最顶层的元素，它下面可以包含0个或者一个<description>和多个<bean>以及<import>或者<alias>,如图

![1564834125595](D:\我的文档\JAVA\框架\Spring解密\images\beans)

<beans> 作为所有<bean>的统帅，它拥有相应的属性(attribute) 对所辖的 <bean>进行统一的默认行为设置，包括如下几个：

1. default-lazy-init	其值可以指定为true或者false，默认值为false。用来标记是否对所有的<bean>进行延迟初始化。
2. default-aotowire   可以取值为no  byName   byType  constructor 以及 autodetect。默认值为no，如果使用自动绑定的话，用来标志全体bean使用哪一种默认绑定方式。
3. default-dependency-check   可以取值为none、objects、simple以及all，默认值为none，即不做依赖检查。
4. default-init-method  如果所管辖的<bean>按照某种规则，都有同样名称的初始化方法的话，可以在这里统一指定这个初始化方法名，而不用在每一个<bean>上都重复单独指定。
5. default-destroy-method   与 default-init-method相对应，如果所管辖的bean有按照某种规则使用了相同名称的对象销毁方法，可以通过这个属性统一指定。

可省略的一些标签

<description>	

​	基本没卵用，写一些描述信息上去

<import>
通常情况下，可以根据模块功能或者层次关系，将配置信息分门别类地放到多个配置文件中。在 5
想加载主要配置文件，并将主要配置文件所依赖的配置文件同时加载时，可以在这个主要的配置文件
中通过<import>元素对其所依赖的配置文件进行引用。比如，如果A.xml中的<bean>定义可能依赖
B.xml中的某些<bean>定义，那么就可以在A.xml中使用<import>将B.xml引入到A.xml，以类似于
<import resource="B.xml"/>的形式。 

<alias>

可以通过<alias>为某些<bean>起一些“外号”（别名），通常情况下是为了减少输入。比如，
假设有个<bean>，它的名称为dataSourceForMasterDatabase，你可以为其添加一个<alias>，像
这样<alias name="dataSourceForMasterDatabase" alias="masterDataSource"/>。以后通过
dataSourceForMasterDatabase或者masterDataSource来引用这个<bean>都可以，只要你觉得方便
就行。  其实讲道理这个取别名感觉还是可以的，**要紧记的是用了多个取再多的名字对的都是同一个Bean**



# 容器背后的秘密