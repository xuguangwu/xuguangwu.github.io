---
title: dubbo中的一致性hash
categories:
 - dubbo
tags: 一致性hash
---

dubbo中提供了四种负载策略，本篇主要讲解一下ConsistentHashLoadBalance。

dubbo的配置文件中可以配置hash.nodes和hash.arguments这样的两个参数，
来配置虚拟节点数以及hash求key的参数。结合代码来看
````
ConsistentHashSelector(List<Invoker<T>> invokers, String methodName, int identityHashCode) {
            this.virtualInvokers = new TreeMap<Long, Invoker<T>>();
            this.identityHashCode = identityHashCode;
            URL url = invokers.get(0).getUrl();
            //默认虚拟节点数160个
            this.replicaNumber = url.getMethodParameter(methodName, "hash.nodes", 160);
            //对url中的第一个或几个参数进行hash得到索引，默认取第一个参数
            String[] index = Constants.COMMA_SPLIT_PATTERN.split(url.getMethodParameter(methodName, "hash.arguments", "0"));
            argumentIndex = new int[index.length];
            for (int i = 0; i < index.length; i++) {
                argumentIndex[i] = Integer.parseInt(index[i]);
            }
            for (Invoker<T> invoker : invokers) {
                String address = invoker.getUrl().getAddress();
                //
                for (int i = 0; i < replicaNumber / 4; i++) {
                    byte[] digest = md5(address + i);
                    for (int h = 0; h < 4; h++) {
                        long m = hash(digest, h);
                        virtualInvokers.put(m, invoker);
                    }
                }
            }
        }

````
