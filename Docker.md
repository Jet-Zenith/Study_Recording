# 🐳 Docker 基础到进阶：完整入门指南

## 01. Docker 介绍

Docker 是一个开源的应用**容器（container）**引擎，它允许开发者将应用及其依赖包打包到一个轻量级、可移植的**镜像（image）**中，然后发布到任何流行的操作系统的机器上运行。

- **核心痛点：** 彻底解决“代码在我电脑上运行得好好的，怎么部署到服务器就不行了”的环境不一致问题。
- **核心优势：**  **轻量化：** 比传统虚拟机更节省资源。
  - **秒级启动：** 容器启动通常只需几秒甚至毫秒。
  - **隔离性：** 容器之间相互隔离，互不影响。

------

## 02. Docker 的安装

无论是在 CentOS 还是 Ubuntu 环境下，安装 Docker 的核心逻辑是一致的。

**安装核心步骤概述：**

- **卸载旧版本**（如果系统自带老旧版本）

```shell
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine \
    docker-selinux 
```

- **设置仓库源**（国内通常建议配置阿里云或清华源，否则下载速度极慢）

```bash
# 安装一个yum工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 安装成功后，执行命令，配置Docker的yum源（已更新为阿里云源）：
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo

# 更新yum，建立缓存
sudo yum makecache fast
```

- **安装 Docker Engine** 及其依赖

```bash
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- **启动服务**

```bash
# 启动Docker
systemctl start docker

# 停止Docker
systemctl stop docker

# 重启
systemctl restart docker

# 设置开机自启
systemctl enable docker

# 执行docker ps命令，如果不报错，说明安装启动成功
docker ps
```

- **配置镜像加速器：** 这是国内开发者**必做**的一步。在 `/etc/docker/daemon.json` 中配置国内云厂商提供的加速地址，否则后续拉取镜像会非常痛苦

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://dockerhub.icu",
    "https://docker.m.daocloud.io"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

------

## 03. 快速入门 - 部署 MySQL

学习理论不如直接动手。使用 Docker 部署的第一个应用：

```Bash
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:5.7
```

执行完这段代码后，你的机器上就已经运行了一个纯净的 MySQL 5.7 数据库！

![示意图](images\流程图.png)

------

## 04. 快速入门 - 命令解读

```Bash
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123 \
  mysql:5.7
