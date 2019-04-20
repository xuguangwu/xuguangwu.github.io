---
title: kubernates学习笔记
categories:
 - linux
tags: linux
---

Kubernetes

在Docker之前的日子里，我们有物理的或者虚拟的机器。每一个机器（物理的或者虚拟的）经常只负责一个功能。而容器化后我们面对的是一个资源池（内存/cpu/网络），这是一个我们集群所有运行的机器的资源总量。容器编排引擎后，我们对资源的理解应该在集群维度进行考量，而不是在考虑单机的利用率。

我们对容器编排系统(Orchestration)的期望是：

基本发布自动化功能：

	编排过程包含分配机器，拉取镜像，启动／停止／更新容器，存活监控，容器数量扩展和收缩

声明式定义服务栈：

	提供一种机制，可以用配置文件来声明服务的网络端口，镜像及版本，在需要的时候通过配置可再现的创建出一整套服务。

服务发现：

	提供DNS和负载均衡，一个容器启动之后，需要其他服务能够访问到它，一个容器终止运行之后，需要保证流量不会再导向它。

状态检查：

	需要持续监控系统是否符合配置中声明的状态，比如一台宿主机挂了，需要把上面运行的容器在其他健康的节点上启动起来，如果一个容器挂了，需要把它重新启动。

什么是Kubernetes

Kubernetes是Google开源的容器集群管理系统，其提供应用部署、维护、 扩展机制等功能，利用Kubernetes能方便地管理跨机器运行容器化的应用，其主要功能如下：

	1. 使用Docker对应用程序包装(package)、实例化(instantiate)、运行(run)。
	2. 以集群的方式运行、管理跨机器的容器。
	3. 解决Docker跨机器容器之间的通讯问题。
	4. Kubernetes的自我修复机制使得容器集群总是运行在用户期望的状态。

Kubernetes架构

主要包括kubecfg、Master API Server、Kubelet、Minion(Host)以及Proxy。

一个Kubernetes集群包括**至少一个主节点和多个计算节点**。

Master：暴露应用程序接口（API），调度部署和管理整个集群。
Master定义了Kubernetes 集群Master/API Server的主要声明，包括Pod Registry、Controller Registry、Service Registry、Endpoint Registry、Minion Registry、Binding Registry、RESTStorage以及Client, 是client(Kubecfg)调用Kubernetes API，管理Kubernetes主要构件Pods、Services、Minions、容器的入口。Master由API Server、Scheduler以及Registry等组成。
￼
从下图可知Master的工作流主要分以下步骤：
	1. Kubecfg将特定的请求，比如创建Pod，发送给Kubernetes Client。
	2. Kubernetes Client将请求发送给API server。
	3. API Server根据请求的类型，比如创建Pod时storage类型是pods，然后依此选择何种REST Storage API对请求作出处理。
	4. REST Storage API对的请求作相应的处理。
	5. 将处理的结果存入高可用键值存储系统Etcd中。
	6. 在API Server响应Kubecfg的请求后，Scheduler会根据Kubernetes Client获取集群中运行Pod及Minion信息。
	7. 依据从Kubernetes Client获取的信息，Scheduler将未分发的Pod分发到可用的Minion节点上。
￼
**Minion Registry**：Minion Registry负责跟踪Kubernetes 集群中有多少Minion(Host)。Kubernetes封装Minion Registry成实现Kubernetes API Server的RESTful API接口REST，通过这些API，我们可以对Minion Registry做Create、Get、List、Delete操作，由于Minon只能被创建或删除，所以不支持Update操作，并把Minion的相关配置信息存储到etcd。除此之外，Scheduler算法根据Minion的资源容量来确定是否将新建Pod分发到该Minion节点。

**Pod Registry**：Pod Registry负责跟踪Kubernetes集群中有多少Pod在运行，以及这些Pod跟Minion是如何的映射关系。将Pod Registry和Cloud Provider信息及其他相关信息封装成实现Kubernetes API Server的RESTful API接口REST。通过这些API，我们可以对Pod进行Create、Get、List、Update、Delete操作，并将Pod的信息存储到etcd中，而且可以通过Watch接口监视Pod的变化情况，比如一个Pod被新建、删除或者更新。

**Service Registry**：Service Registry负责跟踪Kubernetes集群中运行的所有服务。根据提供的Cloud Provider及Minion Registry信息把Service Registry封装成实现Kubernetes API Server需要的RESTful API接口REST。利用这些接口，我们可以对Service进行Create、Get、List、Update、Delete操作，以及监视Service变化情况的watch操作，并把Service信息存储到etcd。

