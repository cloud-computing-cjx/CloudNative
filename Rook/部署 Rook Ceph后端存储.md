# 部署 Rook Ceph后端存储
Rook 项目是一个基于 Ceph 的 Kubernetes 存储插件（它后期也在加入对更多存储实现的支持）。不过，不同于对 Ceph 的简单封装，Rook 在自己的实现中加入了水平扩展、迁移、灾难备份、监控等大量的企业级功能，使得这个项目变成了一个完整的、生产级别可用的容器存储插件。
## 安装
这里使用1.2版本，再高级的版本条件会高一点
```
git clone --single-branch --branch release-1.2 https://github.com/rook/rook.git
cd cluster/examples/kubernetes/ceph
kubectl create -f common.yaml
kubectl create -f operator.yaml
kubectl create -f cluster-test.yaml

需要等一段时间，才能创建完成
[root@bigdata-pro04 ceph]# kubectl -n rook-ceph get pod
NAME                                                      READY   STATUS      RESTARTS   AGE
csi-cephfsplugin-kxw4j                                    3/3     Running     0          5m25s
csi-cephfsplugin-provisioner-558b4777b-jvzk6              4/4     Running     0          5m25s
csi-cephfsplugin-provisioner-558b4777b-m446c              4/4     Running     0          5m25s
csi-rbdplugin-6b5np                                       3/3     Running     0          5m25s
csi-rbdplugin-provisioner-55494cc8b4-98fxs                5/5     Running     0          5m25s
csi-rbdplugin-provisioner-55494cc8b4-xnglv                5/5     Running     0          5m25s
rook-ceph-crashcollector-bigdata-pro03-687f6d6b9d-gpbt4   1/1     Running     0          115s
rook-ceph-mgr-a-57dd7c7999-96v4w                          1/1     Running     0          2m29s
rook-ceph-mon-a-5545dcfc95-lwkmm                          1/1     Running     0          2m45s
rook-ceph-operator-85bcfbf89-xrmmk                        1/1     Running     0          7m19s
rook-ceph-osd-0-7797f5db79-wjkfj                          1/1     Running     0          116s
rook-ceph-osd-prepare-bigdata-pro03-94skp                 0/1     Completed   0          2m8s
rook-discover-2rztw                                       1/1     Running     0          5m28s
```