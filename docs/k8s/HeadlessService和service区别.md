Headless Service 和普通的 Service（通常指 ClusterIP 类型）在 Kubernetes 中虽然都属于 Service 资源，但它们在**是否分配 ClusterIP、DNS 行为、负载均衡机制、使用场景**等方面有明显区别：

---

### ✅ 核心区别对比

|特性|普通 Service（ClusterIP）|Headless Service|
|:--|:--|:--|
|**ClusterIP**|有，Kubernetes 会分配一个虚拟 IP|无，需显式设置 `clusterIP: None`|
|**DNS 解析**|返回 Service 的虚拟 IP|返回后端 Pod 的 IP 列表|
|**负载均衡**|由 kube-proxy 自动完成|无负载均衡，客户端需自行选择 Pod|
|**Pod 访问方式**|通过 Service IP 转发|可直接访问具体 Pod（如 `pod-name.service-name.ns.svc.cluster.local`）|
|**适用场景**|无状态服务，如 Web API|有状态服务，如 Kafka、Zookeeper、MySQL 主从|

---

### ✅ 举个例子

- **普通 Service**：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - port: 80
```
DNS 查询 `web.default.svc.cluster.local` 会返回一个虚拟 IP，比如 `10.96.0.10`，流量由 kube-proxy 转发到某个 Pod。

- **Headless Service**：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: kafka
spec:
  clusterIP: None
  selector:
    app: kafka
  ports:
    - port: 9092
```
DNS 查询 `kafka.default.svc.cluster.local` 会返回所有 Pod 的 IP，比如：
10.0.0.1
10.0.0.2
10.0.0.3

---

### ✅ 总结一句话

> **普通 Service** 提供统一入口和负载均衡，适合无状态服务；  
> **Headless Service** 暴露每个 Pod 的 IP，适合需要直接访问具体 Pod 的有状态服务或自定义负载均衡场景。