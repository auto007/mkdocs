## 场景需求

在 `dev` 命名空间里，我们希望：

1. 每个 Pod/Container 必须合理配置资源：
    
    - CPU：最少 100m，最多 1 核
        
    - 内存：最少 128Mi，最多 1Gi
        
    - 如果没写 requests/limits，就默认给：requests=200m/256Mi，limits=500m/512Mi
        
2. 整个 namespace 的总资源：
    
    - requests.cpu ≤ 4 核
        
    - requests.memory ≤ 8Gi
        
    - limits.cpu ≤ 8 核
        
    - limits.memory ≤ 16Gi
        
    - Pod 总数 ≤ 20
        

---

## 🔹 YAML 配置

### 1. LimitRange（单 Pod 约束 + 默认值）

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limit-range
  namespace: dev
spec:
  limits:
  - type: Container
    max:
      cpu: "1"
      memory: "1Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
    defaultRequest:
      cpu: "200m"
      memory: "256Mi"
    default:
      cpu: "500m"
      memory: "512Mi"

```

### 2. ResourceQuota（命名空间总量约束）

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-resource-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"

```

---

## 🔹 验证过程

1. 创建 namespace：
    

`kubectl create ns dev`

2. 应用配置：
    

`kubectl apply -f limitrange.yaml kubectl apply -f resourcequota.yaml`

3. 创建一个 **没写资源的 Pod**：
    

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx

```

4. 查看 Pod 的实际资源：
    

`kubectl get pod nginx-test -n dev -o yaml | grep -A6 resources:`

你会看到自动补上：

```yaml
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

```

5. 再查看 namespace 的配额情况：
    

`kubectl describe quota dev-resource-quota -n dev`

会显示类似：

```shell
Resource        Used   Hard
--------        ----   ----
pods            1      20
requests.cpu    200m   4
requests.memory 256Mi  8Gi
limits.cpu      500m   8
limits.memory   512Mi  16Gi

```

如果创建 Pod 超过配额，就会报错：

`Error from server (Forbidden): exceeded quota: dev-resource-quota, requested: limits.memory=2Gi, used: limits.memory=15Gi, limited: limits.memory=16Gi`

---

✅ **总结：**

- `LimitRange` 保证 **单 Pod 有合理的默认资源配置**。
    
- `ResourceQuota` 保证 **整个命名空间不会超量使用资源**。
    

这样就能既防止“一个 Pod 独占资源”，也防止“整个 namespace 把集群资源吃光”。