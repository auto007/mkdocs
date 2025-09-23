## 安装
将prometheus安装在k8s集群中，使用 https://github.com/prometheus-operator/kube-prometheus

因测试环境k8s版本是 v1.31.10，选择安装 ## [v0.16.0](https://github.com/prometheus-operator/kube-prometheus/releases/tag/v0.16.0)

根据服务器配置改下副本数 replicas
kube-prometheus-0.16.0/manifests/alertmanager-alertmanager.yaml
kube-prometheus-0.16.0/manifests/prometheus-prometheus.yaml

改成自己的私有镜像仓库地址
kube-prometheus-0.16.0/manifests/grafana-deployment.yaml


```shell
kubectl apply --server-side -f manifests/setup
kubectl wait \
    --for condition=Established \
    --all CustomResourceDefinition \
    --namespace=monitoring
kubectl apply -f manifests/


kubectl get po -n monitoring

kubectl edit svc grafana -n monitoring
```
## 访问 grafana NodePort端口不通
```shell
kubectl edit svc grafana -n monitoring
root@k8s-master01 ~]# kubectl exec -it grafana-6dc5b577d4-ts474 -n monitoring -- netstat -tlnp | grep 3000
tcp        0      0 :::3000                 :::*                    LISTEN      1/grafana
[root@k8s-master01 ~]# ipvsadm -Ln --stats | grep 31411
TCP  172.16.32.128:31411                 0        0        0        0        0
TCP  192.168.124.60:31411              144      722        0    37544        0
TCP  192.168.124.65:31411                0        0        0        0        0


kubectl exec -it grafana-6dc5b577d4-ts474 -n monitoring -- curl -v http://127.0.0.1:3000

网上查资料发现是网络限制，测试环境直接用下面命令搞定
kubectl delete networkpolicy --all -n monitoring
```
## 云原生应用 Etcd 监控
```shell
cat /etc/kubernetes/manifests/etcd.yaml
curl -s --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key https://192.168.124.60:2379/metrics -k

kubectl get svc -A |grep etcd

# 创建etcd的svc
[root@k8s-master01 monitor]# cat etcd_svc.yaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    app: etcd-prom
  name: etcd-prom
  namespace: kube-system
spec:
  selector:               # 关键：让 Service 自动找到 Pod
    component: etcd
    tier: control-plane
  ports:
  - name: https-metrics
    port: 2379            # Service 端口
    targetPort: 2379      # Pod 的容器端口
    protocol: TCP
  type: ClusterIP         # 建议加上，做 headless service，方便直接访问各 etcd 实例


curl -s --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key https://10.96.43.75:2379/metrics -k

#创建secret
kubectl create secret generic etcd-ssl --from-file=/etc/kubernetes/pki/etcd/ca.crt --from-file=/etc/kubernetes/pki/etcd/server.crt --from-file=/etc/kubernetes/pki/etcd/server.key -n monitoring


#查看证书是否挂载（任意一个Prometheus 的 Pod 均可）
kubectl exec -n monitoring prometheus-k8s-0 -c prometheus -- ls /etc/prometheus/secrets/etcd-ssl/


#创建 Etcd 的 ServiceMonitor

apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: etcd
  namespace: monitoring
  labels:
    app: etcd
spec:
  jobLabel: k8s-app
  endpoints:
  - interval: 30s
    port: https-metrics   # 这个 port 对应 Service.spec.ports.name
    scheme: https
    tlsConfig:
      caFile: /etc/prometheus/secrets/etcd-ssl/ca.crt   # 证书路径
      certFile: /etc/prometheus/secrets/etcd-ssl/server.crt
      keyFile: /etc/prometheus/secrets/etcd-ssl/server.key
      insecureSkipVerify: true   # 关闭证书校验
  selector:
    matchLabels:
      app: etcd-prom   # 跟 svc 的 labels 保持一致
  namespaceSelector:
    matchNames:
    - kube-system
	
kubectl get servicemonitor -n monitoring
```