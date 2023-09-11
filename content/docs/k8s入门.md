# k8s入门

## k8s基础

> 官方教程: https://kubernetes.io/zh-cn/docs/tutorials/

### 命令行

```bash
# 查看节点的状态
kubectl get nodes

#创建一个dev的命名空间
kubectl create ns dev #kubectl create namespace dev
# 删除dev的命名空间 注意：一个命名空间一旦被删除，那么它下面所有的pod都会被删除
kubectl delete ns dev

# 查看pod的状态
kubectl get pod --all-namespces
# 查看pod更多的信息
kubectl get pod --all-namespaces -o wide
# 查看pod的具体情况
kubectl describe pod #查看namespace=default下所有的pod
kubectl describe pod <PodName>  #--namesapce=keub-sys 指定命名空间 查看指定的pod
# 部署一个应用 kubectl run <PodName> --image=<PodImage> --replicas=<pod副本的数量>
kubectl run httpd-app --image=httpd --replicas=2

kubectl run nginx-deployment --image=nginx:1.7.9 --replicas=2 #将部署包含2个副本的Deployment nginx-deployment，容器的image为nginx:1.7.9.
#通过kubectl get deployment命令查看nginx-deployment
的状态，输出显示两个副本正常运行
#用kubectl describe deployment了解更详细的信息
#用kubectl describe replicaset查看详细信息

# 删除pod
kubectl delete pod <PodName> # kubectl delete -f <pod 的配置文件> 如：kubectl delete -f test-pod.yaml

#查看Deployment的状态
kubectl get deployments


```

### 配置文件

> 通过配置文件和kubectl apply创建。

```yaml
#nginx.yaml
---
apiVersion: extensions/v1beta1 #apiVersion指定当前配置格式的版本
kind: Deployment #kind是要穿建的资源类型
metadata: #metadata是该资源的元数据，name是必须的元数据选项。
  name: nginx-deployment
spec: #spec描述Deployment的规格说明。
  replicas: 2 #指定副本的数量，默认为1
  template:   #定义pod的模板
    metadata:  #定义pod的元数据，至少需要一个label。label的key和value可以任意指定。
      labels:  # labels pod 的标签是一个map类型，k-v随便定义
       app: web   #为该pod定义了一个label key=app，value=web
    spec:  #描述pod的规格，此部分定义pod中每一个容器的属性，name和image是必须的。
      containers:  #一个pod可以有多个容器，containers表示容器列表，是一个数组
        - name: frond-end #容器的名称
          images: nginx   #容器的镜像
          ports:          #端口，也是一个数组，因为一个容器可以暴露多个端口
           - containerPort: 80 #容器中暴露的端口

```

```bash
kubectl apply -f nginx.yaml   #kubectl create -f  nginx.yaml 
```

> kubectl apply不但能够创建Kubernetes资源，也能对资源进行更新，非常方便。不过Kubernets还提供了几个类似的命令，例如kubectl create、kubectl replace、kubectl edit和kubectl patch。

### 创建Pod

> 一般不建议直接创建一个pod

```yaml
---
apiVersion: v1 # 指定kube api 的版本
kind: Pod      # 指定资源类型
metadata:      # 资源的元数据
  name: kebu100-site # 为该Pod取一个名称
  labels:      # labels pod 的标签是一个map类型，k-v随便定义
    app: web   #为该pod定义了一个label key=app，value=web
spec:          # 资源的规格,即定义资源的模板
  containers:  #一个pod可以有多个容器，containers表示容器列表，是一个数组
    - name: frond-end-100 #容器的名称
      images: nginx   #容器的镜像
      ports:          #端口，也是一个数组，因为一个容器可以暴露多个端口
        - containerPort: 80 #容器中暴露的端口
```

> 在这个例⼦中，这是⼀个简单的最⼩定义：⼀个名字（front-end），基于 nginx 的镜像，以及容器 将会监听的⼀个端⼝（80）。在这些当中，只有名字是⾮常需要的，你也可以指定⼀个更加复杂的属性，例如在容器启动时运⾏的命令，应使⽤的参数，⼯作⽬录，或每次实例化时是否拉取映像的新副本。以下是⼀些容器可选的设置属性：
>
> * name
> * image
> * command
> * args
> * workingDir
> * ports
> * env
> * resources
> * volumeMounts
> * livenessProbe
> * readinessProbe
> * livecycle
> * terminationMessagePath
> * imagePullPolicy
> * securityContext
> * stdin
> * stdinOnce
> * tty
>
> 在Kubernetes集群中除了我们经常使⽤到的普通的 Pod 外，还有⼀种特殊的 Pod，叫做 Static Pod ，就是我们说的静态 Pod，静态 Pod 有什么特殊的地⽅呢？
>
> 静态 Pod 直接由特定节点上的 kubelet 进程来管理，不通过 master 节点上的apiserver 。⽆法与我们常⽤的控制器 Deployment 或者 DaemonSet 进⾏关联，它由 kubelet 进程⾃⼰来监控，当 pod 崩溃时重启该 pod ， kubelete 也⽆法对他们进⾏健康检查。静态 pod 始终绑定在某⼀个 kubelet ，并且始终运⾏在同⼀个节点上。 kubelet 会⾃动为每⼀个静态 pod 在 Kubernetes 的 apiserver 上创建⼀个镜像 Pod（Mirror Pod），因此我们可以在 apiserver 中查询到该 pod，但是不能通过 apiserver 进⾏控制（例如不能删除）。
>
> 创建静态 Pod 有两种⽅式：配置⽂件和 HTTP 两种⽅式
>
> ### 配置⽂件
>
> 配置⽂件就是放在特定⽬录下的标准的 JSON 或 YAML 格式的 pod 定义⽂件。⽤ kubelet --pod-manifest-path=`<the directory>` 来启动 kubelet 进程，kubelet 定期的去扫描这个⽬录，根据这个⽬录下出现或消失的 YAML/JSON ⽂件来创建或删除静态 pod。
>
> ⽐如我们在 node01 这个节点上⽤静态 pod 的⽅式来启动⼀个 nginx 的服务。我们登录到node01节点上⾯，可以通过下⾯命令找到kubelet对应的启动配置⽂件
>
> $ systemctl status kubelet
>
> 配置⽂件路径为：
>
> $ /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
>
> 打开这个⽂件我们可以看到其中有⼀条如下的环境变量配置：
>
> Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"
>
> 所以如果我们通过 kubeadm 的⽅式来安装的集群环境，对应的 kubelet 已经配置了我们的静态 Pod⽂件的路径，那就是 /etc/kubernetes/manifests ，所以我们只需要在该⽬录下⾯创建⼀个标准的 Pod
>
> 的 JSON 或者 YAML ⽂件即可：
>
> 如果你的 kubelet 启动参数中没有配置上⾯的 --pod-manifest-path 参数的话，那么添加上这个参数然
>
> 后重启 kubelet 即可。
>
> ```bash
> [root@ node01 ~] $ cat <<EOF >/etc/kubernetes/manifest/static-web.yaml
> ```

