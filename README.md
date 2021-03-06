# dubboc

## 为什么要做这个项目
dubbo作为一款优秀的分布式服务框架（SOA），已被很多互联网公司使用，但是dubbo在跨语言方面一直缺乏相应的支持，
虽然dubbox支持rest调用，在一定程度上解决了跨语言的问题，但是rest也存在诸多缺陷，比如http的性能问题、安全性
问题以及http有限动词无法应对复杂场景等。   
我所在的互联网公司虽然没有直接使用dubbo/dubbox，但是自研的分布式服务框架却是深度借鉴、精简之后的dubbo，作为
刚毕业的应届生，我有幸负责了这款分布式服务框架的跨语言调用的设计和实现。在完成了公司的项目之后，我有了开源的想法
，但是项目在设计之初并没有考虑到架构设计的合理性和扩展性，因此和dubbo架构相比简直是一团乱麻，为此我决定模仿dubbo
架构重写这个项目，这个决定诞生之后，我才意识到这个决定所带来的巨大挑战，虽然之前对dubbo的源码已经有了初步了解，但
是dubbo巨大的代码量、抽象的设计模式、java与c++固有的语言区别等问题，让我在将java语言写的dubbo"翻译"为c++语言写
的dubboc过程中面临了诸多挑战。

## 挑战
如果要我说java与c++在语言层面的最大区别，那就是c++语言没有内置反射特性，这导致在java中很容易使用的功能在c++中无法
实现，比如dubbo中最重要的设计思想：微内核+插件式，这个特性严重依赖SPI特性，而SPI特性有严重依赖反射功能。另一个c++
中没有的特性就是java中的字节码生成技术，这直接影响到c++无法使用动态代理技术，而dubbo中动态代理、字节码生成是一个很
重要的环节。为此，这些语言级别的却别，都需要寻找相应的解决方案甚至是让步。


## 解决方案
1、关于微内核+插件式实现方案：通过实现一个类型工厂（IocContainer），让所有要被反射的类在进程初始化时注册到类型工厂
中（注册类型名、构造器），然后在运行时便可以使用类型名称反射获取这个类的实例，这虽然并不是一个完整的反射实现方案，但是
可以满足一个插件的动态注册和动态获取功能。为此在IocContainer的基础上，我进一步封装了ExtensionLoader，负责插件的查找、
加载，以及对adaptive和wrapper的处理等。  

2、关于c++无法动态代理、字节码生成的解决方案：dubbo中第一处用到的动态代理的地方就是用于封装调用细节的consumer端和
provider端的proxy，这里的解决方案是，dubboc中使用泛化调用和泛化实现的方式，所有的proxy本质都是一样的，那就是GenericService
接口，因此这个proxy可以提前固话在代码中，无需动态生成；dubbo中第二处需要动态生成代码的地方是，dubbo中的ExtensionLoader在
自动装配（DI）扩展点实现时，并不是直接装载具体的扩展点实现，而是装载了一个adaptive实现，这个adaptive实现就是根据扩展点接口
动态生成的，它的功能就是运行时根据url中的配置参数动态选择调用哪一个具体扩展实现，由于dubbo中的扩展点是固定的，因此这里的adpative
也可以提前写好并固话在代码中。


## 暂定功能

1、支持c++泛化调用和c++泛化服务暴露。  

2、采用微内核、插件式设计模式。  
  
3、框架强依赖folly、stl。  
   
4、remoting层提供基于wangle标准实现,基于hessian序列化。  
  
5、protocol层提供兼容dubbo的标准协议。  

6、monitor层提供dubbo标准监控实现。  

7、cluster层提供dubbo所有集群策略。  

8、registry层提供基于zookeeper标准实现。  
 
9、config层提供基于api和xml两种配置方式(xml用于模拟spring配置方式)。  

10、admin采用react+springMVC实现，完全实现dubbo admin现有所有功能。  
  
 
## 注意
项目正在重写中，我会加快进度，但是由于工作量巨大，如果你有任何想法，欢迎与我联系。
