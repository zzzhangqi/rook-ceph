# 在 Kubernetes 上部署 Rook Ceph

**Rook** 是一种开源的云原生存储编排工具，提供了为云原生环境而生的平台框架级复杂存储解决方案。

> 当前部署的 Rook Ceph 版本为 **v1.11.9** 。
## 前提条件

* Kubernetes **v1.21 **或更高版本。
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

3. 为运行ceph-mgr的节点，打上：ceph-mgr=enabled

> ceph-mgr最多只能运行2个

```bash
$ kubectl label nodes {kube-node1,kube-node2} ceph-mgr=enabled
```

#### 创建集群

创建前修改 `storage.node`字段中的节点名称及对应盘符

```bash
$ kubectl apply -f cluster.yaml
```

### 验证 Ceph 集群

通过命令行查看以下pod启动，表示成功：

```bash
$ kubectl get pod -n rook-ceph
```

```bash
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
$ c
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
