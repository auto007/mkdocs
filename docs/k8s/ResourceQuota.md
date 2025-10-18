## ResourceQuota 是什么

`ResourceQuota` 是 **命名空间级别的资源配额管理工具**，用来控制一个 namespace 内所有 Pod、Service、PVC 等资源的**总量上限**。

它的作用是：

- 防止某个团队/用户在 namespace 里无限制创建资源，把整个集群搞挂。
    
- 和 `LimitRange` 搭配使用，既能限制单个 Pod 的资源，又能限制整个命名空间的总资源。
    

---

## 🔹 常见限制类型

1. **计算资源配额**
    
    - `requests.cpu`、`requests.memory`
        
    - `limits.cpu`、`limits.memory`  
        限制 namespace 内所有 Pod 请求和限制的总和。
        
2. **对象数量配额**
    
    - `pods`、`services`、`configmaps`、`secrets`、`persistentvolumeclaims` …  
        限制命名空间中某种对象的数量。
        
3. **存储相关配额**
    
    - `requests.storage`：PVC 总申请存储容量
        
    - `persistentvolumeclaims`：PVC 数量
        

---

## 🔹 示例 1：限制计算资源
```shell
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "2"        # 所有 Pod 请求 CPU 不能超过 2 核
    requests.memory: 4Gi     # 所有 Pod 请求内存不能超过 4Gi
    limits.cpu: "4"          # 所有 Pod 限制 CPU 不能超过 4 核
    limits.memory: 8Gi       # 所有 Pod 限制内存不能超过 8Gi

```
## 🔹 示例 2：限制对象数量
```shell
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-quota
  namespace: dev
spec:
  hard:
    pods: "10"                  # namespace 内最多 10 个 Pod
    configmaps: "20"            # 最多 20 个 configmap
    persistentvolumeclaims: "5" # 最多 5 个 PVC
    services: "10"              # 最多 10 个 Service

```
## 🔹 示例 3：限制存储资源
```shell
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-quota
  namespace: dev
spec:
  hard:
    requests.storage: "100Gi"          # 所有 PVC 总申请不能超过 100Gi
    persistentvolumeclaims: "10"       # 最多 10 个 PVC
    storageclass.standard.storageclass.storage.k8s.io/requests.storage: "50Gi"  # 指定存储类限制

```
## 验证效果

1. 创建 ResourceQuota：
    

`kubectl apply -f quota.yaml`

2. 查看配额情况：
    

`kubectl get resourcequota -n dev kubectl describe resourcequota compute-quota -n dev`

会显示：

`Resource                    Used  Hard --------                    ----  ---- requests.cpu                1     2 requests.memory             1Gi   4Gi limits.cpu                  2     4 limits.memory               3Gi   8Gi`

3. 如果超出配额，Pod 创建会失败：
    

`Error from server (Forbidden): exceeded quota: compute-quota, requested: limits.memory=2Gi, used: limits.memory=7Gi, limited: limits.memory=8Gi`

---

## 🔹 ResourceQuota vs LimitRange

- **LimitRange**：约束 **单个 Pod/Container** 的资源（默认值 + 最小/最大）。
    
- **ResourceQuota**：约束 **整个 namespace** 的资源总量。  
    👉 一般两个会配合使用：
    
- `LimitRange` 保证 Pod 有合理的 requests/limits。
    
- `ResourceQuota` 保证 namespace 不会无限制占资源。