> ```yaml
> # 静态pod
> apiVersion: v1
> kind: Pod
> metadata:
>   name: static-web
>   labels:
>     app: static
> spec:
>   containers:
>     - name: web
>       image: nginx
>       ports:
>          - name: web
>            containerPort: 80
> EOF
> ```
>
> ### 通过 HTTP 创建静态 Pods
>
> kubelet 周期地从 –manifest-url= 参数指定的地址下载⽂件，并且把它翻译成 JSON/YAML 格式的pod 定义。此后的操作⽅式与 –pod-manifest-path= 相同，kubelet 会不时地重新下载该⽂件，当⽂件变化时对应地终⽌或启动静态 pod。
>
> #### 静态pods的动作⾏为
>
> kubelet 启动时，由 --pod-manifest-path= or --manifest-url= 参数指定的⽬录下定义的所有 pod 都会⾃动创建，例如，我们示例中的 static-web。（可能要花些时间拉取nginx 镜像，耐⼼等待…）
>
> ```bash
> [root@node01 ~] $ docker ps
> CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
> f6d05272b57e nginx:latest "nginx" 8 minutes ago Up 8 minutes k8s_web.6f802af4_static-web-fk-node1_default_67e24ed9466ba55986d120c867395f3c_378e5f3c
> ```
>
> 现在我们通过 kubectl ⼯具可以看到这⾥创建了⼀个新的镜像 Pod：
>
> ```bash
> [root@node01 ~] $ kubectl get pods
> NAME READY STATUS RESTARTS AGE
> static-web-my-node01 1/1 Running 0 2m
> ```
>
> 静态 pod 的标签会传递给镜像 Pod，可以⽤来过滤或筛选。 需要注意的是，我们不能通过 API 服务器来删除静态 pod（例如，通过kubectl命令），kebelet 不会删除它。
>
> ```bash
> [root@node01 ~] $ kubectl delete pod static-web-my-node01
> [root@node01 ~] $ kubectl get pods
> NAME READY STATUS RESTARTS AGE
> static-web-my-node01 1/1 Running 0 12s
> ```
>
> 我们尝试⼿动终⽌容器，可以看到kubelet很快就会⾃动重启容器。
>
> ```bash
> [root@node01 ~] $ docker ps
> CONTAINER ID IMAGE COMMAND CREATED ...
> 5b920cbaf8b1 nginx:latest "nginx -g 'daemon of 2 seconds ago ...
> ```
>
> #### 静态pods的动态增加和删除
>
> 运⾏中的kubelet周期扫描配置的⽬录（我们这个例⼦中就是/etc/kubernetes/manifests）下⽂件的变化，当这个⽬录中有⽂件出现或消失时创建或删除pods。
>
> ```bash
> [root@node01 ~] $ mv /etc/kubernetes/manifests/static-web.yaml /tmp
> [root@node01 ~] $ sleep 20
> [root@node01 ~] $ docker ps
> // no nginx container is running
> [root@node01 ~] $ mv /tmp/static-web.yaml /etc/kubernetes/manifests
> [root@node01 ~] $ sleep 20
> [root@node01 ~] $ docker ps
> CONTAINER ID IMAGE COMMAND CREATED ...
> e7a62e3427f1 nginx:latest "nginx -g 'daemon of 27 seconds ago
> ```
>
> 其实我们⽤ kubeadm 安装的集群，master 节点上⾯的⼏个重要组件都是⽤静态 Pod 的⽅式运⾏的，我们登录到 master 节点上查看 /etc/kubernetes/manifests ⽬录：
>
> ```bash
> [root@master ~]# ls /etc/kubernetes/manifests/
> etcd.yaml kube-apiserver.yaml kube-controller-manager.yaml kube-scheduler.yaml
> ```
>
> 现在明⽩了吧，这种⽅式也为我们将集群的⼀些组件容器化提供了可能，因为这些 Pod 都不会受到apiserver 的控制，不然我们这⾥ kube-apiserver 怎么⾃⼰去控制⾃⼰呢？万⼀不⼩⼼把这个 Pod 删掉了呢？所以只能有 kubelet ⾃⼰来进⾏控制，这就是我们所说的静态 Pod。

### 创建 Deployment

