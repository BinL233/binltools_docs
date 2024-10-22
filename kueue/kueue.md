Kueue 是一个 kubernetes 原生系统，用于管理 quotas 以及 jobs 如何消耗 quotas。Kueue 决定 job 什么时候应该等待、什么时候应允许 job 启动（如创建 pod）以及 job 什么时候应该抢占作业（如删除活动 pod）。
Kueue 的优势有：
- Kueue 不会替换任何 K8s 组件，易于部署。
- 通过 borrowing 和 preemption 语义来达到资源配额管理。
- 在异构集群内资源可互换。
- 支持多种 job： BatchJob、MPIJob、JobSet、Pod、RayJob...
- 支持自定义 job CRDs 通过 Extension points 和 Libraries。

![theory-of-operation](https://github.com/BinL233/my_docs/blob/main/kueue/images/theory-of-operation.png)

## 一、创建 Resource Flavor

Resource Flavor 允许我们定义一个对象来描述集群中可用的资源。通常 ResourceFlavor 与一组节点的特征相关联。它可以区分资源的不同特征，例如：
- 架构：x86、ARM
- 型号：V100S、T4S

Resource Flavor 允许使用 `labels`, `taints` 以及 `tolerations` 来和 Node 联系起来。
我们可以这样创建：

```YAML
apiVersion: kueue.x-k8s.io/v1beta1
kind: ResourceFlavor
metadata:
  name: "rf-test1"
spec:
  nodeLabels:
    kueue-test-type: "1"
  nodeTaints:
  - effect: NoSchedule
    key: kueue-test
    value: "true"
  tolerations:
  - key: "kueue-test-taint"
    operator: "Exists"
    effect: "NoSchedule"
---
apiVersion: kueue.x-k8s.io/v1beta1
kind: ResourceFlavor
metadata:
  name: "rf-test2"
spec:
  nodeLabels:
    kueue-test-type: "2"
```
**解释：**
- `.spec.nodeLabels` （需要 K8s 版本大于等于 1.23）
  - 这个字段用于匹配 Node 中的 `labels` 字段。
  - 当 Workload 进入 ClusterQueue 时，用于确认此 Workload 是否和这个 ResourceFlavor 对应的 Node 匹配。
- `.spec.nodeTaints`
  - 这个字段用于匹配 Node 中的 `taints` 字段。
  - 和 `.spec.nodeLabels` 目的基本一致。

如果集群中有同质资源（Homogeneous resources），或者不需要单独管理不同类型资源的配额，可以创建一个空的 ResourceFlavor。这个 ResourceFlavor 不含有任何 `labels` 以及 `taints`，例如：

```YAML
apiVersion: kueue.x-k8s.io/v1beta1
kind: ResourceFlavor
metadata:
  name: rf-default
```


## 二、创建 Cluster Queue

ClusterQueue 用于管理资源，可以定义：
- 资源使用限制（Usage limit）：管理 ResourceFlavor 的资源配额、使用限制以及用量大小。
- 公平共享规则（Fair sharing rules）：在集群中的多个 ClusterQueue 的公平分享规则。

ClusterQueue 是集群范围的组件，用于管理资源，例如：pods，CPU，内存以及硬件加速器等。

这是 ClusterQueue、ResourceFlavor、Cohort 以及 Node 的关系图：
![cluster-queue.svg](https://github.com/BinL233/my_docs/blob/main/kueue/images/cluster-queue.svg)

Cohort 是一个组，由 ClusterQueues 组成。在同一个 Cohort 中，ClusterQueue 可以相互借用未使用的资源。ClusterQueue 可以给不同的 flavor 设置不同的配额（CPU, memory, GPUs, pods 等等）。Flavor 代表一个资源的不同变种（例如不同的 GPU 型号）。我们通过 ResourceFlavor 来定义 Flavor 如何映射到一组 Node 中，我们还可以为每一个 Flavor 定义提供的资源量大小，创建如下：

```YAML
apiVersion: kueue.x-k8s.io/v1beta1
kind: ClusterQueue
metadata:
    name: "cluster-queue-1"
spec:
    namespaceSelector: {} # 匹配所有 namespace
    cohort: "cohort-1" # 定义所属 cohort
    resourceGroups:
      - coveredResources: ["cpu", "memory", "pods"] # 设置资源组（Resource Group）
        flavors:
          - name: "rf-test1"
            resources:
              - name: "cpu"
                nominalQuota: 5 # 限制 CPU 请求总量 <= 5
              - name: "memory" 
                nominalQuota: 6Gi # 限制内存请求总量 <= 6Gi
              - name: "pods"
                nominalQuota: 10 # 限制 Pod 请求总量 <= 10
          - name: rf-test2
            resources:
              - name: cpu
                nominalQuota: 3
              - name: memory
                nominalQuota: 6Gi
              - name: pods
                nominalQuota: 10
```

这其中，对于资源请求涉及几个新的字段：
- `nominalQuota`：代表在一个特定时间，ClusterQueue 可以使用的资源量。
- `borrowingLimit`：代表这个 ClusterQueue 可以从其他 ClusterQueue 未使用的 nominal quota 内能够借用的最大资源量。（必须在同一个 Cohort 中）
- `lendingLimit`：代表这个 ClusterQueue 可以允许同一个 Cohort 中其他 ClusterQueue 从自己未使用的 nominal quota 中能够借用的最大资源量。（必须在同一个 Cohort 中）

现在我们通过查看已创建的 ClusterQueue 的 YAML，我们可以看到一些新的字段：

- `.spec.namespaceSelector`：

  我们可以通过这个字段来限制哪些 namespaces 可以在 ClusterQueue 中接纳 Workloads：
  ```YAML
  namespaceSelector:
    matchLabels:
      kubernetes.io/metadata.name: team-a
  ```
  我们甚至可以通过给 Namespace 资源添加 label 并让 `matchLabels` 一次匹配多个 namespaces：
  ```YAML
  # 给 team-a 添加 label
  apiVersion: v1
  kind: Namespace
  metadata:
    name: team-a
    labels:
      research-cohort: team-a-b
  ```
  ```YAML
  # 给 team-b 添加 label
  apiVersion: v1
  kind: Namespace
  metadata:
    name: team-a
    labels:
      research-cohort: team-a-b
  ```
  ```YAML
  # 一次性匹配
  namespaceSelector:
    matchLabels:
      research-cohort: team-a-b
  ```

- `.spec.queueingStrategy`

  我们可以通过z这个字段设置不同的队列策略。QueueingStrategy 决定了 ClusterQueue 中的 workloads 是如何排列的，以及它们是如何在准入（Admission）尝试失败后重新加入队列的。目前有2个策略：
  - `StrictFIFO`：Workloads 首先按照优先级排序，然后根据 `.metadata.creationTimestamp`。无法被准入（Admitted）的 workloads 将会阻塞新的 workloads。
  - `BestEffortFIFO`（默认）：Workloads 排序和 `StrictFIFO` 一样，但是无法被准入的（Admitted）的 workloads 不会阻塞符合可用配额的新 workloads。

- `.spec.preemption`

  当一个 ClusterQueue 或者它的 Cohort 没有足够资源配额时，即将到来的 workload 会根据 ClusterQueue 中的政策抢占（Preemption）之前准入的 workloads。抢占的定义如下：
  - `reclaimWithinCohort`：待处理的 Workload 是否可以从同一个 cohort 中其他 ClusterQueues 中抢占多于自身 nominal quota 的 workload：
      - `Never`*（默认）*
      - `LowerPriority`
      - `Any`
  - `borrowWithinCohort`：当待处理的 workload 需要借用时，是否可以从同一个 cohort 中其他 ClusterQueues 中抢占 workload：
      - `Never`*（默认）*
      - `LowerPriority`
  - `withinClusterQueue`：当一个没有符合 ClusterQueue 声明的 nominal quota 的待处理 Workload，是否可以占用这个 ClusterQueue 中其他活跃的 Workloads：
      - `Never`*（默认）*
      - `LowerPriority`
      - `LowerOrNewerEqualPriority`

- `.spec.flavorFungibility`

  当 ResourceFlavor 中的资源 nominal quota 不足时，传入的 workload 可以借用配额，或者抢占 ClusterQueue 或 Cohort 中正在运行的 workload。Kueue 按顺序评估 ClusterQueue 中的 flavor。通过设置 `flavorFungibility` 字段，您可以决定在尝试容纳下一个 flavor 中的 Workload 之前，是否优先考虑抢占或借用某个 flavor。
  - `whenCanBorrow`：如果 workload 可以通过借用当前 ResourceFlavor 内获得足够的资源，它是否应停止寻找更好的作业分配：
    - `Borrow`*（默认）*
    - `TryNextFlavor`
  - `whenCanPreempt`：workload 是否应在尝试下一个 ResourceFlavor 之前尝试在当前 ResourceFlavor 中抢占：
    - `Preempt`
    - `TryNextFlavor`*（默认）*

- `.spec.stopPolicy`

  通过设置此字段，可以允许集群管理员暂时定制 workload 的准入：
  - `Hold`
  - `HoldAndDrain`
  - `None`*（默认）*

## 三、创建 Local Queue

LocalQueue 是一个在 namespace 下的队列。它只能在一个 namespace 下。一个 LocalQueue 指向一个 ClusterQueue ，从中分配资源去运行 Workloads。

我们现在来创建一个 LocalQueue：

```YAML
apiVersion: kueue.x-k8s.io/v1beta1
kind: LocalQueue
metadata:
    namespace: team-a # 指定一个已创建的 namespace
    name: team-a-queue
spec:
    clusterQueue: cluster-queue-1
```
    
用户需要提交 jobs 到 LocalQueue 而不是直接到 ClusterQueue。

## 四、创建 Job

现在，我们来创建一个 Job 交给 Kueue 执行。需要注意几点：
- Job 必须设置为 suspended 状态。
- 需要给 Job 设置队列。
- 对于每一个 Job pod，需要包含资源请求。

按照这个创建：
```YAML
apiVersion: batch/v1
kind: Job
metadata:
  generateName: sample-job
  namespace: team-a
  labels:
    kueue.x-k8s.io/queue-name: team-a-queue # 定义想要放入的队列
spec:
  parallelism: 3
  completions: 3
  suspend: true # 一定要为 true
  template:
    spec:
      containers:
      - name: dummy-job
        image: uhub.service.ucloud.cn/yingkaile/sleep:latest
        args: ["30s"]
        resources:
          requests:
            cpu: 1
            memory: "200Mi"
      restartPolicy: Never
```
ueue 会提取我们的 job 信息，然后创建成 Workload。
创建完成之后，我们输入指令 `kubectl -n team-a get workloads`，得到：
```bash
NAME                        QUEUE          RESERVED IN       ADMITTED   FINISHED   AGE
job-sample-jobfrjsv-c9d31   team-a-queue   cluster-queue-1   True                  5m53s
```
至此，我们使用 Kueue 完成了 Job。


## 五、创建 Workload

Workload 是一个应用，它会运行到完成。它是 Kueue 中准入的单位。Workload 由一个或多个 pods 组成。Kueue 不会直接操作 Job，而是通过管理 workload 来间接性操作 Job。**Kueue 会为每一个 job 创建一个 workload。**

创建以下 workload：
```YAML
apiVersion: kueue.x-k8s.io/v1beta1
kind: Workload
metadata:
  name: sample-workload
  namespace: team-a
spec:
  active: true # 是否要继续运行这个 workload
  queueName: team-a-queue # 指明 Workload 进入哪一个 LocalQueue
  podSets: # Workload 可以由多个 pods 组成，就会有多个 podSpec 需要声明
    - count: 3
      name: main
      template:
        spec:
          containers:
            - image: uhub.service.ucloud.cn/yingkaile/sleep:latest
              imagePullPolicy: Always
              name: container
              resources:
                  requests:
                      cpu: "1"
                      memory: 200Mi
          restartPolicy: Never
```

创建好之后，我们可以通过 `kubectl get -n team-a localqueues` 或者 `kubectl get -n team-a queues` 来列出 namespace 中的 LocalQueue 以及绑定的 workload：
```bash
$ kubectl get -n team-a localqueues
NAME           CLUSTERQUEUE      PENDING WORKLOADS   ADMITTED WORKLOADS
team-a-queue   cluster-queue-1   0                   1
```

我们输入指令 `kubectl -n team-a get workloads`，得到：
```bash
NAME                        QUEUE          RESERVED IN       ADMITTED   FINISHED   AGE
sample-workload             team-a-queue   cluster-queue-1   True                  2m5s
```

## 七、优先级和抢占
- **Case 1**: 有两个 ClusterQueue (team-a-cq & team-b-cq) 在同一个 cohort, yaml 中设置了 `borrowingLimit`
    
    ```YAML
    apiVersion: kueue.x-k8s.io/v1beta1
    kind: ClusterQueue
    metadata:
      name: team-a-cq
    spec:
      cohort: all-teams
      resourceGroups:
      - coveredResources: ["cpu"]
        flavors:
        - name: default
          resources:
          - name: cpu
            nominalQuota: 40
            borrowingLimit: 20
      preemption:
        reclaimWithinCohort: Any
    ```
    
    ![cohortPreemption.png](https://github.com/BinL233/my_docs/blob/main/kueue/images/cohortPreemption.png)
    
  team-a-cq 的 `borrowingLimit` 为 20，它可以向其他 ClusterQueue 借用 20 cpu，总数可达到 60。
  如果 team-b-cq 需要它自己原本的 10 cpu，team-a-cq 需要释放借来的资源。

- **Case 2**: 有两个 ClusterQueue (team-a-cq & team-b-cq) 在同一个 cohort, yaml 中没有设置 `borrowingLimit`
    
    ```YAML
    apiVersion: kueue.x-k8s.io/v1beta1
    kind: ClusterQueue
    metadata:
      name: team-b-cq
    spec:
      cohort: all-teams
      resourceGroups:
        - coveredResources: ["cpu"]
          flavors:
          - name: default
            resources:
            - name: cpu
              nominalQuota: 40
      preemption:
        reclaimWithinCohort: Any
        withinClusterQueue: LowerPriority
    ```
    
    ![Prioity.png](https://github.com/BinL233/my_docs/blob/main/kueue/images/Prioity.png)

  team-a-cq 没有设置 `borrowingLimit`，无法向 team-b-cq 借用资源。
  team-a-cq 只能替换自己的 Workload。
  根据每个 Workload 的 priority 替换低优先级的 Workload。