```

- **`docker run`**: 创建并启动一个新的容器。
- **`-d`**: （Detach）后台运行容器，不阻塞当前终端。
- **`--name mysql`**: 给这个容器起个名字叫 `mysql`，方便以后管理。
- **`-p 3306:3306`**: 端口映射。将宿主机的 3306 端口（冒号前）映射到容器内的 3306 端口（冒号后）。
- **`-e KEY=VALUE`**: 设置环境变量（environment）。这里是初始化 MySQL 的 root 密码。
- **`repository:tag`**: 指定要使用的镜像名称和版本标签（Tag）。

------

## 05. Docker 基础 - 常见命令

| **命令**       | **说明**                       | **文档地址**                                                 |
| :------------- | :----------------------------- | :----------------------------------------------------------- |
| docker pull    | 拉取镜像                       | [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) |
| docker push    | 推送镜像到DockerRegistry       | [docker push](https://docs.docker.com/engine/reference/commandline/push/) |
| docker images  | 查看本地镜像                   | [docker images](https://docs.docker.com/engine/reference/commandline/images/) |
| docker rmi     | 删除本地镜像                   | [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/) |
| docker run     | 创建并运行容器（不能重复创建） | [docker run](https://docs.docker.com/engine/reference/commandline/run/) |
| docker stop    | 停止指定容器                   | [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) |
| docker start   | 启动指定容器                   | [docker start](https://docs.docker.com/engine/reference/commandline/start/) |
| docker restart | 重新启动容器                   | [docker restart](https://docs.docker.com/engine/reference/commandline/restart/) |
| docker rm      | 删除指定容器                   | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/rm/) |
| docker ps      | 查看容器                       | [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) |
| docker logs    | 查看容器运行日志               | [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) |
| docker exec    | 进入容器                       | [docker exec](https://docs.docker.com/engine/reference/commandline/exec/) |
| docker save    | 保存镜像到本地压缩文件         | [docker save](https://docs.docker.com/engine/reference/commandline/save/) |
| docker load    | 加载本地压缩文件到镜像         | [docker load](https://docs.docker.com/engine/reference/commandline/load/) |
| docker inspect | 查看容器详细信息               | [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) |

![命令示意](images\命令示意.png)

------

## 06. Docker 基础 - 命令别名

有时候 `docker ps` 打印出来的信息格式比较乱，或者进入容器的命令太长。我们可以通过配置操作系统的 `alias` （别名）来偷懒。

配置示例（在 Linux/Mac 的 `~/.bashrc` 结尾添加）：

```bash
# 修改/root/.bashrc文件
vi /root/.bashrc
内容如下：
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias dps='docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"'
alias dis='docker images'

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi
```

配置后执行 `source ~/.bashrc` 生效，以后直接输入 `dps` 即可获得排版精美的容器列表。

------

## 07. Docker 基础 - 数据卷挂载 (Volume)

容器是临时的，如果删除了容器，里面产生的数据（如数据库记录）也会灰飞烟灭。**数据卷**就是为了解决数据持久化问题。

- **原理：** 数据卷是交由 Docker 管理的宿主机虚拟目录，它与容器内的目录实时双向同步。

- **操作：**

  Bash

  ```
  # 提前创建数据卷
  docker volume create html-data
  # 挂载到容器内 (格式: -v 卷名:容器内路径)
  docker run -d --name nginx -v html-data:/usr/share/nginx/html nginx
  ```

------

## 08. Docker 基础 - 本地目录挂载 (Bind Mount)

如果你正在开发阶段，经常需要改动代码，用数据卷就不够直观了。这时候可以用**本地目录挂载**。

- **原理：** 直接将宿主机的一个绝对路径映射到容器内。宿主机修改文件，容器内立刻生效，极其适合挂载配置文件或前端代码。

- **操作格式：** `-v /宿主机绝对路径:/容器内路径`

- **示例：**

  Bash

  ```
  docker run -d \
    -p 80:80 \
    -v /mydata/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    nginx
  ```

------

## 09. Docker 基础 - Dockerfile 语法

当官方镜像不能满足需求（比如你要部署自己写的 Java/Python 项目）时，就需要写 **Dockerfile** 来构建镜像。它就是一个包含构建指令的文本文件。

**常用核心指令：**

- **`FROM`**: 站在巨人的肩膀上，指定基础镜像（如 `FROM java:8`）。
- **`ENV`**: 设置环境变量（如时区等）。
- **`COPY`**: 将宿主机的代码、文件拷贝到镜像内部。
- **`RUN`**: 在构建镜像的过程中执行的 Shell 命令（如 `RUN yum install -y vim`）。
- **`EXPOSE`**: 仅仅是声明容器打算监听的端口（方便看文档），不会真正映射端口。
- **`ENTRYPOINT` / `CMD`**: 决定容器启动时默认执行的命令（比如 `java -jar app.jar`）。

------

## 10. Docker 基础 - 自定义镜像

写好 Dockerfile 后，最后一步就是把它变成一个真实的镜像并跑起来。

**实战构建步骤：**

1. 将项目打包好的文件（如 `app.jar`）与编写好的 `Dockerfile` 放在同一个目录下。

2. 在该目录下执行构建命令：

   Bash

   ```
   # -t 指定镜像的名称和版本标签，注意最后有一个点（.），代表当前上下文目录
   docker build -t my-springboot-app:1.0 .
   ```

3. 构建完成后，用 `docker images` 就能看到你的大作了。

4. 运行它：

   Bash

   ```
   docker run -d -p 8080:8080 --name myapp my-springboot-app:1.0
   ```