> 现在我们可以来创建⼀个真正的 Deployment。在上⾯的例⼦中，我们只是单纯的创建了⼀个 POD 实例，但是如果这个 POD 出现了故障的话，我们的服务也就挂掉了，所以 kubernetes 提供了⼀个 Deployment 的概念，可以让 kubernetes 去管理⼀组 POD 的副本，也就是副本集，这样就可以保证⼀定数量的副本⼀直可⽤的，不会因为⼀个 POD 挂掉导致整个服务挂掉。我们可以这样定义⼀个Deployment:

```yaml
#deployment.yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  name: kube101-site
spec: 
  replicas: 2
  template: 
     metadata:
  
```

部署：

```bash
kubectl create -f deployment.yaml
```

### Pod Hook

> 我们知道 Pod 是 Kubernetes 集群中的最⼩单元，⽽ Pod 是有容器组组成的，所以在讨论 Pod 的⽣命周期的时候我们可以先来讨论下容器的⽣命周期。
>
> 实际上 Kubernetes 为我们的容器提供了⽣命周期钩⼦的，就是我们说的 Pod Hook ，Pod Hook 是由kubelet 发起的，当容器中的进程启动前或者容器中的进程终⽌之前运⾏，这是包含在容器的⽣命周期之中。我们可以同时为 Pod 中的所有容器都配置 hook。
>
> Kubernetes 为我们提供了两种钩⼦函数：
>
> * PostStart：这个钩⼦在容器创建后⽴即执⾏。但是，并不能保证钩⼦将在容器 ENTRYPOINT 之前运⾏，因为没有参数传递给处理程序。主要⽤于资源部署、环境准备等。不过需要注意的是如果钩⼦花费太⻓时间以⾄于不能运⾏或者挂起， 容器将不能达到 running 状态。
> * PreStop：这个钩⼦在容器终⽌之前⽴即被调⽤。它是阻塞的，意味着它是同步的， 所以它必须在删除容器的调⽤发出之前完成。主要⽤于优雅关闭应⽤程序、通知其他系统等。如果钩⼦在执⾏期间挂起， Pod阶段将停留在 running 状态并且永不会达到 failed 状态。
>
> 如果 PostStart 或者 PreStop 钩⼦失败， 它会杀死容器。所以我们应该让钩⼦函数尽可能的轻量。当然有些情况下，⻓时间运⾏命令是合理的， ⽐如在停⽌容器之前预先保存状态。
>
> 另外我们有两种⽅式来实现上⾯的钩⼦函数：
>
> * Exec - ⽤于执⾏⼀段特定的命令，不过要注意的是该命令消耗的资源会被计⼊容器。
> * HTTP - 对容器上的特定的端点执⾏ HTTP 请求。
>
> ### 示例1 环境准备
>
> 以下示例中，定义了⼀个Nginx Pod，其中设置了 PostStart 钩⼦函数，即在容器创建成功后，写⼊⼀句话到 /usr/share/message ⽂件中。
>
> ```yaml
> apiVErsion: v1
> kind: Pod
> metadata:
>   name: hook-demo1
> spec: 
>   containers:
>     - name: hook-demo1
>       image: nginx
>       lifecycle: 
>         postStart:
>            exec: 
>              command: ["bin/sh","-c","echo Hello from the postStart handler > /usr/share/message"]
> ```
>
> ### 示例2 优雅删除资源对象
>
> 当⽤户请求删除含有 pod 的资源对象时（如Deployment等），K8S 为了让应⽤程序优雅关闭（即让应⽤程序完成正在处理的请求后，再关闭软件），K8S提供两种信息通知：
>
> * 默认：K8S 通知 node 执⾏ docker stop 命令，docker 会先向容器中 PID 为1的进程发送系统信号 SIGTERM ，然后等待容器中的应⽤程序终⽌执⾏，如果等待时间达到设定的超时时间，或者默认超时时间（30s），会继续发送 SIGKILL 的系统信号强⾏ kill 掉进程。
> * 使⽤ pod ⽣命周期（利⽤ PreStop 回调函数），它执⾏在发送终⽌信号之前。
>
> 默认所有的优雅退出时间都在30秒内。kubectl delete 命令⽀持 --grace-period=`<seconds>` 选项，这个选项允许⽤户⽤他们⾃⼰指定的值覆盖默认值。值'0'代表 强制删除 pod. 在 kubectl 1.5 及以上的版本⾥，执⾏强制删除时必须同时指定 --force --grace-period=0 。
>
> 强制删除⼀个 pod 是从集群状态还有 etcd ⾥⽴刻删除这个 pod。 当 Pod 被强制删除时， api 服务器不会等待来⾃ Pod 所在节点上的 kubelet 的确认信息：pod 已经被终⽌。在 API ⾥ pod 会被⽴刻删除，在节点上， pods 被设置成⽴刻终⽌后，在强⾏杀掉前还会有⼀个很⼩的宽限期。
>
> 以下示例中，定义了⼀个Nginx Pod，其中设置了 PreStop 钩⼦函数，即在容器退出之前，优雅的关闭 Nginx:
>
> ```yaml
> apiVersion: v1
> kind: Pod
> metadata: 
>   name: hook-demo2
> spec: 
>   containers:
>      - name: hook-demo2
>        image: nginx
>        lifecycle:
>          preStop:
>            exec:
>              command: ["use.sbin/nginx","-s","quit"]
> ---
> apiVersion: v1
> kind: Pod
> metadata: 
>   name: hook-demo2
>   lables: 
>     app: hook
> spec: 
>   containers:
>     - name: hook-demo2
>       image: nginx
>       ports:
>         - name: webport
>           containerPort: 80
>       valumeMounts:
>         - name: message
>           mountPath: /usr/share/
>       lifecycle:
>         preStop:
>           exec:
>             command: ["/bin/sh","-c","echo Hello from the preStop Handler > /usr/share/message"]
>   volumes: 
>     - name: message
>       hostPath:
>         path: /tmp
>
>
> ```

