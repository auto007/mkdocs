ls .kube/config

添加 Secret file 类型凭据
cat /etc/kubernetes/admin.conf

Harbor
添加用户名密码凭据

gitlab 私钥添加凭据

假设 k8s-node01 作为 Slave 节点：
```shell
kubectl label node k8s-node01 build=true
```

构建镜像时，需要使用 Docker，k8s-node01 安装docker环境


Manage Jenkins => Security => TCP port for inbound agents  50000

Manage Jenkins => System Configuration => Clouds 添加k8s集群