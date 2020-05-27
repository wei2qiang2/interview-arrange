拉取镜像

```shell
docker pull elasticsearch:7.4.2
```

创建挂载文件夹

```shell
mkdir -p /mydata/elasticsearch/config
mkdir -p /mydata/elasticsearch/data
```

创建配置文件

```shell
echo "http.host:0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml
```

创建实例

```shell
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPS="-Xms64m -Xmx128m" -v /mydata/elasticsearch/config/elasticsearch.yml:/url/share/elasticsearch/config/elasticsearch.yml -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.4.2
```

查看启动失败日志

```shell
docker logs (容器名称)
```



挂载文件夹的权限修改

```shell
chmod -R 777 /mydata/elasticsearch
```

#### ES的内存太小了，重新创建个大点内存的ES容器

```shell
docker stop elasticsearch # 容器ID
docker rm elasticsearch # 移除容器
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPS="-Xms64m -Xmx512m" -v /mydata/elasticsearch/config/elasticsearch.yml:/url/share/elasticsearch/config/elasticsearch.yml -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.4.2 # 创建容器并运行，此处挂载的文件目录不变，那么原来的数据就存在
```

