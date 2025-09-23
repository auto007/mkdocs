## 1. 基本作用

- **CMD**
    
    - 给容器提供**默认命令或参数**。
        
    - 可以被 `docker run <image> <command>` **完全覆盖**。
        
- **ENTRYPOINT**
    
    - 定义容器的**主命令**（入口点）。
        
    - 即使 `docker run` 后跟参数，也不会替换，而是把这些参数**传给 ENTRYPOINT 命令**。
        

---

## 2. 形式

两者都支持：

- **exec 形式**（推荐）：`["可执行文件", "参数1", "参数2"]`
    
- **shell 形式**：`command param1 param2`（会通过 `/bin/sh -c` 执行）
    

---

## 3. 使用区别示例

### 例子 1：CMD

`FROM ubuntu CMD ["echo", "hello"]`

运行：

`docker run myimage # 输出 hello`

但是如果指定命令：

`docker run myimage ls / # 会执行 ls /，CMD 完全被覆盖`

---

### 例子 2：ENTRYPOINT

`FROM ubuntu ENTRYPOINT ["echo", "hello"]`

运行：

`docker run myimage # 输出 hello`

如果加参数：

`docker run myimage world # 实际执行的是 echo hello world`

这里，`world` 被当作参数传给了 ENTRYPOINT，而不是替换它。

---

### 例子 3：结合使用 ENTRYPOINT + CMD

`FROM ubuntu ENTRYPOINT ["echo"] CMD ["hello"]`

运行：

`docker run myimage # 输出 hello`

加参数：

`docker run myimage world # 输出 world`

这里 CMD 提供默认参数，ENTRYPOINT 定义主命令。

---

## 4. 总结对比表

|特性|CMD|ENTRYPOINT|
|---|---|---|
|定义作用|默认命令或参数|固定主命令|
|`docker run <command>`|覆盖 CMD|把 `<command>` 作为参数传递|
|推荐用途|设置默认参数|设置容器主进程|
|常见用法|`CMD ["--port", "80"]`|`ENTRYPOINT ["nginx"]`|

---

👉 简单记：

- 想让镜像像一个命令行工具（固定执行某个程序，可以加参数） → 用 **ENTRYPOINT**
    
- 想给镜像设置**默认行为**，但允许用户轻易覆盖 → 用 **CMD**
    
- 最佳实践：**ENTRYPOINT + CMD** 配合，ENTRYPOINT 指定命令，CMD 提供默认参数。