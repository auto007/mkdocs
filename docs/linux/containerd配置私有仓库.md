从 containerd v1.5 开始，`[plugins."io.containerd.grpc.v1.cri".registry]` 下面的 `mirrors` 字段就被标记为“弃用”，在即将发布的 containerd v2.1 里会彻底删除。如果升级前不改成新的 `config_path` 方式，containerd 会直接启动失败。

---

## 一、旧的（已弃用）写法

/etc/containerd/config.toml 片段示例：
```toml
[plugins."io.containerd.grpc.v1.cri".registry]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = ["https://mirror.baidubce.com", "https://registry-1.docker.io"]
```

---

## 二、新的（官方推荐）写法

1. 在主配置文件里只保留一行，告诉 containerd “镜像配置目录”在哪：
    

/etc/containerd/config.toml

```toml
[plugins."io.containerd.grpc.v1.cri".registry]
  config_path = "/etc/containerd/certs.d"
```

2. 然后给每个仓库单独建一个目录和 `hosts.toml` 文件，不再在主配置里写 `mirrors`。
    

---

## 三、把旧配置迁移成新文件

以 Docker Hub 为例：
```bash
# 1. 创建目录
mkdir -p /etc/containerd/certs.d/docker.io

# 2. 写入新配置
cat > /etc/containerd/certs.d/docker.io/hosts.toml <<'EOF'
server = "https://registry-1.docker.io"

[host."https://mirror.baidubce.com"]
  capabilities = ["pull", "resolve"]

[host."https://registry-1.docker.io"]
  capabilities = ["pull", "resolve", "push"]
EOF

### 我用的配置
cat > /etc/containerd/certs.d/docker.io/hosts.toml <<'EOF'
server = "http://192.168.124.70"
EOF
```

如果还有其他仓库（例如 `registry.k8s.io`、`quay.io`），同理再建对应的目录和 `hosts.toml` 即可。

---

## 四、重启 containerd 生效

```bash
systemctl restart containerd
```

---

## 五、确认警告消失
```bash
journalctl -u containerd -e
# 或者
containerd --log-level=warn 2>&1 | head
```

看不到 `DEPRECATION: The mirrors property ...` 就说明改好了。

---

## 六、一键检查

```bash
grep -q mirrors /etc/containerd/config.toml && \
  echo "❌ 仍使用已弃用的 mirrors 字段！" || \
  echo "✅ 已切换到 config_path，没问题"
```
## 七、拉取测试
```shell
docker tag registry.cn-hangzhou.aliyuncs.com/xxx/nginx:1.29 192.168.124.70/kubernetes/nginx:1.29
docker login 192.168.124.70
docker push 192.168.124.70/kubernetes/nginx:1.29

#拉取测试
ctr -n k8s.io image pull  192.168.124.70/kubernetes/nginx:1.29 --plain-http --user admin:Harbor12345
ctr -n k8s.io i ls

```
