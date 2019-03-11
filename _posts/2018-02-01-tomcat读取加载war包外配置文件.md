---
title: tomcat读取加载war包外配置文件
categories:
 - Java
tags: 
 - Java
---

在生产环境中，我们的配置文件是不会配置死了再war包中，所以需要定制
tomcat启动配置来读取war包外指定目录下的配置文件。

通过阅读catalina.sh可以查看到会取读取setclasspath.sh,
setclasspath.sh主要是去修改CLASSPATH环境变量，
那么只要在setclasspath.sh末，配置上我们的配置文件路径，
如：export CLASSPATH=$CLASSPATH:/Users/xxx/software/tomcat_config
在tomcat启动的时候就会扫描配置的classpath所指定的路径。












