###  Headless Service
用于 StatefulSet 内部 Pod DNS 解析
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    app: redis
spec:
  ports:
    - port: 6379
      name: redis
  clusterIP: None   # Headless Service
  selector:
    app: redis

```
### redis-empty
测试环境不持久化数据，使用emptyDir
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: "redis"
  replicas: 3   # 1 主 + 2 从
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7.2
        ports:
        - containerPort: 6379
          name: redis
        command:
          - sh
          - -c
          - |
            if [ "$(hostname)" = "redis-0" ]; then
              echo "我是主节点，启动 Redis"
              exec redis-server --appendonly yes
            else
              echo "我是从节点，启动 Redis 并连接主节点"
              exec redis-server --appendonly yes --replicaof redis-0.redis 6379
            fi
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        emptyDir: {}
```
### 外部访问 Service（可选）
```shell
apiVersion: v1
kind: Service
metadata:
  name: redis-access
spec:
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
  type: ClusterIP

```
### 这个外部访问Service 不能从集群外部直接访问，那为什么还需要呢
#### 1. **内部访问入口统一**

- StatefulSet Pod 有固定的域名，比如 `redis-0.redis.default.svc.cluster.local`、`redis-1.redis.default.svc.cluster.local`。
    
- 但是如果你只想「不管主从，随便找一个 Redis 用」，就可以访问 Service：
    
    `redis-access.default.svc.cluster.local:6379`
    
- 这样，集群内的 **应用程序（如 Web、微服务）** 就能直接用 Service 的名字，而不用管 Pod IP。
    

---

#### 2. **方便应用部署**

很多应用（Java、Python、Go 之类的微服务）在配置时更习惯填一个 **固定的地址**，而不是 StatefulSet Pod 的子域名。

- 如果没有 Service，就必须写：
    
    `redis-0.redis.default.svc.cluster.local`
    
- 有了 Service，就写：
    
    `redis-access.default.svc.cluster.local`
    

---

#### 3. **集群内 LB（负载均衡）作用**

虽然 Redis 主从不能做「写请求负载均衡」，但：

- Service 依然能在多个从节点之间随机分发 **读请求**；
    
- 这样，你可以设计成：
    
    - 写请求 → `redis-0`（主）
        
    - 读请求 → `redis-access`（ClusterIP，连到任意从）
        

---

#### 4. **可升级成外部访问**

如果以后你真的要从集群外访问，只需要改一下类型：

`type: NodePort # 或者 type: LoadBalancer`

就能开放出去，而不需要再重新写一个 Service。

---

📌 **总结**：

- `redis-access` 这个 Service 主要是给 **集群内部应用用的**，不是给宿主机直接访问的。
    
- 它的价值在于：**固定入口、解耦 Pod 名称、方便配置、支持负载分发**。
    
- 如果你要从集群外访问，再把它改成 `NodePort` 或 `LoadBalancer` 就行。