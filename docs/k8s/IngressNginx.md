### ingress nginx安装
下载安装包
```shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

#因网络问题，将文件下载下来安装
https://github.com/kubernetes/ingress-nginx/tags
https://codeload.github.com/kubernetes/ingress-nginx/zip/refs/tags/helm-chart-4.14.0

#4.14.0   k8s supported version  1.34, 1.33, 1.32, 1.31, 1.30

cd /root/ingress-nginx/ingress-nginx-helm-chart-4.14.0/charts/ingress-nginx

```
###镜像上传到私有仓库
values.yaml 配置文件3个镜像地址上传到私有仓库
```shell
registry.k8s.io/ingress-nginx/controller:v1.14.0
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.6.4
registry.k8s.io/defaultbackend-amd64:1.5
```
修改values.yaml 配置文件
- 镜像地址修改并注释掉镜像的hash
- hostNetwork: true  改成true
- dnsPolicy: ClusterFirstWithHostNet
- nodeSelector 添加 ingress: "true" 标签部署至指定节点
- kind: DaemonSet   默认值是Deployment
- ingressClassResource   default: false      default改成 true

安装
```shell
kubectl label node k8s-node02 ingress=true
kubectl create ns ingress-nginx
helm install ingress-nginx -n ingress-nginx .

helm list -n ingress-nginx

kubectl get po -n ingress-nginx -owide

kubectl get ds -n ingress-nginx
```
### 访问测试
```shell
kubectl create ns study-ingress

#deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: study-ingress
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: registry.cn-hangzhou.aliyuncs.com/xxx/nginx:1.29
        ports:
        - containerPort: 80
          
#svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: study-ingress
spec:
  selector:
    app: nginx   # 这里要和 Deployment 的 label 匹配
  ports:
  - port: 80        # Service 暴露端口
    targetPort: 80  # Pod 容器端口
    
    
#ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: study-ingress
spec:
  rules:
  - host: nginx.test.com
    http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```
nginx.test.com 解析到节点机器ip即可访问

### 域名重定向
```shell
#redirect.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-redirect
  namespace: study-ingress
  annotations:
    nginx.ingress.kubernetes.io/permanent-redirect: https://www.baidu.com
spec:
  rules:
  - host: nginx.redirect.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80

```

### 前后端分离 Rewrite
访问 nginx.test.com/aip-a  跳转到 backend-api
```shell
kubectl create deploy backend-api --image=registry.cn-hangzhou.aliyuncs.com/xxx/nginx-custom:v1 -n study-ingress

kubectl expose deploy backend-api --port 80 -n study-ingress

#rewrite.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: backend-api
  namespace: study-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: nginx.test.com
    http:
      paths:
      - path: /api-a(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: backend-api
            port:
              number: 80
```
### 错误代码重定向
```shell
#修改 values.yaml

config:  
  apiVersion: v1  
  client_max_body_size: 20m  
  custom-http-errors: "404,415,503"
  
helm upgrade ingress-nginx -n ingress-nginx .
```
### Nginx SSL
```shell
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=nginx.test.com"

kubectl create secret tls ca-secret --cert=tls.crt --key=tls.key -n study-ingress

#ingress-ssl.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  creationTimestamp: null
  name: nginx-ingress
  namespace: study-ingress
  # annotations:
  #   kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx  # for k8s >= 1.22+
  tls:
  - hosts:
    - nginx.test.com
    secretName: ca-secret
  rules:
  - host: nginx.test.com
    http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
              
kubectl get ingress -n study-ingress
```
### 匹配请求头
```shell
# laptop-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: laptop
  namespace: study-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/server-snippet: |
      set $agentflag 0;
      if ($http_user_agent ~* "(Android|iPhone|Windows Phone|UC|Kindle)" ){
        set $agentflag 1;
      }
      if ($agentflag = 1) {
        return 301 http://m.test.com;
      }
spec:
  rules:
  - host: test.com
    http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: laptop
            port:
              number: 80

```
### 基本认证
```shell
htpasswd -c auth foo

kubectl create secret generic basic-auth --from-file=auth -n study-ingress

#ingress-with-auth.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-with-auth
  namespace: study-ingress
  annotations:
    # kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/auth-realm: "Please Input Your Username and Password"
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-type: basic
spec:
  ingressClassName: nginx   # for k8s >= 1.22+
  rules:
  - host: auth.test.com
    http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```
### 速率限制
```shell
# auth-rate-limit.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-with-auth
  namespace: study-ingress
  annotations:
    # kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/auth-realm: "Please Input Your Username and Password"
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/limit-connections: "1"
spec:
  ingressClassName: nginx   # for k8s >= 1.22+
  rules:
  - host: auth.test.com
    http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
              
ab -c 10 -n 100 http://nginx.test.com/ | grep requests

#限制每秒的连接，单个 IP：  
nginx.ingress.kubernetes.io/limit-rps
  
#限制每分钟的连接，单个 IP：  
nginx.ingress.kubernetes.io/limit-rpm
  
#限制客户端每秒传输的字节数， 单位为 K，需要开启 proxy-buffering：  
nginx.ingress.kubernetes.io/limit-rate
  
# 速率限制白名单  
nginx.ingress.kubernetes.io/limit-whitelist
```