**Controller Registry**：Controller Registry负责跟踪Kubernetes集群中所有的Replication Controller，Replication Controller维护着指定数量的pod 副本(replicas)拷贝，如果其中的一个容器死掉，Replication Controller会自动启动一个新的容器，如果死掉的容器恢复，其会杀死多出的容器以保证指定的拷贝不变。通过封装Controller Registry为实现Kubernetes API Server的RESTful API接口REST， 利用这些接口，我们可以对Replication Controller进行Create、Get、List、Update、Delete操作，以及监视Replication Controller变化情况的watch操作，并把Replication Controller信息存储到etcd

**Endpoints Registry**：Endpoints Registry负责收集Service的endpoint，比如Name："mysql"，Endpoints: ["10.10.1.1:1909"，"10.10.2.2:8834"]，同Pod Registry，Controller Registry也实现了Kubernetes API Server的RESTful API接口，可以做Create、Get、List、Update、Delete以及watch操作。

**Binding Registry**：Binding包括一个需要绑定Pod的ID和Pod被绑定的Host，Scheduler写Binding Registry后，需绑定的Pod被绑定到一个host。Binding Registry也实现了Kubernetes API Server的RESTful API接口，但Binding Registry是一个write-only对象，所有只有Create操作可以使用， 否则会引起错误。

**Scheduler**：Scheduler收集和分析当前Kubernetes集群中所有Minion节点的资源(内存、CPU)负载情况，然后依此分发新建的Pod到Kubernetes集群中可用的节点。由于一旦Minion节点的资源被分配给Pod，那这些资源就不能再分配给其他Pod， 除非这些Pod被删除或者退出， 因此，Kubernetes需要分析集群中所有Minion的资源使用情况，保证分发的工作负载不会超出当前该Minion节点的可用资源范围。
(Scheduler做以下工作：
1) 实时监测Kubernetes集群中未分发的Pod。
2) 实时监测Kubernetes集群中所有运行的Pod，Scheduler需要根据这些Pod的资源状况安全地将未分发的Pod分发到指定的Minion节点上。
3) Scheduler也监测Minion节点信息，由于会频繁查找Minion节点，Scheduler会缓存一份最新的信息在本地。
4) 最后，Scheduler在分发Pod到指定的Minion节点后，会把Pod相关的信息Binding写回API Server。)

下图为Kubernetes详细构件
￼
**Kubelet**：Kubelet是Kubernetes集群中每个Minion和Master API Server的连接点，Kubelet运行在每个Minion上，是Master API Server和Minion之间的桥梁，接收Master API Server分配给它的commands和work，与持久性键值存储etcd、file、server和http进行交互，读取配置信息。Kubelet的主要工作是管理Pod和容器的生命周期，其包括Docker Client、Root Directory、Pod Workers、Etcd Client、Cadvisor Client以及Health Checker组件。
工作流程：
	1. 通过Worker给Pod异步运行特定的Action。
	2. 设置容器的环境变量。
	3. 给容器绑定Volume。
	4. 给容器绑定Port。
	5. 根据指定的Pod运行一个单一容器。
	6. 杀死容器。
	7. 给指定的Pod创建network 容器。
	8. 删除Pod的所有容器。
	9. 同步Pod的状态。
	10. 从Cadvisor获取container info、 pod info、root info、machine info。
	11. 检测Pod的容器健康状态信息。
	12. 在容器中运行命令。

**Proxy**：Proxy是为了解决外部网络能够访问跨机器集群中容器提供的应用服务而设计的，Proxy服务也运行在每个Minion上。Proxy提供TCP/UDP sockets的proxy，每创建一种Service，Proxy主要从etcd获取Services和Endpoints的配置信息，或者也可以从file获取，然后根据配置信息在Minion上启动一个Proxy的进程并监听相应的服务端口，当外部请求发生时，Proxy会根据Load Balancer将请求分发到后端正确的容器处理。

**Node**：每个节点都有一个容器运行时，如Docker或Rocket，以及与主节点通信的agent 。节点还运行日志、监视、服务发现和可选附加组件的其他组件。节点是Kubernetes集群的工作负荷节点，他们给应用提供计算，网络，存储资源。节点可能是云上的虚拟机或数据中心的裸机。

