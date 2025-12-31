# **1️⃣ 删除系统自带旧版本（如果有）**

`dnf remove -y docker docker-client docker-client-latest docker-common \     docker-latest docker-latest-logrotate docker-logrotate docker-engine`

---

# **2️⃣ 安装必要依赖**

`dnf install -y yum-utils device-mapper-persistent-data lvm2`

---

# **3️⃣ 添加 Docker 官方仓库**

`yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

> Rocky 基于 RHEL9，可以使用 CentOS 的这个 repo，官方支持。

---

# **4️⃣ 安装 Docker Engine**

`dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

---

# **5️⃣ 启动并设置开机自启**

`systemctl enable --now docker`

---

# **6️⃣ 验证安装**

```shell
docker version
docker info
docker run hello-world
```

---

# **7️⃣（可选）允许当前用户不用 sudo 运行 docker**

`usermod -aG docker $USER`

然后退出重新登录生效。

下面是 Rocky9 通用的阿里云 Docker CE repo 配置方式。

---

# ✅ **1. 添加阿里云 Docker CE 仓库（适用于 Rocky9/RHEL9）**

阿里云镜像是同步官方的 CentOS/RHEL仓库，可以直接用。

### 创建 repo 文件：

```shell
sudo tee /etc/yum.repos.d/docker-ce.repo << 'EOF'
[docker-ce-stable]
name=Docker CE Stable - $basearch - mirrors.aliyun.com
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/9/$basearch/stable
enabled=1
gpgcheck=0

[docker-ce-test]
name=Docker CE Test - $basearch - mirrors.aliyun.com
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/9/$basearch/test
enabled=0
gpgcheck=0

[docker-ce-nightly]
name=Docker CE Nightly - $basearch - mirrors.aliyun.com
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/9/$basearch/nightly
enabled=0
gpgcheck=0
EOF

```

---

# ✅ **2. 安装 Docker**

```shell
dnf install -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin

```

---

# ✅ **3. 启动 Docker 服务**

`systemctl enable --now docker`

---

# ✅ **4. 测试**

`docker run hello-world`

---

# 🎯 （可选）配置国内镜像加速（推荐）

在 `/etc/docker/daemon.json` 添加阿里云加速器：

```shell
mkdir -p /etc/docker

tee /etc/docker/daemon.json << 'EOF'
{
    "registry-mirrors": [
        "https://registry.aliyuncs.com",
        "https://docker.mirrors.sjtug.sjtu.edu.cn",
        "https://mirror.ccs.tencentyun.com"
    ]
}
EOF

```

重启 Docker：
```shell
systemctl restart docker
docker-compose version
docker compose --help
```
配置私有仓库
```shell
cat /etc/docker/daemon.json
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "insecure-registries": ["192.168.124.70:80"]
}
#默认情况下，Docker 只信任HTTPS的镜像仓库。
#如果你有一个自建的私有仓库（比如用 Harbor 或简单 registry），而且没有配置 HTTPS，Docker 会拒绝连接。
#把 IP 加入insecure-registries后，Docker 就会允许通过 HTTP 连接这个地址，不再报错。
```



