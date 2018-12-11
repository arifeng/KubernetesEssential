## Ceph存储

这里使用了两种Ceph的存储类，RBD和CephFS。RBD格式为Kuberentes内建支持，而CephFS需要安装额外的provisioner。

RBD和CephFS使用起来各有优劣，见下表：

|      比较    |    RBD    |  CephFS   |
|-------------|-----------|------------|
| 读写效率     |    高     |    稍低     |
|  安全性      |    高     |    低      |
|  空间限制    |   支持     |   不支持   |
| ReadWriteMany |  不支持  |    支持    |

上面说RBD比CephFS更安全，是因为Ceph可以为K8S创建专用的存储池，而一个Ceph集群只有一个CephFS文件系统，因此所有挂载了CephFS的系统都可以访问K8S管理的卷内容。

另外，在许多教程中（包括Kuberentes官方示例）都是使用ceph的admin账号进行示范，用来测试还可以，但是用于生产环境肯定不安全，所以本项目一律没有使用admin账号，并遵循最小权限原则进行账号授权。

#### RBD存储类

我们将在Ceph中专门创建一个`kube`池供K8S使用，并创建ceph普通账号`kuber`。需要在Ceph集群的服务器上执行以下操作：

```
ceph osd pool create kube 8  # 创建存储池kube
rbd pool init kube  # 初始化存储池kube
ceph osd pool set kube size 3 # 设置kube存储群的副本数为2
ceph osd pool application enable kube rbd

ceph auth add client.kuber mon 'allow r' osd 'allow rwx pool=kube'  # 添加普通用户kuber，使其尽可以访问kube存储池
ceph auth get client.kuber > /etc/ceph/ceph.client.kuber.keyring  # 生成kuber用户的授权文件，此文件需分发到所有k8s服务器
ceph auth print-key client.kuber | base64  # ceph-rbd-secret.yml中的key值
```

账号创建好后，就可以创建存储类了。首先将`ceph-rbd-secret.yml.example`文件重命名为`ceph-rbd-secret.yml`，修改其中key值为上面最后一步的输出，同时修改`ceph-rbd-storage.yml`中的集群地址，然后执行命令：

    kubectl create -f ceph/ceph-rbd-secret.yml
    kubectl create -f ceph/ceph-rbd-storage.yml

现在创建一个持久卷测试一下：

    kubectl create -f ceph/ceph-rbd-test.yml
    kubectl get pvc | grep ceph-rbd-test   # 如果状态为`Bound`说明创建成功了。

#### CephFS存储类

先在Ceph集群上创建一个普通用户`cephfs`用于挂载CephFS，避免直接使用admin账号。在Ceph集群服务器上进行操作：

    ceph auth add client.cephfs mds 'allow rwp' mon 'allow rwx' osd 'allow rw pool=cephfs_data'
    ceph auth print-key client.cephfs | base64  # ceph-fs-secret.yml中的key值

> 关于这个普通用户的权限问题参考了 # https://github.com/kubernetes-incubator/external-storage/issues/941

然后需要在K8S集群上安装[ceph-fs-provisioner](https://github.com/kubernetes-incubator/external-storage/tree/master/ceph/cephfs/)。在安装之前需要提前创建好`cephfs`命名空间，然后执行:

    kubectl create -f ceph/ceph-fs-privisioner

> 该目录下的 clusterrole.yml 和 deployment.yml 已做修改，否则无法正常运行，修改的部分已添加注释。

将`ceph-fs-secret.yml.example`重命名为`ceph-fs-secret.yml`，修改其中的key值，同时修改`ceph-fs-storage.yml`中的集群地址，然后执行：

    kubectl create -f ceph/ceph-fs-secret.yml
    kubectl create -f ceph/ceph-fs-storage.yml

现在进行测试：

    kubectl create -f ceph/ceph-fs-test.yml
    kubectl get pvc | grep ceph-fs-test   # 如果状态为`Bound`说明创建成功了。