Kubernetes对象（如pods，replica sets和services。）被提交给主节点，基于需求定义和资源可用性，主节调度到指定节点的pod。该节点从容器镜像仓库中提取镜像并协调本地容器运行时启动容器。

**Pods**：kubernetes的基本操作单元，把相关的一个或多个容器构成一个Pod，通常Pod里的容器运行相同的应用。Pod包含的容器运行在同一个Minion(Host)上，看作一个统一管理单元，共享相同的volumes和network namespace/IP和Port空间。(A Pod encapsulates(包含) an application container, storage resources, a unique network IP, and options  that govern how the container(s) should run.Restarting a container in a Pod should not be confused with restarting the Pod. The Pod itself does not run, but is an environment the containers run in and persists until it is deleted(pod本身是不运行的，不存在重启pod一说).
**Services**：Kubernetes的基本操作单元，是真实应用服务的抽象，每一个服务后面都有很多对应的容器来支持，通过Proxy的port和服务selector决定服务请求传递给后端提供服务的容器，对外表现为一个单一访问接口，外部不需要了解后端如何运行，这给扩展或维护后端带来很大的好处。
**Replication Controllers** (ReplicaSets is the next-generation ReplicationController)：确保任何时候Kubernetes集群中有指定数量的pod副本(replicas)在运行， 如果少于指定数量的pod副本(replicas)，Replication Controller会启动新的Container，反之会杀死多余的以保证数量不变。通过始终维护一组预定义的pods，提供所需的规模和可用性。单个pod或replica set能通过services暴露给内部或外部消费者。services通过特定的标准将一组Pods关联来实现对pods的发现。通过键值对的标签和选择器来关联pods，任何与标签匹配的新pod将自动被服务发现。（A Controller can create and manage multiple Pods for you, handling replication and rollout and providing self-healing capabilities at cluster scope. ）Replication Controller使用预先定义的pod模板创建pods，一旦创建成功，pod 模板和创建的pods没有任何关联，可以修改pod 模板而不会对已创建pods有任何影响，也可以直接更新通过Replication Controller创建的pods。对于利用pod 模板创建的pods，Replication Controller根据label selector来关联，通过修改pods的label可以删除对应的pods。
	(Replication Controller主要有如下用法：
	1) Rescheduling
	如上所述，Replication Controller会确保Kubernetes集群中指定的pod副本(replicas)在运行， 即使在节点出错时。
	2) Scaling
	通过修改Replication Controller的副本(replicas)数量来水平扩展或者缩小运行的pods。
	3) Rolling updates
	Replication Controller的设计原则使得可以一个一个地替换pods来rolling updates服务。
	4) Multiple release tracks
	如果需要在系统中运行multiple release的服务，Replication Controller使用labels来区分multiple release tracks。)
Labels：Labels是用于区分Pod、Service、Replication Controller的key/value键值对，Pod、Service、 Replication Controller可以有多个label，但是每个label的key只能对应一个value。Labels是Service和Replication Controller运行的基础，为了将访问Service的请求转发给后端提供服务的多个容器，正是通过标识容器的labels来选择正确的容器。同样，Replication Controller也使用labels来管理通过pod 模板创建的一组容器，这样Replication Controller可以更加容易，方便地管理多个容器，无论有多少容器。

**Etcd**：etcd是CoreOS开源，分布式健值数据库，做为kubernetes集群所有组件的（single source of truth ） SSOT。主节点需要etcd获得节点状态，pods状态和容器状态参数信息。(etcd is a highly-available key value store for shared configuration and service discovery.which Kubernetes uses for persistent storage of all of its REST API objects).



Kubernetes框架通过创建应用和底层架构的抽象层，达到模块化和可扩展。 
￼


￼




Although Pods each have a unique IP address, those IPs are not exposed outside the cluster without a Service.

Services can be exposed in different ways by specifying a type in the ServiceSpec:
* ClusterIP (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
* NodePort - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using :. Superset of ClusterIP.
* LoadBalancer - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
* ExternalName - Exposes the Service using an arbitrary name (specified by externalName in the spec) by returning a CNAME record with the name. No proxy is used. This type requires v1.7 or higher of kube-dns


Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones. The new Pods will be scheduled on Nodes with available resources.

￼

![kubernetes architecture](https://raw.githubusercontent.com/xuguangwu/blog/master/_posts/images/kubernetes_architecture.png)






















