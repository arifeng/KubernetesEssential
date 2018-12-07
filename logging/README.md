### 日志收集和浏览

主要包括以下服务:

* elasticsearch: 用于保存集群和应用生成的日志
* kibana: 图形化展示日志和监控
* fluentd: 统一处理应用手动记录的日志
* fluentd-daemonset: 自动记录集群和容器生成的日志
* apm-server: 自动记录应用日志服务

所有的服务位于`logging`命令空间下。

### 部署流程

部署用户应具有logging命名空间的读写权限。

```
    kubectl create -f elasticsearch.yml
    kubectl create -f fluentd.yml
    kubectl create -f fluentd-daemonset.yml # 需要集群管理员权限
    kubectl create -f kibana.yml
    kubectl create -f elastic-apm-server.yml
```

如果想要测试部署的日志模块是否正常，可以部署以下测试应用：

    kubectl create -f nodejs-fluentd-apm-test.yml

在浏览器中多次访问 http://192.168.1.100:13000 和 http://192.168.1.100:13000/home 两个API以采集apm数据。

把kibana服务端口映射到本地：

    kubectl port-forward -n logging svc/kibana 5601:5601

然后打开浏览器访问 http://127.0.0.1:5601 即可。

测试完成后删除测试应用：

    kubectl delete -f nodejs-fluentd-apm-test.yml

### 注意

* 日志和其他应用使用的 Elasticsearch 是两个实例，本实例在logging命名空间下，应用所用的实例在default命令空间下，不要混淆。
* 本模块的ES服务名必须是`elasticsearch`，因为kibana,fluentd,fluentd-daemonset和apm-server都会使用`elasticsearch`作为主机名去连接ES服务。
* Kibana服务的服务名称必须是`kibana`，因为apm-server会以此作为Kibana服务的主机名。
* 为考虑安全性，本模块内的所有服务均只能集群内访问，如果外部需要访问可以使用kubectl将服务映射到本地（见上面的 kubectl port-forward）。
