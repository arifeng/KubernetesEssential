有时我们想要使用K8S节点本身的存储（考虑到Ceph的性能有时不能满足需求），最简单的办法是在Pod配置里直接挂载hostPath。但是因为Pod可能会被调度到其他机器上，就会导致以前数据无法访问。因此，我们需要创建一个网络存储，此存储的数据保存在指定的节点上，可供其他任意节点的Pod使用。这里基于NFS协议实现此功能。

首先需要创建 [nfs-provisioner](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs)。当通过StorageClass申请PVC时，provisioner收到此事件后就启动一个NFS服务器，并在已经配置好的服务器目录（provisioner运行所在节点的本地目录）下创建一个卷。因此，一个privisioner只能共享一个节点下的存储，如果需要分别使用多个节点的本地存储，需要对应创建多个privisioner，以及对应的存储类。

假设我们要使用node1的本地存储，可以如下操作：

    kubectl create -f nfs-provisioner/psp.yml
    kubectl create -f nfs-provisioner/rbac.yml
    kubectl create -f nfs-provisioner/provisioner-node1.yaml # 执行前需修改nodeSelector字段的主机名，使privisioner被调度到此主机上，并且hostPath指定的目录/srv必须存在。

如果就可以创建存储类和动态卷了：

    kubectl create -f nfs-node1-storage.yml
    kubectl create -f nfs-node1-test.yml

如果卷的状态为`Bound`说明创建成功了。这时到指定的节点上可以看到/srv目录下创建了相应的动态卷。

如果也要使用node2下的存储怎么办？只需要将`provisioner-node1.yaml`复制到`provisioner-node2.yaml`，并将其中的`node1`替换为`node2`，主机名替换为此节点的主机名，然后创建即可。当然也要创建对应的存储类，此处不再赘述。