### Pod的健康检查

> kubelet 通过使⽤ liveness probe 来确定你的应⽤程序是否正在运⾏，通俗点将就是是否还活着。⼀般来说，如果你的程序⼀旦崩溃了， Kubernetes 就会⽴刻知道这个程序已经终⽌了，然后就会重启这个程序。⽽我们的 liveness probe 的⽬的就是来捕获到当前应⽤程序还没有终⽌，还没有崩溃，如果出现了这些情况，那么就重启处于该状态下的容器，使应⽤程序在存在 bug 的情况下依然能够继续运⾏下去。
>
> ### 存活探针的使⽤:
>
> ⾸先我们⽤ exec 执⾏命令的⽅式来检测容器的存活，如下:
>
> ```yaml
> ---
> apiVersion: v1
> kind:Pod
> metadata:
>   name:liveness-exec
>   lables:
>      app: liveness
> spec:
>   containers:
>     - name: liveness
>       image: busybox
>       agrs:
>         - /bin/sh
>         - -c
>         - touch /tmp/headlthy;sleep 30;rm -f /tmp/headlthy; sleep 600
>       livenessProbe:
> 	exec:
> 	  command:
> 	    - cat
> 	    - /tmp/healthy
> 	initialDelaySeconds: 5
> 	periodSeconds: 5
> ```
>
> 我们这⾥需要⽤到⼀个新的属性： livenessProbe ，下⾯通过 exec 执⾏⼀段命令，其
>
> 中 periodSeconds 属性表示让 kubelet 每隔5秒执⾏⼀次存活探针，也就是每5秒执⾏⼀次上⾯的 cat /tmp/healthy 命令，如果命令执⾏成功了，将返回0，那么 kubelet 就会认为当前这个容器是存活的并且很监控，如果返回的是⾮0值，那么 kubelet 就会把该容器杀掉然后重启它。另外⼀个属性 initialDelaySeconds 表示在第⼀次执⾏探针的时候要等待5秒，这样能够确保我们的容器能够有⾜够的时间启动起来。⼤家可以想象下，如果你的第⼀次执⾏探针等候的时间太短，是不是很有可能容器还没正常启动起来，所以存活探针很可能始终都是失败的，这样就会⽆休⽌的重启下去了，对吧？所以⼀个合理的 initialDelaySeconds ⾮常重要。
>
> 另外我们在容器启动的时候，执⾏了如下命令：
>
> ```bash
> ~ /bin/sh -c "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600"
> ```
>
> 意思是说在容器最开始的30秒内有⼀个 /tmp/healthy ⽂件，在这30秒内执⾏ cat /tmp/healthy 命令都会返回⼀个成功的返回码。30秒后，我们删除这个⽂件，现在执⾏ cat /tmp/healthy 是不是就会失败了，这个时候就会重启容器了。
>
> 我们来创建下该 Pod ，在30秒内，查看 Pod 的 Event ：
>
> ```bash
> ~ kubectl describe pod liveness-exec
> ```
>
> 我们可以观察到容器是正常启动的，在隔⼀会⼉，⽐如40s后，再查看下 Pod 的 Event ，在最下⾯有⼀条信息显示 liveness probe 失败了，容器被删掉并重新创建。
>
> 然后通过 kubectl get pod liveness-exec 可以看到 RESTARTS 值加1了。
>
> 同样的，我们还可以使⽤ HTTP GET 请求来配置我们的存活探针，我们这⾥使⽤⼀个 liveness 镜像来验证演示下:
>
> ```yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   labels:
>     test: liveness
>    name: liveness-http
> spec:
>   containers:
>     - name: liveness
>       image: cnych/liveness
>       args:
>        - /server
>       livenessProbe:
>          httpGet:
>            path: /healthz
>            port: 8080
>            httpHeaders:
>              - name: X-Custom-Header
>                value: Awesome
>          initialDelaySeconds: 3
>          periodSeconds: 3
> ```
>
> 同样的，根据 periodSeconds 属性我们可以知道 kubelet 需要每隔3秒执⾏⼀次 liveness probe ，该探针将向容器中的 server 的8080端⼝发送⼀个 HTTP GET 请求。如果 server 的 /healthz 路径的handler 返回⼀个成功的返回码， kubelet 就会认定该容器是活着的并且很健康,如果返回失败的返回码， kubelet 将杀掉该容器并重启它。。 initialDelaySeconds 指定 kubelet 在该执⾏第⼀次探测之前需要等待3秒钟。
>
> 通常来说，任何⼤于200⼩于400的返回码都会认定是成功的返回码。其他返回码都会被认为是失败的返回码。
>
> 通过端⼝的⽅式来配置存活探针，使⽤此配置， kubelet 将尝试在指定端⼝上打开容器的套接字。 如果可以建⽴连接，容器被认为是健康的，如果不能就认为是失败的。
>
> ```yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   name: goproxy
>   labels:
>     app: goproxy
> spec:
>   containers:
>     - name: goproxy
>       image: cnych/goproxy
>       ports:
>         - containerPort: 8080
>       readinessProbe:
>         tcpSocket:
>           port: 8080
>         initialDelaySeconds: 5
>         periodSeconds: 10
>       livenessProbe:
>         tcpSocket:
>           port: 8080
>         initialDelaySeconds: 15
>         periodSeconds: 20
> ```
>
> 我们可以看到，TCP 检查的配置与 HTTP 检查⾮常相似，只是将 httpGet 替换成了 tcpSocket 。 ⽽且我们同时使⽤了 readiness probe 和 liveness probe 两种探针。 容器启动后5秒后， kubelet 将发送第⼀个 readiness probe （可读性探针）。 该探针会去连接容器的8080端，如果连接成功，则该Pod 将被标记为就绪状态。然后 Kubelet 将每隔10秒钟执⾏⼀次该检查。除了 readiness probe 之外，该配置还包括 liveness probe 。 容器启动15秒后， kubelet 将运⾏第⼀个 liveness probe 。 就像 readiness probe ⼀样，这将尝试去连接到容器的8080端⼝。如果 liveness probe 失败，容器将重新启动。
>
> 有的时候，应⽤程序可能暂时⽆法对外提供服务，例如，应⽤程序可能需要在启动期间加载⼤量数据或配置⽂件。 在这种情况下，您不想杀死应⽤程序，也不想对外提供服务。 那么这个时候我们就可以使⽤ readiness probe 来检测和减轻这些情况。 Pod中的容器可以报告⾃⼰还没有准备，不能处理Kubernetes服务发送过来的流量。
>
> 从上⾯的 YAML ⽂件我们可以看出 readiness probe 的配置跟 liveness probe 很像，基本上⼀致的。唯⼀的不同是使⽤ readinessProbe ⽽不是 livenessProbe 。两者如果同时使⽤的话就可以确保流量不会到达还未准备好的容器，准备好过后，如果应⽤程序出现了错误，则会重新启动容器。
>
> 另外除了上⾯的 initialDelaySeconds 和 periodSeconds 属性外，探针还可以配置如下⼏个参数：
>
> * timeoutSeconds：探测超时时间，默认1秒，最⼩1秒。
> * successThreshold：探测失败后，最少连续探测成功多少次才被认定为成功。默认是 1，但是如果是 `liveness`则必须是 1。最⼩值是 1。
> * failureThreshold：探测成功后，最少连续探测失败多少次才被认定为失败。默认是 3，最⼩值是 1。
>
> 这就是 liveness probe （存活探针）和 readiness probe （可读性探针）的使⽤⽅法。在 Pod 的⽣命周期当中，我们已经学习了容器⽣命周期中的钩⼦函数和探针检测。

