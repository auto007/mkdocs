### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
  labels:
    app: mysql
data:
  my.cnf: |
    [mysqld]
    # ⚙️ 内存控制参数
    innodb_buffer_pool_size=64M
    innodb_log_buffer_size=4M
    max_connections=30
    tmp_table_size=8M
    performance_schema=0
    table_open_cache=64
    thread_cache_size=8
    query_cache_size=0
    innodb_flush_method=O_DIRECT
    innodb_flush_log_at_trx_commit=2

    # ⚙️ 常规设置
    sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
    character-set-server=utf8mb4
    collation-server=utf8mb4_unicode_ci
    
    open_files_limit = 6553
```
### mysql
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: registry.cn-hangzhou.aliyuncs.com/caixu/mysql:5.7.23
          command: ["/bin/sh", "-c"]
          args:
              - |
                ulimit -n 6553
                exec docker-entrypoint.sh mysqld --defaults-file=/etc/mysql/my.cnf
          ports:
            - containerPort: 3306
              name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "123456"
          resources:
            requests:
              cpu: 100m
              memory: 1Gi
            limits:
              cpu: 300m
              memory: 2Gi
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
            - name: config
              mountPath: /etc/mysql/my.cnf
              subPath: my.cnf
      volumes:
        - name: config
          configMap:
            name: mysql-config
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: nfs-client    # ✅ 使用你的 NFS 动态存储
        resources:
          requests:
            storage: 1Gi
```
