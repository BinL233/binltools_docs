# 组件（Components）

- 原生组件
    
    ![截屏2024-09-29 17.36.39.png](/kubernetes_manual/images/截屏2024-09-29%2017.36.39.png)
    

## API Server（Master）

### 功能

1. API Server 是唯一和 etcd 通信的组件，任何组件需要与 etcd 交互都需要通过 API Server
2. 提供以 RESTful API 为基础的 CRUD interface 来查询和更改集群状态，这些状态存在 etcd 中

### 流程

1. Authentication（认证）
    - 通过检查 HTTP 请求来完成认证
    - 身份认证插件会获取：username、user ID、用户的 group
2. Authorization（鉴权）
    - 授权插件会确认用户是否有权限对请求的资源进行相应动作
3. Admission
    - 当请求是 Create/Update/Delete 时，这个请求会进入 Admission Control plugins
    - 这些插件可以用默认值填补缺失的 spec 或者覆盖 spec；可以修改相关资源
    - Admission Control plugins examples
        - AlwaysPullImages：覆盖 pod 内的 imagePullPolicy 到 Always
        - ServiceAccount：将默认的 Service Account 应用于没有声明 service account 的 pod
        - NamespaceLifecycle：防止 pod 创建时使用的 ns 正在删除或不存在
        - ResourceQuota：确保 pod 使用的 CPU 和 memory 的量
4. Validation
    - APIServer 验证 object，存入 etcd 中，并返回一个响应给ke hu

### 特性

1. API Server 可以拥有多个实例，可以做到同时并行
2. API Server 可以被直接部署到系统中，也可以以 Pod 的形式运行

### [localhost](http://localhost) 连接到 API Server

```go
// 获取 API Server 的 IP Addr，此时无法直接访问（无权限）
kubectl cluster-info 

// 开启代理，proxy 拥有权限
kubectl proxy

// 访问地址，可以连接了
curl 127.0.0.1:8001
```

## etcd（Master）

### 功能

1. etcd 是一个快速、分布式、稳定的 key-value 存储组件
2. 存储方式
    - v2
        - 等级制度（Hierarchical）的键空间，使 key-value pair 像一个文件系统。
        - 每一个 key 就是一个目录，内部包含了其他 key 或者 value
            
            ```yaml
            $ etcdctl ls /registry
            /registry/configmaps
            /registry/daemonsets
            /registry/deployments
            /registry/events
            /registry/namespaces
            /registry/pods
            ...
            ```
            
        - 使用轮询模式
            - HTTP/1.x + JSON
            - Client 通过 HTTP/1.1 协议长连接定时轮询（Watch） Server，获取最新的数据变化事件（客户端例如 Scheduler）
            - 缺点：如果 Watcher 太多会消耗大量内存、Socket等资源
    - v3
        - 使用推送模式
            - 使用 Protobuf（类 json，属于结构化数据，占用小，但是是二进制 less human-readable）
            - 使用 HTTP/2 的 gRPC 与 Server 进行通信
            - 具有双向流，Server 端可以推送 data
            - 可以多路复用，减少负载

### 特性

1. etcd 可以拥有多个实例，可以做到同时并行
2. etcd 可以被直接部署到系统中，也可以以 Pod 的形式运行
3. 只有 API Server 与 etcd 通信
    1. Robust optimistic locking system
    2. 验证 (Validation)
4. 一般 etcd 的实例个数为奇数
    - 偶数个节点集群不可用风险更高，表现在选主过程中，有较大概率获得等额选票，从而触发下一轮选举
    - 奇数个节点和偶数（奇数+1）个节点允许宕机的节点数是一样的，偶数个节点浪费资源
    - majority：超过半数节点接受投票的节点数量

## Scheduler（Master）

### 功能

