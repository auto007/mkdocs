## 安装 Podman

### 1. 更新系统

`sudo dnf update -y`

### 2. 安装 Podman

`sudo dnf install -y podman`

### 3. 验证版本

`podman --version`

输出类似 `podman version 4.x.x` 就说明安装成功。

---

## 🛠 Podman 常用用法（对照 Docker）

Podman 的命令几乎与 Docker 一致，只是默认不需要守护进程。

|Docker 命令|Podman 等价命令|
|---|---|
|`docker run hello-world`|`podman run hello-world`|
|`docker ps`|`podman ps`|
|`docker images`|`podman images`|
|`docker exec -it <id> sh`|`podman exec -it <id> sh`|
|`docker-compose up -d`|`podman-compose up -d`|

---

## 📦 Podman-Compose（替代 docker-compose）

如果你需要 `docker-compose` 的功能，可以安装 **podman-compose**：

`sudo dnf install -y podman-compose`

使用方式和 `docker-compose` 一样：

`podman-compose up -d`

## 容器开机自启

### 1. 确认容器存在

先看看你现在有哪些容器：

`podman ps -a`

假设输出里有个容器叫 `nginx-test`，那么你要把 `mycontainer` 换成 `nginx-test`。

---

### 2. 使用 `generate systemd`（旧方法）

如果只是想先试用旧方法（还能用，但未来会弃用）：

`podman generate systemd --name nginx-test --files --new`

会生成一个 `container-nginx-test.service` 文件。你可以把它复制到合适位置：

```shell
cp container-nginx-test.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable container-nginx-test.service
systemctl --user start container-nginx-test.service
```

这样容器就能随 systemd 启动。

---

### 3. 使用 **Quadlet**（推荐方法）

Podman 4.x 推荐用 **Quadlet**（systemd unit 的扩展）。

创建一个 `.container` 文件，例如：

```shell
# ~/.config/containers/systemd/nginx-test.container
[Unit]
Description=My Nginx Container

[Container]
Image=nginx:latest
Name=nginx-test
PublishPort=8080:80

[Service]
Restart=always

[Install]
WantedBy=default.target

```
然后执行：
```shell
systemctl --user daemon-reload
systemctl --user enable nginx-test.service
systemctl --user start nginx-test.service

```

> 注意：`.container` 文件要放在 `~/.config/containers/systemd/` 目录下，systemd 会自动识别。