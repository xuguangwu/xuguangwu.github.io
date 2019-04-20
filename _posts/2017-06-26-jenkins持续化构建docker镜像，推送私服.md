---
title: jenkins自动化构建docker镜像,kubernetes滚动升级部署
categories:
 - linux
tags: linux
---

在开发联调过程中，频繁打包到测服正服，再去手动部署是非常麻烦且浪费精力的事情。
于是我就想尝试自动化打包，自动化部署升级。
于是就有了jenkins，kubernetes的学习。
以下记录我的配置过程。

1.Install jerkins

  ● 配置jdk和maven，配置jenkins的全局工具
![jenkins_global_config](https://github.com/xuguangwu/blog/blob/master/_posts/images/web_hook.png?raw=true)
  
  ● 部署jenkins，建议使用jenkins.war，使用-httpPort=8081执行访问端口
  
  ● 下载插件，选择default 的，方便。后面按需添加，
  由于我需要打包maven工程，版本控制使用的是gitlab,所以搜索相关插件，
  这里用到了Maven Integration plugin，
  以及gitlab的插件（下载所有gitlab开头的插件就对了）,如果想实现自动部署还需要安装Deploy to container Plugin。
  
  ● 再配置gitlab认证。官方提供了多种方式，
  gitlab  api  token,user-passwd以及SSH userName  with  private  key的方式。
  这里我选用的是最后一种，
  在jenkins部署的机器上生成ssh-key,ssh-keygen -t rsa  -C "your_email@example.com"，
  将生成的~/.ssh/id_rsa.pub中的内容配置到gitlab的ssh  keys中。
  再将其配置在jenkins中
![source_code_manage](https://github.com/xuguangwu/blog/blob/master/_posts/images/source_code_manage.png?raw=true)
  
  ● 新建free-style工程，
  
  ● 配置源码管理的时候注意，仅需要配置源码地址，
  采用private  key的认证方式的时候Credentials不用选择，
  其余方式需要指定你所使用的用户或者api token。
  
  ● 构建触发器，选择你需要的触发方式，我采用的是gitlab 的 webhook，
  当发生push操作的时候触犯自动构建，也可以使用定时器，用cron表达式来约束自动构建时间。
  注意，在选择gitlab  trigger的时候jenkins会自动生成一个回调url ,
  如GitLab CI Service URL: http://{host}:8005/project/${project_name}，
  需要将该回调url配置到gitlab中。见下图
![source_code_manage](https://github.com/xuguangwu/blog/blob/master/_posts/images/chufaqi.png?raw=true)
![source_code_manage](https://github.com/xuguangwu/blog/blob/master/_posts/images/web_hook.png?raw=true)
  
  ● 最后就是我们的构建脚本。

我设计的自动化发布过程是这样的：gitlab自动触发jenkins构建，
在jenkins中打包成jar包再用shell脚本构建成docker镜像，
最后推向私服。这一切都是为了方便后面在kubernetes中做滚动升级，直接配置新的镜像tag就可以了。

```
#!/bin/bash
REGISTRY=120.76.XX.XX   #私服地址
JENKINS_ROOT=/mnt/jenkins
PROJECT_NAME=XXX
export JENKINS_HOME=$JENKINS_ROOT/jenkins_home
DIR="${JENKINS_HOME}/workspace/${PROJECT_NAME}"
cd ${DIR}
mvn -DskipTests clean package  #构建jar包
tag=$(date +%Y%m%d)
docker build -t ${PROJECT_NAME}:${tag} -f Dockerfile .
docker tag ${PROJECT_NAME}:${tag} ${REGISTRY}/library/${PROJECT_NAME}:${tag}
docker login -u=admin -p=Harbor12345 ${REGISTRY}  #登陆私服
docker push ${REGISTRY}/library/michun:${tag} #推送私服准备发布测试
```

2.在harbor中查看自动构建镜像
![harbor](https://github.com/xuguangwu/blog/blob/master/_posts/images/harbor.png?raw=true)

3.在kubernetes中配置最新镜像版本，滚动升级

4.额外补充搭建docker  registry过程中遇到的问题















