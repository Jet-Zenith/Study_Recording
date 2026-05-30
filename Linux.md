# Linux 学习笔记

## 一、 Linux 概述

### 1. 介绍 & 系统安装

- **Linux 简介**：一种开源、免费的类 Unix 操作系统，具有高稳定性、高安全性、多用户多任务等特点。广泛应用于服务器端、嵌入式领域。
- **核心组成**：
  - **内核 (Kernel)**：系统的心脏，负责管理硬件、内存、进程等核心功能。
  - **壳 (Shell)**：命令解释器，充当用户与内核之间的桥梁（如 Bash）。
  - **文件系统**：管理文件的组织方式。
  - **应用程序**：运行在系统上的各种软件。
- **系统安装**：
  - **常规做法**：在 Windows/macOS 上安装虚拟机软件（如 **VMware Workstation** 或 **VirtualBox**），然后在虚拟机中安装 Linux 发行版（如 **CentOS**、**Ubuntu** 或 **Rocky Linux**）。
  - **网络配置**：通常选择 **NAT 模式**，方便虚拟机共享主机的网络上网。

### 2. 远程连接 & 目录结构

- **远程连接**：
  - 由于服务器通常在机房，开发和运维人员需要通过网络进行远程操作。
  - **常用工具**：**Xshell**、**SecureCRT**、**MobaXterm**、**FinalShell**。
  - **连接要素**：服务器的 IP 地址、SSH 端口号（默认 **22**）、用户名（如 `root`）和密码。
- **目录结构 (树状结构)**：
  - Linux 没有 Windows 的盘符概念（C盘、D盘），一切皆文件，所有文件都从根目录 `/` 开始。
  - **核心目录说明**：
    - `/bin` 和 `/sbin`：存放系统可执行命令（`sbin` 多为系统管理员使用的指令）。
    - `/etc`：存放系统和软件的**配置文件**。
    - `/home`：普通用户的家目录（如用户 tom 的家目录是 `/home/tom`）。
    - `/root`：系统管理员（超级用户）的家目录。
    - `/usr`：用户安装的应用程序和文件（类似于 Windows 的 `Program Files`）。
    - `/var`：存放经常变化的文件，如**系统日志** (`/var/log`)。
    - `/opt`：大型附加软件的安装目录。

## 二、 Linux 常用命令

### 1. 目录 & 文件操作

- `pwd`：显示当前工作目录的绝对路径。
- `ls`：列出目录内容。
  - `ls -l` (可简写为 `ll`)：以列表形式显示详细信息。
  - `ls -a`：显示所有文件，包括以 `.` 开头的隐藏文件。
- `cd`：切换目录。
  - `cd ~`：回到当前用户的家目录。
  - `cd ..`：回到上一级目录。
- `mkdir`：创建新目录。
  - `mkdir -p path/to/dir`：递归创建多级目录。
- `touch`：创建一个空文件。
- `rm`：删除文件或目录。
  - `rm -r`：递归删除目录。
  - `rm -f`：强制删除，不提示（**注意：`rm -rf /\*` 极其危险，切勿执行**）。

### 2. 拷贝移动 & 打包压缩

- `cp`：复制文件或目录。
  - `cp source_file dest_file`：复制文件。
  - `cp -r source_dir dest_dir`：递归复制整个目录。
- `mv`：移动文件/目录，或者重命名。
  - `mv old_name new_name`：重命名。
  - `mv file.txt /target/dir/`：移动文件。
- `tar`：打包与解压命令（最常用）。
  - **打包压缩**：`tar -zcvf archive.tar.gz directory_to_compress`
  - **解压文件**：`tar -zxvf archive.tar.gz`（若要指定解压目录，加 `-C /target/dir`）。
  - *参数释义：`-z` 用 gzip 压缩/解压，`-c` 创建新档案，`-x` 解压，`-v` 显示过程，`-f` 指定档案名。*

### 3. 文本编辑 & 查找

