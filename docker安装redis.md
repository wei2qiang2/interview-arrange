拉取redis镜像

```shell
docker pull redis
```

启动redis

创建配置文件，否则启动的时候配置文件挂载找不到会把redis.conf当做文件夹

```shell
mkdir -p /mydata/redis/conf
touch /mydata/redis/conf/redis.conf
```

```shell
docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf
```

使用redis镜像执行redis-cli命令连接

```shell
docker exec -it redis redis-cli
```

当前的redis是没有开启持久化的，重启之后数据全部丢失，所以需要修改redis的配置实现持久化

在redis.conf中添加

```shell
appendonly yes # 开启aof的持久化策略
```



用redis-deskop可视化界面连接redis

redis配置文件参考地址：raw.githubusercontent.com/antirez/redis/4.0/redis.conf



环境搭建

maven和jdk版本

maven的镜像仓库地址改为阿里云的

编译环境改为1.8的



vscode安装的插件

auto close tag

auto rename tag

Chinese

Eslint

HTML Css Support

HTML Snippets

JavaScript(ES6)

live server

open in browser

Vetur