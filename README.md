# 在 Kubernetes 上部署 Rook Ceph

**Rook** 是一种开源的云原生存储编排工具，提供了为云原生环境而生的平台框架级复杂存储解决方案。

> 当前部署的 Rook Ceph 版本为 **v1.11.9** 。
## 前提条件

* Kubernetes **v1.21** 或更高版本。
* 集群最少拥有3个节点用来部署 Rook Ceph 存储集群。
* 建议的最低内核版本为 **5.4.13** 。如果内核版本低于 5.4.13则需要升级内核。
* 存储集群的服务器之间需要进行同步时间，官方要求0.05秒，并且做定时任务。
* 要配置 Ceph 存储集群，存储至少需要以下选项之一：
  - 原始设备（无分区或格式化文件系统）
  - 原始分区（无格式化文件系统）
  - LVM 逻辑卷（无格式化文件系统）

- 安装lvm包

  - CentOS

    ```bash
    sudo yum install -y lvm2
    ```

  - Ubuntu

    ```bash
    sudo apt-get install -y lvm2
    ```

## 部署 Rook Ceph

### 部署 Rook Operator

```bash
$ git clone https://github.com/inative-io/rook-ceph.git
$ cd rook-ceph/v1.11.9
$ kubectl apply -f crds.yaml -f common.yaml -f operator.yaml
```

为Running则Operator部署成功

```bash
$ kubectl get pod -n rook-ceph
NAME                                 READY   STATUS    RESTARTS   AGE
rook-ceph-operator-b89545b4f-j64vk   1/1     Running   0          4m20s
```

### 创建 Ceph 集群

#### 给节点打标签

> {kube-node1,kube-node2,kube-node3} 此处应替换为当前集群节点的 node 名称。

1. 为运行ceph-mon的节点打上：ceph-mon=enabled

```bash
$ kubectl label nodes {kube-node1,kube-node2,kube-node3} ceph-mon=enabled
```

2. 为运行ceph-osd的节点，也就是存储节点，打上：ceph-osd=enabled

```bash
$ kubectl label nodes {kube-node1,kube-node2,kube-node3} ceph-osd=enabled
```

3. 为运行ceph-mgr的节点，打上：`ceph-mgr=enabled`

> ceph-mgr最多只能运行2个

```bash
$ kubectl label nodes {kube-node1,kube-node2} ceph-mgr=enabled
```

#### 创建集群

创建前需修改 `vim cluster.yaml` 中的 `storage.node` 字段中的节点名称及对应盘符，例如：

1. 如果您使用裸盘：

```yaml
storage:
  nodes:
  - name: "172.31.98.242"
    config:
      storeType: bluestore
    devices:
    - name: "vdb"
    - name: "vdc"
  - name: "172.31.98.243"
    config:
      storeType: bluestore
    devices:
    - name: "vdb"
    - name: "vdc"
  - name: "172.31.98.244"
    config:
      storeType: bluestore
    devices:
    - name: "vdb"
    - name: "vdc"
```

2. 如果您使用分区：

```yaml
storage:
  nodes:
  - name: "172.31.98.242"
    config:
      storeType: bluestore
    devices:
    - name: "vdb1"
    - name: "vdc1"
  - name: "172.31.98.243"
    config:
      storeType: bluestore
    devices:
    - name: "vdb1"
    - name: "vdc1"
  - name: "172.31.98.244"
    config:
      storeType: bluestore
    devices:
    - name: "vdb1"
    - name: "vdc1"
```

3. 如果您使用 LVM：

```yaml
storage:
  nodes:
  - name: "172.31.98.242"
    config:
      storeType: bluestore
    devices:
    - name: "/dev/vg/lv"
  - name: "172.31.98.243"
    config:
      storeType: bluestore
    devices:
    - name: "/dev/vg/lv"
  - name: "172.31.98.244"
    config:
      storeType: bluestore
    devices:
    - name: "/dev/vg/lv"
```

创建 rook ceph 集群
```bash
$ kubectl apply -f cluster.yaml
```

### 验证 Ceph 集群

通过命令行查看以下pod启动，表示成功：

```bash
$ kubectl get pod -n rook-ceph
```