- **Vim 文本编辑器**：
  - **三种模式**：命令模式（默认）、插入模式（按 `i` 进入）、底行模式（按 `:` 进入）。
  - **常用操作**：
    - 进入编辑：按 `i`。
    - 退出编辑回到命令模式：按 `Esc`。
    - 保存并退出：在底行模式下输入 `:wq`。
    - 强制退出不保存：在底行模式下输入 `:q!`。
- **查看文本**：
  - `cat`：一次性查看文件所有内容。
  - `more` / `less`：分屏查看大文件。
  - `tail -f filename`：**实时查看**文件末尾内容（极常用于看系统/项目日志）。
- **查找**：
  - `find`：在文件系统中搜索文件。如 `find / -name "*.txt"`。
  - `grep`：在文件内过滤指定的文本行。常配合管道符 `|` 使用，如 `ps -ef | grep java`（查找 Java 进程）。

## 三、 软件安装

在企业级环境中，软件安装通常通过 **包管理器（如 yum/dnf/apt）**、**解压版二进制包** 或 **源码编译** 来完成。

### 1. JDK 安装 (以解压版为例)

1. 下载 JDK 的 Linux 版本（`.tar.gz`）。

2. 通过远程工具上传到 Linux（如 `/usr/local/` 目录）。

3. 解压：`tar -zxvf jdk-xxx.tar.gz`。

4. **配置环境变量**：编辑 `/etc/profile` 文件，在末尾添加：

   ```Bash
   export JAVA_HOME=/usr/local/jdk-xxx
   export PATH=$JAVA_HOME/bin:$PATH
   ```

5. 刷新配置：`source /etc/profile`。

6. 验证：`java -version`。

### 2. MySQL 详细安装与防坑配置 (CentOS 7/8 示例)

在 Linux 上安装 MySQL，推荐使用官方的 YUM 仓库安装，这样会自动处理复杂的依赖关系。

#### （1） 清理环境

CentOS 系统通常默认自带了 `mariadb`（MySQL 的一个分支），如果不卸载，会导致后续安装 MySQL 失败。

- **检查是否存在 mariadb**：

  ```Bash
  rpm -qa | grep mariadb
  ```

- **如果输出有内容，强制卸载它**（假设输出为 `mariadb-libs-xxxxx`）：

  ```Bash
  rpm -e --nodeps mariadb-libs-xxxxx
  ```

#### （2）下载并添加 MySQL 官方 YUM 源

因为默认的 Linux 软件库里没有最新的官方 MySQL，我们需要去官网抓取它的 YUM 配置文件。

```Bash
# 下载 MySQL 8.0 的 YUM 源安装包
wget https://dev.mysql.com/get/mysql80-community-release-el7-7.noarch.rpm

# 安装该 YUM 源
rpm -ivh mysql80-community-release-el7-7.noarch.rpm
```

#### （3）正式安装 MySQL

有了官方源后，直接使用 `yum` 进行安装。

```Bash
# 这步会下载大量包，请保持网络畅通
yum -y install mysql-community-server
```

> ⚠️ **大坑提示 (GPG 密钥过期报错)**：
>
> 如果安装时遇到 `Failing package...` 提示密钥过期，可以使用以下命令跳过检查或更新密钥：
>
> ```Bash
> yum -y install mysql-community-server --nogpgcheck
> ```

#### （4）启动 MySQL 并获取初始密码

```Bash
# 1. 启动 MySQL 服务
systemctl start mysqld

# 2. 设置开机自启（防止服务器重启后网页连不上数据库）
systemctl enable mysqld

# 3. MySQL 启动后会生成一个临时的随机密码，必须去日志里抓取它
grep 'temporary password' /var/log/mysqld.log
```

*记住控制台输出最后那串长长的、奇形怪状的字符串，那就是你的初始密码。*

#### （5）首次登录与修改密码策略

```Bash
# 登录数据库（回车后输入刚才抓取到的临时密码，输入时屏幕不显示，输完直接回车）
mysql -u root -p
```

