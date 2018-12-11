这个项目是关于Kuberentes集群的一些基础配置，如使用Ceph存储类（rbd和cephfs），集群日志管理，集群监控等。

要使用这些配置，需要有一个可用的Kubernetes和Ceph集群。如果还没有，可以参照 [部署Ceph+Kubenetes容器运行集群](https://bbs.forensix.cn/topic/177/) 进行部署。

本项目的内容是在学习Kubernetes过程中的总结，所有配置都经过实践，踩过不少坑。基于的Kubernetes版本为1.11.3，Ceph版本为luminious。

## 使用Ceph存储

见ceph目录下的[README](ceph/README.md)

## 使用NFS(本地)存储

见nfs目录下的[README](nfs/README.md)

## 日志管理

见logging目录下的[README](logging/README.md)

## 集群监控

见monitoring目录下的[README](monitoring/README.md)
