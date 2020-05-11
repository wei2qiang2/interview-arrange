[docker镜像网站]: hub.docker.com
[docker安装文档]: https://docs.docker.com/engine/install/centos/`

```shell
# 移除原来的安装的docker
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
```

```shell
# 安装yum工具
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
# 配置docker下载地址
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
    yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```shell
$ sudo yum-config-manager --enable docker-ce-nightly # 可选
```

```shell
$ sudo yum-config-manager --enable docker-ce-test # 可选
```

```shell
$ sudo yum-config-manager --disable docker-ce-nightly # 可选
```

```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io # 安装docker相关的包
```

```shell
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
```

```shell
$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

```shell
# 启动docker
$ sudo systemctl start docker
# 查看镜像
sudo docker images
# 开机自启动
sudo systemctl enable docker
# 查看已有容器
docker ps -a
# docker启动的时候自动启动docker中的容器
docker update redis --restart=always
docker update mysql --restart=always
# 下mysql5.7镜像
docker pull mysql:5.7
```

配置docker镜像加速（默认从docker hub下载），阿里云镜像

![](F:\桌面文件\学习文件\images\docker镜像加速配置.png)

```shell
$ sudo docker run hello-world
```

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://vvovjoni.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

```shell
$ sudo yum install /path/to/package.rpm
```

```shell
$ sudo systemctl start docker
```

```shell
$ sudo docker run hello-world	
```

npm配置淘宝镜像

npm config set registry http://registry.npm.taobao.org/