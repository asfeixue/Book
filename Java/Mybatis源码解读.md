Mybatis源码解读.md

##1.日志
logging package下的类图如下：
![日志类图](../static/logClassDiagram.png)

作为基础框架，本身的日志独立于任意日志框架，避免强绑定带来的依赖传递，同时又尽可能支持现有的任意日志框架，对具体使用的业务环境更友好。

mybatis按照如下顺序尝试加载具体日志框架：
    
    slf4j》commons-logging》log4j2》log4j》jdk-logging》no-log（丢弃所有日志）

从这里可以看出来，mybatis并未支持logback，如果日志用logback，最好还是通过slf4j来驱动。

##2.缓存
###BlockingCache
为每一个cache的数据项，映射一个ReentrantLock。然后在获取指定key的数据时，阻塞并等待对应的锁。阻塞可以设定一定的超时时间，也可以持续等待。