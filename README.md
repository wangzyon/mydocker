# 1 摘要

使用Dockerfile、nvidia-docker、docker-compose技术实现开发、部署环境的快速搭建，安装、启动、停止、卸载均是一条指令。提供若干Dockerfile模板，满足不同环境配置需求。
| DOCKFILE                                      | SOURCE                                                       |
| --------------------------------------------- | ------------------------------------------------------------ |
| Dockerfile-cu111-cp38-torch1.8.2-linux_x86_64 | torch-1.8.2+cu111-cp38-cp38-linux_x86_64.whl<br/>torchaudio-0.8.2-cp38-cp38-linux_x86_64.whl<br/>torchvision-0.9.2+cu111-cp38-cp38-linux_x86_64.whl<br/>Anaconda3-2020.07-Linux-x86_64.sh |
| Dockerfile-cu115-cp39-torch1.11-linux_x86_64  | torch-1.11.0+cu115-cp39-cp39-linux_x86_64.whl<br/>torchaudio-0.11.0+cu115-cp39-cp39-linux_x86_64.whl<br/>torchvision-0.12.0+cu115-cp39-cp39-linux_x86_64.whl<br/>Anaconda3-2022.05-Linux-x86_64.sh |
| Dockerfile-cpu-cp38-linux_x86_64              | Anaconda3-2020.07-Linux-x86_64.sh                            |

# 2 Docker安装（ubuntu）

## 2.1 卸载旧版本

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

## 2.2 安装新版本

1. 更新

```bash
 sudo apt-get update

 sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

2. Add Docker’s official GPG key

```bash
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

3. 构建docker仓库

```bash
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

4. 安装docker engine

```bash
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
```

5. 验证安装是否成功

```bash
docker version
```

6. docker用户组添加普通用户

```bash
# 切换至普通用户
su wzy
# 添加到docker用户组
sudo usermod -aG docker $USER
# 更新
newgrp docker
# 重启docker
sudo service docker restart
```

7. 查看docker服务

```bash
# docker是C-S模式，docker操作要启动docker服务
service docker status
service docker start
```

8. 安装nvidia-docker

安装nvidia工具：[官方安装步骤](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)

```shell
curl https://get.docker.com | sh \
  && sudo systemctl --now enable docker
  
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   
curl -s -L https://nvidia.github.io/nvidia-container-runtime/experimental/$distribution/nvidia-container-runtime.list | sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
   
apt-get update

apt-get install -y nvidia-docker2
```

修改/etc/docker/daemon.json，添加runtimes部分

```bash
{
   "registry-mirrors": [
      "https://7hsq3dad.mirror.aliyuncs.com",
      "https://dockerhub.azk8s.cn",
      "https://reg-mirror.qiniu.com",
      "http://hub-mirror.c.163.com"
    ],
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
         }	
    }
}
```

启动测试

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker

docker run --runtime=nvidia --rm nvidia/cuda:10.2-base nvidia-smi

# 若出现错误，应更新安装NVIDIA显卡驱动（NVIDIA官网下载.run驱动，执行安装）
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: process_linux.go:545: container init caused: Running hook #1:: error running hook: exit status 1, st: nvidia-container-cli: initialization error: driver error: failed to process request: unknown.

```

## 2.3 docker常用命令

```bash
# 停止所有容器
docker stop $(docker ps -q)

# 删除所有容器
docker stop $(docker ps -aq)

# 进入容器 
docker exec -it <container ID> /bin/bash
```

## 2.4 docker-compose安装

```bash
# v2.6.0可以替换为新版本，新版本号在https://github.com/docker/compose中查看，github访问太慢，可以用daocloud下载
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/v2.6.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

# 查看安装是否成功
docker-compose --version
```

# 3 文件准备

## 3.1 拉取目录

```bash
git clone https://github.com/wangzyon/mydocker.git
```

拉取后目录如下：

```
mydocker
    └─components													
    	├─dockerfile_xxx1                                        # Dockfile
    	└─dockerfile_xxx2                                        # Dockfile
    └─volume                                                     # 数据卷,存放项目代码和数据
    └─docker-compose.yml                                         # docker-compose配置文件
```

## 3.2 选择容器类型

根据使用场景，选择dockerfile，并将dockerfile对应文件资源拷贝至`components`目录；

| DOCKFILE                                      | SOURCE                                                       |
| --------------------------------------------- | ------------------------------------------------------------ |
| Dockerfile-cu111-cp38-torch1.8.2-linux_x86_64 | torch-1.8.2+cu111-cp38-cp38-linux_x86_64.whl<br/>torchaudio-0.8.2-cp38-cp38-linux_x86_64.whl<br/>torchvision-0.9.2+cu111-cp38-cp38-linux_x86_64.whl<br/>Anaconda3-2020.07-Linux-x86_64.sh |
| Dockerfile-cu115-cp39-torch1.11-linux_x86_64  | torch-1.11.0+cu115-cp39-cp39-linux_x86_64.whl<br/>torchaudio-0.11.0+cu115-cp39-cp39-linux_x86_64.whl<br/>torchvision-0.12.0+cu115-cp39-cp39-linux_x86_64.whl<br/>Anaconda3-2022.05-Linux-x86_64.sh |
| Dockerfile-cpu-cp38-linux_x86_64              | Anaconda3-2020.07-Linux-x86_64.sh                            |

## 3.3 容器编排

根据使用场景编辑容器编排配置文件`vim docker-compose.yml`：

1. dockerfile参数修改3.2节中指定dockerfile文件名称；
2. 修改image、container_name参数，自定义镜像和容器名称
3. 根据需求修改端口映射ports参数；
4. networks用于多容器网络通信，不涉及多容器通信可删除字段前后两处networks字段；
5. CPU容器应删除字段devices；

```shell
version: '3'

services:
    dev:
        image: wzy_image
        container_name: wzy_container
        build: 
            context: ./components
            dockerfile: Dockerfile-cu111-cp38-torch1.8.2-linux_x86_64
        shm_size: '64gb'
        cap_add:
            - ALL
        ports:
            - "12009:6379"
            - "12010:12010"
            - "12011:12011"
            - "12012:12012"
            - "12022:22"
        volumes:
            - ./volume:/volume
        networks:
            - trt_net
        stdin_open: true
        tty: true
        deploy:
          resources:
            reservations:
              devices:
                - driver: nvidia
                  count: 1
                  capabilities: [gpu]
                  
networks:
  trt_net:
```

# 4 安装与卸载

## 4.1 安装

```bash
# 切换至docker-compose.yml同级目录执行
docker-compose build
```

## 4.2 启动和停止

```bash
# 切换至install目录
# 启动
docker-compose up
# docker-compose up -d后台启动

# 停止
docker-compose stop
```

## 4.3 卸载

```bash
# 切换至install目录
# 卸载
docker-compose down --rmi local
```

# 5 端口转发

在不重新创建容器前提下，修改端口映射

```bash
# 添加
iptables -t nat -A PREROUTING -i eno1 -p tcp --dport 8805 -j DNAT --to 192.168.0.94:8805

iptables -t nat -A PREROUTING -i eno1 -p tcp --dport 8900 -j DNAT --to 192.168.0.94:8900
# 查看
sudo iptables -t nat -L -n
# 删除
iptables -t 表名 -F 链名
iptables -t nat -F PREROUTING
```

