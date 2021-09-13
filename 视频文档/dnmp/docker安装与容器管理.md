### docker安装

一、 官方安装脚本
```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

二、 daocloud安装脚本
```shell
curl -sSL https://get.daocloud.io/docker | sh
```

### 手动安装docker

一、 卸载旧版本
```shell
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

二、 安装所需的软件包
```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

三、 安装docker
```shell
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo && sudo yum install docker-ce docker-ce-cli containerd.io
```

### 测试安装

一、 查看命令
```shell
docker 
```

二、 服务管理
```shell
# 启动服务
sudo systemctl start docker
# 查看服务转固态
sudo systemctl status docker
# 停止服务
sudo systemctl stop docker
```

### 安装docker-compose

一、 安装docker compose
```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

二、 设置可执行权限
```shell
sudo chmod +x /usr/local/bin/docker-compose
```

三、 建立软连接
```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

四、 测试安装
```shell
docker-compose --version
```

### 启动容器

一、 操作容器
```shel
# 启动容器
docker start redis;
# 重启容器
docker restart redis;
# 停止容器
docker stop redis;
# 删除容器
docker rm redi;
```

二、 进入容器
```shell
docker exec -it redis /bin/sh
```