```bash
NAME                                                     READY   STATUS      RESTARTS   AGE
csi-cephfsplugin-dxrzk                                   2/2     Running     0          4m3s
csi-cephfsplugin-lpt5s                                   2/2     Running     0          4m3s
csi-cephfsplugin-provisioner-6b98f5875d-b2fvz            5/5     Running     0          4m3s
csi-cephfsplugin-provisioner-6b98f5875d-t4s9x            5/5     Running     0          4m3s
csi-cephfsplugin-xwr99                                   2/2     Running     0          4m3s
csi-rbdplugin-bgf97                                      2/2     Running     0          4m3s
csi-rbdplugin-fpt6j                                      2/2     Running     0          4m3s
csi-rbdplugin-provisioner-665575874b-mpwn2               5/5     Running     0          4m3s
csi-rbdplugin-provisioner-665575874b-tqzl8               5/5     Running     0          4m3s
csi-rbdplugin-w7knx                                      2/2     Running     0          4m3s
rook-ceph-crashcollector-172.31.98.242-7d979cd6f-2mpkb   1/1     Running     0          87s
rook-ceph-crashcollector-172.31.98.243-cbd487598-68gq9   1/1     Running     0          84s
rook-ceph-crashcollector-172.31.98.244-fbfc6649f-rdrlg   1/1     Running     0          116s
rook-ceph-mgr-a-68fb998d66-n5bzd                         3/3     Running     0          2m13s
rook-ceph-mgr-b-6d6d794c75-246nx                         3/3     Running     0          2m10s
rook-ceph-mon-a-54f9fcd5-9slvj                           2/2     Running     0          5m37s
rook-ceph-mon-b-7d64475c8f-4mqbc                         2/2     Running     0          5m12s
rook-ceph-mon-c-6554987f-lwq2h                           2/2     Running     0          2m26s
rook-ceph-operator-866f998d48-6k26s                      1/1     Running     0          11m
rook-ceph-osd-0-7796655c86-lk4l5                         2/2     Running     0          88s
rook-ceph-osd-1-57c55b9894-pqspd                         2/2     Running     0          87s
rook-ceph-osd-2-55c8bb4577-2cx5h                         2/2     Running     0          84s
rook-ceph-osd-prepare-172.31.98.242-g6wrq                0/1     Completed   0          97s
rook-ceph-osd-prepare-172.31.98.243-ncfv5                0/1     Completed   0          96s
rook-ceph-osd-prepare-172.31.98.244-xwt5q                0/1     Completed   0          96s
```

安装工具箱，其中包含了 Ceph 集群管理所需要的命令行工具：

```bash
$ kubectl apply -f toolbox.yaml
$ kubectl get pod -l app=rook-ceph-tools -n rook-ceph
NAME                               READY   STATUS    RESTARTS   AGE
rook-ceph-tools-7c8b5c5cfc-gvlcf   1/1     Running   0          59s
```

使用如下命令确定 Ceph 集群状态：

```bash
$ kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
$ ceph status
  cluster:
    id:     8bb6bbd4-ec90-4707-85d7-903551d08991
    health: HEALTH_OK #出现该字样则集群正常

  services:
    mon: 3 daemons, quorum a,b,c (age 77s)
    mgr: b(active, since 27s), standbys: a
    osd: 6 osds: 6 up (since 37s), 6 in (since 56s)

  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   31 MiB used, 750 GiB / 750 GiB avail #磁盘总容量，确认是否与实际容量相同
    pgs:     1 active+clean
```

## 提供存储

Rook-ceph提供了三种类型的存储，三种存储类型在 Rainbond （版本要求不低于v5.7.0-release）中均可以对接使用。：