- Scheduler 会监听没有绑定 node 的 pod，然后它会更新 pod 的定义，让 pod 和对应的 node 绑定（这里 Scheduler 没有让 node 去运行 pod，需要 kubelet）
- Scheduler 寻找 node 的方法
    
    ![截屏2024-10-08 11.39.29.png](/kubernetes_manual/images/截屏2024-10-08%2011.39.29.png)
    
    - 在选择最优 node 时，如果出现同分，则 Round-Robin
    - 选择 node 的条件
        - Hardware resources
        - Label 匹配 node selector
        - 绑定指定的 host port
        - 指定类型的 volume
        - tolerate taints
        - node and / or pod affinity or anti-affinity rules

### 特性

1. Scheduler 在同一时间只能运行一个实例
2. Scheduler 可以被直接部署到系统中，也可以以 Pod 的形式运行

## Controller Manager（Master）

### 功能

1. Controllers 会监听来自 API Server 提供的资源变动，并对每一个变动执行操作
2. Controllers 还会周期性执行 re-list 操作去确保没有丢失部分没有监听到的变动
3. Controllers 会运行 Reconciliation loop（调协循环），将 actual state 和 desired state 进行调谐

### 特性

1. Controller 之间不会有任何联系，也不知道彼此的存在，也不知道 kubelet 的存在
2. Controller 只会在 API Server 上更新资源
3. Controller Manager 在同一时间只能运行一个实例
4. Controller Manager 可以被直接部署到系统中，也可以以 Pod 的形式运行

### Controllers Cooperate

![截屏2024-10-11 13.43.51.png](/kubernetes_manual/images/截屏2024-10-11%2013.43.51.png)

### Controller Examples

- Replication Manager
    
    ![截屏2024-10-08 17.39.58.png](/kubernetes_manual/images/截屏2024-10-08%2017.39.58.png)
    
- ReplicaSet, DaemonSet, Job controllers
    - 类似 Replication Manager
- Deployment controller
    - 在 Replication Manager 的基础上增加了 Rollout 的功能
- StatefulSet controller
    - 类似 ReplicaSet
- Node controller
    - 管理的是 Worker nodes
    - 确保 nodes 的列表和实际机器上运行的 nodes 一致
    - 监视 node 的健康，从无法连接的 node 中驱逐 pods
- Service controller
    - 当 create/delete LoadBalancer Service 时，Service controller 是从基础设施中请求和释放load balancer 的控制器。
- Endpoint controller
    - Service 不是直接连接 pod 的，而是持有一个 endpoints（IP & Ports）的列表的
    - Endpoint Controller 负责确保 endpoint list 是更新的（pod 的 ip & port 与 label selector 匹配）
    - Endpoint Controller 同时监视 Service Resources 和 Pod Resources，并改变 Endpoint Resources
        
        ![截屏2024-10-08 17.53.25.png](/kubernetes_manual/images/截屏2024-10-08%2017.53.25.png)
        
- Namespace controller
    - 当一个 ns resource 被删除了，Namespace controller 确保这个 ns 下所有资源删除
- PersistentVolume controller
    - PersistentVolume controller 会帮创建好的 PVC 寻找合适的 PV
    - 当 PVC 删除时，还会让 PV 解绑并根据 reclaim policy 作出操作
- etc.

## Kubelet（Worker Node）

### 功能

1. 在 API Server 中创建 Node 资源
2. 监听 API Server，确保 pods 已经被调度到此 node 中，然后启动 pod 的容器（通过通知configured container runtime（Docker…））
3. 监听运行中的容器，向 API Server 报告状态、事件、资源消耗
4. 运行 Container liveness probes，当 probes 失败时重启容器
5. 当 pod 被终止时，终止容器
6. 可以不通过 API Server，直接使用本地的 Pod manifest 创建 pod

### 特性

1. Kubelet 只能直接部署在系统中
2. Kubelet 对 Worker node 中的一切负责

## kube-proxy（Worker Node）

全称：Kubernetes Service Proxy

### 功能

