---
title: Docker快速入门
tags:
  - Docker
  - 快速入门
categories:
  - 编程笔记
headimg: https://pic.zeng.cyou/wide/202409042025520.webp
abbrlink: 2763
date: 2024-09-01 18:29:11
---

## 环境搭建

### Docker

{% tabs tab-1 %}
<!-- tab CentOS -->
1. 卸载旧版本

```shell
sudo yum remove docker \
      docker-client \
      docker-client-latest \
      docker-common \
      docker-latest \
      docker-latest-logrotate \
      docker-logrotate \
      docker-engine
```

2. yum安装gcc相关

```shell
sudo yum -y install gcc

sudo yum -y install gcc-c++
```

3. 安装需要的软件包

```shell
sudo yum install -y yum-utils

sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

4. 更新yum软件包索引

```shell
sudo yum makecache fast
```

5. 安装docker engine

```shell
sudo yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
<!-- endtab -->

<!-- tab Ubuntu -->
1. 卸载旧版本

```shell
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

2. 设置docker的apt存储库

```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

> 如果您使用Ubuntu衍生发行版，例如Linux Mint，则可能需要使用`UBUNTU_CODENAME`而不是`VERSION_CODENAME`

3. 安装docker engine

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
<!-- endtab -->
{% endtabs %}

### Docker-Compose

运行以下命令可以安装Docker Compose的当前稳定版本：

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

> 要安装其他版本的Compose，请替换2.21.0为要使用的Compose版本

将可执行权限应用于二进制文件：

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

> 如果安装后命令失败可以创建指向/usr/bin或路径中任何其他目录的符号链接

```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

测试安装：

```shell
docker-compose --version
```

### Registry

> 访问页面：<http://ip:5000/v2/_catalog>

```shell
docker run -d --name registry \
      -p 5000:5000 \
      -v /data/registry:/usr/local/registry \
      --restart=always \
      registry

vim /etc/docker/daemon.json

systemctl daemon-reload
systemctl restart docker
```

配置文件：

```json
"insecure-registries":["registry.access.redhat.com","quay.io","ip:5000"],
"exec-opts":["native.cgroupdriver=systemd"],
"live-restore":true
```

## 命令指南

### 基础命令

#### 启动docker

```shell
systemctl start docker
```

#### 关闭docker

```shell
systemctl stop docker
```

#### 重启docker

```shell
systemctl restart docker
```

#### 自启docker

```shell
systemctl enable docker
```

### 镜像命令

#### 在仓库搜索镜像

```shell
docker search [OPTIONS] <search_term>
```

- `-f`：根据指定的过滤条件进行搜索
- `--format`：指定输出的格式
- `--limit`：限制搜索结果的数量
- `--no-trunc`：显示完整的镜像描述
- `--stars`：根据指定的星级评分筛选搜索结果

#### 下载指定的镜像

```shell
docker pull [OPTIONS] <image_name>
```

- `-a`：拉取指定镜像名称的所有标签
- `-q`：只显示拉取的镜像ID
- `--disable-content-trust`：禁用内容信任验证

#### 将本地的镜像推送到镜像仓库

```shell
docker push [OPTIONS] <repository_address>/<namespace>/<image_name>:<tag>
```

- `-q`：只显示推送的镜像ID
- `--tls`：与仓库建立TLS连接
- `--all-tags`：同时推送镜像的所有标签
- `--disable-content-trust`：禁用内容信任验证

#### 列出本地上的所有镜像

```shell
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

- `-a`：显示所有镜像，包括中间层镜像
- `-q`：只显示镜像的ID
- `--no-trunc`：显示完整的镜像ID
- `--digests`：显示镜像的摘要信息
- `--filter`：根据指定的条件过滤镜像
- `--format`：指定输出的格式
- `REPOSITORY`：指定仓库名称
- `TAG`：指定镜像的标签

#### 删除指定的本地镜像

```shell
docker rmi [OPTIONS] <image_name>
```

