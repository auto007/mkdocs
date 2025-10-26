用的是 **StatefulSet + Headless Service**，所以 Redis Pod 会有 **固定的 DNS 域名**，不用自己去记 IP。
### 1. 查看 Service

先看你定义的 Headless Service 名字（一般就是 `redis`）：

`kubectl get svc`

你应该能看到类似：

```shell
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
redis           ClusterIP   None         <none>        6379/TCP   10m

```

这里的 `redis` 就是你 StatefulSet 的 `serviceName`。

---

### 2. Pod 的域名规则

StatefulSet Pod 的域名是：

`<pod-name>.<service-name>.<namespace>.svc.cluster.local`

比如你的 Pod：

- `redis-0` → 域名：
    
    `redis-0.redis.default.svc.cluster.local`
    
- `redis-1` → 域名：
    
    `redis-1.redis.default.svc.cluster.local`
    
- `redis-2` → 域名：
    
    `redis-2.redis.default.svc.cluster.local`
    

如果你就在 `default` namespace 下，可以直接用短域名：

```shell
redis-0.redis:6379
redis-1.redis:6379
redis-2.redis:6379
```

---

### 3. 验证解析

你可以在集群里起一个 busybox 容器测试：

`kubectl run -it --rm dns-test --image=busybox:1.28 --restart=Never -- nslookup redis-0.redis`

或者直接解析：

`kubectl exec -it redis-0 -- nslookup redis-1.redis`

---

✅ 总结：

- 你的 Redis 主节点访问地址是：`redis-0.redis.default.svc.cluster.local:6379`
    
- 从节点分别是：`redis-1.redis.default.svc.cluster.local:6379`、`redis-2.redis.default.svc.cluster.local:6379`