# kubernetes

> 安装教程看官方：https://kubernetes.io/zh-cn/docs/tasks/tools/install-kubectl-linux/

### 部署集群

> minikube是本地Kubernetes，致力于让Kubernetes易于学习和开发
> 您只需要Docker(或类似的兼容)容器或虚拟机环境，Kubernetes只需执行一个命令即可:minikube start
>
> What you’ll need
>
> * 2 CPUs or more
> * 2GB of free memory
> * 20GB of free disk space
> * Internet connection
> * Container or virtual machine manager, such as: [Docker](https://minikube.sigs.k8s.io/docs/drivers/docker/), [QEMU](https://minikube.sigs.k8s.io/docs/drivers/qemu/), [Hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/), [Hyper-V](https://minikube.sigs.k8s.io/docs/drivers/hyperv/), [KVM](https://minikube.sigs.k8s.io/docs/drivers/kvm2/), [Parallels](https://minikube.sigs.k8s.io/docs/drivers/parallels/), [Podman](https://minikube.sigs.k8s.io/docs/drivers/podman/), [VirtualBox](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/), or [VMware Fusion/Workstation](https://minikube.sigs.k8s.io/docs/drivers/vmware/)
>
> 安装
>
> ```bash
> curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
> sudo install minikube-linux-amd64 /usr/local/bin/minikube
>
> ```
>
> 启动集群：minikube start 时报错
>
> ❌  Exiting due to DRV_AS_ROOT: The "docker" driver should not be used with root privileges.
>
> 解决minikube start --force --driver=docker
>
> 更多：https://minikube.sigs.k8s.io/docs/start/

### 入门参考

> https://www.topgoer.cn/docs/kubernetes_foundation/kubernetes_foundation-1drs06hb8fg67

### k8s对象及spec

> k8s对象是k8s系统持久化的实体，一个k8s对代表用户的一个意图。

配置例子:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.7.9
          ports:
            - containerPort: 80
```

如上的一个 YAML 文件，对应到 Kubernetes 中，就是一个 API Object（API 对象）。当你为这个对象的各个字段填好值并提交给 Kubernetes 之后，Kubernetes 就会负责创建出这些对象所定义的容器或者其他类型的 API 资源。

### Deployment

> 使用 Deployment 运行一个无状态应用。无状态就是我们的应用不需要持久化存储任何数据。像mysql，MongoDB这些应用是需要持久化存储的，就不能部署为Deployment.

### StatefulSet

> 运行一个有状态的应用程序.