- `-f`：强制删除镜像
- `-q`：只显示被删除的镜像ID
- `--no-prune`：不自动删除与镜像关联的未被使用的中间层镜像

#### 构建一个新的镜像

```shell
docker build [OPTIONS] <path_to_dockerfile>
```

- `-t`：为镜像指定标签
- `-f`：指定要使用的Dockerfile路径
- `-q`：只显示构建过程中生成的镜像ID
- `--build-arg <variable=value>`：为构建过程中的ARG指令传递变量和值

#### 为现有的本地镜像添加一个新的标签

```shell
docker tag <image_name> <new_image_name>
```

#### 显示指定镜像的历史记录

```shell
docker history [OPTIONS] <image_name>
```

- `--format`：指定输出的格式
- `--human`：以人类可读的格式显示文件大小
- `--no-trunc`：显示完整的命令和结果

#### 将指定的镜像保存到一个tar文件中

```shell
docker save <image_name> -o <output_file.tar>
```

#### 从一个tar文件中加载一个镜像

```shell
docker load -i <input_file.tar>
```

### 容器命令

#### 创建并运行一个新容器

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

- `-d`：在后台以守护进程方式运行容器
- `-p`：将容器的端口映射到主机的端口
- `-v`：挂载主机目录或数据卷到容器
- `-e`：设置环境变量
- `-it`：以交互模式运行容器，并分配一个伪终端
- `--rm`：当容器退出时自动删除容器
- `--name`：为容器指定一个名称
- `--network`：指定容器连接的网络模式

#### 启动一个已创建的容器

```shell
docker start [OPTIONS] <container_id>
```

- `-a`：将标准输入（STDIN）、标准输出（STDOUT）和标准错误输出（STDERR）与容器进行关联
- `-it`：以交互模式运行容器，并分配一个伪终端

#### 停止一个正在运行的容器

```shell
docker stop [OPTIONS] <container_id>
```

- `-t`：指定停止容器前等待的时间（默认 10s）

#### 重启一个容器

```shell
docker restart [OPTIONS] <container_id>
```

- `-t`：指定停止容器前等待的时间（默认 10s）
- `-s`：指定用于关闭容器的信号
- `--time-format="<time_format>"`：指定等待时间的格式

#### 删除一个停止的容器

```shell
docker rm [OPTIONS] <container_id>
```

- `-f`：强制删除运行中的容器
- `-v`：删除关联的卷
- `-l`：删除容器与其他容器的链接

#### 列出正在运行的容器

```shell
docker ps [OPTIONS]
```

- `-a`：显示所有容器，包括已停止的容器
- `-q`：只显示容器的ID

#### 查看容器的日志

```shell
docker logs [OPTIONS] <container_id>
```

- `-f`：实时跟踪容器日志的输出
- `-t`：在日志中显示时间戳
- `--tail`：仅显示最后几行的日志
- `--details`：显示更详细的日志输出

#### 在正在运行的容器中执行命令

```shell
docker exec [OPTIONS] <container_id> <command>
```

- `-i`：以交互模式运行容器，并分配一个伪终端
- `-t`：分配一个伪终端

#### 检查容器的详细信息

```shell
docker inspect [OPTIONS] <container_id>
```

- `-f`：指定输出的格式
- `--type`：指定要检查的对象类型
- `--size`：显示相关对象的大小信息

### 数据卷命令

#### 创建一个数据卷

```shell
docker volume create <volume_name>
```

#### 列出所有数据卷

```shell
docker volume ls
```

#### 查看特定数据卷的详细信息

```shell
docker volume inspect <volume_name>
```

#### 删除一个数据卷

```shell
docker volume rm <volume_name>
```

#### 删除未使用的数据卷

```shell
docker volume prune
```

#### 将数据卷绑定到容器的指定路径

```shell
docker run -v <volume_name>:<container_path>
```

#### 将本地路径挂载为数据卷

```shell
docker run -v <host_path>:<container_path>
```

#### 将数据卷复制到本地路径

```shell
docker run -v <volume_name>:/<container_path> -v <host_path>:<container_path>
```
