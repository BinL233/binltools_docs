## Cloud Disk

### Workflow

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
      - The provisioner in the Controller watches for PVCs. When a new PVC appears, the Controller creates a volume.
      - The provisioner then creates a corresponding PV based on the PVC’s request.

2. Controller Publish Volume
      - The Controller attaches the volume, establishing the connection between the cloud disk and the Node.

3. Node Stage Volume
      - Check whether the disk is already formatted. If not, it formats the disk.
      - The Node mounts the device to the `globalmount`.

4. Node Publish Volume
      - The Node bind-mounts the `globalmount` point to the `/data` path inside the Pod.

### Topology (topology-aware dynamic provisioning**)**

- Ensure that each Pod and its attached Cloud Disk are in the same Availability Zone.

- When creating a cluster or adding new nodes, each Node is labeled with a Region and Zone to indicate its geographic region and availability zone.

- When creating a Cloud Disk, the CSI Controller uses the Region and Zone information to ensure that the Pod and its Cloud Disk are in the same Region and Zone.

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
      - The provisioner in the Controller watches for PVCs. When a new PVC appears, the Controller creates a volume.
      - The provisioner then creates a corresponding PV based on the PVC’s request.

2. Node Publish Volume
      - The Node mounts the `S3` bucket to the `/data` path in the Pod using the `s3fs` protocol.

### s3fs

S3 is an object storage service (key-value based), and s3fs is a file system interface that allows exporting buckets from object storage as a file system.

It helps users mount an S3 bucket to the local file system, making it as convenient to operate on data in the bucket as if they were working with local files.

## NFS vs Samba

### NFS (Network File System)
- Communication model: Between an NFS client and an NFS server.

    - The client uses Remote Procedure Calls (RPC) to request files or directories from the server.

    - The server then checks:

      - Whether the file or directory is available.

      - Whether the client has the required access permissions.

- The server mounts the file or directory remotely on the client and shares access through a virtual connection.
- A stateful file sharing protocol based on Unix systems.

### Samba
- A default file sharing protocol for DOS and Windows operating systems.

- Communication model: The client-server communication process is generally similar to NFS.
  - However, the file system is not mounted locally on the SMB client. Instead, the client accesses the network share hosted on the SMB server via a network path.

### Similarities
- Both operate in a client-server model.

- Both support CRUD operations.

- Both can be used across different operating systems.

### Differences
- Shared Resources

  - Samba can share various network resources (files, print services, storage devices, virtual machine storage).

  - NFS only supports sharing files and directories.

- Client Communication

  - Samba allows clients to communicate with each other and share files via the server acting as an intermediary.

  - NFS only supports direct client-server interactions.