## Pod的管理

> Kubernetes 就为我们提供了这样的资源对象：
>
> * Replication Controller：⽤来部署、升级 Pod
> * Replica Set：下⼀代的 Replication Controller
> * Deployment：可以更加⽅便的管理 Pod 和 Replica Set

### Replication Controller(RC)

> Replication Controller 简称 RC ， RC 是 Kubernetes 系统中的核⼼概念之⼀，简单来说， RC 可以保证在任意时间运⾏ Pod 的副本数量，能够保证 Pod 总是可⽤的。如果实际 Pod 数量⽐指定的多那就结束掉多余的，如果实际数量⽐指定的少就新启动⼀些 Pod ，当 Pod 失败、被删除或者挂掉后， RC 都会去⾃动创建新的 Pod 来保证副本数量，所以即使只有⼀个 Pod ，我们也应该使⽤ RC 来管理我们的 Pod 。
>
> 现在我们来使⽤ RC 来管理我们前⾯使⽤的 Nginx 的 Pod ， YAML ⽂件如下：
>
> ```yaml
> ---
> apiVersion: v1
> kind: ReplicationController
> metadata:
>   name: rc-demo
>   lables:
>     app: rc
> spec:
>   replicas: 3 #指定副本的数量，默认为1
>   selector: #rc 通过该属性来筛选要控制的Pod
>     name: rc
>   template: #当rc控制的pod数量不满足我们定义的要求时，rc通过该属性来创建pod，此时无需定义apiVersion和kind
>     metadata: 
>       #name:pod-name  #这里的name是无需定义的，假设我们定义了pod的名称，而这些pod又运行在同一个节点上，那么名称就冲突了，如果我们不定义的话，它会自动生成随机的名称。
>       labels:
>         name: rc  #注意这里Pod的labels要和spec.selector相同，这样rc就可以来控制当前这个pod了
>     spec: 
>       containers:
>         - name: nginx-demo #容器名称
>           image: nginx #pod依赖的镜像
>           ports:
>             - containerPort: 80 #容器中对外暴露的端口
> ```
>
> 上⾯的 YAML ⽂件相对于我们之前的 Pod 的格式：
>
> kind： ReplicationController
>
> spec.replicas: 指定 Pod 副本数量，默认为1
>
> spec.selector: RC 通过该属性来筛选要控制的 Pod
>
> spec.template: 这⾥就是我们之前的 Pod 的定义的模块，但是不需要 apiVersion 和 kind 了
>
> spec.template.metadata.labels: 注意这⾥的 Pod 的 labels 要和 spec.selector 相同，这
>
> 样 RC 就可以来控制当前这个 Pod 了。
>
> 这个 YAML ⽂件中的意思就是定义了⼀个 RC 资源对象，它的名字叫 rc-demo ，保证⼀直会有3个 Pod 运⾏， Pod 的镜像是 nginx 镜像。
>
>> 注意 spec.selector 和 spec.template.metadata.labels 这两个字段必须相同，否则会创建失败的，当然我们也可以不写 spec.selector ，这样就默认与 Pod 模板中的 metadata.labels 相同了。所以为了避免不必要的错误的话，不写为好。
>>
>
> 然后我们来创建上⾯的 RC 对象(保存为 rc-demo.yaml):
>
> ```bash
> $ kubectl create -f rc-demo.yaml
> ```
>
> 查看 RC ：
>
> ```bash
> $ kubectl get rc
> ```
>
> 查看具体信息：
>
> ```bash
> $ kubectl describe rc rc-demo
> ```
>
> 然后我们通过 RC 来修改下 Pod 的副本数量为2：
>
> ```bash
> $ kubectl apply -f rc-demo.yaml
> ```
>
> 或者
>
> ```bash
> $ kubectl edit rc rc-demo
> ```
>
> ⽽且我们还可以⽤ RC 来进⾏滚动升级，⽐如我们将镜像地址更改为 nginx:1.7.9 :
>
> ```bash
> $ kubectl rolling-update rc-demo --image=nginx:1.7.9
> ```
>
> 但是如果我们的 Pod 中多个容器的话，就需要通过修改 YAML ⽂件来进⾏修改了:
>
> ```bash
> $ kubectl rolling-update rc-demo -f rc-demo.yaml
> ```
>
> 如果升级完成后出现了新的问题，想要⼀键回滚到上⼀个版本的话，使⽤ RC 只能⽤同样的⽅法把镜像地址替换成之前的，然后重新滚动升级。

