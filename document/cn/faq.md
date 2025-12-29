# 常见问题

## aimde 开发环境需要多少硬盘空间？
  aimde 镜像大小约为 18GB，考虑到安装过程中的临时文件和后续开发所需空间，建议保留 30GB 以上的可用磁盘空间。

## docker 命令需要 sudo 才能执行
### 普通用户执行失败
```
  $ docker ps
  permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock

  # 使用 sudo 可以成功执行
  $ sudo docker ps
  CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
### 问题原因
当前用户不在 docker 用户组中，没有访问 Docker daemon socket 的权限。Docker daemon 默认绑定到 Unix socket (/var/run/docker.sock)，该 socket 归 root 用户和 docker 组所有。
### 解决方案：
```
# 步骤1: 创建 docker 用户组（如果不存在）
sudo groupadd docker

# 步骤2: 将当前用户添加到 docker 组
sudo usermod -aG docker $USER

# 步骤3: 重启系统
sudo reboot
```

## 系统找不到 Docker 服务
### 现象描述
```
sudo systemctl start docker
Failed to start docker.service: Unit docker.service not found
```

### 解决方案
```
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```
