3台主节点 /etc/kubernetes/manifests/kube-scheduler.yaml 修改监听ip为--bind-address=0.0.0.0 ，修改完成pod会自动重启

创建 Service 和 Endpoints
```shell
apiVersion: v1
kind: Service
metadata:
  name: kube-scheduler-prom
  namespace: kube-system
  labels:
    app.kubernetes.io/name: kube-scheduler
spec:
  ports:
  - name: https-metrics
    port: 10259
    targetPort: 10259
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: kube-scheduler-prom
  namespace: kube-system
  labels:
    app.kubernetes.io/name: kube-scheduler
subsets:
- addresses:
  - ip: 192.168.124.60
  - ip: 192.168.124.61
  - ip: 192.168.124.62
  ports:
  - name: https-metrics
    port: 10259

```
测试
```shell
curl -k \
  --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt \
  --key /etc/kubernetes/pki/apiserver-kubelet-client.key \
  https://192.168.124.60:10259/metrics
```
