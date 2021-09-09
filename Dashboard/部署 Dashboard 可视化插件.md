# 部署 Dashboard 可视化插件
一个可视化的 Web 界面来查看当前集群的各种信息。
## 安装
官网：https://github.com/kubernetes/dashboard
```
[root@bigdata-pro04 ~]# kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml

[root@bigdata-pro04 ~]# kubectl get pods -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-856586f554-nqs86   1/1     Running   0          83s
kubernetes-dashboard-67484c44f6-5jm9h        1/1     Running   0          83s
```
