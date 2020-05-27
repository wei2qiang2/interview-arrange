1. 随便启动一个nginx实例，只是为了复制出配置

   ```shell
   docker run -p 80:80 --name nginx -d nginx:1.10
   ```

2. 在mydata文件夹下面创建nginx文件夹

   ```shell
   cd /mydata
   mkdir nginx
   ```

3. 将容器内的配置文件拷贝到当前文件，别忘了后面的点

   ```shell
   docker container cp nginx:/etc/nginx . # 没找到会自动下载
   ```

4. 修改文件夹名称：nginx修改为conf

   ```shell
   mv nginx conf
   ```

5. 重新创建nginx文件夹

   ```shell
   mkdir nginx
   ```

6. 把conf文件移动到mydata的nginx文件夹下

   ```shell
   mv nginx.conf /nginx
   ```

7. 终止原容器

   ```shell
   docker stop nginx
   ```

8. 执行命令删除原容器

   ```shell
   docker rm $ContainerId
   ```

9. 创建新的nginx容器

   ```json
   docker run -p 80:80 --name nginx -v /mydata/nginx/html:/usr/share/nginx/html -v /mydata/nginx/logs:/var/log/nginx -v /mydata/nginx/conf:/etc/nginx -d nginx:1.10
   ```

   

10. 在nginx的html文件夹下面创建index.html

    ```shell
    cd /mydata/nginx/html
    vi index.html
    <h1>GUlimall</h1>
    ```

11. 在html文件夹下面创建es文件夹，存放es所需相关资源

    ```shell
    vi fenci.text
    尚硅谷
    乔碧萝
    ```

12. 访问

    ```shell
    192.168.56.10/es/fenci.txt
    ```

13. 修改IK分析器的配置

    ```shell
    cd /mydata/elasticsearch/plugins/ik/config
    ```

14. 修改配置文件

    ```shell
    vi IKAnalyzer.cfg.xml
    ```

    ```xml
    <properties>
        <comment>IK Analyzer扩展配置</comment>
        <!-- 配置扩展词典 -->
        <entry key="ext_dict"></entry>
        <!-- 配置扩展停止词词典 -->
        <entry key="key_stopwords"></entry>
        <!-- 配置远程扩展词典 -->
        <entry key="remote_ext_dict">http://192.168.56.10/es/fenci.txt</entry>
        <!--<entry key="remote_ext_stopwords">words_location</entry>-->
    </properties>
    ```

15. 重启es

    ```shell
    docker restart elasticsearch
    ```

16. 测试分词效果

    POST /_analyze

    ```json
    {
        "analyzer": "ik_max_word",
        "text": ["乔碧萝点下"] // 乔碧萝，殿下
    }
    ```

    


​    

​    

​    

​    

​    

​    