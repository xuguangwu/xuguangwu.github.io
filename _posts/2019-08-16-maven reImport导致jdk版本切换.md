---
title: maven reImport导致jdk版本切换
categories:
 - Java
---

在maven的setting.xml文件中添加配置
````
<profiles>  
    <profile>  
         <id>jdk-1.8</id>  
         <activation>  
             <activeByDefault>true</activeByDefault>  
             <jdk>1.8</jdk>  
          </activation>  
          <properties>  
              <maven.compiler.source>1.8</maven.compiler.source>  
              <maven.compiler.target>1.8</maven.compiler.target>  
              <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>  
          </properties>  
   </profile>  
</profiles> 
````



