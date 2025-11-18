### 准备自定义页面
例如创建一个 `index.html`
```shell
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>My Custom Nginx Page</title>
</head>
<body>
<h1>欢迎来到我的 Nginx 自定义首页！</h1>
<p>这是基于官方 nginx 镜像构建的。</p>
</body>
</html>

```
### 编写 Dockerfile
```Dockerfile
FROM nginx:latest

# 删除默认首页（可选）
RUN rm -f /usr/share/nginx/html/index.html

# 复制自定义首页
COPY index.html /usr/share/nginx/html/index.html

```
构建镜像
```shell
docker build -t my-nginx-custom:v1 .

podman tag my-nginx-custom:v1 registry.cn-hangzhou.aliyuncs.com/xxx/nginx-custom:v1

podman push registry.cn-hangzhou.aliyuncs.com/xxx/nginx-custom:v1
```

