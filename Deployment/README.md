# 运行
```
$ kubectl create -f nginx-deployment.yaml
```
# 获取
```
$ kubectl get pods -l app=nginx
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-67594d6bf6-9gdvr   1/1       Running   0          10m
nginx-deployment-67594d6bf6-v6j7w   1/1       Running   0          10m
```
kubectl get 指令的作用，就是从 Kubernetes 里面获取（GET）指定的 API 对象。可以看到，在这里我还加上了一个 -l 参数，即获取所有匹配 app: nginx 标签的 Pod。需要注意的是，**在命令行中，所有 key-value 格式的参数，都使用“=”而非“:”表示。**
## 查看一个 API 对象的细节
 kubectl describe 
 ```
$ kubectl describe pod nginx-deployment-67594d6bf6-9gdvr
Name:               nginx-deployment-67594d6bf6-9gdvr
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               node-1/10.168.0.3
Start Time:         Thu, 16 Aug 2018 08:48:42 +0000
Labels:             app=nginx
                    pod-template-hash=2315082692
Annotations:        <none>
Status:             Running
IP:                 10.32.0.23
Controlled By:      ReplicaSet/nginx-deployment-67594d6bf6
...
Events:

  Type     Reason                  Age                From               Message

  ----     ------                  ----               ----               -------
  
  Normal   Scheduled               1m                 default-scheduler  Successfully assigned default/nginx-deployment-67594d6bf6-9gdvr to node-1
  Normal   Pulling                 25s                kubelet, node-1    pulling image "nginx:1.7.9"
  Normal   Pulled                  17s                kubelet, node-1    Successfully pulled image "nginx:1.7.9"
  Normal   Created                 17s                kubelet, node-1    Created container
  Normal   Started                 17s                kubelet, node-1    Started container
 ```
在 kubectl describe 命令返回的结果中，你可以清楚地看到这个 Pod 的详细信息，比如它的 IP 地址等等。其中，有一个部分值得你特别关注，它就是** Events（事件）。**

在 Kubernetes 执行的过程中，对 API 对象的所有重要操作，都会被记录在这个对象的 Events 里，并且显示在 kubectl describe 指令返回的结果中。

比如，对于这个 Pod，我们可以看到它被创建之后，被调度器调度（Successfully assigned）到了 node-1，拉取了指定的镜像（pulling image），然后启动了 Pod 里定义的容器（Started container）。

所以，这个部分正是我们将来进行 Debug 的重要依据。**如果有异常发生，你一定要第一时间查看这些 Events**，往往可以看到非常详细的错误信息。

## 对这个 Nginx 服务进行升级
修改这个 YAML 文件
```
...    
    spec:
      containers:
      - name: nginx
        image: nginx:1.8 #这里被从1.7.9修改为1.8
        ports:
      - containerPort: 80
```
使用 kubectl replace 指令来完成这个更新：
```
 $ kubectl replace -f nginx-deployment.yaml
 ```
 推荐使用 kubectl apply 命令，来统一进行 Kubernetes 对象的创建和更新操作。

 这样的操作方法，是 Kubernetes“声明式 API”所推荐的使用方法。也就是说，作为用户，你不必关心当前的操作是创建，还是更新，你执行的命令始终是 kubectl apply，而 Kubernetes 则会根据 YAML 文件的内容变化，自动进行具体的处理。

 # 声明一个 Volume
 在 Kubernetes 中，Volume 是属于 Pod 对象的一部分。

 修改这个 YAML 文件里的 template.spec 字段
 ```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: nginx-vol
      volumes:
      - name: nginx-vol
        emptyDir: {}
```
在 Deployment 的 Pod 模板部分添加了一个 volumes 字段，定义了这个 Pod 声明的所有 Volume。它的名字叫作 nginx-vol，类型是 emptyDir。
## 什么是 emptyDir 类型
Docker 的隐式 Volume 参数，即：不显式声明宿主机目录的 Volume。所以，Kubernetes 也会在宿主机上创建一个临时目录，这个目录将来就会被绑定挂载到容器所声明的 Volume 目录上。

Kubernetes 也提供了显式的 Volume 定义，它叫作 hostPath。比如下面的这个 YAML 文件：
```
 ...   
    volumes:
      - name: nginx-vol
        hostPath: 
          path:  " /var/data"
```
这样，容器 Volume 挂载的宿主机目录，就变成了 /var/data。

更新这个 Deployment:
```
$ kubectl apply -f nginx-deployment.yaml
```
使用 kubectl describe 查看一下最新的 Pod
```
$ k describe pod nginx-deployment-748c6fff66-kxndv
```
会发现 Volume 的信息已经出现在了 Container 描述部分：
```
Containers:
  nginx:
    Container ID:   docker://058a92bb208a8b14e1afc9fe5c468d2e311626a921f4785e597613061390ea63
    Image:          nginx:1.8
    Image ID:       docker-pullable://docker.io/nginx@sha256:c97ee70c4048fe79765f7c2ec0931957c2898f47400128f4f3640d0ae5d60d10
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 12 Sep 2021 21:52:20 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from nginx-vol (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n24nr (ro)
...
Volumes:
  nginx-vol:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-n24nr:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
```
使用 kubectl exec 指令，进入到这个 Pod 当中（即容器的 Namespace 中）查看这个 Volume 目录：
```
$ k exec -it  nginx-deployment-748c6fff66-kxndv -- /bin/bash
root@nginx-deployment-748c6fff66-kxndv:/# 
```
删除这个 Nginx Deployment 
```
$ kubectl delete -f nginx-deployment.yaml
```
# 思考题
在实际使用 Kubernetes 的过程中，相比于编写一个单独的 Pod 的 YAML 文件，我一定会推荐你使用一个 replicas=1 的 Deployment。请问，这两者有什么区别呢？
# 1
推荐使用replica=1而不使用单独pod的主要原因是pod所在的节点出故障的时候 pod可以调度到健康的节点上，单独的pod只能在节点健康的情况下由kubelet保证pod的健康状况

# Q
emptyDir创建一个临时目录，pod删除之后临时目录也会被删除。在平时的使用下，有哪些场景会用到这种类型volume呢？
# A
临时写文件，又不想提交到镜像里。另外，volume并不跟pod同生命周期，不会删的这么快。

# Q
yaml文件中如何使用变量，不是环境变量env那种，而是我在yaml定义一个版本号的变量，当版本发生变更，我只需要修改版本号变量，或者外部传参就行了。不希望频繁修改yaml文件。
# A
可以使用placeholder，或者yaml模板jinja