### Replication Set（RS）

> Replication Set 简称 RS ，随着 Kubernetes 的⾼速发展，官⽅已经推荐我们使⽤ RS 和 Deployment 来代替 RC 了，实际上 RS 和 RC 的功能基本⼀致，⽬前唯⼀的⼀个区别就是 RC 只⽀持基于等式的 selector （env=dev或environment!=qa），但 RS 还⽀持基于集合的 selector （version in (v1.0, v2.0)），这对复杂的运维管理就⾮常⽅便了。
>
> kubectl 命令⾏⼯具中关于 RC 的⼤部分命令同样适⽤于我们的 RS 资源对象。不过我们也很少会去单独使⽤ RS ，它主要被 Deployment 这个更加⾼层的资源对象使⽤，除⾮⽤户需要⾃定义升级功能或根本不需要升级 Pod ，在⼀般情况下，我们推荐使⽤ Deployment ⽽不直接使⽤ Replica Set 。
>
> 最后我们总结下关于 RC / RS 的⼀些特性和作⽤吧：
>
> * ⼤部分情况下，我们可以通过定义⼀个 RC 实现的 Pod 的创建和副本数量的控制
> * RC 中包含⼀个完整的 Pod 定义模块（不包含 apiversion 和 kind ）
> * RC 是通过 label selector 机制来实现对 Pod 副本的控制的
> * 通过改变 RC ⾥⾯的 Pod 副本数量，可以实现 Pod 的扩缩容功能
> * 通过改变 RC ⾥⾯的 Pod 模板中镜像版本，可以实现 Pod 的滚动升级功能（但是不⽀持⼀键回滚，需要⽤相同的⽅法去修改镜像地址）

### Deployment的使⽤（重点）