1. Kube-proxy 确保与 Service 的 IP 和 Port 的连接最终在支持该 Service 的其中一个 pod。当服务由多个 pod 支持时，proxy 会跨这些 pod 执行 Load Balancing。Kube-proxy 维护节点上的网络规则，使发往 Service 的流量（通过 ClusterIP 和端口）负载均衡到正确的后端 Pod。
2. iptables proxy mode：使用 iptables 的规则去重定向数据包到随机的 pod 中，而不是将数据包通过实际的 proxy server
    
    ![截屏2024-10-09 09.59.51.png](/kubernetes_manual/images/截屏2024-10-09%2009.59.51.png)
    
3. userspace mode：老版本。等待连接，并为每一个连接开启一个新的连接

### 特性

1. kube-proxy 可以被直接部署到系统中，也可以以 Pod 的形式运行
2. 是一个实际的代理

## Container Runtime（Worker Node）

## CSI （Add-on）

查看 [Kubernetes CSI](https://www.notion.so/Kubernetes-CSI-7ab6ca25a6ab416cae8482c5b7d280cb?pvs=21) 

## CNI（Add-on）

## DNS server（Add-on）

1. 所有的 pod 都默认使用集群内部的 DNS Server，这允许 pod 可以通过名字查找 Service
2. Service 的 ip 都记录在每个容器的 /etc/resolv.conf 文件中
3. kube-dns pod 使用 API Server 的监听机制来观察 Services 和 Endpoints 的更新，然后更新 DNS 记录

## Dashboard（Add-on）

## Ingress controller（Add-on）

1. 提供反向代理服务器（Reverse proxy server）
2. 确保服务器时刻为最新状态（需要监听 Ingress、 Services 和 Endpoints 的更新）
3. Ingress Controller 负责解析 Ingress 反向代理规则。在 Ingress Controller 收到请求后，它会根据 Ingress 的规则将请求转发到对应 Service 的 Pod 中

## Heapster（Add-on）

## Provisioner（Add-on）

用于建立 PV 和实际的 Volume（DIsk）

决定使用哪一个卷来制备 PV

![image.png](/kubernetes_manual/images/image.png)

### ProvisionController

- 提供**动态供应**
    
    ![截屏2024-09-24 14.08.11.png](/kubernetes_manual/images/截屏2024-09-24%2014.08.11.png)
    
    - 静态供应：需要手动在服务器上创建存储卷，然后要创建 PV，最后要创建 PVC 与 PV 绑定
    - 动态供应：用户无需关心PV资源创建以及存储服务器上的存储卷创建。ProvisionController 根据 StorageClass 资源中的信息以及 PVC 的请求容量，自动创建存储卷和 PV 资源

### CapacityController

![截屏2024-09-24 14.14.04.png](/kubernetes_manual/images/截屏2024-09-24%2014.14.04.png)

- ProvisionController 完成 PV 创建操作后，CapacityController 更新 CSIStorageCapacity 消耗容量
- ProvisionController 完成 PV 删除操作后，CapacityController 更新 CSIStorageCapacity 释放空闲容量

### **CloningProtectionController**

ProvisionController 在克隆存储卷时，会对 dataSource 指向的 PVC 设置克隆保护

之后，CloningProtectionController 会监听克隆保护的 PVC，发现目标 PVC 已完成克隆操作，则会删除克隆 finalizer 成员。这样可以避免 PVC 在进行克隆卷创建时，出现源 PVC 被删除的情况。

**Finalizer**：Finalizer 是带有命名空间的键，告诉 Kubernetes 等到特定的条件被满足后， 再完全删除被标记为删除的资源。 Finalizer 提醒[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)清理被删除的对象拥有的资源。

# 资源（Resources）

![截屏2024-09-23 11.39.28.png](/kubernetes_manual/images/截屏2024-09-23%2011.39.28.png)

## Pod

![未命名.png](/kubernetes_manual/images/未命名.png)

一个 Pod 下所有 Containers 共用一个 Linux Namespace 和 Network

### 创建 pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual
spec:
  containers:
  - image: luksa/kubia
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
```

### Pause Container

- Pod 中一定会有 Pause Container，而且它会比 pod 内其他容器更早起来
- Pause Container 让所有此 pod 中的容器都共用一个网络、一个 Linux Namespace
- 如果 Pause 被 kill 了，kubelet 会重新创建 Pause 和其他所有的容器

### Pod 在同个 Node 中通信

- Virtual Ethernet Interface Pair（veth pair） 服务于容器
- Veth pair 中，一个接口保留在 host 的 namespace 中；一个在容器的 network namespace 中
- 这两个虚拟接口像管道的两端，一边进了就从另一边出
- Network Packet 可以从容器中的接口传入，并到达 Bridge，最后可以送到任意一个连接到 Bridge 的容器中

![截屏2024-10-11 16.11.30.png](/kubernetes_manual/images/截屏2024-10-11%2016.11.30.png)

### Pod 在不同 Node 中通信

- 集群中，Pod IP Address 是必须唯一的，所以不会有 IP Address 冲突
- Node 的 Physical network interface 需要连接到 Bridge
- Node 中的 Routing Table 需要确保对应的 ip 地址 route 到目标 Node

![截屏2024-10-11 16.21.58.png](/kubernetes_manual/images/截屏2024-10-11%2016.21.58.png)

### Pod 绑核

- Kubelet 使用 CFS（完全公平策略）算法来为 Pod 分配 CPU
- kubelet 通过 Linux 的 cpusets 来实现 Pod 独占 CPU（即绑核）
- 可以避免跟其它 Pod 争抢 CPU 降低性能
- 操作方法：
    1. 驱逐 Node：`kubectl drain <NODE_NAME>` 
    2. 停止 Kubelet： `systemctl stop kubelet`
    3. 修改 Kubelet 参数：`--cpu-manager-policy="static”`
        - --cpu-manager-policy：
            - [`none`](https://kubernetes.io/docs/tasks/administer-cluster/cpu-management-policies/#none-policy)：默认策略，通过 Linux 默认的 CFS quota 实现 Guaranteed 和 Burstable 的 CPU 使用限制。
            - [`static`](https://kubernetes.io/docs/tasks/administer-cluster/cpu-management-policies/#static-policy)：允许 `requests` 中 CPU 为整数的 *Guaranteed* Pod 独占节点上的 CPU，通过 Linux cpuset cgroup 实现。
    4. 删除旧的 CPU 管理器状态文件：`rm var/lib/kubelet/cpu_manager_state`
    5. 启动 Kubelet：`systemctl start kubelet`

## Namespace

### 创建 Namespace

```yaml

apiVersion: v1
kind: Namespace
metadata:
  name: custom-namespace
```

或者：**`kubectl create namespace custom-namespace`**

## ReplicationController

### 操作流程：

![截屏2024-09-23 11.30.59.png](/kubernetes_manual/images/截屏2024-09-23%2011.30.59.png)

### 内部结构：

![截屏2024-09-23 11.32.59.png](/kubernetes_manual/images/截屏2024-09-23%2011.32.59.png)

确定标签，查看 replicas 是否符合数量，通过 pod template 创建新 pod

## RelipcaSet

### 创建 RelipcaSet

```yaml
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
```

## Deployment

启动 Deployment 时，Deployment 会启动 ReplicaSet

![截屏2024-09-29 15.47.57.png](/kubernetes_manual/images/截屏2024-09-29%2015.47.57.png)

### Deployment vs ReplicaSet

1. Deployment 拥有滚动更新的机制，而 ReplicaSet 没有
2. Deployment 可以回滚，ReplicaSet 不可以

### 回滚（Rolling Back）与 Revision

- Deployment 的 .spec.template 改动后，会创建一个 Revision
- Revision 用于 Rolling Back，可以回滚到之前的 revision
- 更改 revisionHistoryLimit 可以改变 Revision 的存储量

### 滚动更新（Rolling Update）

- rollingUpdate Strategy
    
    ```yaml
    spec:
      strategy:
        rollingUpdate:
          maxSurge: 1  # 允许 pod 数量超 Replica 设置的个数
          maxUnavailable: 0  # 允许 Unavaliable Pod 的数量，默认 25% pod 数量
        type: RollingUpdate
    ```
    

### 创建 Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubia
spec:
  replicas: 3  # 副本数量
  template:
    metadata:
      name: kubia
      labels:
        app: kubia
    spec:
      containers:
      - image: luksa/kubia:v1
        name: nodejs
```

## DaemonSet

![截屏2024-09-23 17.22.17.png](/kubernetes_manual/images/截屏2024-09-23%2017.22.17.png)

- DaemonSet 会保证每个 Node 有一个指定的 Pod
- 通过  `Node selector`  来指定需要放置 Pod 的 Nodes（Nodes 中添加 Label）

### 创建 DaemonSet

```yaml
apiVersion: apps/v1beta2
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disk: ssd
      containers:
      - name: main
        image: luksa/ssd-monitor
```

## StatefulSet

### StatefulSet vs Deployment & ReplicaSet & ReplicationController

- StatefulSet 内的 Pod 是有状态的，而另外三个资源的 Pod 是无状态的
    - 有状态指 StatefulSet 存有 Pod 的信息和状态
    - 当 Pod 消失时，StatefulSet 会创建一个新 Pod 并让该 Pod 持有原来 Pod 的信息和状态
    - 而另外三个资源的 Pod 是没有信息和状态的，它们会创建一个新的随机 Pod
- StatefulSet 通过给每个 Pod 名称后添加序列号来确保 Pod 拥有稳定的网络身份（Network Identity）
    - 并且当一个 Pod 消失了，它会创建一个新 Pod 并持有消失 Pod 的名字
    
    ![截屏2024-09-29 16.50.52.png](/kubernetes_manual/images/截屏2024-09-29%2016.50.52.png)
    
- StatefulSet Scaling
    - Scaling Up：新创建的 Pod 名字序号会是下一个没有用过的号码
    - Scaling Down：删除 Pod 的顺序是从序号高位到地位
- StatefulSet 拥有 Volume Claim Templates
    - 可以给每个 Pod 创建不同需求的 PVC

### 创建 StatefulSet

```yaml
kind: List
apiVersion: v1
items:
- apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-a
  spec:
    capacity:
      storage: 1Mi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Recycle
    gcePersistentDisk:
pdName: pv-a
      fsType: nfs4
- apiVersion: v1
  kind: PersistentVolume
  metadata:
name: pv-b
...
```

## Job

### 创建 Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    metadata:
      labels:
        app: batch-job
    spec:
      restartPolicy: OnFailure  # 不可以是 Always
			completions: 5  # 设置完成次数
		  parallelism: 2  # 设置并行 pods 数量
      containers:
      - name: main
        image: luksa/batch-job
```

## CronJob

### 创建CronJob

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
API group is batch, version is v1beta1
  name: batch-job-every-fifteen-minutes
spec:
  schedule: "0,15,30,45 * * * *“  # 分钟、小时、几号、几月、周几。
																	# 这里是每天每小时的 0、15、30、45 分
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: periodic-batch-job
        spec:
          restartPolicy: OnFailure
          containers:
          - name: main
            image: luksa/batch-job
```

## Service

### 特性

- Service 有自己稳定的 IP 地址和 Port 用于客户端来连接（一般是 Pod）
- 这个 IP 地址是虚拟的（VIP），没有实际连接到任何一个网络端口，它不会被列为 Destination IP Addr
- 当 Service 创建时，VIP 就立刻分配给它了
- API Server 会通知所有运行在 Worker Node 的 kube-proxy，Service 已经创建
- Kube-proxy 通过设置 iptables 规则来确保 Service 的 ip 地址可以连接到

### 创建Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
		app: kubia  # 所有 app=kubia 的 pod 才能使用这个 svc
```

### Pod 内部访问 Service

![截屏2024-09-23 17.47.48.png](/kubernetes_manual/images/截屏2024-09-23%2017.47.48.png)

### Cluster类型

1. ClusterIP（默认）：从 IP 地址池中分配一个 IP 地址
2. NodePort：额外分配一个端口，使得集群外部可以访问服务
3. LoadBalancer：外部请求会先通过 LB，来自 LB 的流量将被直接重定向到后端各个 Pod 上，云平台决定如何进行负载平衡
    1. 多个 svc 可以被一个 Ingress 暴露
        
        ![截屏2024-09-23 18.21.50.png](/kubernetes_manual/images/截屏2024-09-23%2018.21.50.png)
        

## Endpoint

![image.png](/kubernetes_manual/images/image%20(1).png)

## Volume（Pod / PersistentVolume）

### Non-persistent Storage

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune
spec:
containers:
- image: luksa/fortune
  name: html-generator
  volumeMounts:
  - name: html
    mountPath: /var/htdocs
- image: nginx:alpine
  name: web-server
  volumeMounts:
  - name: html
    mountPath: /usr/share/nginx/html
		readOnly: true
  ports:
  - containerPort: 80
    protocol: TCP
volumes:  # Volume 直接建立在 pod 内，会随着 pod 的消失而消失
- name: html
  emptyDir: {}
```

### Persistent Storage

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 1Gi  # 定义 PV 的大小
	accessModes:
	- ReadWriteOnce  # 允许单个用户读写
	- ReadOnlyMany  # 或者允许多个用户读
	persistentVolumeReclaimPolicy: Retain
	gcePersistentDisk:
		pdName: mongodb
		fsType: ext4
```

- PV 不属于任何 Namespace，所有 Namespace 中的 pod 都可以获取到

### Access Modes

- RWO - ReadWriteOnce
- ROX - ReadOnlyMany
- RWX - ReadWriteMany
- RWOP - ReadWriteOncePod

## PersistentVolumeClaim

![截屏2024-09-24 13.46.55.png](/kubernetes_manual/images/截屏2024-09-24%2013.46.55.png)

### 创建PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  resources:
    requests:
      storage: 1Gi  # 请求了 1GB 资源
  accessModes:
  - ReadWriteOnce  # 允许单个用户读写
  storageClassName: ""  # 指名 sc
												# 如果是空，则被视为要请求的是没有设置存储类的 PV 卷，因此这一 PVC 申领只能绑定到未设置存储类的 PV 卷
```

## StorageClass

- 不同的类型可能会映射到不同的服务质量等级或备份策略，相当于配置文件
- 需要包含 `provisioner`、`parameters` 和 `reclaimPolicy` 字段
- 你可以将某个 StorageClass 标记为集群的默认存储类
- 当一个 PVC 没有指定 `storageClassName` 时，会使用默认的 StorageClass
    - 改为默认：`kubectl patch storageclass <your-class-name> -p '{"metadata {"annotations":{"[storageclass.kubernetes.io/is-default-class":"true](http://storageclass.kubernetes.io/is-default-class%22:%22true)"}}}'`

### 创建 StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: low-latency
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: csi-driver.example-vendor.example
reclaimPolicy: Retain # 默认值是 Delete
allowVolumeExpansion: true
mountOptions:
  - discard # 这可能会在块存储层启用 UNMAP/TRIM
volumeBindingMode: WaitForFirstConsumer
parameters:
  guaranteedReadWriteLatency: "true" # 这是服务提供商特定的
```

## Lease

### Node Heartbeats

每个 Node在 `kube-node-lease` Namespace 中都有一个具有匹配名称的 Lease 对象。 在此基础上，每个 kubelet 心跳都是对该 `Lease` 对象的 **update** 请求，更新该 Lease 的 `spec.renewTime` 字段。

### Leader Election

确保在一个组件中，只有一个实例在运行。

一般用于 Controller 或 Scheduler。

### 查看 Lease YAML

```yaml
apiVersion: coordination.k8s.io/v1
kind: Lease
metadata:
  name: example-lease
  namespace: default
spec:
  holderIdentity: "" # 当前持有该 Lease 的实体
  leaseDurationSeconds: 15 # 超过 15 秒没有反应 lease 失效，表示持有者不再存活
  acquireTime: null # 租约被创建的时间
  renewTime: null # 租约的最后一次创建时间
  leaseTransitions: 0 # 租约持有者的更变次数
```

## Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
data:
  username: test
  password: 123456
```

### ConfigMap, Secret, emptyDir Graph

![截屏2024-09-27 16.42.17.png](/kubernetes_manual/images/截屏2024-09-27%2016.42.17.png)

## ServiceAccount

### 特性

- 一个 Pod 只可以使用同一个 namespace 下的 ServiceAccount
- 如果不给一个 Pod 声明 ServiceAccount，那么默认使用 namespace 下的 default ServiceAccount
- 用于和 Role/ClusterRole 绑定

## Role

### 特性

- 定义 verbs 以及针对的 resources

### 创建 Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1kind: Role
metadata:
  namespace: foo
  name: service-reader
rules:
- apiGroups: [""]
  verbs: ["get", "list"]
  resources: ["services"]
```

```bash
$ kubectl create -f service-reader.yaml -n foo

$ kubectl create role service-reader --verb=get --verb=list --resource=services -n bar
```

## RoleBinding

![截屏2024-10-25 11.27.28.png](/kubernetes_manual/images/截屏2024-10-25%2011.27.28.png)

- 将 Roles/ClusterRoles 与 users/groups/ServiceAccounts 绑定

### 创建 RoleBinding

```bash
$ kubectl create rolebinding test --role=service-reader --serviceaccount=foo:default -n foo
```

## Downward API (Volume)

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: downward
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "9999999"]
    resources:
      requests:
        cpu: 15m
        memory: 100Ki
      limits:
				cpu: 100m
        memory: 4Mi
       env:
       - name: POD_NAME
				 valueFrom:
				   fieldRef:
				     fieldPath: metadata.name  # referencing, not absolute value
			 - name: POD_NAMESPACE
				 valueFrom:
				   fieldRef:
				     fieldPath: metadata.namespace
			 - name: POD_IP
				 valueFrom:
				   fieldRef:
				     fieldPath: status.podIP
			 - name: NODE_NAME
				 valueFrom:
				   fieldRef:
				     fieldPath: spec.nodeName
			 - name: SERVICE_ACCOUNT
				 valueFrom:
				   fieldRef:
				     fieldPath: spec.serviceAccountName
			 - name: CONTAINER_CPU_REQUEST_MILLICORES
				 valueFrom:
				   resourceFieldRef:
				     resource: requests.cpu
			       divisor: 1m
			 - name: CONTAINER_MEMORY_LIMIT_KIBIBYTES
			   valueFrom:
		       resourceFieldRef:
				     resource: limits.memory
				  	 divisor: 1Ki
```

# 字段

## Label（标签可以自定义起）

![截屏2024-09-23 10.55.42.png](/kubernetes_manual/images/截屏2024-09-23%2010.55.42.png)

- app: 属于的类别
- rel: 当前的版本状态（stable, beta, canary）

Label 一个 node：**`kubectl label node test1 gpu=true`**

**根据 node 的 label 创建 pod （nodeSelector）：**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-gpu
spec:
  nodeSelector:
    gpu: "true"
  containers:
  - image: luksa/kubia
name: kubia
```

## Label vs Taint

- Label
    - Node 打上 `label`，yaml 内规定 `nodeSelector`
    - Pod 只会部署在对应的 Node
- Taint
    - Node 打上 `taint` ，yaml 内必须要有 `toleration` 才能把 Pod 部署到该 Node 上

## Annotation

没有像 Label 一样，可以通过 command 来查找过滤的功能。

用于添加 Description

## Liveness probe

通过三种方式检测 pod 健康状态：

1. HTTP GET
2. TCP Socket
3. Exec (发送指令，查看返回码是否为0)

### 创建 Liveness probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5  # 执行第一次探测前应该等待 5 秒
      periodSeconds: 5  # 每 5 秒执行一次
```

## Readiness Probe

### 创建Readiness Probe

```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Command & Args

### e.g.

```yaml
kind: Pod
  spec:
    containers:
    - image: some/image
      command: ["/bin/command"]
      args: ["arg1", "arg2", "arg3"]
```

### command

- 相当于 Docker 的 ENTRYPOINT，在容器内执行可执行的文件

### args

- 相当于 Docker 的 CMD，传递给可执行文件的参数

## Env

```yaml
kind: Pod
spec:
 containers:
 - image: luksa/fortune:env
   env:
   - name: INTERVAL
		 value: "30"
   name: html-generator
...
```

## ConfigMap

### 创建ConfigMap

1. 根据文件内容创建
    
    **`kubectl create configmap my-config --from-file=config-file.conf`**
    
    ![截屏2024-09-24 14.53.57.png](/kubernetes_manual/images/截屏2024-09-24%2014.53.57.png)
    
2. 根据 YAML 文件创建
**`kubectl create -f fortune-config.yaml`**

### Pod 使用 ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune-env-from-configmap
spec:
  containers:
  - image: luksa/fortune:env
		env:
		- name: INTERVAL  # 环境变量名称
			valueFrom:
			  configMapKeyRef:
			    name: fortune-config  # ConfigMap 名称
			    key: sleep-interval  # 对应 ConfigMap 中的 key
...
```

# 流程

## 创建资源

### 通过 kubectl 载入 YAML 文件创建 Deployment

![截屏2024-10-11 13.48.17.png](/kubernetes_manual/images/截屏2024-10-11%2013.48.17.png)

### 创建 Service

- PodA 想要发送 Packet 给 Service
- Packet 的目的地起初设置为 Service 的 IP 和 port（172.30.0.1:80）
- Packet 首先由 Node A 的 kernel 根据 iptables 规则进行处理
- 如果 packet 匹配了 iptables 上的规则，packet 的 destination IP 和 port 就会被更换为随机选中的 Pod 的 IP 和 port（例子中改为了 10.1.2.1:8080）

![截屏2024-10-12 10.50.04.png](/kubernetes_manual/images/截屏2024-10-12%2010.50.04.png)

# 指令

```go
// 创建新资源（重复配置会报错）（需要yaml/yml/json文件）
kubectl create -f <file's name>

// 配置应用于资源（重复配置不会报错）（需要yaml/yml/json文件）
kubectl apply -f <file's name> 

// 创建容器镜像
kubectl run <name> --image=<image's name> 

// 列出pods/nodes/ingresses/deployments/secrets/namespaces/services...
kubectl get pods / nodes / ingresses （ing） / deployments / secrets / namespaces (ns) / services (svc) ... 

// 列出所有pods
kubectl get pods -A 

// 列出pods并显示详细信息
kubectl get pods -o wide 

// 查看 yaml 文件
kubectl get po <name> -o yaml

// 列出指定namespace的pod
kubectl get pods --ns=<namespace> 

// 实时查看pods的信息
kubectl get pods --watch

// 查看 pod 并显示 label
kubectl get po --show-labels

// 查看带有 creation_method 和 env 标签的 pod
kubectl get po -L creation_method,env

// 查看标签有 creation_method=manual 的 pod
kubectl get po -l creation_method=manual

// 查看 k8s 组件状态
kubectl get componentstatuses

// 删除目标pod
kubectl delete pod <pod's name> 

// 显示pod详细信息
kubectl describe pod <pod's name> 

// 干进行，只会在客户端执行命令，但不会向Kubernetes API 发送实际的请求
-dry-run=client 

// 修改 pod 的镜像
kubectl set image POD/<POD_NAME> <CONTAINER_NAME>=<IMAGE_NAME>:<TAG> 

// help
kubectl explain po.spec

// get pod logs
kubectl logs <pod's name>

// get container logs
kubectl logs <container id>

// pod 端口转发
kubectl port-forward <pod name> 8888:8080

// 回滚到先前的 Deployment 版本
kubectl rollout undo deployment/abc
  
// 检查 Daemonset 的部署状态
kubectl rollout status daemonset/foo
  
// 重启 Deployment
kubectl rollout restart deployment/abc
  
// 重启带有 'app=nginx' 标签的 Deployment
kubectl rollout restart deployment --selector=app=nginx
```