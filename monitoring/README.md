## 集群和应用监控

本监控方案使用 Metricbeat + ElasticSearch + Kibana ，并重用日志模块部署的 ElasticSearch 和 Kibana，所以需要先部署日志模块。将部署在`monitoring`命名空间下。

## 部署步骤

#### 部署kube-steate-metrics

Metricbeat依赖`kube-steate-metrics`获取部分数据，需要首先部署之。

    kubectl create -f kube-state-metrics  # 需要集群管理员权限

注：kube-state-metrics目录来自于 https://github.com/kubernetes/kube-state-metrics/tree/master/kubernetes , 除了将命名空间修改成了monitoring，还有更换了镜像源。

## 安装 Kibana Dashboard

首先将集群中的Elasticsearch和Kibana服务使本地可以访问，两个命令需要分别在两个终端中执行，并且不能关闭终端。

    kubectl port-forward svc/elasticsearch 9200:9200 -n logging
    kubectl port-forward svc/kibana 5601:5601 -n logging

然后访问 http://127.0.0.1:5601/app/kibana#/home/tutorial/kubernetesMetrics 并根据提示下载Metricbeat。修改配置文件`metricbeat.yml`将elasticsearch数据库地址设置为`localhost:9200`，Kibana地址设置为`localhost:5601`，然后执行命令安装Dashboard:

    ./metricbeat setup

> 注：我们只需要运行setup安装dashboard，不必执行 ./metricbeat modules enable kubernetes 和 ./metricbeat -e 命令，因为在下一步要将Metricbeat部署到集群里运行。

## 部署 Metricbeat

> 参考：https://www.elastic.co/guide/en/beats/metricbeat/current/running-on-kubernetes.html

    kubectl create -f metricbeat-kubernetes.yaml # 需要集群管理员权限

注：`metricbeat-kubernetes.yaml`在原版的基础上，将命名空间更换为monitoring，同时将ES数据库的位置指定成`elasticsearch.logging`。

部署完成之后，访问 `http://127.0.0.1:5601/app/kibana#/dashboards/` ，选择"
[Metricbeat Kubernetes] Overview"即可。
