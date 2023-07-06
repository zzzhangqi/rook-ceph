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
git clone https://github.com/inative-io/rook-ceph.git
$ cd v1.11.9
$ kubectl create -f crds.yaml -f common.yaml -f operator.yaml
```

为Running则Operator部署成功

```bash
$ kubectl get pod -n rook-ceph
NAME                                 READY   STATUS    RESTARTS   AGE
rook-ceph-operator-b89545b4f-j64vk   1/1     Running   0          4m20s
```