进入 MySQL 终端后（看到 `mysql>` 开头），**必须先修改密码才能做其他操作**。但 MySQL 8.0 对密码强度有极高要求，如果你想设一个简单的密码（比如 `123456`）方便学习，需要先降低安全策略：

```sql
-- 1. 修改密码强度验证等级为 LOW（只验证长度）
SET GLOBAL validate_password.policy=0;

-- 2. 修改密码最小长度为 4 位
SET GLOBAL validate_password.length=4;

-- 3. 现在可以修改成你喜欢的简单密码了
ALTER USER 'root'@'localhost' IDENTIFIED BY '1234';
```

#### （6）开启远程连接权限与防火墙

默认情况下，MySQL 的 root 用户只允许在 Linux 本机（localhost）登录，我们在 Windows 用 Navicat 或代码是连不上的。

```SQL
-- 切换到 mysql 核心数据库
USE mysql;

-- 将 root 用户的允许登录主机改为 '%'（代表任意 IP 都可以连接）
UPDATE user SET host = '%' WHERE user = 'root';

-- 刷新权限使其生效
FLUSH PRIVILEGES;

-- 退出 MySQL 终端
EXIT;
```

最后，退出到 Linux 终端，别忘了放行防火墙的 **3306** 端口（前一节笔记提过的命令）：

```Bash
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

### 3. Nginx 详细编译安装与服务配置

Nginx 官方虽然也提供编译好的二进制包，但企业中 90% 以上都会选择**源码编译安装**。因为这样可以自由选择安装路径、根据业务需求随时追加各种第三方插件和模块。

#### (1)安装编译所需要的“四大金刚”依赖

Nginx 是纯 C 语言写的，编译它必须先准备好编译器和一些处理文本、加密的底层库。

```Bash
yum -y install gcc gcc-c++ make pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

- `gcc` / `gcc-c++`：C/C++ 编译器。
- `pcre`：支持 Nginx 在配置文件里写正则表达式（比如匹配路径）。
- `zlib`：支持网页内容在传输时进行 Gzip 压缩。
- `openssl`：支持 HTTPS 加密传输。

#### (2)下载并解压 Nginx 源码

```Bash
# 习惯上把下载的源码包放在 /usr/local/src 目录下
cd /usr/local/src

# 下载 Nginx 官方源码包（可以去官网看最新稳定版版本号）
wget http://nginx.org/download/nginx-1.24.0.tar.gz

# 解压
tar -zxvf nginx-1.24.0.tar.gz

# 进入解压后的源码目录
cd nginx-1.24.0
```

#### (3)编译三部曲（核心步骤）

- **第一步：配置 (`./configure`)**

  用于检查系统环境，并指定安装路径和开启哪些功能。

  ```Bash
  ./configure --prefix=/usr/local/nginx --with-http_ssl_module
  ```

  - `--prefix=/usr/local/nginx`：指定待会儿编译好后，Nginx 安装到哪里。
  - `--with-http_ssl_module`：顺便把支持 HTTPS 功能的模块勾选上（以后用得着）。

- **第二步：编译 (`make`)**

  将 C 语言源码翻译成 Linux 能看懂的二进制可执行文件。

  ```Bash
  make
  ```

- **第三步：安装 (`make install`)**

  将编译好的文件、配置文件、默认网页拷贝到我们刚才指定的 `--prefix` 路径中。

  ```Bash
  make install
  ```

#### (4)目录结构与启动验证

此时，你可以彻底离开 `/usr/local/src/nginx-1.24.0` 这个源码目录了，以后再也不用管它。我们要去的是 `--prefix` 指定的**安装目录**：

```Bash
cd /usr/local/nginx
ls
# 你会看到 4 个核心目录：
# conf (放配置文件)  html (放前端静态网页)  logs (放日志)  sbin (放启动脚本)
```

- **启动 Nginx**：

  ```Bash
  ./sbin/nginx
  ```

