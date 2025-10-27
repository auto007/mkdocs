**mysqld_exporter v0.15.0 起**（现在用的 v0.18.0），**废弃了 `DATA_SOURCE_NAME` 环境变量**
### 创建监控用户
```shell
kubectl exec -ti mysql-0 -- bash
CREATE USER 'exporter'@'%' IDENTIFIED BY 'exporter' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';
```

### 创建secret
```shell
kubectl -n monitoring create secret generic mysql-exporter-secret \
  --from-literal=.my.cnf='[client]
user=exporter
password=exporter
host=mysql.default.svc.cluster.local
port=3306'

kubectl get secret -n monitoring
```

### mysqld-exporter.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-exporter
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql-exporter
  template:
    metadata:
      labels:
        app: mysql-exporter
    spec:
      containers:
      - name: mysql-exporter
        image: registry.cn-hangzhou.aliyuncs.com/xxx/mysqld-exporter:latest
        imagePullPolicy: IfNotPresent
        args:
          - --config.my-cnf=/etc/mysql/.my.cnf
          - --web.listen-address=:9104
          - --collect.global_status
          - --collect.engine_innodb_status
          - --collect.info_schema.processlist
          - --collect.info_schema.tables
          - --collect.info_schema.tablestats
          - --collect.info_schema.userstats
          - --collect.perf_schema.eventsstatements
          - --collect.perf_schema.eventsstatementssum
        ports:
        - containerPort: 9104
        volumeMounts:
        - name: mysql-exporter-config
          mountPath: /etc/mysql/.my.cnf
          subPath: .my.cnf
      volumes:
      - name: mysql-exporter-config
        secret:
          secretName: mysql-exporter-secret
```

### 创建service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-exporter
  namespace: monitoring
  labels:
    app: mysql-exporter
spec:
  selector:
    app: mysql-exporter
  ports:
  - name: http
    port: 9104
    targetPort: 9104
```
curl 10.96.177.111:9104/metrics

### ServiceMonitor
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mysql-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: mysql-exporter   
  endpoints:
  - port: http
    interval: 30s
```
查看prometheus后台  http://192.168.124.60:32713/targets  状态是不是up

kubectl get servicemonitor  -n monitoring

### Grafana导入仪表盘
https://grafana.com/grafana/dashboards/7362
可选中文版 https://grafana.com/grafana/dashboards/14057
