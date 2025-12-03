在 **Windows 系统**里，`mkvirtualenv.bat` 来自 **virtualenvwrapper-win**（是 Linux 上 virtualenvwrapper 的 Windows 版）。
### **1. 安装 virtualenvwrapper-win**

`pip install virtualenvwrapper-win`

> ⚠️确保安装前已经有 Python & pip，并且都加入 PATH。

---

### **2. 安装完成后，命令自动可用**

执行：

`mkvirtualenv myenv`

如果正常，会创建一个虚拟环境，并自动进入。

---

# 📌 **安装后常见命令**

| 命令                    | 功能     |
| --------------------- | ------ |
| `mkvirtualenv <name>` | 创建虚拟环境 |
| `workon <name>`       | 进入虚拟环境 |
| `lsvirtualenv`        | 列出虚拟环境 |
| `rmvirtualenv <name>` | 删除虚拟环境 |
| `deactivate`          | 退出虚拟环境 |