- **检查 Nginx 是否启动成功**：

  ```Bash
  ps -ef | grep nginx
  ```

  *如果看到一个 `master process` 和一个或多个 `worker process`，说明它完美跑起来了！*

- **放行防火墙 80 端口**：

  ```Bash
  firewall-cmd --zone=public --add-port=80/tcp --permanent
  firewall-cmd --reload
  ```

  现在在你的 Windows 浏览器里输入 Linux 的 IP 地址，就能看到那句亲切的 `Welcome to nginx!` 了。

#### (5)Nginx 常用运维命令

在 Nginx 的安装目录下（`/usr/local/nginx`）：

- 停止服务（优雅关闭）：`./sbin/nginx -s quit`
- 强制停止：`./sbin/nginx -s stop`
- **热重载配置文件**（修改完 `nginx.conf` 后不用重启，直接生效）：`./sbin/nginx -s reload`
- 检查配置文件是否写错（改完配置后、重载前一定要运行一下这个）：`./sbin/nginx -t`

这样拆解下来，是不是对整个安装流程心中有底多了？建议你在虚拟机操作时，每敲完一步，都用 `echo $?` 或者 `ls`、`ps` 验证一下当前步骤是否成功，这样排查问题会非常快。

## 四、 项目部署

在前后端分离的架构中，前端和后端项目通常需要分开部署，通过 Nginx 建立连接。

### 1. 前端项目部署（基于 Nginx）

前端项目（如 Vue/React）打包后为纯静态文件（`.html`, `.js`, `.css`），需要使用 Nginx 进行托管和反向代理。

1. **放置静态资源**：

   - 在本地开发环境中执行打包命令（如 `npm run build`），生成 `dist` 文件夹。
   - 将 `dist` 内的所有静态资源上传到 Linux 上 Nginx 的 `html` 目录中（默认路径通常为 `/usr/local/nginx/html`）。

2. **修改 Nginx 配置文件 (`conf/nginx.conf`)**：

   ```nginx
   server {
       listen       80;               # 监听 80 端口（浏览器默认访问端口）
       server_name  localhost;        # 域名或服务器 IP
   
       client_max_body_size 10m;      # 允许客户端上传的最大文件大小为 10MB
   
       # 1. 托管前端静态资源
       location / {
           root   html;               # 静态资源根目录指向刚才存放文件的 html 文件夹
           index  index.html index.htm; # 默认首页
           try_files $uri $uri/ /index.html; # 解决 Vue/React 路由刷新 404 的关键配置
       }
   
       # 2. 反向代理后端接口（解决跨域问题）
       location ^~ /api/ {
           # 路径重写：把请求路径中的 /api 前缀去掉。
           # 例如：请求 /api/user/login 变成 /user/login
           rewrite ^/api/(.*)$ /$1 break; 
   
           # 将请求转发给真正的后端 Java 服务（假设后端运行在 8080 端口）
           proxy_pass http://localhost:8080; 
       }
   }
   ```

3. **运行命令**：

   - 必须在 Nginx 的安装目录下执行：
   - **首次启动**：`sbin/nginx`
   - **修改配置后热重载（不停止服务）**：`sbin/nginx -s reload`

### 2. 后端项目部署（以 Spring Boot 的 Jar 包为例）

后端项目主要负责处理业务逻辑和提供 API 接口。

1. **上传文件**：在本地开发环境中将项目打包成 `jar` 包（如 `app.jar`），并上传到 Linux 服务器的指定目录。

2. **启动项目**：

   - 前台启动（测试用）：`java -jar app.jar`（关闭终端连接项目就会停止）。

   - **后台守护启动（生产环境用）**：

     Bash

     ```
     nohup java -jar app.jar > server.log 2>&1 &
     ```

     - `nohup`：不挂断地运行命令。
     - `> server.log`：将日志重定向输出到指定文件。
     - `&`：让程序在后台运行。

3. **关闭项目**：

   - 查找项目进程号：`ps -ef` 或 `ps -ef | grep java`
   - 终止进程：`kill -9 进程号(PID)`。

### 