# k8s常用操作

### 命令行

```bash

# 查看节点的状态
kubectl get nodes
# 查看pod的状态

kubectl get pod --all-namespces
# 查看pod更多的信息
kubectl get pod --all-namespaces -o wide
# 查看pod的具体情况
kubectl describe pod #查看namespace=default下所有的pod
kubectl describe pod <PodName>  #--namesapce=keub-sys 指定命名空间 查看指定的pod
# 部署一个应用 kubectl run <PodName> --image=<PodImage> --replicas=<pod副本的数量>
kubectl run httpd-app --image=httpd --replicas=2

kubectl run nginx-deployment --image=nginx:1.7.9 --replicas=2 #将部署包含两个副本的Deployment nginx-deployment，容器的image为nginx:1.7.9.
#通过kubectl get deployment命令查看nginx-deployment
的状态，输出显示两个副本正常运行
#用kubectl describe deployment了解更详细的信息
#用kubectl describe replicaset查看详细信息



```

### 配置文件

> 通过配置文件和kubectl apply创建。

```yaml
#nginx.yaml
apiVersion: extensions/v1beta1 #apiVersion指定当前配置格式的版本
kind: Deployment #kind是要穿建的资源类型
metadata: #metadata是该资源的元数据，name是必须的元数据选项。
  name: nginx-deployment
spec: #spec描述Deployment的规格说明。
  replicas: 2 #指定副本的数量，默认为1
  template:   #定义pod的模板
    metadata:  #定义pod的元数据，至少需要一个label。label的key和value可以任意指定。
      labels:
        app: web_server
    spec:  #描述pod的规格，此部分定义pod中每一个容器的属性，name和image是必须的。
      containers:
      - name: nginx
        image: nginx:1.7.9

```

```bash
kubectl apply -f nginx.yaml 
```

> kubectl apply不但能够创建Kubernetes资源，也能对资源进行更新，非常方便。不过Kubernets还提供了几个类似的命令，例如kubectl create、kubectl replace、kubectl edit和kubectl patch。