- **[Block](https://rook.io/docs/rook/v1.8/ceph-block.html)**：创建由 Pod (RWO) 使用的块存储。Rainbond 平台中的有状态类型服务组件，可以在添加存储时选择 `rook-ceph-block` 申请并挂载块存储。
- **[共享文件系统](https://rook.io/docs/rook/v1.8/ceph-filesystem.html)**：创建一个跨多个 Pod 共享的文件系统 (RWX)。安装 Rainbond 时可按照后文中的方式进行对接。Rainbond 平台中的服务组件在添加存储时选择默认的共享存储，即可对接 cephfs 共享文件系统。
- **[Object](https://rook.io/docs/rook/v1.8/ceph-object.html)**：创建一个可以在 Kubernetes 集群内部或外部访问的对象存储。该存储类型可以对接 Rainbond 的对象存储设置，为云端备份恢复等功能提供支持。

### 创建共享存储

为运行 mds 的节点添加标签：role=mds-node，通常为 Ceph 的三个节点

```bash
$ kubectl label nodes {kube-node1,kube-node2,kube-node3} role=mds-node
```

```bash
$ kubectl apply -f filesystem.yaml
$ kubectl get pod -l app=rook-ceph-mds -n rook-ceph
NAME                                        READY   STATUS    RESTARTS   AGE
rook-ceph-mds-sharedfs-a-f748485d7-7r4ps    2/2     Running   0          42s
rook-ceph-mds-sharedfs-b-867f554f9d-bcpjd   2/2     Running   0          41s
```

在 Rook 开始供应存储之前，需要根据文件系统创建一个 StorageClass。这是 Kubernetes 与 CSI 驱动程序互操作以创建持久卷所必需的。

```bash
$ kubectl apply -f storageclass-sharedfs.yaml
$ kubectl get sc
NAME          PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-cephfs   rook-ceph.cephfs.csi.ceph.com   Delete          Immediate           false                  72m
```

### 创建块存储

```bash
$ kubectl apply -f storageclass-block.yaml
$ kubectl get sc rook-ceph-block
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   22s
```

获取到 storageclass 后已创建成功，在平台上有状态组件可选择块存储进行使用。

### 创建对象存储

添加标签

```bash
# 添加节点标签
$ kubectl label nodes {kube-node1,kube-node2,kube-node3} rgw-node=enabled
# Create the object store
$ kubectl apply -f object.yaml
# To confirm the object store is configured, wait for the rgw pod to start
$ kubectl get pod -l app=rook-ceph-rgw -n rook-ceph
NAME                                       READY   STATUS    RESTARTS   AGE
rook-ceph-rgw-my-store-a-f67d85bf6-wjxfs   2/2     Running   0          53s
```

创建 StorageClass

```bash
$ kubectl apply -f storageclass-bucket-delete.yaml
```

创建 Bucket Claim

```bash
$ kubectl apply -f object-bucket-claim-delete.yaml
```

#### 验证对象存储

创建完成后，下面来验证。

```bash
$ kubectl get svc rook-ceph-rgw-my-store -n rook-ceph
NAME                     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
rook-ceph-rgw-my-store   ClusterIP   10.43.159.5   <none>        80/TCP    96s
# 出现以下返回即证明bucket创建完成
$ curl 10.43.159.5
<?xml version="1.0" encoding="UTF-8"?><ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/"><Owner><ID>anonymous</ID><DisplayName></DisplayName></Owner><Buckets></Buckets></ListAllMyBucketsResult>
```

```bash
# 获取连接信息
$ kubectl get cm ceph-bucket -n rook-ceph -o jsonpath='{.data.BUCKET_HOST}'
rook-ceph-rgw-my-store.rook-ceph.svc
$ kubectl get secret ceph-bucket -n rook-ceph -o jsonpath='{.data.AWS_ACCESS_KEY_ID}' | base64 --decode
UR4V1U3WZ84PKURSG4GV
$ kubectl get secret ceph-bucket -n rook-ceph -o jsonpath='{.data.AWS_SECRET_ACCESS_KEY}' | base64 --decode
AmVruTFoWJdjMD7QINZCekezh3hHdGCJUekyxDEn
# 获取bucket name
$ kubectl get ObjectBucketClaim ceph-bucket -n rook-ceph -o yaml|grep "bucketName: ceph-bkt"
  bucketName: ceph-bkt-88f7a768-226d-4ac2-8d6d-ee56825b6827
```

将该 `SVC` 地址在平台以第三方组件形式运行并打开对外访问策略，即可通过对外访问策略进行访问

```bash
$ kubectl get service rook-ceph-rgw-my-store -n rook-ceph
NAME                     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
rook-ceph-rgw-my-store   ClusterIP   10.43.159.5   <none>        80/TCP    7m2s
```

## 访问 Dashboard

获取 `svc`，在平台上使用第三方组件的形式代理

```bash
$ kubectl get service rook-ceph-mgr-dashboard -n rook-ceph
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
rook-ceph-mgr-dashboard   ClusterIP   10.43.250.183   <none>        7000/TCP   20m
```

访问到仪表板后，默认用户为`admin`，在服务器执行以下命令获取密码：

```bash
kubectl get secret rook-ceph-dashboard-password -n rook-ceph -o jsonpath="{['data']['password']}" | base64 --decode && echo
```
