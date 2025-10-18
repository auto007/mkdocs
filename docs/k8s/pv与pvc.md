- **PV**：是由管理员（或存储类 StorageClass）提供的存储资源，属于集群层级的资源。它就像是一个已经准备好的“硬盘”。
    
- **PVC**：是用户提出的存储申请（请求），Pod 不直接用 PV，而是通过 PVC 来申请 PV。它就像用户说“我需要一个 10Gi 的硬盘”。

### 一、PV（PersistentVolume）

- 表示一个实际存在的存储资源，比如：
    
    - 节点本地磁盘（hostPath）
        
    - NFS
        
    - Ceph
        
    - 云厂商的存储（AWS EBS、阿里云云盘等）
        
- PV 是 **静态创建** 或者通过 **StorageClass 动态创建**。
    

📌 示例（静态创建 PV）：

```shell
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany   # 多个节点可读写
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs
    server: 192.168.1.100

```

---

### 二、PVC（PersistentVolumeClaim）

- 用户通过 PVC 请求存储，比如大小、访问模式等。
    
- Kubernetes 会去匹配已有的 PV，如果符合条件就绑定。
    
- Pod 里挂载 PVC，就能用存储。
    

📌 示例（申请 PVC）：
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi

```
### 三、Pod 使用 PVC

Pod 不直接写 PV，而是通过挂载 PVC 来使用。

📌 Pod 示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-pvc
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: html-storage
  volumes:
    - name: html-storage
      persistentVolumeClaim:
        claimName: pvc-nfs

```

---

### 四、核心关系

1. 管理员提供 PV（存储资源池）。
    
2. 用户写 PVC（存储需求声明）。
    
3. K8s 匹配并绑定 PV ↔ PVC。
    
4. Pod 通过挂载 PVC 使用存储。



