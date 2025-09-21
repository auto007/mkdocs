```yaml
apiVersion: v1
kind: Endpoints
metadata:
labels:
app: etcd-prom
name: etcd-prom
namespace: kube-system
subsets:
- addresses:
- ip: YOUR_ETCD_IP01
- ip: YOUR_ETCD_IP02
- ip: YOUR_ETCD_IP03
ports:
- name: https-metrics
port: 2379 # etcd 端口
protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
labels:
app: etcd-prom
name: etcd-prom
namespace: kube-system
spec:
ports:
- name: https-metrics
port: 2379
protocol: TCP
targetPort: 2379
type: ClusterIP
```
这里为什么要手动创建endpoint呢，不是创建svc的时候会自动创建ep吗?

正常情况下，Kubernetes **Service** 创建以后，会自动创建对应的 **Endpoints**（ep），但这是有前提条件的：

1. **Service 的 selector 匹配到 Pod**
    
    - 比如你写了：
        
        `spec:   selector:     app: my-app`
        
        那么 kube-controller-manager 会去找带 `app=my-app` 的 Pod，把它们的 IP 填到 Endpoint 里。
        
2. **Pod 必须由 K8s 管理**
    
    - 即 Pod 必须在集群里运行，才能被自动关联。
        

---

但你这里的情况不同：

- etcd 是 **集群核心组件**，一般是作为 **静态 Pod** 或 **独立进程** 在 master 节点上运行的，而不是像普通业务 Pod 一样由 Deployment/StatefulSet 管理。
    
- 这些 etcd Pod 通常不会带上 `label`，也没有 selector 能够匹配到它们。
    
- Service 自然无法自动找到这些 IP，自动生成的 Endpoint 就是空的。
    

所以，你需要 **手动写 Endpoints**，把 etcd 实际的 IP 和端口填进去，这样 Service 才能代理到它们。

---

### 总结

- 普通应用：创建 Service → 自动生成 Endpoints（因为 selector 匹配到了 Pod）。
    
- 核心组件（如 etcd、apiserver）：因为没有 Pod/selector，必须手动写 Endpoints 来告诉 Service 后端地址。
    

👉 你的 `etcd-prom` Service + Endpoints，就是为了让 Prometheus 可以通过 `etcd-prom.kube-system.svc:2379` 统一访问到 etcd，而不用直接写死节点 IP。