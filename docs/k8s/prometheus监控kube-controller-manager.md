3台主节点 /etc/kubernetes/manifests/kube-controller-manager.yaml  修改监听ip为--bind-address=0.0.0.0
重启服务 systemctl restart kubele
```shell
kubectl get servicemonitor -n monitoring kube-controller-manager -oyaml

#查看得到的标签是
  #selector:
    #matchLabels:
      #app.kubernetes.io/name: kube-controller-manager

kubectl get svc -n kube-system -l app.kubernetes.io/name=kube-controller-manager
#No resources found in kube-system namespace.
#找不到svc，所以prometheus监控kube-controller-manager没有数据
```
可以看到并没有此标签的 Service，所以导致了找不到需要监控的目标，此时可以手动创建  
该 Service 和 Endpoint 指向自己的 Controller Manager
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: kube-controller-manager-prom
  namespace: kube-system
  labels:
    app.kubernetes.io/name: kube-controller-manager
subsets:
- addresses:
  - ip: 192.168.124.60
  - ip: 192.168.124.61
  - ip: 192.168.124.62
  ports:
  - name: https-metrics
    port: 10257
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: kube-controller-manager-prom
  namespace: kube-system
  labels:
    app.kubernetes.io/name: kube-controller-manager
spec:
  ports:
  - name: https-metrics
    port: 10257
    protocol: TCP
    targetPort: 10257
  sessionAffinity: None
  type: ClusterIP
```

提取出证书文件
```shell
# 1. 提取 controller-manager 客户端证书
grep "client-certificate-data" /etc/kubernetes/controller-manager.conf | awk '{print $2}' | base64 -d > /etc/kubernetes/pki/cm-client.crt

# 2. 提取 controller-manager 客户端私钥
grep "client-key-data" /etc/kubernetes/controller-manager.conf | awk '{print $2}' | base64 -d > /etc/kubernetes/pki/cm-client.key

```
`system:kube-controller-manager` 添加 RBAC 访问权限
controller-manager-metrics-rbac.yaml
```shell
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kube-controller-manager-metrics
rules:
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-controller-manager-metrics-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-controller-manager-metrics
subjects:
- kind: User
  name: system:kube-controller-manager
  apiGroup: rbac.authorization.k8s.io

```
测试访问
```shell
curl -k \
  --cert /etc/kubernetes/pki/cm-client.crt \
  --key /etc/kubernetes/pki/cm-client.key \
  https://192.168.124.60:10257/metrics

```





