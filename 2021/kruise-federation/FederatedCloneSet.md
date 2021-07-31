## FederatedCloneSet思路

首先得明确 FederatedCloneSet 的主要功能：任务分发至集群的 cloneset

看到这个目标，第一个反应便是直接分发配置文件给各集群的 cloneset。此处的配置文件包含了副本数与 cloneset 的增强功能配置。支持使用 clusterName ，clusterLabel筛选不同的集群采用不同的配置方案，未被指定的集群使用同一默认方案。

理想总是美好的。考虑可行性的时候总会遇到问题。因为放到实际情况中你会发现有些东西跟你想的完全不一样。

### 原生 FederatedDeployment 实现

在阅读相关代码之后，我大致了解了其实现过程：获取集群信息和配置信息进行权重计算，然后在原 fed(map) 的基础上插入新的 map （？？？）

map[string]int64

就是这么个东西。我猜()

上面的 map 是一个 replica 的 kubernetesDeployment (KD) 的配置文件的抽象。将新的 map 插入后，某样东西检测到原来的 map 发生了变化（因为真的只是做了插入操作，没发现其触发啥新函数），就会让各 KD 根据新的 map 来创建 pod。

### 结合实际的最终思路

如果上面对于原生实现的猜想是正确的。最终要实现就是在分配到特定集群的 map 中加入特定的键值对。在原算法的基础上改进（如果某 cluster 已经溢出则将未能分配的集群优先分配到配置文件相同的 cluster 上），增加 label selector 。
