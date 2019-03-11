---
title: kubernetes集群搭建问题汇总
categories:
 - DevOps
tags: DevOps
---

将我在搭建kubernetes集群过程中遇到的一些比较棘手的问题记录下来。

1.docker.service failed.

journalctl -xn 查看内核日志

Error starting daemon: error initializing graphdriver: driver not supported
[graphdriver] prior storage driver devicemapper failed: driver not supported
devmapper: Udev sync is not supported. This will lead to data loss and unexpected behavior.

解决方案:

Answer: I managed to resolve this on my side **by deleting /var/lib/docker and restarting the docker service. **
I'm really not sure what caused it and it may be due to packaging issue in my case.
**Note: Removing /var/lib/docker will delete all your downloaded images!**

尤其注意最后一句，会将本地的镜像全部删除，因为需要最好备份工作，将镜像导出，再docker服务重新启动后再load回来。