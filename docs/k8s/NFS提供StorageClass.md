Kubernetes 本身不会直接管理 NFS，但可以通过安装一个 **NFS Provisioner（动态存储提供器）** 来实现  
当 Pod 或 StatefulSet 申请 PVC 时，Provisioner 会自动在 NFS 上创建目录并返回给 Kubernetes 使用。

### nfs搭建
```shell
dnf install -y nfs-utils

mkdir -p /data/nfs
chmod -R 777 /data/nfs

vim /etc/exports
/data/nfs 192.168.1.0/24(rw,sync,no_root_squash)


#参数解释：
rw 读写
sync 同步写入（保证数据安全）
no_root_squash 客户端 root 用户拥有服务端目录 root 权限

systemctl enable --now nfs-server
systemctl enable --now rpcbind

#应用配置
exportfs -rv

#查看共享是否生效：
exportfs -s


firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload

#客户端安装
dnf install -y nfs-utils

mount -t nfs 192.168.1.100:/data/nfs /mnt/nfs
```

### 使用官方 NFS Subdir External Provisioner
```shell
#每个集群节点上安装 NFS 模块
yum install nfs-common -y

wget https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/releases/download/nfs-subdir-external-provisioner-4.0.18/nfs-subdir-external-provisioner-4.0.18.tgz

cd /root/mysql/nfs-subdir-external-provisioner
helm install nfs-storage .  --set nfs.server=192.168.124.63   --set nfs.path=/data/nfs

kubectl get storageclass

```