> Deployment 同样也是 Kubernetes 系统的⼀个核⼼概念，主要职责和 RC ⼀样的都是保证 Pod 的数量和健康，⼆者⼤部分功能都是完全⼀致的，我们可以看成是⼀个升级版的 RC 控制器，那 Deployment ⼜具备那些新特性呢？
>
> * RC 的全部功能： Deployment 具备上⾯描述的 RC 的全部功能
> * 事件和状态查看：可以查看 Deployment 的升级详细进度和状态
> * 回滚：当升级 Pod 的时候如果出现问题，可以使⽤回滚操作回滚到之前的任⼀版本
> * 版本记录：每⼀次对 Deployment 的操作，都能够保存下来，这也是保证可以回滚到任⼀版本的基础
> * 暂停和启动：对于每⼀次升级都能够随时暂停和启动
>
> 作为对⽐，我们知道 Deployment 作为新⼀代的 RC ，不仅在功能上更为丰富了，同时我们也说过现在官⽅也都是推荐使⽤ Deployment 来管理 Pod 的，⽐如⼀些官⽅组件 kube-dns 、 kube-proxy 也都是使⽤的 Deployment 来管理的，所以当⼤家在使⽤的使⽤也最好使⽤ Deployment 来管理 Pod 。
>
> #### 创建Deployment
>
> 下⾯创建⼀个Deployment，它创建了⼀个Replica Set来启动3个nginx pod，yaml⽂件如下：
>
> ```yaml
> apiVersion: apps/v1
>   kind: Deployment
>   metadata:
>     name: nginx-deploy
>     labels:
>       k8s-app: nginx-demo
> spec:
>   replicas: 3
>   template:
>     metadata:
>       labels:
>         app: nginx
>     spec:
>       containers:
>         - name: nginx
>           image: nginx:1.7.9
>           ports:
>             - containerPort: 80 #目标容器暴露的端口
>               name: http #为改端口起一个名称
>    
> ```
>
> 将上⾯内容保存为: nginx-deployment.yaml，执⾏命令:
>
> ```bash
> $ kubectl create -f nginx-deployment.yaml
> deployment "nginx-deploy" created
> ```
>
> 然后执⾏⼀下命令查看刚刚创建的Deployment
>
> ```bash
> $ kubectl get deployments
> NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
> nginx-deploy 3 0 0 0 1s
> ```
>
> 隔⼀会再次执⾏上⾯命令：
>
> ```bash
> $ kubectl get deployments
> NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
> nginx-deploy 3 3 3 3 4m
> ```
>
> 我们可以看到Deployment已经创建了1个Replica Set了，执⾏下⾯的命令查看rs和pod:
>
> ```bash
> $ kubectl get rs
> NAME DESIRED CURRENT READY AGE
> nginx-deploy-431080787 3 3 3 6m
> ```
>
> ```bash
> $ kubectl get pod --show-labels
> NAME READY STATUS RESTARTS AGE LABELS
> nginx-deploy-431080787-53z8q 1/1 Running 0 7m app=nginx,pod-temp
> late-hash=431080787
> nginx-deploy-431080787-bhhq0 1/1 Running 0 7m app=nginx,pod-temp
> late-hash=431080787
> nginx-deploy-431080787-sr44p 1/1 Running 0 7m app=nginx,pod-temp
> late-hash=431080787
> ```
>
> 上⾯的Deployment的yaml⽂件中的 replicas:3 将会保证我们始终有3个POD在运⾏。
>
> 由于 Deployment 和 RC 的功能⼤部分都⼀样的，我们上节课已经和⼤家演示了⼤部分功能了，我们这⾥重点给⼤家演示下 Deployment 的滚动升级和回滚功能。
>
> ### 滚动升级
>
>> 现在我们将刚刚保存的yaml⽂件中的nginx镜像修改为 nginx:1.13.3 ，然后在spec下⾯添加滚动升级策略：
>>
>
> ```yaml
> minReadySeconds: 5
> strategy:
>   # indicate which strategy we want for rolling update
>   type: RollingUpdate
>   rollingUpdate:
>     maxSurge: 1
>     maxUnavailable: 1
> ```
>
> * minReadySeconds:
>
> 1. Kubernetes在等待设置的时间后才进⾏升级
> 2. 如果没有设置该值，Kubernetes会假设该容器启动起来后就提供服务了
> 3. 如果没有设置该值，在某些极端情况下可能会造成服务服务正常运⾏
>
> * maxSurge:
>
> 1. 升级过程中最多可以⽐原先设置多出的POD数量,例如：maxSurage=1，replicas=5,则表示Kubernetes会先启动1⼀个新的Pod后才删掉⼀个旧的POD，整个升级过程中最多会有5+1个POD。
>
> * maxUnavaible:
>
> 1. 升级过程中最多有多少个POD处于⽆法提供服务的状态
> 2. 当 maxSurge 不为0时，该值也不能为0,例如：maxUnavaible=1，则表示Kubernetes整个升级过程中最多会有1个POD处于⽆法服务的状态。
>
> 然后执⾏命令：
>
> ```bash
> $ kubectl apply -f nginx-deployment.yaml
> deployment "nginx-deploy" configured
> ```
>
> 然后我们可以使⽤ rollout 命令：
>
> * 查看状态：
>
> ```bash
> $ kubectl rollout status deployment/nginx-deploy
> Waiting for rollout to finish: 1 out of 3 new replicas have been updated..
> deployment "nginx-deploy" successfully rolled out
> ```
>
> * 暂停升级
>
> ```bash
> $ kubectl rollout pause deployment <deployment>
> ```
>
> * 继续升级
>
> ```bash
> $ kubectl rollout resume deployment <deployment>
> ```
>
> 升级结束后，继续查看rs的状态：
>
> ```bash
> $ kubectl get rs
> NAME DESIRED CURRENT READY AGE
> nginx-deploy-2078889897 0 0 0 47m
> nginx-deploy-3297445372 3 3 3 42m
> nginx-deploy-431080787 0 0 0 1h
> ```
>
> 根据AGE我们可以看到离我们最近的当前状态是：3，和我们的yaml⽂件是⼀致的，证明升级成功了。
>
> ⽤ describe 命令可以查看升级的全部信息。
>
> ### 回滚Deployment
>
>> 我们已经能够滚动平滑的升级我们的Deployment了，但是如果升级后的POD出了问题该怎么办？我们能够想到的最好最快的⽅式当然是回退到上⼀次能够提供正常⼯作的版本，Deployment就为我们提供了回滚机制。
>>
>
> ⾸先，查看Deployment的升级历史：
>
> ```bash
> $ kubectl rollout history deployment nginx-deploy
> deployments "nginx-deploy"
> REVISION CHANGE-CAUSE
> 1 <none>
> 2 <none>
> 3 kubectl apply --filename=Desktop/nginx-deployment.yaml --record=true
> ```
>
> 从上⾯的结果可以看出在执⾏ Deployment 升级的时候最好带上 record 参数，便于我们查看历史版本信息。
>
> 默认情况下，所有通过 kubectl xxxx --record 都会被 kubernetes 记录到 etcd 进⾏持久化，这⽆疑会占⽤资源，最重要的是，时间久了，当你 kubectl get rs 时，会有成百上千的垃圾 RS 返回给你，那时你可能就眼花缭乱了。
>
> 上⽣产时，我们最好通过设置Deployment的 .spec.revisionHistoryLimit 来限制最⼤保留的 revision number ，⽐如15个版本，回滚的时候⼀般只会回滚到最近的⼏个版本就⾜够了。其实 rollout history 中记录的 revision 都和 ReplicaSets ⼀⼀对应。如果⼿动 delete 某个ReplicaSet，对应的 rollout history 就会被删除，也就是还说你⽆法回滚到这个 revison 了。
>
> rollout history 和 ReplicaSet 的对应关系，可以在 kubectl describe rs $RSNAME 返回的 revision 字段中得到，这⾥的 revision 就对应着 rollout history 返回的 revison 。
>
> 同样我们可以使⽤下⾯的命令查看单个 revison 的信息：
>
> ```bash
> $ kubectl rollout history deployment nginx-deploy --revision=3
> deployments "nginx-deploy" with revision #3
> Pod Template:
> Labels: app=nginx
> pod-template-hash=3297445372
> Annotations: kubernetes.io/change-cause=kubectl apply --filename=nginx-deployment.yaml
> --record=true
> Containers:
> nginx:
> Image: nginx:1.13.3
> Port: 80/TCP
> Environment: <none>
> Mounts: <none>
> Volumes: <none>
> ```
>
> 假如现在要直接回退到当前版本的前⼀个版本：
>
> ```bash
> $ kubectl rollout undo deployment nginx-deploy
> deployment "nginx-deploy" rolled back
> ```
>
> 当然也可以⽤ revision 回退到指定的版本：
>
> ```bash
> $ kubectl rollout undo deployment nginx-deploy --to-revision=2
> deployment "nginx-deploy" rolled back
> ```
>

