`LimitRange` 是 Kubernetes 里的 **资源限制策略对象**，用来给 **Pod 或 Container** 在 **命名空间级别**上设定默认的 CPU / 内存请求（request）、限制（limit），以及资源使用的最大最小值。

它的作用可以总结为三点：

1. **默认值**：如果 Pod/Container 没写 request/limit，就用 LimitRange 里配置的默认值。
    
2. **上下限**：防止某个容器申请的资源过大或过小。
    
3. **约束规则**：保证一个命名空间内的 Pod 有统一的资源使用规范。
    

---

### 1️⃣ 定义一个 LimitRange 示例

假设我们有个命名空间 `dev`，我们希望：

- 每个容器至少要申请 100m CPU，最多 1 CPU。
    
- 每个容器内存至少 128Mi，最多 1Gi。
    
- 如果没写 request/limit，就自动分配默认值：CPU=200m，内存=256Mi。
    

可以这样写 YAML：

```shell
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
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
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "200m"
      memory: "256Mi"

```

---

### 2️⃣ 验证效果

1. 创建 LimitRange：
    

`kubectl apply -f limitrange.yaml`

2. 新建一个 Pod **不写资源**：
    
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx

```

3. 查看 Pod 实际资源：
    

`kubectl get pod test-pod -n dev -o yaml | grep -A 6 resources:`

你会发现 Pod 自动被加上了：

```shell
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

```

---

### 3️⃣ 常见使用场景

- **多人共用集群**：防止某人写个 Pod 要 100 核 CPU 把集群占光。
    
- **强制规范**：比如要求所有 Pod 最少写 `128Mi` 内存 request，否则不允许创建。
    
- **默认保障**：开发人员省略资源配置时，系统帮忙填好。


### 疑问
你问的这个

```shell
default:
  cpu: "500m"
  memory: "512Mi"

```

就是 **LimitRange 给 Pod/Container 自动填充的默认资源限制（limit）**。

---

### 🔹 解释：

- `default` 表示：如果 Pod 里 **没有写 limit**，Kubernetes 会自动帮你加上这里定义的值。
    
- 上面这段意思是：
    
    - 默认 CPU 限制是 **500m** （= 0.5 核）
        
    - 默认内存限制是 **512Mi**
        

换句话说，`default` 会填到 Pod 的 `resources.limits` 里。

---

### 🔹 对比一下几个字段：

- `min`：要求容器的 **request/limit** 不能低于这个值。
    
- `max`：要求容器的 **request/limit** 不能高于这个值。
    
- `defaultRequest`：如果没写 `requests`，自动填这个值。
    
- `default`：如果没写 `limits`，自动填这个值。
    

---

### 🔹 举个例子

#### 定义的 LimitRange：

```shell
defaultRequest:
  cpu: "200m"
  memory: "256Mi"
default:
  cpu: "500m"
  memory: "512Mi"

```

#### Pod YAML（没写资源）：

```shell
containers:
- name: nginx
  image: nginx

```

#### Kubernetes 自动加上的效果：

```shell
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

```

### default defaultRequest 为什么要引入2个呢
这个问题问得非常好 👍，很多人第一次接触 `LimitRange` 时都会疑惑：  
**为什么要有 `default` 和 `defaultRequest` 两个字段，不是一个就够了吗？**

---

## 🔹 背景：K8s 里有两个概念

1. **requests**：  
    Pod 调度时使用的“保证值”。
    
    - K8s 调度器会根据 requests 去决定 Pod 能不能放到某个节点。
        
    - 比如 `requests.cpu=200m`，调度器会认为这个 Pod 至少需要 0.2 核。
        
2. **limits**：  
    Pod 运行时的“上限值”。
    
    - 容器最多只能用这么多资源。
        
    - 如果超过 limit：
        
        - CPU：会被 **限流**（不会被杀掉）。
            
        - 内存：会触发 **OOMKill**。
            

⚠️ **调度器只看 requests，不看 limits！**

---

## 🔹 为什么要区分 `defaultRequest` 和 `default`

K8s 的设计理念是：**调度和运行分开考虑**。

- `defaultRequest` → 给调度器看的（缺省 requests）。
    
- `default` → 给 kubelet/运行时看的（缺省 limits）。
    

### 场景举例

1. 如果只有 `default`（limit）：  
    Pod 被分配了 **0.5 核 limit**，但调度时没有 requests，调度器会默认当成 **0**，可能把一堆 Pod 都挤到一个节点上，运行时才发现资源不够。
    
    → 结果：节点超卖严重，Pod OOM 或抢占。
    
2. 如果只有 `defaultRequest`（request）：  
    Pod 有 requests，可以被合理调度，但没有 limit，可能无限制吃资源，把节点搞挂。
    
    → 结果：单个 Pod 抢光整个节点资源。
    

所以 K8s 需要两个字段：

- `defaultRequest` 保证调度合理。
    
- `default` 保证运行时不会“吃爆”。
    

---

## 🔹 总结成一句话

- **`defaultRequest`** = Pod 没写 `requests` 时，调度器用这个值。
    
- **`default`** = Pod 没写 `limits` 时，运行时用这个值。
    

👉 二者配合起来，才能保证：

1. **调度合理**（不会过度超卖）。
    
2. **运行安全**（不会无限制吃资源）。

## LimitRange 的 `type`
在 `LimitRange` 里，`type: Container` 指的是 **针对 Pod 里的单个容器** 设置的限制，而不是整个 Pod。

---

## 🔹 为什么是容器级别？

Kubernetes 的资源单位是 **Container**：

- 你在 Pod spec 里写的 `resources.requests/limits`，都是在 **容器** 上，而不是 Pod 上。
    
- 调度时，K8s 会把 Pod 里所有容器的 `requests` 加起来，作为 Pod 的整体调度需求。
    

所以 `LimitRange` 的类型必须区分清楚。

---

## 🔹 LimitRange 的 `type` 有几种

1. **Container**
    
    - 作用对象：Pod 里的每个容器
        
    - 限制内容：CPU/Memory requests & limits（最常见）
        
2. **Pod**
    
    - 作用对象：整个 Pod 的 **总资源**
        
    - 限制内容：Pod 的 `ephemeral-storage`（临时存储，比如 `/tmp`）
        
3. **PersistentVolumeClaim**
    
    - 作用对象：PVC
        
    - 限制内容：申请存储的大小（`requests.storage`）
        

---

## 🔹 举例说明

### 针对容器：

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: container-limit
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

```

👉 限制的是 **Pod 里每个容器** 的最小/最大值。  
如果一个 Pod 里有 2 个容器，它们每个容器都必须在 [100m, 1CPU] & [128Mi, 1Gi] 之间。

---

### 针对 Pod：

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: pod-limit
  namespace: dev
spec:
  limits:
  - type: Pod
    max:
      ephemeral-storage: "2Gi"
    min:
      ephemeral-storage: "1Gi"

```

👉 限制的是 **Pod 总的临时存储大小**。

---

### 针对 PVC：

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: pvc-limit
  namespace: dev
spec:
  limits:
  - type: PersistentVolumeClaim
    min:
      storage: "1Gi"
    max:
      storage: "10Gi"

```

👉 限制的是 **单个 PVC 的最小/最大大小**。

---

✅ **结论**：

- `type: Container` = 限制 **Pod 内每个容器** 的 CPU/内存。
    
- 如果要控制整个 Pod 或 PVC，则用 `type: Pod`、`type: PersistentVolumeClaim`。