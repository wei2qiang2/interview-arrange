拉取mysql镜像

docker pull mysql:5.7

查看当前所有镜像

docker images

启动mysql

```shell
 docker run -p 3306:3306 --name mysql -v /mydata/mysql/log:/var/log/mysql -v /mydata/mysql/data:/var/lib/mysql -v /mydata/mysql/conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
 
 docker run -p 3306:3306 --name mysql \ -v /mydata/mysql/log:/var/log/mysql \ -v /mydata/mysql/data:/var/lib/mysql \ -v /mydata/mysql/conf:/etc/mysql \ -e MYSQL_ROOT_PASSWORD=root \ -d mysql:5.7 
```

进入容器内部

```shell
docker exec -it mysql（容器的名称或者ID） /bin/bash
```

进入容器之后查看mysql的位置

```shell
whereis mysql
```

退出容器

```shell
exit
```

修改mysql的配置文件

```shell
cd /mydata/mysql/conf
vi my.cnf
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='SET collation_connection=utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

重启mysql

```shell
docker restart mysql
docker exec -it mysql /bin/bash
cd /etc/mysql
ls
cat my.cnf
```

