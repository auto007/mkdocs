很多人第一次接触 **Taint/Toleration**（污点和容忍） 和 **NodeSelector**（节点选择器）时都会混淆，其实两者目标相似（控制 Pod 调度位置），但机制和使用场景不同。

---

## 🔹 区别总结

|特性|**Taints & Tolerations（污点与容忍）**|**NodeSelector / NodeAffinity**|
|---|---|---|
|**作用对象**|污点在 **节点** 上，容忍在 **Pod** 上|节点选择器在 **Pod** 上|
|**逻辑方向**|**节点说：“没有通行证就别来”**（排斥）|**Pod 说：“我只去满足条件的节点”**（吸引）|
|**默认行为**|有污点的节点 **会拒绝 Pod**，除非 Pod 容忍|没设置 nodeSelector 的 Pod **可以去任何节点**|
|**强制性**|污点是 **节点级别的限制**，影响所有 Pod|nodeSelector 是 **Pod 自己的选择**，不会影响其他 Pod|
|**使用场景**|- 隔离关键节点（专用节点、系统组件）  <br>- 故障驱逐（NoExecute）  <br>- 临时禁止调度|- Pod 必须运行在 GPU 节点、SSD 节点、某个可用区|
|**灵活性**|容忍匹配即可，但不会“强制调度”到该节点|nodeSelector **必须匹配**，不匹配就无法调度|
|**进阶用法**|污点 effect 有 `NoSchedule`、`PreferNoSchedule`、`NoExecute`|nodeSelector 过于简单，一般用 **NodeAffinity**（支持 AND/OR、范围匹配等）|

---

## 🔹 举个例子

### 1. 使用 **污点 + 容忍**

```shell
# 给 node1 打上污点
kubectl taint nodes node1 dedicated=database:NoSchedule

```

```yaml
# 只有能容忍的 Pod 才能上去
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "database"
  effect: "NoSchedule"

```

👉 节点限制别人，只有带“通行证”的 Pod 能进去。

---

### 2. 使用 **NodeSelector**

```yaml
# Pod 选择必须有 dedicated=database 标签的节点
nodeSelector:
  dedicated: database

```

👉 Pod 自己挑选目标节点，没这个标签的节点就不会考虑。

---

## 🔹 类比

- **NodeSelector / NodeAffinity**：Pod 说“我要去某些节点”。（主动挑）
    
- **Taints & Tolerations**：节点说“你不符合条件别来”。（被动拒）