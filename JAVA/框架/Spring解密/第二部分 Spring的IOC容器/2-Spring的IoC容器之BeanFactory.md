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
//拉响启航的汽笛。在BeanFactory出现之前，我们通常会直接在应用程序的入口类的main方法中，
//自己实例化相应的对象并调用之，如以下代码所示： 
FXNewsProvider newsProvider = new FXNewsProvider();
newsProvider.getAndPersistNews(); 
不过，现在既然有了BeanFactory，我们通常只需将“生产线图纸”交给BeanFactory，让
BeanFactory为我们生产一个FXNewsProvider，如以下代码所示： 13

BeanFactory container = new XmlBeanFactory(new ClassPathResource("配置文件路径"));
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider");
newsProvider.getAndPersistNews();
//或者如以下代码所示：
ApplicationContext container = new ClassPathXmlApplicationContext("配置文件路径");
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider");
newsProvider.getAndPersistNews(); 
//亦或如以下代码所示：
ApplicationContext container = new FileSystemXmlApplicationContext("配置文件路径");
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
//代码清单4-4 通过编码方式使用BeanFactory实现FX新闻相关类的注册及绑定
public static void main(String[] args)
{
DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
BeanFactory container = (BeanFactory)bindViaCode(beanRegistry);
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider");
newsProvider.getAndPersistNews();
}
public static BeanFactory bindViaCode(BeanDefinitionRegistry registry)
{
AbstractBeanDefinition newsProvider = new RootBeanDefinition(FXNewsProvider.class,true);
AbstractBeanDefinition newsListener = new RootBeanDefinition(DowJonesNewsListener.class,true);
AbstractBeanDefinition newsPersister = new RootBeanDefinition(DowJonesNewsPersister.class,true);
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
//在Spring的术语中，把BeanFactory需要使用的对象注册和依赖绑定信息称为Configuration Metadata。我们这里所
//展示的，实际上就是组织这些Configuration Metadata的各种方式。因此这个标题才这么长。
MutablePropertyValues propertyValues = new MutablePropertyValues();
propertyValues.addPropertyValue(new PropertyValue("newsListener",newsListener));
propertyValues.addPropertyValue(new PropertyValue("newPersistener",newsPersister));
newsProvider.setPropertyValues(propertyValues);
// 绑定完成 2
return (BeanFactory)registry;	//看了下源代码感觉有问题..这样能拿到bean吗
//这样写确实是没问题的，面向接口编程，但是具体的实现类里面又（间接）实现了BeanFactory这个接口，所以是可以实现互转的
    
```

关于代码的解释很精彩 信手拈来。

## 外部配置文件方式

你也可以引入自己的文件格式，前提是真的需要。 Spring的IoC容器支持两种配置文件格式：Properties文件格式和XML文件格式。当然，也可以引入自己的文件格式，前提是真的需要。

采用外部配置文件时，Spring的IoC容易有一个统一的处理方式。通常情况下，需要根据不同的外部配置文件格式，给出相应的BeanDefinitionReader的实现类，由BeanDefinitionReader的相应实现类负责将相应的配置文件内容读取并映射到BeanDefinition，然后将映射后的BeanDefinition注册到一个BeanDefinitionRegistry中，之后，BeanDefinitionRegistry即完成Bean的**注册和加载**。

​	

![1564831764058](D:\我的文档\JAVA\框架\Spring解密\images\配置文件加载)

后面讲的是XML的方式，现在发现认真看（也就是一个字一个字的看并没有什么难懂的）关键是写了配置文件还是要在启动类中加载才回去使用

## 注解方式

如果需要注解标注的方式为FXNewsProvider注入所需要的依赖，现在可以使用 @Autowired 以及 @Componet对相关类进行标记。下面是使用了注解之后的代码

![1564832664418](C:\Users\lwx\AppData\Roaming\Typora\typora-user-images\1564832664418.png)

@AutoWired是这里的主角，它的存在将告知Sring容器需要为当前对象注入哪些依赖对象

@Component说是要配合的 **classpath-scanning**来做（也就是在XML中加一些东西），但是在SpringBoot中应该是不需要了，需要注意的是单单使用spring还是要写XML的。

这里我觉得还是不能忽略掉，毕竟这涉及到了 classpath的概念

![1565059874474](D:\我的文档\JAVA\框架\Spring解密\images\类路径扫描)

# BeanFactory的XML之旅

## 1. < beans >  之唯我独尊

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

## 2. 孤孤单单一个 < bean>

每个业务对象，在spring的XML配置文件中是与<bean>元素一一对应的。摸头一个之后其他的都好解决了。

![1564877664570](D:\我的文档\JAVA\框架\Spring解密\images\bean)

### id属性

​	通常情况下，每个注册到容器的对象需要一个唯一标志将与其“共处一室”的“兄弟们”区分开来。通过这个id属性来指定当前注册对象的beanName是什么。实际上，并非任何情况下都需要指定每个<bean>的id，有些情况下，id可以省略，比如后面会提到的内部<bean> 以及不需要根据 beanName明确依赖关系的场合等。

​	除了可以使用id来指定<bean>在容器中的标志，还可以使用name属性来指定<bean>的别名(alias)。比如，以上定义，我们还可以像如下代码这样，为其添加别名：

```xml
<bean id="djNewsListener"name="/news/djNewsListener,dowJonesNewsListener" class="..impl.DowJonesNewsListener">
</bean>
```

与id属性相比，name属性的灵活之处在于，name可以使用id不能使用的一些字符，比如	/	。而且还可以通过逗号、宫格或者冒号分割指定多个name。name的作用跟使用<alias>为id 指定多个别名基本相同：

![1564878220999](D:\我的文档\JAVA\框架\Spring解密\images\alias)

### class属性

​	每个注册到容器的对象都需要通过<bean>元素的class属性指定其类型。在大部分情况下这个属性是必须的，仅在少数情况下不需要指定，如后面将提到的在使用抽象配置模板的情况下。

## 3. Help me , Help you

​	在大部分情况下，你不太可能选择单独“作战”，业务对象也是；各个业务对象之间会相互协作来更好的完成同一使命。这时，各个业务对象之间的相互依赖就是无法避免的。对象之间需要相互协作，在横向上它们存在一定的依赖性。而现在我们就是要看一下，在Spring的IoC的XML配置中，应该如何表达这种依赖性。

### 	1.构造方法注入的XML之道

按照Spring的IoC容器配置格式，要通过构造方法注入方式，为当前业务对象注入其所依赖的对象，

需要使用<constructor-arg>。正常情况下，如以下代码所示： 

![1564878925184](D:\我的文档\JAVA\框架\Spring解密\images\构造方法注入)

对于<ref>元素，稍后会进行详细说明。这里你只需要知道，通过这个元素来指明容器将为
djNewsProvider这个<bean>注入通过<ref>所引用的Bean实例。这种方式可能看起来或者编写起来
不是很简洁，最新版本的Spring也支持配置简写形式，如以下代码所示： 

![1564879003517](D:\我的文档\JAVA\框架\Spring解密\images\简写)

有时候，容器在加载XML配置的时候，因为某些原因，无法明确配置项与对象的构造方法参数列表的一一对应关系，就需要请<constructor-arg>的 type 或者 index 属性出马。比如，对象存在多个构造方法，**当参数列表数目相同而类型不同的时候**，容器无法区分应该使用哪个构造方法来实例化对象，或者构造方法可能同时传入最少两个类型相同的对象。

#### type属性

​	假设有一个对象定义如代码清单4-12所示。 

![1564880067462](D:\我的文档\JAVA\框架\Spring解密\images\某一个业务对象)

该类声明了两个构造方法，分别都只是传入一个参数，且参数类型不同。这时，我们可以进行配置，如下面代码所示

![1564880940997](D:\我的文档\JAVA\框架\Spring解密\images\bean1)

**如果从BeanFactory** 取得该对象并调用toString() 查看的话，我们会发现Spring调用的是第一个构造方法，因为输出是如下内容：

![1564881049419](D:\我的文档\JAVA\框架\Spring解密\images\输出1)

但是，如果我们想调用的却是第二个传入int类型的参数的构造方法，又该如何呢？可以使用 type属性，通过指定构造方法的参数类型来解决这一问题，配置内容如下代码所示 

![1564881132596](D:\我的文档\JAVA\框架\Spring解密\images\bean2)

现在，我们得到了自己想要的对象实例，如下的控制台输出信息印证了这一点：

![1564881182313](D:\我的文档\JAVA\框架\Spring解密\images\输出2)

#### index属性

​	当某个业务对象的构造方法同时传入多个类型相同的参数时，Spring 又该如何将这些配置中的信息与实际对象的参数一 一 对应呢？好在，如果配置项信息和对象参数可以按照顺序初步对应的话，Spring还是可以正常工作的，如代码清单4-13所示 

![1564881381385](D:\我的文档\JAVA\框架\Spring解密\images\对象2)

![1564881409175](D:\我的文档\JAVA\框架\Spring解密\images\index)



### 2. setter方法注入的XML之道

与构造方法注入可以使用<constructor-arg>注入配置相对应， Spring为setter方法注入提供了
<property>元素 

<property>有一个name属性（attribute），用来指定该<property>将会注入的对象所对应的实例
变量名称。之后通过value或者ref属性或者内嵌的其他元素来指定具体的依赖对象引用或者值，如以
下代码所示：

![1564881588036](D:\我的文档\JAVA\框架\Spring解密\images\set) 

**当然，如果只是使用<property> 进行依赖注入的话，请确保你的对象提供了默认的构造方法，也就是一个参数都没有那个**

以上配置形式还可以简化为如下形式：

![1564881699446](D:\我的文档\JAVA\框架\Spring解密\images\简化1)

使用<property>的setter方法注入和使用<constructor-arg>的构造方法注入并不是水火不容 7
的。实际上，如果需要，可以同时使用这两个元素： 

![1564881760635](D:\我的文档\JAVA\框架\Spring解密\images\混合1)

当然，现在需要MockBusinessObject提供一个只有一个String类型参数的构造方法，并且为
dependency2提供了相应的setter方法。代码清单4-14演示了符合条件的一个业务对象定义 

![1564881807982](D:\我的文档\JAVA\框架\Spring解密\images\对象3)

也就是说两种注入方式都一起用了吧

### 3.< property>和< constructor-arg>中可用的配置项 

之前我们看到，可以通过在<property>和<constructor-arg>这两个元素内部嵌套<value>或者
<ref>，来指定将为当前对象注入的简单数据类型或者某个对象的引用。不过，为了能够指定多种注
入类型， Spring还提供了其他的元素供我们使用，这包括bean、 ref、 idref、 value、 null、 list、 set、 map、
props。下面我们来逐个详细讲述它们 

**提示 以下涉及的所有内嵌元素，对于<property>和<constructor-arg>都是通用的** 

#### (1) < value>

通过value为主题对象注入简单的数据类型，不但可以指定String类型的数据，而且可以指定其他Java语言中的原始类型以及它们的包装器（wrapper）类型，比如int、 Integer等 。容器在注入的时候，会做适当的转换工作（我们会在后面揭示转换的奥秘）。你之前已经见过如何使用<value>了，不过让我们通过如下代码来重新认识一下它： 

![1564882097682](D:\我的文档\JAVA\框架\Spring解密\images\value)

当然，如果愿意，你也可以使用如下的简化形式（不过这里的value是以上一层元素的属性身份
出现）： 

![1564882156171](D:\我的文档\JAVA\框架\Spring解密\images\简写2)

#### （2）< ref>

​	使用ref来引用容器中其他的对象实例，可以通过 ref 的 local 、parent  和 bean属性来指定引用的对象的 beanName是什么。代码清单4-15演示了ref及其三个对应属性的使用情况。 

![1564882388631](D:\我的文档\JAVA\框架\Spring解密\images\ref)

loal	parent 和 bean 的区别在于：

local 只能指定与当前配置的对象在同一个配置文件的对象定义的名称。（可以获得XML解析器
的id约束验证支持）； 

parent 则只能指定位于当前容器的父容器中定义的对象引用。

@@注意@@ ，BeanFactory可以分层次（通过实现HierarchicalBeanFactory 接口），容器A在初始化的时候，可以首先加载容器B中的所有对象定义，然后再加载自身的对象定义，这样，容器B就成为了容器A的父容器，容器A可以引用容器B中所有对象定义:

![1564883014641](D:\我的文档\JAVA\框架\Spring解密\images\父容器)

然而时过境迁，XmlBeanFactory已经被弃用。



**<bean>则基本上通吃，所以，通常情况下，直接使用bean来指定对象引用就可以了 **

<ref>的定义为<!ELEMENT ref EMPTY>，也就是说，它下面没有其他子元素可用了，别硬往人家
肚子里塞东西哦 



#### （3） < idref>  

如果要为当前对象注入所依赖的对象的名称，而不是引用，那么通常情况下，可以使用 <value>来达到这个目的

![1564883252720](D:\我的文档\JAVA\框架\Spring解密\images\ref3)

但这种场合下，使用idref才是最为合适的。因为使用idref，容器在解析配置的时候就可以帮 你检查这个beanName到底是否存在，而不用等到运行时才发现这个beanName对应的对象实例不存在。毕竟，输错名字的问题很常见。以下代码演示了idref的使用：  （所以说启动报错反而是件好事? 起码是提前治疗而不用等到病入膏肓的时候）

![1564883364193](D:\我的文档\JAVA\框架\Spring解密\images\idref)

这段配置跟上面使用<value>达到了相同的目的，**不过更加保险**。如果愿意，也可以通过local而
不是bean来指定最终值，不过， bean比较大众化哦。

#### （4）内部 < bean>

使用<ref>可以引用容器中独立定义的对象定义。但有时，可能我们所依赖的对象只有当前一个对象引用，或者某个对象定义我们不想其他对象通过<ref> 引用到它。这时，我们可以使用内嵌的<bean>，将这个私有的对象定义仅局限在当前对象。对于FX新闻的DowJonesNewsListener而言 实际上只有道琼斯的FXNewsProvider会使用它。而且，我们也不想让其他对象引用到它。为此完全可以像代码清单4-16这样，将它配置为内部<bean>的形式。 

![1564883648932](D:\我的文档\JAVA\框架\Spring解密\images\内部bean)

这样，该对象实例就只有当前的djNewsProvider 可以使用，其他对象无法取得该对象的引用。

![1564883966361](D:\我的文档\JAVA\框架\Spring解密\images\内部bean1)

内部<bean>的配置只是在位置上有所差异，但配置项上与其他的<bean>是没有任何差别的。也
就是说， <bean>内嵌的所有元素，内部<bean>的<bean>同样可以使用。如果内部<bean>对应的对象
还依赖于其他对象，你完全可以像其他独立的<bean>定义一样为其配置相关依赖，没有任何差别。 

#### （5） < list>

​		<list>  对应注入对象类型为 java.util.List及其子类或者数组的依赖对象。通过<list> 可以有序地为当前对象注入以collection形式声明的依赖。 代码清单4-17给出了一个使用<list>的实例演示 

​	![1564884222546](D:\我的文档\JAVA\框架\Spring解密\images\注入list)

注意，<list> 内部可以嵌套其他元素，并且可以像 param1所展示的那样夹杂配置。但是，从好的编程实践来说，这样的处理并不妥当，除非你真的知道自己在做什么！（以上只是出于演示的目的才会如此配置 ）

#### （6） < set>

如果说<list>可以帮你有序地注入一系列依赖的话，那么<set>就是无序的，而且，对于set来说，元素的顺序本来就是无关紧要的。 <set>对应注入Java Collection中类型为java.util.Set或者其子类的依赖对象。代码清单4-18演示了通常情况下<set>的使用场景。 

![1564884504482](D:\我的文档\JAVA\框架\Spring解密\images\set1)

例子就是例子，只是为了给你演示这个元素到底有多少能耐。从配置上来说，这样多层嵌套、多元素混杂配置是完全没有问题的。不过，各位在具体编程实践的时候可要小心了。如果你真的想这么夹杂配置的话，不好意思，估计ClassCastException会很愿意来亲近你，而这跟容器或者配置一点儿关系也没有 

#### (7) 	< map>

与列表（list）使用数字下标来标识元素不同，映射（map）可以通过指定的键（key来获取相应的值。如果说在<list>中混杂不同元素不是一个好的实践（方式）的话，你就应该求助<map>。<map>与<list>和<set>的相同点在于，都是为主体对象注入Collection类型的依赖，不同点在于它对应注入java.util.Map或者其子类类型的依赖对象。代码清单4-19演示了<map>的通常使用场景。 

![1564884703204](D:\我的文档\JAVA\框架\Spring解密\images\map)

 

对于<map>来说， 它可以内嵌任意多个<entry>，每一个<entry>都需要为其指定一个键和一个值，
就跟真正的java.util.Map所要求的一样。 

指定entry的键。 可以使用<entry>的属性——key或者key-ref来指定键，也可以使用<entry>的内嵌元素<key>来指定键，这完全看个人喜好，但两种方式可以达到相同的效果。在<key>内部，可以使用以上提到的任何元素来指定键，从简单的<value>到复杂的Collection，只要映射需要，你可以任意发挥。

指定entry对应的值。 <entry>内部可以使用的元素，除了<key>是用来指定键的，其他元素可以任意使用，来指定entry对应的值。除了之前提到的那些元素，还包括马上就要谈到的<props>。如果对应的值只是简单的原始类型或者单一的对象引用，也可以直接使用<entry>的value或者value-ref这两个属性来指定，从而省却多敲入几个字符的工作量。 

![1564884980507](D:\我的文档\JAVA\框架\Spring解密\images\map2)



   所以，如果你不想敲那么些字符，可以像代码清单4-20所展示的那样使用<map>进行依赖注入的 配置。


![1564885058053](D:\我的文档\JAVA\框架\Spring解密\images\map3)

#### (8) < props>

<props>。 <props>是简化后了的<map>，或者说是特殊化的map，该元素对应配置类型为java.util.Properties的对象依赖。因为Properties只能指定String类型的键（key）和值，所以，<props>的配置简化很多，只有固定的格式，见代码清单4-21 

![1564885322965](D:\我的文档\JAVA\框架\Spring解密\images\prop)

每个<props>可以嵌套多个<prop>，每个<prop>通过其key属性来指定键，在<prop>内部直接指定其所对应的值。<prop>内部没有任何元素可以使用，只能指定字符串，这是由java.util.Properties的语意决定的。 

#### (9) < null/>

最后一个提到的元素是</null>，这是最简单的一个元素，因为它只是一个空元素，而且通常使用到它的场景也不是很多。对于String类型来说，如果通过value以这样的方式指定注入，即<value></value>，那么，得到的结果是""，而不是null 。所以，如果需要为这个string对应的值注入null的话，请使用<null/>。当然，这并非仅限于String类型，如果某个对象也有类似需求，请不要犹豫。代码清单4-22演示了一个使用<null/>的简单场景 

![1564885798805](D:\我的文档\JAVA\框架\Spring解密\images\null)



### 4.  depends-on  

​	通常情况下，可以直接通过之前提到的所有元素，来显示地指定bean之间的依赖关系。这样，容器在初始化当前bean定义的时候，会根据这些元素所标记的依赖关系，首先实例化当前bean定义所依赖的其他bean定义。但是，**如果某些时候，我们没有通过类似<ref>的元素明确指定对象A依赖于对象B的话，如果让容器在实例化对象A之前首先实例化对象B呢？**

![1564886036128](D:\我的文档\JAVA\框架\Spring解密\images\构建顺序)

​	系统中所有需要日志记录的类，都需要在这些类使用之前首先初始化log4j。那么，就会非显式地依赖于SystemConfigurationSetup的静态初始化块。如果ClassA需要使用log4j，那么就必须在bean定义中使用depends-on来要求容器在初始化自身实例之前首先实例化SystemConfigurationSetup，以保证日志系统的可用，如下代码演示的正是这种情况 ：

![1564886129512](D:\我的文档\JAVA\框架\Spring解密\images\depend)

举log4j在静态代码块（static block）中初始化的例子在实际系统中其实不是很合适，因为通常在应用程序的主入口类初始化日志就可以了。这里主要是给出depends-on可能的使用场景，大部分情况下，是那些拥有静态代码块初始化代码或者数据库驱动注册之类的场景。 

如果说ClassA拥有多个类似的**非显式依赖关系**，那么，你可以在ClassA的depends-on中通过逗号分割各个beanName，如下代码所示： 

![1564886352426](D:\我的文档\JAVA\框架\Spring解密\images\depend1)



### 5. autowire(这个应该是重中之重了)

​	除了可以通过配置明确指定bean之间的依赖关系，Spring还提供了根据bean定义的某些特点 将相互依赖的某些bean直接自动绑定的功能。通过<bean>的autowire属性，可以指定当前bean定义采用某种类型的自动绑定模式。这样，**你就无需手工明确指定该bean定义相关的依赖关系，从而也可以免去一些手工输入的工作量**

Spring提供了5种自动绑定模式，即no、 byName、 byType、 constructor和autodetect，下面是它们的具体介绍。 

#### 1. no

容器默认的自动绑定模式，也就是不采用任何形式的自动绑定，完全依赖手工明确配置各个bean之间的依赖关系，以下代码演示的两种配置是等效的： 

​	![1564886667635](D:\我的文档\JAVA\框架\Spring解密\images\no)

#### 2. byName

按照类种声明的实例变量的名称，与XML配置文件中声明的bean定义的beanName的值进行匹配，相匹配的bean的定义将被自动绑定到当前实例变量上。这种方式对类定义和配置的bean定义有一定的限制。假设我们有如下所示的类定义 ：

![1564886809565](D:\我的文档\JAVA\框架\Spring解密\images\byname)

需要注意两点，**第一，我们并没有明确指定fooBean的依赖关系，而仅指定了它的autowire属性为byName；第二，第二个bean定义的 id 为 emphasisAttribute ，与Foo类中的实例变量名称相同 **

#### 3.byType

​	如果指定当前bean定义的 aotowire模式为 byType，那么，容器会根据当前bean定义类型，分析其相应的依赖对象类型，然后到容器所管理的所有bean定义中寻找与依赖对象类型相同的bean定义，然后将找到的符合条件的bean自动绑定到当前bean定义。

​	对于byName模式中的实例类Foo来说，容器会在其所管理的所有bean定义中寻找类型为Bar的bean定义。如果找到，则将找到的bean绑定到Foo的bean定义；如果没有找到，则不做设置。**但如果找到多个，容器会告诉你它解决不了“该选用哪一个”的问题**，你只好自己查找原因，并自己修正该问题。所以，**byType只能保证，在容器中只存在一个符合条件的依赖对象的时候才会发挥最大的作用，如果容器中存在多个相同类型的bean定义，那么不好意思，采用手动明确配置吧！**

@注意@在java的继承树上，所有的类型本质上都是同一种类型！ 不过讲道理，如果看的是父类的话，那大家不都是Object类型了吗？这样不是就一定冲突了吗？我觉得本质上应该是加载了两个类型真相同但是名字不相同的bean导致了这个问题，应该不是简单的继承关系

@尝试后的结果@ **神奇的是如果把Bar改成接口的话，就不会出现说找到多个的问题**

而且如果依赖的bean不存在的话也不会报错，只是说会注入不到，值为空而已

![1564887639923](D:\我的文档\JAVA\框架\Spring解密\images\byType)

#### 4.constructor 

byName 和 byType类型的自动绑定模式是针对 property 的自动绑定，**而constructor类型则是针对构造方法参数的类型而进行的自动绑定**，它同样是 byType类型的绑定模式。不过，constructor 是匹配构造方法的参数类型，而不是实例属性的类型，与byType模式类似，如果找到不止一个符合条件的bean定义，那么，容器会返回错误。使用上也与byType没有太大差别，只不过是应用到需要使用构造方法注入的bean定义之上，代码清单4-23给出了一个使用construtor模式进行自动绑定的简单场景演示。 

![1564897638321](D:\我的文档\JAVA\框架\Spring解密\images\构造器)

同理，如果跟byType类似的话，被注入的类型不能有子类。

#### 5.autodetect 

这种模式是byType和constructor模式的结合体，如果对象拥有默认无参数的构造方法，容器会优先考虑byType的自动绑定模式。否则，会使用constructor模式。当然，如果通过构造方法注入绑定后还有其他属性没有绑定，容器也会使用byType对剩余的对象属性进行自动绑定。
![1564898396894](D:\我的文档\JAVA\框架\Spring解密\images\小心)

![1564898591288](D:\我的文档\JAVA\框架\Spring解密\images\自动绑定的利与弊)

@反思!!!!@所以说在我们之前的干的项目中，全都用的autowire，用的是爽了但是控制能力弱了很多，**所以现在重点需要研究的是SpringBoot 中是怎样解决这个问题的 或者说问的仔细点就是 Sring Boot 中是否给我们使用配置文件来控制Bean的能力**

通常情况下，只要有良好的XML编辑器支持，我不会介意多敲那几个字符。起码自己可以对整个系统的行为有完全的把握。当然，任何事物都不绝对，只要根据相应场景找到合适的就可以。 

噢，对了，差点儿忘了！作为所有<bean>的统帅， <beans>有一个default-autowire属性，它可以帮我们省去为多个<bean>单独设置autowire属性的麻烦， default-autowire的默认值为no，即不进行自动绑定。如果想让系统中所有的<bean>定义都使用byType模式的自动绑定，我们可以使用如下配置内容： 

![1564898964137](D:\我的文档\JAVA\框架\Spring解密\images\beans全局)



#### 6.dependency-check 

我们可以使用每个<bean>的dependency-check属性对其所依赖的对象进行最终检查，就好像电影里每队美国大兵上战场之前，带队的军官都会朝着士兵大喊“检查装备， check， recheck”是一个道理。 

该功能主要与自动绑定结合使用，可以保证当自动绑定完成后，最终确认每个对象所依赖的对象是否按照所预期的那样被注入。当然，并不是说不可以与平常的明确绑定方式一起使用。 

**总地来说，控制得力的话，这个依赖检查的功能我们基本可以不考虑使用 **

也就是说可以忽略一波了。。

#### 7。lazy-init

​	延迟初始化( lazy-init ) 这个特性的作用，主要是可以针对 ApplicationContext 容器的bean初始化行为施以更多控制。与BeanFactory 不同，ApplicationContext 在容器启动的时候，**就会马上对所有的 singleton 的 bean 定义**进行实例化操作。通常这种默认行为是好的，因为如果系统有问题的话，可以在第一时间发现这些问题，但有时，我们不想某些bean定义在容器启动后就直接实例化，可能出于容器启动时间的考虑，也可能出于其他原因的考虑。总之，我们想改变某个或者某些bean定义在ApplicationContext 容器中的默认实例化时机。这时，就可以通过<bean>的lazy-init属性来控制这种初始化行为，如下代码所示： 

![1564899608238](D:\我的文档\JAVA\框架\Spring解密\images\延迟初始化)

这样， ApplicationContext容器在启动的时候，只会默认实例化not-lazy-init-bean而不会实例化lazy-init-bean。

当然， 仅指定lazy-init-bean的lazy-init为true，并不意味着容器就一定会延迟初始化该bean的实例。如果某个非延迟初始化的bean定义依赖于lazy-init-bean，那么毫无疑问，按照依赖决计的顺序，容器还是会首先实例化lazy-init-bean，然后再实例化后者，如下代码演示了这种相互牵连导致延迟初始化失败的情况：  

![1564899684299](D:\我的文档\JAVA\框架\Spring解密\images\延迟失败)

虽然lazy-init-bean是延迟初始化的，但因为依赖它的not-lazy-init-bean并不是延迟初始化，所以lazy-init-bean还是会被提前初始化，延迟初始化的良好打算“泡汤”。如果我们真想保证lazy-init-bean一定会被延迟初始化的话，就需要保证依赖于该bean定义的其他bean定义也同样设置为延迟初始化。在bean定义很多时，好像工作量也不小哦。不过不要忘了， <beans>可是所有<bean>的统领啊，让它一声令下吧！如代码清单4-24所演示的，在顶层由<beans>统一控制延迟初始化行为即可。
![1564899762735](D:\我的文档\JAVA\框架\Spring解密\images\全部配置延迟初始化)

这样我们就不用每个<bean>都设置一遍，省事儿多了不是吗？ 

## 4.继承？我也会！ 

除了单独存在的bean以及多个bean之间的横向依赖关系，我们也不能忽略“纵向上”各个bean之间的关系。确切来讲，我其实是想说“**类之间的继承关系”**。不可否认，继承可是在面向对象界声名远扬啊 

假设我们某一天真的需要对FXNewsProvider使用继承进行扩展，那么可能会声明如下代码所示的子类定义： 

![1564899984668](D:\我的文档\JAVA\框架\Spring解密\images\extends)

实际上，我们想让该子类与FXNewsProvider使用相同的IFXNewsPersister，即DowJonesNewsPersister，那么可以使用如代码清单4-25所示的配置 

![1564900126564](D:\我的文档\JAVA\框架\Spring解密\images\继承后的配置)

但实际上，这种配置存在冗余，而且也没有表现两者之间的纵向关系。所以，我们可以引入XML中的bean的“继承”配置，见代码清单4-26。 

![1564900178400](D:\我的文档\JAVA\框架\Spring解密\images\去除冗余)

仔细观察之后发现共同的部分被去掉了

我们在声明subNewsProvider的时候，使用了parent属性，将其值指定为superNewsProvider，这样就继承了superNewsProvider定义的默认值，只需要将特定的属性进行更改，而不要全部又重新定义一遍。 

parent属性还可以与abstract属性结合使用，达到将相应bean定义模板化的目的。比如，我们还可以像代码清单4-27所演示的这样声明以上类定义。 

![1564900353995](D:\我的文档\JAVA\框架\Spring解密\images\模板)

newsProviderTemplate的bean定义通过abstract属性声明为true，说明这个bean定义不需要实例化。实际上，这就是之前提到的可以不指定class属性的少数场景之一（当然，同时指定class和abstract="true"也是可以的）。该bean定义只是一个配置模板，不对应任何对象 。superNewsProvider和subNewsProvider通过parent指向这个模板定义，就拥有了该模板定义的所有属性配置。当多个bean定义拥有多个相同默认属性配置的时候，你会发现这种方式可以带来很大的便利。 

另外，既然这里提到abstract，对它就多说几句。容器在初始化对象实例的时候，不会关注将abstract属性声明为true的bean定义。如果你不想容器在初始化的时候实例化某些对象，那么可以将其abstract属性赋值true，以避免容器将其实例化。对于ApplicationContext容器尤其如此，**因为默认情况下， ApplicationContext会在容器启动的时候就对其管理的所有bean进行实例化，只有标志为abstract的bean除外**。 

## **5.bean 的 scope**

BeanFactory除了拥有作为IoC Service Provider的职责，作为一个轻量级容器，它还有着其他一些职责，其中就包括对象的生命周期管理。 

本节主要讲述容器中管理的对象的scope这个概念，多数中文资料在讲解bean的scope时喜欢用“作用域”这个名词，应该还算贴切吧。不过，我更希望告诉你scope这个词到底代表什么意思，至于你怎么称呼它反而不重要。 

scope用来声明容器中的对象所应该处的限定场景或者说该对象的存活时间，即容器在对象进入其相应的scope之前，生成并装配这些对象，在该对象不再处于这些scope的限定之后，容器通常会销毁这些对象。打个比方吧！我们都是处于社会（容器）中，如果把中学教师作为一个类定义，那么当容器初始化这些类之后，中学教师只能局限在中学这样的场景中；中学，就可以看作中学教师的scope。 

Spring容器最初提供了两种bean的scope类型： singleton和prototype，但发布2.0之后，又引入了另外三种scope类型，即request、 session和global session类型。不过这三种类型有所限制，只能在Web应用中使用。也就是说，只有在支持Web应用的ApplicationContext中使用这三个scope才是合理的。 

我们可以通过使用<bean>的singleton或者scope属性来指定相应对象的scope，其中， scope属性只能在XSD格式的文档声明中使用，类似于如下代码所演示的形式： 

![1564901666262](D:\我的文档\JAVA\框架\Spring解密\images\生命周期)

让我们来看一下容器提供的这几个scope是如何限定相应对象的吧。

### 1.singleton

配置中的bean定义可以看作是一个模板，容器会根据这个模板来构造对象。但是要根据这个模板构造多少对象实例，又该让这些构造完的对象实例存活多久，则由容器根据bean定义的scope语意来决定。 

标记为拥有singleton scope的对象定义，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的“寿命”。 

图4-5是Spring参考文档中所给出的singleton的bean的实例化和注入语意演示图例，或许可以更形象地说明问题。 

![1564901774513](D:\我的文档\JAVA\框架\Spring解密\images\单例)

**需要注意的一点是，不要因为名字的原因而与GoF所提出的 Singleton模式相混淆，二者的语意是不同的：标记为 singleton 的bean 是由容器来保证这种类型的bean在同一个容器中只存在一个共享实例；而Singleton模式则是保证在同一个 Classloader  中只存在一个这种类型的实例**

可以从两个方面来看待singleton的bean所具有的特性。 

对象实例数量。 singleton类型的bean定义，在一个容器中只存在一个共享实例，所有对该类型 bean的依赖都引用这一单一实例。这就好像每个幼儿园都会有一个滑梯一样，这个幼儿园的小朋友共同使用这一个滑梯。而对于该幼儿园容器来说，滑梯实际上就是一个singleton的bean 

对象存活时间。 singleton类型bean定义， 从容器启动，到它第一次被请求而实例化开始，只要容器不销毁或者退出，该类型bean的单一实例就会一直存活。通常情况下，如果你不指定bean的scope， singleton便是容器默认的scope，所以，下面三种配置形式实际上达成的是同样的效果：  

![1564902130271](D:\我的文档\JAVA\框架\Spring解密\images\全局生命)

### 2.prototype

针对声明为拥有prototype scope的bean定义。容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器扶着的，但是只要准备完毕，并且对象实例返返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。也就是说，容器每次返回给请求方一个新的对象实例之后，就任由这个对象实例“自生自灭”了。 

让我们继续幼儿园的比喻，看看prototype在这里应该映射到哪些事物。儿歌里好像有句“排排坐，分果果”，我们今天要分苹果咯！将苹果的bean定义的scope声明为prototype，在每个小朋友领取苹果的时候，我们都是分发一个新的苹果给他。发完之后，小朋友爱怎么吃怎么吃，爱什么时候吃什么时候吃。但是，吃完后要记得把果核扔到果皮箱哦！ 而如果你把苹果的bean定义的scope声明为singleton会是什么情况呢？如果第一个小朋友比较谦让，那么他可能对这个苹果只咬一口，但是下一个小朋友吃多少就不知道了。当吃得只剩一个果核的时候，下一个来吃苹果的小朋友肯定要哭鼻子的。


所以， 对于那些请求方不能共享使用的对象类型，应该将其bean定义的scope设置为prototype。这样，每个请求方可以得到自己对应的一个对象实例，而不会出现上面“哭鼻子”的现象。通常，声明为prototype的scope的bean定义类型，都是一些有状态的，比如保存每个顾客信息的对象。

从Spring 参考文档上的这幅图片（见图4-6），你可以再次了解一下拥有prototype scope的bean定义，在实例化对象并注入依赖的时候，它的具体语意是个什么样子。



![1564902765502](D:\我的文档\JAVA\框架\Spring解密\images\原型)

你用以下形式来指定某个bean定义的scope为prototype类型，效果是一样的：
<!-- DTD -->
<bean id="mockObject1" class="...MockBusinessObject" singleton="false"/>
<!-- XSD -->
<bean id="mockObject1" class="...MockBusinessObject" scope="prototype"/> 

### 3.request 、session 和 global session

这三个scope类型是Spring 2.0之后新增加的，它们不像之前的 singleton 和 prototype 那么 ”通用“，因为它们只适用于Web应用程序，通常是与XmlWebApplicationContext共同使用，而这些将在第6部分详细讨论。不过，既然它们也属于scope的概念，这里就简单提几句。 

![1564906309034](D:\我的文档\JAVA\框架\Spring解密\images\其他)

 1.request

request通常的配置形式如下：
<bean id="requestProcessor" class="...RequestProcessor" scope="request"/> 

Spring 容 器 ， 即XmlWebApplicationContext会 为 每 个 HTTP 请 求 创建 一 个 全 新的RequestProcessor对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。当同时有10个HTTP请求进来的时候，容器会分别针对这10个请求返回10个全新的RequestProcessor对象实例，且它们之间互不干扰。从不是很严格的意义上说， request可以看作prototype的一种特例，除了场景更加具体之外，语意上差不多。 

2. session

对于Web应用来说，放到session中的最普遍的信息就是用户的登录信息，对于这种放到session中
的信息，我们可使用如下形式指定其scope为session： 

<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/> 

Spring容器会为每个独立的session创建属于它们自己的全新的UserPreferences对象实例。与request相比，除了拥有session scope的bean的实例具有比 request scope的bean可能更长的存活时间，其他方面真是没什么差别。

3.global session

还是userPreferences，不过scope对应的值换一下，如下所示：
<bean id="userPreferences" class="com.foo.UserPreferences" scope="globalSession"/> 

global session只有应用在基于portlet的Web应用程序中才有意义，它映射到portlet的global范围的 3
session。如果在普通的基于servlet的Web应用中使用了这个类型的scope，容器会将其作为普通的session
类型的scope对待。 说白了就是现在没什么卵用了

### 4.自定义scope类型

在Spring 2.0之后的版本中，容器提供了对scope的扩展点，这样，你可以根据自己的需要或者应用的场景，来添加自定义的scope类型。需要说明的是，默认的singleton和prototype是硬编码到代码中的，而request、 session和global session，包括自定义scope类型，则属于可扩展的scope行列，它们都实现了 rg.springframework.beans.factory.config.Scope接口，该接口定义如下： 

```java
public interface Scope {
Object get(String name, ObjectFactory objectFactory); 7
Object remove(String name);
void registerDestructionCallback(String name, Runnable callback);
String getConversationId();
} 
```

要实现自己的scope类型，首先需要给出一个Scope接口的实现类，接口定义中的4个方法并非都是必须的，但get和remove方法必须实现。我们可以看一下http://www.jroller.com/eu/entry/implementing_efficinet_id_generator中提到的一个ThreadScope的实现（见代码清单4-28）。 

```java
public class ThreadScope implements Scope {		//面向接口编程无处不在
private final ThreadLocal threadScope = new ThreadLocal() { 
protected Object initialValue() {
return new HashMap();
}
};
public Object get(String name, ObjectFactory objectFactory) {
Map scope = (Map) threadScope.get();
Object object = scope.get(name);
if(object==null) {
object = objectFactory.getObject();
scope.put(name, object); 
}
return object;
} 
public Object remove(String name) {
Map scope = (Map) threadScope.get();
return scope.remove(name); 17
}54 Spring 的 IoC 容器
public void registerDestructionCallback(String name, Runnable callback) {
}
...
}
```

更多Scope相关的实例，可以参照同一站点的一篇文章“More fun with Spring scopes”（http://jroller.com/eu/entry/more_fun_with_spring_scopes），其中提到PageScope的实现。 

有了Scope的实现类之后，我们需要把这个Scope注册到容器中，才能供相应的bean定义使用。通常情况下，我们可以使用ConfigurableBeanFactory的以下方法注册自定义scope： 

void registerScope(String scopeName, Scope scope); 

其中，参数scopeName就是使用的bean定义可以指定的名称，比如Spring框架默认提供的自定义scope
类型request或者session。参数scope即我们提供的Scope实现类实例。 

对于以上的ThreadScope，如果容器为BeanFactory类型（当然，更应该实现ConfigurableBeanFactory），我们可以通过如下方式来注册该Scope：
Scope threadScope = new ThreadScope();
beanFactory.registerScope("thread",threadScope); 

之后，我们就可以在需要的bean定义中直接通过“thread”名称来指定该bean定义对应的scope
为以上注册的ThreadScope了，如以下代码所示：
<bean id="beanName" class="..." scope="thread"/> 

除了直接编码调用ConfigurableBeanFactory的registerScope来注册scope， Spring还提供了一个专门用于统一注册自定义scope的BeanFactoryPostProcessor实现（有关BeanFactoryPostProcessor的更多细节稍后将详述），即org.springframework.beans.factory.config.CustomScopeConfigurer。对于ApplicationContext来说，因为它可以自动识别并加载BeanFactoryPostProcessor，所以我们就可以直接在配置文件中，通过这个CustomScopeConfigurer注册来ThreadScope（如代码清单4-29所示）。 

![1564907461631](D:\我的文档\JAVA\框架\Spring解密\images\自定义scope)

在以上工作全部完成之后，我们就可以在自己的bean定义中使用这个新增加到容器的自定义scope
“thread”了，如下代码演示了通常情况下“thread”自定义scope的使用：
<bean id="beanName" class="..." scope="thread">
< aop:scoped-proxy/>
</bean>
由于<aop:scoped-proxy/>涉及Spring AOP相关知识，这里不会详细讲述。需要注意的是，使用了自定义scope的bean定义，需要该元素来为其在合适的时间创建和销毁相应的代理对象实例。对于request、 session和global session来说，也是如此。 （这里好像涉及到了什么代理模式 我就有点看不懂了）



## 6 工厂方法与 FactoryBean

在强调”面向接口编程“的同时，有一点需要注意：虽然对象可以通过上面接口来避免对特定接口实现类的过度耦合，但总归需要一种方式将声明依赖接口的对象与接口实现类关联起来。否则，**只依赖一个不做任何事情的接口是没有任何用处的。**  假设我们有一个像代码清单4-30所声明的Foo类，它声明了一个BarInterface依赖。 

![1564907936273](D:\我的文档\JAVA\框架\Spring解密\images\面向接口编程)

如果该类是由我们设计并开发的，那么还好说，我们可以通过依赖注入，让容器帮助我们解除接口与实现类之间的耦合性。但是，有时，我们需要依赖第三方库（大量依赖好吗...），需要实例化并使用第三方库中的相关类。这时，接口与实现类的耦合性需要其他方式来避免。

通常的做法是通过使用工厂方法(Factory Method)模式，提供一个工厂类来实例化具体的接口实现类，这样，主体对象只需要依赖工厂类，具体使用的实现类有变更的话，只是变更工厂类，而主体对象不需要做任何变动。代码清单4-31演示了这种做法。 

![1564908202825](D:\我的文档\JAVA\框架\Spring解密\images\工厂方法)

针对使用工厂方法模式实例化对象的方式， Spring的IoC容器同样提供了对应的集成支持。我们所要做的，只是将工厂类所返回的具体的接口实现类注入给主体对象（这里是Foo）。 

### 1.  静态工厂方法 (Static Factory Method)

​	假设某个第三方库发布了BarInterface ，为了向使用该接口的客户端对象屏蔽以后可能对BarInterface 实现类的变动，同时还提供了一个静态的工厂方法实现类StaticBarInterfaceFactory ，代码如下：

```java
public class StaticBarInterfaceFactory
{
	public static BarInterface getInstance(){
		return new BarInterfaceImpl();
	}
}
```

为了将该静态工厂方法类返回的实现注入Foo，我们使用以下方式进行配置（通过setter方法注入
方式为Foo注入BarInterface的实例）： 

```xml
<bean id="foo" class="...Foo">
<property name="barInterface">
<ref bean="bar"/>
</property>
</bean>
<bean id="bar" class="...StaticBarInterfaceFactory" factory-method="getInstance"/>
```

class指定静态方法工厂类，factory-method指定工厂方法名称，然后，容器调用该静态方法工厂类的指定工厂方法（getInstance），并返回方法调用后的解雇，即BarInterfaceImpl的实例，即方法调用后的结果，而不是静态工厂方法类StaticBarInterfaceFactory。我们可以实现自己的静态工厂方法类返回任意类型的对象实例，但工厂方法类的类型与工厂方法返回的类型没有必然的相同关系。

​	某些时候，有的工厂类的工厂方法可能需要参数来返回相应实例。而不一定非要像我们的getInstance()这样没有任何参数。对于这种情况，可以通过<constructor-arg>来指定工厂方法需要的参数，比如现在StaticBarInterfaceFactory需要其他依赖来返回某个BarInterface的实现，其定义可能如下：

![1564917805158](D:\我的文档\JAVA\框架\Spring解密\images\工厂方法1)

为了让包含方法参数的工厂方法能够预期返回相应的实现类实例，我们可以像代码清单4-32所演示的那样， 通过<constructor-arg> 为工厂方法传入相应的参数。

![1564917883892](D:\我的文档\JAVA\框架\Spring解密\images\工厂方法传参)

唯一需要注意的就是，针对静态工厂方法实现类的bean定义，使用<constructor-arg>传入的是工厂方法的参数，而不是静态工厂方法实现类的构造方法的参数（况且，静态工厂方法实现类也没有提供显示的构造方法。）

### 2. 非静态工厂方法（Instance Factory Method）

​	既然可以将静态工厂方法实现类的工厂方法调用结果作为bean注册到容器中，我们同样可以针对基于工厂类实例的工厂方法调用结果应用相同的功能，只不过，表达方式可能需要稍微变一下了。

​	现在为 BarInterface 提供非静态的工厂方法实现类，该类定义如下代码所示：

![1564918277825](D:\我的文档\JAVA\框架\Spring解密\images\非静态工厂)

因为工厂方法为非静态的，我们只能通过某个NonStaticBarInterfaceFactory 实例来调用该方法（哦，错了，是容器来调用），那么也就有了如下的配置内容：

![1564918535174](D:\我的文档\JAVA\框架\Spring解密\images\非静态工厂配置)

NonStaticBarInterfaceFactory 是作为正常的bean注册到容器的，而bar的定义则与静态工厂方法的定义有些不同（多了factory-bean ）。现在使用 factory-bean属性来指定工厂方法所在的工厂类实例，而不是通过class属性来指定工厂方法所在类的类型(确实，连class属性都没有了 )。指定工厂方法名则相同，都是通过factory-method属性进行的。

​	如果非静态工厂方法调用时也需要提供参数的话，处理方式是与静态的工厂方法相似的，都可以通过

<constructor-arg>	来指定方法调用参数

### 3.	FactoryBean(这个牛B)

​	FactoryBean是Spring容器提供的一种可以拓展容器对象实例化逻辑的接口，请不要将其与容器名称BeanFactory相混淆。 FactoryBean，其主语是 Bean，定语为 Factory，也就是说，它本身与其他注册到容器的对象一样，只是一个Bean而已，只不过，这种类型的Bean本身就是生产对象的工厂(Factory)

​	当某些对象的实例化过程过于烦琐，通过XML配置过于复杂，使我们宁愿使用Java代码来完成这个实例化过程的时候，或者，**某些第三方库不能直接注册到Spring容器的时候**，就可以使用org.springframework.beans.factory.FactoryBean接口，给出自己的对象实例化逻辑代码。当然，不使用FactoryBean，而像通常那样实现自定义的工厂方法类也是可以的。不过，FactoryBean可是Spring提供的对付这种情况的“制式装备”哦！

要实现并使用自己的 FactoryBean其实很简单，org.springframework.beans.factory.FactoryBean只定义了三个方法，如以下代码所示：



```java
public interface FactoryBean {
	Object getObject() throws Exception;
	Class getObjectType();
	boolean isSingleton();
}
```

getObject()方法会返回该 FactoryBean “生产” 的对象实例，我们需要实现该方法以给出自己的对象实例化逻辑；getObjectType() 方法仅返回getObject()方法所返回的对象的类型，如果预先无法确定，则返回 null；isSingleton()方法的返回结果表明，工厂方法( getObject() ) 所 “生产” 的对象是否要以 singleton形式存在于容器中。如果以 singleton形式存在，则返回true，否则返回false；

​	如果我们想每次得到的日期都是第二天，可以实现一个如代码清单4-33所示的FactoryBean 

![1564924177705](D:\我的文档\JAVA\框架\Spring解密\images\工厂bean)

很简单的实现，不是嘛？

要使用 NextDayDateFactoryBean ,只需要如下这样将其注册到容器即可：

![1564924374641](D:\我的文档\JAVA\框架\Spring解密\images\注册实例工厂Bean)

配置上看不出于平常的bean定义有何不同，不过，只有当我们看到 NextDayDateFactoryDsiplayer的定义的时候，才会知道FactoryBean的魔力到底在哪。NextDayDateFactoryDsiplayer的定义如下：

```java
public class NextDayDateDisplayer
{
private DateTime dateOfNextDay;
// 相应的setter方法
// ...
}
```

看到了吗？NextDayDateDisplayer所声明的依赖dateOfNextDay的类型为DateTime，而不是 NextDayDateFactoryBean。也就是说 FactoryBean类型的bean定义，通过正常的id引用，容器返回的是 FactoryBean所“生产”的对象类型，而非 FactoryBean本身。

如果一定要取得FactoryBean本身的话，可以通过在bean定义的id之前加前缀 & 来达到目的。代码清单4-34展示了获取FactoryBean本身与获取FactoryBean“生产”的对象之间的差别。 

![1565187930994](D:\我的文档\JAVA\框架\Spring解密\images\factoryBean)

## 7.偷梁换柱之术

​	在学习以下内容之前，先提一下有关bean的scope的使用“陷阱”，特别是 prototype在容器中的使用，以此引出本节将要介绍的Spring容器较为独特的功能特性：方法注入(Method Injection) 以及方法替换 (Method Replacement)。

我们知道，拥有prototype类型scope的bean，在请求方每次向容器请求该类型对象的时候，容器都会返回一个全新的该对象的实例。为了简化问题的叙述，我们直接将 FXNews系统中的 FXNewsBean定义注册到容器中，并将其scope设置为prototype。因为它是有状态的类型，每条新闻都应该是新的独立个体：同时，我们给出MockNewsPersister 类，使其实现 IFXNewsPersister 接口，以模拟注入 FXNewsBean 实例后的情况。这样，我们就有了代码4-35所展示的类声明和相关配置 。

![1565189061953](D:\我的文档\JAVA\框架\Spring解密\images\code)

![1565189077603](D:\我的文档\JAVA\框架\Spring解密\images\凑得)

![1565189826932](D:\我的文档\JAVA\框架\Spring解密\images\一厢情愿)

从输出看，对象实例是相同的，而这与我们的初衷是相悖的。因为每次调用persistNews都会调用getNewsBean()方法并返回一个FXNewsBean实例，而FXNewsBean实例是prototype类型的，因此每次不是应该输出不同的对象实例嘛？ 

好了，问题实际上不是出在FXNewsBean的scope类型是否是prototype的，而是出在实例的取得方式上面。虽然FXNewsBean拥有prototype类型的scope，但当容器将一个FXNewsBean的实例注入MockNewsPersister之后， MockNewsPersister就会一直持有这个FXNewsBean实例的引用。虽然每次输出都调用了getNewsBean()方法并返回了 FXNewsBean 的实例，但实际上每次返回的都是MockNewsPersister持有的容器第一次注入的实例。这就是问题之所在。换句话说，第一个实例注入后， MockNewsPersister再也没有重新向容器申请新的实例。所以，容器也不会重新为其注入新的FXNewsBean类型的实例。 

### 1.  方法注入

​	Spring容器提出了一种叫做方法注入（Method Injection）的方式，可以帮助我们解决上述问题。我们所要做的很简单，只要让 getNewsBean 方法声明符合规定的格式，并在配置文件中通知容器，当该方法被调用的时候，每次返回指定类型的对象实例即可。方法声明需要符合规定定义如下：

![1565190065634](D:\我的文档\JAVA\框架\Spring解密\images\抽象了点)

也就是说，该方法必须能够被子类实现或者覆写，因为容器会为我们要进行方法注入的对象使用Cglib 动态生成一个子类实现，从而替代当前对象。既然我们的 getNewsBean() 方法已经满足以上方法声明格式，剩下唯一要做的就是配置该类，配置内容如下：

![1565190264788](D:\我的文档\JAVA\框架\Spring解密\images\奇奇怪怪)

![1565190392340](D:\我的文档\JAVA\框架\Spring解密\images\一脸懵逼)

![1565190535061](D:\我的文档\JAVA\框架\Spring解密\images\注意)

### 2.殊途同归

除了使用方法注入来达到“每次调用都让容器返回新的对象实例”的目的，还可以使用其他方式达到相同的目的。下面给出其他两种解决类似问题的方法，供读者参考。 

#### 1.使用BeanFactoryAware接口 

我们知道，即使没有方法注入， 只要在实现getNewsBean()方法的时候，能够保证每次调用BeanFactory的getBean("newsBean")，就同样可以每次都取得新的FXNewsBean对象实例。现在，我们唯一需要的，就是让MockNewsPersister拥有一个BeanFactory的引用 。

Spring框架提供了一个BeanFactoryAware接口，容器在实例化实现了该接口的bean定义的过程中，会自动将容器本身注入该bean。这样， 该bean就持有了它所处的BeanFactory的引用。BeanFactoryAware的定义如下代码所示： 

![1565190932712](D:\我的文档\JAVA\框架\Spring解密\images\try)

![1565190949299](D:\我的文档\JAVA\框架\Spring解密\images\class)

![1565190999397](D:\我的文档\JAVA\框架\Spring解密\images\修)

实际上，方法注入动态生成的子类，完成的是与以上类似的逻辑，只不过实现细节上不同而已 

#### 2.使用 ObjectFactoryCreatingFactoryBean

ObjectFactoryCreatingFactoryBean 是 Spring提供的一个 FactoryBean实现，它 返 回 一 个ObjectFactory实例。从ObjectFactoryCreatingFactoryBean返回的这个ObjectFactory实例可以为 我 们 返 回 容 器 管 理 的 相 关 对 象 。 实 际 上 ， ObjectFactoryCreatingFactoryBean 实 现 了BeanFactoryAware接口，它返回的ObjectFactory实例只是特定于与Spring容器进行交互的一个实现而已。使用它的好处就是，隔离了客户端对象对BeanFactory的直接引用。 

![1565265610618](D:\我的文档\JAVA\框架\Spring解密\images\讲道理好奇怪)

![1565265809811](D:\我的文档\JAVA\框架\Spring解密\images\类名长到想吐)

![1565265868802](D:\我的文档\JAVA\框架\Spring解密\images\错综复杂)

![1565265965405](D:\我的文档\JAVA\框架\Spring解密\images\难懂)

### 3. 方法替换

​	与方法注入只是通过相应方法为主体对象注入依赖对象不同，方法替换更多体现在方法的实现层面上，它可以灵活替换或者说以新的方法实现覆盖掉原来某个方法的实现逻辑。基本上可以认为，方法替换可以帮助我们实现简单的方法拦截功能。要知道，我们现在可是在不知不觉中迈上了AOP的大道哦！

假设某天我看FXNewsProvider不爽，想替换掉它的getAndPersistNews方法默认逻辑，这时，我就可以用方法替换将它的原有逻辑给替换掉。 

@@小心@@ 这里只是为了演示方法替换（ Method Replacement）的功能，不要真的这么做。要使用
也要用在好的地方，对吧？ 假设我们只是简单记录日志，打印简单信息，那么就可以给出一个类似代码清单4-39所示的MethodReplacer实现类。 

![1565266277181](D:\我的文档\JAVA\框架\Spring解密\images\方法替换)

有了要替换的逻辑之后，我们就可以把这个逻辑通过<replaced-method>配置到FXNewsProvider的bean定义中，使其生效，配置内容如代码清单4-40所示。 	

![1565266394684](D:\我的文档\JAVA\框架\Spring解密\images\又丑又长的配置文件)

![1565266605610](D:\我的文档\JAVA\框架\Spring解密\images\输出)

最后需要强调的是，这种方式刚引入的时候执行效率不是很高，当你充分了解并应用SpringAOP之后，我想你也不会再回头求助这个特色功能。不过，怎么说这也是一个选择，场景合适的话，为何不用呢？ 哦，如果要替换的方法存在参数，或者对象存在多个重载的方法，可以在<replaced-method>内部通过<arg-type>明确指定将要替换的方法参数类型。祝“替换”愉快！  恶心的一。。



## 4.容器背后的秘密

子曰：学而不思则罔。除了了解Spring的IoC容器如何使用，了解Spring的IoC容器都提供了哪些功能，我们也应该想一下， Spring的IoC容器内部到底是如何来实现这些的呢？虽然我们不太可能“重新发明轮子”，但是，如图4-7（该图摘自Spring官方参考文档）所示的那样，只告诉你“Magic HappensHere”，你是否就能心满意足呢？ 

![1565266777311](D:\我的文档\JAVA\框架\Spring解密\images\好一个magic)

好，如果你的答案是“不”（我当然认为你说的是“不想一直被蒙在鼓里”），那么就随我一起来探索一下这个“黑匣子”里面到底有些什么……

Spring的IoC容器所起的作用，就像图4-7所展示的那样，它会以某种方式加载ConfigurationMetadata（通常也就是XML格式的配置信息），然后根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统 。

Spring的IoC容器实现以上功能的过程，基本上可以按照类似的流程划分为两个阶段，即**容器启动阶段**和**Bean实例化阶段**，如图4-8所示。 

Spring的IoC容器在实现的时候，充分运用了这两个实现阶段的不同特点，在每个阶段都加入了相应的容器扩展点，以便我们可以根据具体场景的需要加入自定义的扩展逻辑。

![1565266967114](D:\我的文档\JAVA\框架\Spring解密\images\两个阶段)

### 4.1容器启动阶段

容器启动伊始，首先会通过某种途径加载Configuration MetaData。除了代码方式比较直接，在大部分情况下，容器需要依赖某些工具类（BeanDefinitionReader）对加载的Configuration MetaData 进行解析和分析，并将分析后的信息编组为相应的BeanDefinition，最后把这些保存了bean定义必要信息的BeanDefinition，注册到相应的BeanDefinitionRegistry，这样容器启动工作就完成了。图4-9演示了这个阶段的主要工作。 

![1565267136127](D:\我的文档\JAVA\框架\Spring解密\images\容器启动)

总地来说，该阶段所做的工作可以认为是准备性，重点更加侧重于对象管理信息的收集。当然，一些验证性或者辅助性的工作也可以在这个阶段完成。

### 4.2 Bean的实例化过程

​	经过第一阶段，现在所有的bean定义信息都通过 BeanDefinition的方式注册到了BeanDefinitionRegistry中。当某个请求方通过容器的getBean方法明确地请求某个对象，或者因依赖关系容器需要隐式地调用getBean方法时，就会触发第二阶段的活动。

该阶段，容器会首先检查所请求的对象之前是否已经初始化。如果没有，则会根据如果没有，则会根据注册的BeanDefinition所提供的信息实例化被请求对象，并为其注入依赖。如果该对象实现了某些回调接口，也会根据回调接口的要求来装配它。当该对象装配完毕之后，容器会立即将其返回请求方使用。如果说第一阶段只是根据图纸装配生产线的话，那么第二阶段就是使用装配好的生产线来生产具体的产品了。 

#### 4.2.2插手容器的启动

Spring提供了一种叫做BeanFactoryPostProcessor的容器扩展机制。该机制允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做相应的修改。这就相当于在容器实现的第一阶段最后加入一道工序，让我们对最终的BeanDefinition做一些额外的操作，比如修改其中bean定义的某些属性，为bean定义增加其他信息等。 

如果要自定义实现BeanFactoryPostProcessor，通常我们需要实现org.springframework.beans.factory.config.BeanFactoryPostProcessor接口。同时，因为一个容器可能拥有多个BeanFactoryPostProcessor，这个时候可能需要实现类同时实现Spring的org.springframework.core.Ordered接口，以保证各个BeanFactoryPostProcessor可以按照预先设定的顺序执行（如果顺序紧要的话）。但是，因为Spring已经提供了几个现成的BeanFactoryPostProcessor实现类，所以，大多时候，我们很少自己去实现某个BeanFactoryPostProcessor。其中，org.springframework.beans.factory.config.PropertyPlaceholderConfigurer和org.springframework.beans.factory.
config.Property OverrideConfigurer是两个比较常用的BeanFactoryPostProcessor

另外，为了处理配置文件中的数据类型与真正的业务对象所定义的数据类型转换， Spring还允许我们通过
org.springframework.beans.factory.config.CustomEditorConfigurer来注册自定义的PropertyEditor以补助容器中默认的PropertyEditor。可以参考BeanFactoryPostProcessor的Javadoc来了解更多其实现子类的情况。 

我 们 可 以 通 过 两 种 方 式 来 应 用 BeanFactoryPostProcessor ， 分 别 针 对 基 本 的 IoC 容 器BeanFactory和较为先进的容器ApplicationContext。 

对于BeanFactory来说，我们需要用手动方式应用所有的BeanFactoryPostProcessor，代码清单4-41演示了具体的做法。 

![1565269125814](D:\我的文档\JAVA\框架\Spring解密\images\最后一步)