### Pod ⾃动扩缩容

### Service

> Kubernetes 集群就为我们提供了这样的⼀个对象 - Service ， Service 是⼀种抽象的对象，它定义了⼀组 Pod 的逻辑集合和⼀个⽤于访问它们的策略，其实这个概念和微服务⾮常类似。⼀个 Serivce 下⾯包含的 Pod 集合⼀般是由 Label Selector 来决定的。⽐如我们上⾯的例⼦，假如我们后端运⾏了3个副本，这些副本都是可以替代的，因为前端并不关⼼它们使⽤的是哪⼀个后端服务。尽管由于各种原因后端的 Pod 集合会发送变化，但是前端却不需要知道这些变化，也不需要⾃⼰⽤⼀个列表来记录这些后端的服务， Service 的这种抽象就可以帮我们达到这种解耦的⽬的。
>
> ### 三种IP
>
> 在继续往下学习 Service 之前，我们需要先弄明⽩ Kubernetes 系统中的三种IP这个问题，因为经常有同学混乱。
>
> * Node IP： Node 节点的 IP 地址
> * Pod IP: Pod 的IP地址
> * Cluster IP: Service 的 IP 地址
>
> ⾸先， Node IP 是 Kubernetes 集群中节点的物理⽹卡 IP 地址(⼀般为内⽹)，所有属于这个⽹络的服务器之间都可以直接通信，所以 Kubernetes 集群外要想访问 Kubernetes 集群内部的某个节点或者服务，肯定得通过 Node IP 进⾏通信（这个时候⼀般是通过外⽹ IP 了）
>
> 然后 Pod IP 是每个 Pod 的 IP 地址，它是 Docker Engine 根据 docker0 ⽹桥的 IP 地址段进⾏分配的（我们这⾥使⽤的是 flannel 这种⽹络插件保证所有节点的 Pod IP 不会冲突）
>
> 最后 Cluster IP 是⼀个虚拟的 IP ，仅仅作⽤于 Kubernetes Service 这个对象，由 Kubernetes ⾃⼰来进⾏管理和分配地址，当然我们也⽆法 ping 这个地址，他没有⼀个真正的实体对象来响应，他只能结合 Service Port 来组成⼀个可以通信的服务。
>
> ### 定义Service
>
> 定义 Service 的⽅式和我们前⾯定义的各种资源对象的⽅式类型，例如，假定我们有⼀组 Pod 服务，它们对外暴露了 8080 端⼝，同时都被打上了 app=myapp 这样的标签，那么我们就可以像下⾯这样来定义⼀个 Service 对象：
>
> ```yaml
> #apiVersion: v1
> kind: Service
> metadata:
>   name: myservice  #Service的名称
> spec:
>   selector:
>     app: myapp   #Service代理具有app=myapp的pod
>   type: ClusterIP #Service的类型，默认是ClusterIP
>   ports:
>     - name: http-server #Service的端口名称
>       protocol: TCP #连接Service使用的协议
>       port: 80  #Service的端口
>       targetPort: 80 #映射的pod的端口 也可以是字符串 如http对应代理的pod的的端口名称
>       #nodePort： 30001 #nodePort会在集群上的每一个node节点都会开一个30001端口，然后我们就可以通过节点ip的这个端口访问我们的这个Service pod，前提是Service的type=NodePort,当Service的type=NodePort时，如果不指定nodePort的话会随机生成一个
>    
> ```
>
> 然后通过的使⽤ kubectl create -f myservice.yaml 就可以创建⼀个名为 myservice 的 Service 对象，它会将请求代理到使⽤ TCP 端⼝为 8080，具有标签 app=myapp 的 Pod 上，这个 Service 会被系统分配⼀个我们上⾯说的 Cluster IP ，该 Service 还会持续的监听 selector 下⾯的 Pod ，会把这些 Pod 信息更新到⼀个名为 myservice 的 Endpoints 对象上去，这个对象就类似于我们上⾯说的 Pod 集合了。
>
> 需要注意的是， Service 能够将⼀个接收端⼝映射到任意的 targetPort 。 默认情况下， targetPort 将被设置为与 port 字段相同的值。 可能更有趣的是，targetPort 可以是⼀个字符串，引⽤了 backend Pod 的⼀个端⼝的名称。 因实际指派给该端⼝名称的端⼝号，在每个 backend Pod 中可能并不相同，所以对于部署和设计 Service ，这种⽅式会提供更⼤的灵活性。
>
> 另外 Service 能够⽀持 TCP 和 UDP 协议，默认是 TCP 协议
>
> ### Service 类型
>
> 我们在定义 Service 的时候可以指定⼀个⾃⼰需要的类型的 Service ，如果不指定的话默认是 ClusterIP 类型。
>
> 我们可以使⽤的服务类型如下：
>
> * ClusterIP：通过集群的内部 IP 暴露服务，选择该值，服务只能够在集群内部可以访问，这也是默认的ServiceType。
> * NodePort：通过每个 Node 节点上的 IP 和静态端⼝（NodePort）暴露服务NodePort 服务会路由到 ClusterIP 服务，这个 ClusterIP 服务会⾃动创建。通过请求 :，可以从集群的外部访问⼀个NodePort 服务。
> * LoadBalancer：使⽤云提供商的负载局衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 NodePort 服务和 ClusterIP 服务，这个需要结合具体的云⼚商进⾏操作。
> * ExternalName：通过返回 CNAME 和它的值，可以将服务映射到 externalName 字段的内容（例如， foo.bar.example.com）。没有任何类型代理被创建，这只有 Kubernetes 1.7 或更⾼版本的kube-dns 才⽀持。
>
