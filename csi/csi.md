## Cloud Disk

### 流程

```yaml
   CreateVolume +------------+ DeleteVolume
 +------------->|  CREATED   +--------------+
 |              +---+----^---+              |
 |       Controller |    | Controller       v
+++         Publish |    | Unpublish       +++
|X|          Volume |    | Volume          | |
+-+             +---v----+---+             +-+
                | NODE_READY |
                +---+----^---+
               Node |    | Node
              Stage |    | Unstage
             Volume |    | Volume
                +---v----+---+
                |  VOL_READY |
                +---+----^---+
               Node |    | Node
            Publish |    | Unpublish
             Volume |    | Volume
                +---v----+---+
                | PUBLISHED  |
                +------------+
```

1. Controller Create Volume
    1. Controller 的 Provisioner 监听 PVC，当有新 PVC 出现时，Controller Create Volume。
    2. 随后 Provisioner 根据 PVC 的请求创建对应 PV。
2. Controller Publish Volume
    1. Controller attach volume，完成云盘和 Node 之间的连接。
3. Node Stage Volume
    1. 先检查 disk 是否已经格式化，如果没有则格式化。
    2. Node 将设备 mount 到 globalmount。
4. Node Publish Volume
    1. Node 将 globalmount 和 Pod 中 /data 路径 bind-mount。

### Topology (topology-aware dynamic provisioning**)**

- 保证每个 Pod 与其所挂载的 Cloud Disk 在同一个可用区。
- 在创建集群和新增节点时，会对每个 Node 添加 Region 和 Zone，目的是确定该 Node 的地域和可用区。
- 在创建 Cloud Disk 时，CSI Controller 会通过 Region 和 Zone，确保 Pod 和它挂载的 Cloud Disk 在同一个 Region 和同一个 Zone。

## S3

```yaml
   CreateVolume +------------+ DeleteVolume
 +------------->|  CREATED   +--------------+
 |              +---+----^---+              |
 |             Node |    | Node             v
+++         Publish |    | Unpublish       +++
|X|          Volume |    | Volume          | |
+-+                 |    |                 +-+
                +---v----+---+
                | PUBLISHED  |
                +------------+
```

1. Controller Create Volume
    1. Controller 的 Provisioner 监听 PVC，当有新 PVC 出现时，Controller Create Volume。
    2. 随后 Provisioner 根据 PVC 的请求创建对应 PV。
2. Node Publish Volume
    1. Node 将 s3 bucket 通过 s3fs 协议 mount 到 Pod 中 /data 路径中。

### s3fs 协议

S3 是对象存储（key-value），s3fs 是可以将对象存储中的 bucket 以文件形式导出的文件系统接口。

帮助用户把 S3 bucket mount 到本地文件系统，使得用户操作存储桶里的数据就如同操作本地文件一样方便。

## NFS vs Samba

### NFS (Network File System)

- 基于 Unix 系统的有状态文件共享协议。
- 通信方式：NFS 的客户端与 NFS 服务器。
    - 客户端使用远程过程调用（RPC）向服务器请求文件或目录。
    - 然后，服务器会检查：
        - 文件或目录是否可用。
        - 客户端是否具有所需的访问权限。
    - 服务器将文件或目录远程安装在客户端上，并通过虚拟连接共享访问权限。

### Samba

- 适用于 DOS 操作系统，Windows 操作系统的默认文件共享协议。
- 通信方式：客户端-服务器通信的过程大体上与 NFS 类似。
    - 文件系统不安装在本地 SMB 客户端，而是通过网络路径访问托管在 SMB 服务器上的网络共享。

### 相似点

- 都采用客户端-服务器模式运行。
- 都允许 CRUD。
- 可以在多个不同的操作系统上使用。

### 不同点

- 共享资源
    - Samba 可以共享各种网络资源（文件和打印服务、存储设备和虚拟机存储）。
    - NFS 只支持共享文件和目录。
- 客户端通信
    - Samba 允许客户端使用服务器作为调解器相互通信和共享文件。
    - NFS 仅允许客户端-服务器操作。
