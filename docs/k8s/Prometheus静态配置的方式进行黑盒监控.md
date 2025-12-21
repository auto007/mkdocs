我的 k8s 集群是用kubeadmin 搭建的,  https://github.com/prometheus-operator/kube-prometheus 已安装好
```shell
kubectl get po -n monitoring -l app.kubernetes.io/name=blackbox-exporter
kubectl get svc -n monitoring -l app.kubernetes.io/name=blackbox-exporter
```
### 验证黑盒探测是否可用
```shell
#启动一个测试pod
kubectl run netshoot --rm -it --image=registry.cn-hangzhou.aliyuncs.com/xxx/netshoot:latest -- bash
curl -v "http://blackbox-exporter.monitoring.svc.cluster.local:19115/probe?target=www.baidu.com&module=http_2xx"

```
### 编写自定义 scrape 配置文件
创建一个文件 `blackbox-additional-scrape.yaml`
```shell
# ======= HTTP 探测 =======
- job_name: 'blackbox-http'
  metrics_path: /probe
  params:
    module: [http_2xx]  # 使用 HTTP 模块
  static_configs:
    - targets:
      - https://www.baidu.com
  relabel_configs:
    # 把目标地址放入 __param_target 参数
    - source_labels: [__address__]
      target_label: __param_target
    # 把 target 值同时复制给 instance
    - source_labels: [__param_target]
      target_label: instance
    # 指定 blackbox-exporter 的访问地址
    - target_label: __address__
      replacement: blackbox-exporter.monitoring.svc.cluster.local:19115


# ======= TCP 探测 =======
- job_name: 'blackbox-tcp'
  metrics_path: /probe
  params:
    module: [tcp_connect]   # 使用 TCP 模块
  static_configs:
    - targets:
      - 192.168.124.1:80   # ✅ 在此写你要探测的 IP 和端口
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: blackbox-exporter.monitoring.svc.cluster.local:19115
```
### 创建 Secret 注入 Prometheus
```shell
kubectl create secret generic additional-scrape-configs \
  -n monitoring \
  --from-file=blackbox-additional-scrape.yaml

kubectl get secret -n monitoring additional-scrape-configs

#如果blackbox-additional-scrape.yaml有更新
kubectl create secret generic additional-scrape-configs \
  -n monitoring \
  --from-file=blackbox-additional-scrape.yaml \
  --dry-run=client -o yaml | kubectl apply -f -
```
### 修改 Prometheus CR 关联该 Secret
```bash
kubectl edit prometheus -n monitoring k8s

#找到 spec:下添加（或修改）：
spec:
  additionalScrapeConfigs:
    name: additional-scrape-configs
    key: blackbox-additional-scrape.yaml

# 完整示例（简化片段）
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: k8s
  namespace: monitoring
spec:
  replicas: 2
  serviceAccountName: prometheus-k8s
  serviceMonitorSelector: {}
  additionalScrapeConfigs:
    name: additional-scrape-configs
    key: blackbox-additional-scrape.yaml

```
进入 Prometheus UI 验证状态
### grafana导入模板
https://grafana.com/grafana/dashboards/13659-blackbox-exporter-probe-status/




