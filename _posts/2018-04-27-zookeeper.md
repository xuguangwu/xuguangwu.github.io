
## 指定服务器端口
./zkCli.sh -server 30.4.162.108:48312

因为框架中需要用到zookeeper做文件分发，所以要调整znode的限制。
默认的是1M，根据需要，可以在zkServer.sh中修改配置，如下
````
nohup "$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Djute.maxbuffer=10240000" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}"

````
在环境变量中添加**-Djute.maxbuffer=10240000**。
检测是否配置成功，查看进程中是否带了该参数即可。
ps -ef | grep zookeeper

限制znode的大小主要是因为zookeeper的全量数据都在内存中，znode过大会消耗过多的zk服务器内存。

















