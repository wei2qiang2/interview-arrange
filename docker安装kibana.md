[TOC]

####  安装步骤

##### 拉取镜像(版本需要和`elasticsearch`)

```shell
docker pull kibana:7.4.2
```

##### 创建实例

```shell
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.56.10:9200 -p 5601:5601 -d kibana:7.4.2
```

##### 导入测试数据

https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json



```json
POST /bank/account/_bulk
```

查看索引

```json
GET /_cat/indices
```

`Query DSL`:查询领域对象语言

#### 查询API
GET /bank/_search

```json
{
  "query": { 
    "match_all": {}
    },
  "sort": [
    { "account_number": "asc" },
    { "balance": "desc" }
  ],
  "from": 50,
  "size":50,
  "_source": ["balance", "account_number"]
}
```



##### 字段匹配

GET /bank/_search

```json
{
  "query": {
    "match": {
      "account_number": "20"
    }
  }
}
```

##### 最终按照评分排序（依赖于倒排索引）
GET /bank/_search 

```json
{
  "query": {
    "match": {  
      "address": "Mill lane"
    }
  }
}
```



##### 短语匹配(不进行分词，不区分大小写)
GET /bank/_search

```json
{
  "query": {
    "match_phrase": {
      "address": "mill lane"
    }
  }
}
```

##### 短语匹配(不进行分词,当成精确值来查找)
GET /bank/_search

```json
{
  "query": {
    "match": {
      "address.keyword": "198 Mill Lane"
    }
  }
}
```



##### `multi match`多字段匹配,会进行分词
GET /bank/_search

```json
{
  "query": {
    "multi_match": {
      "query": "mill",
      "fields": ["address", "city"]
    }
  }
}
```



##### `bool`复合查询,合并多个查询条件, `must`必须满足, `must not`,必须不满足,`should `能匹配上最好，匹配上之后得分高一些
GET /bank/_search

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "gender": "m"
          }
        },
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "age": "18"
          }
        }
      ],
      "should": [
        {
          "match": {
            "lastname": "Wallance"
          }
        }
      ]
    }
  }
}
```

##### filter 结果过滤(不会贡献相关性得分)
GET /bank/_search

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "age": {
              "gte": 18,
              "lte": 30
            }
          }
        }
      ]
    }
  }
}
```

##### 相关性得分
GET /bank/_search

```json
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "age": {
            "gte": 18,
            "lte": 30
          }
        }
      }
    }
  }
}
```

##### term查询 精确匹配的用term，文本检索用match
GET /bank/_search

```json
{
  "query": {
    "term": {
      "age": "28"
    }
  }
}
```

#### 聚合分析`API`
```json
"aggregations" : {
    "<aggregation_name>" : {
        "<aggregation_type>" : {
            <aggregation_body>
        }
        [,"meta" : {  [<meta_data_body>] } ]?
        [,"aggregations" : { [<sub_aggregation>]+ } ]?
    }
    [,"<aggregation_name_2>" : { ... } ]*
}
```

##### 搜索address中包含mill的人，并看年龄分布以及平均年龄，但不显示人员详情
GET /bank/_search

```json
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvgAgg":{
      "avg": {
        "field": "age"
      }
    },
    "balanceAvgAgg": {
      "avg": {
        "field": "balance"
      }
    }
  }
}
```

##### 按照年龄聚合，并看这些年龄段的人的平均薪资
GET /bank/_search

```json
{
  "aggs": {
    "aggAvg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "balanceAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

##### 查出所有人的年龄分布，并且这些年龄段中性别分别为M和F的平均薪资以及这个年龄段总体的平均薪资
GET /bank/_search

```json
{
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "bAvg":{
          "avg": {
            "field": "balance"
          }
        },
        "sexAgg": {
          "terms": {
            "field": "gender.keyword",
            "size": 10
          },
          "aggs": {
            "balanceAgg": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
```

#### 映射（Mapping）
##### Mapping Type，6.0之后已经移除了类型了

类型介绍：https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-types.html

##### 查看映射
```json
GET /bank/_mapping
```

```json
{
  "bank" : {
    "mappings" : {
      "properties" : {
        "account_number" : {
          "type" : "long"
        },
        "address" : {
          "type" : "text", // 可全文检索
          "fields" : {
            "keyword" : {
              "type" : "keyword", // 精确匹配
              "ignore_above" : 256
            }
          }
        },
        "age" : {
          "type" : "long"
        },
        "balance" : {
          "type" : "long"
        },
        "city" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "email" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "employer" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "firstname" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "gender" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "lastname" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "state" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}

```

##### 创建索引和映射规则
`PUT /my-index`

```json
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  }, // 精确匹配
      "name":   { "type": "text"  }     // 支持全文检索
    }
  }
}
```

##### 为已有索引添加新的字段映射
`PUT /my-index/_mapping`

```json
{
  "properties": {
     // index控制这个属性的值能否被索引，默认都是false
    "employee":    { "type": "keyword", "index": false } 
  }
}
```

##### 更新已存在的映射：数据迁移，创建新的索引，指定新的映射，迁移数据

##### 查看旧映射

```json
GET /bank/_mapping
```
##### 创建新的索引
PUT /newbank

```json
{
  "mappings": {
    "properties": {
      "account_number": {
        "type": "long"
      },
      "address": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "balance": {
        "type": "long"
      },
      "city": {
        "type": "keyword"
      },
      "email": {
        "type": "keyword"
      },
      "employer": {
        "type": "keyword"
      },
      "firstname": {
        "type": "text"
      },
      "gender": {
        "type": "keyword"
      },
      "lastname": {
        "type": "text"
      },
      "state": {
        "type": "keyword"
      }
    }
  }
}
```

##### 查看新建的映射
`GET /newbank/_mapping`
##### 数据迁移,无类型的版本写法
`POST /_reindex`

```json
{
  "source": {
    "index": "bank"
  },
  "dest": {
    "index": "newbank"
  }
}
```
##### 有版本的写法
`POST /_reindex`

```json
{
  "source": {
    "index": "bank",
    "type": "account"
  },
  "dest": {
    "index": "newbank"
  }
}
```
##### 查看迁移后的数据
```json
GET /newbank/_search
```

#### 分词

`POST /_analyze`

```json
{
    "analyzer": "分词器类型",
    "text": "要分词的文本"
}
```

##### `ik`分词器

https://github.com/medcl/elasticsearch-analysis-ik/releases

#### 安装步骤

- 进入es容器内部的`plugins`目录


```shell
docker exec -it elasticsearch /bin/bash
```

- 进入插件目录


```shell
cd plugins
```

- 下载


```shell
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.4.2/elasticsearch-analysis-ik-7.4.2.zip
```

- 解压


```shell
unzip 文件名
```

- 给`ik`分词器的文件夹权限


```shell
chmod -R 777 ik/
```

- 进入`elasticsearch`容器的bin文件夹下面运行命令

- 查看插件列表

```shell
elasticsearch-plugin list
```

- 装完之后重启容器


```shell
docker restart elasticsearch
```

- 测试


`POST /_analyze`

```json
{
	"analyzer": "ik_smart", // 智能分词： 我，是，中国人
	"text": ["我是中国人"]

}	
```

`POST/_analyze`

```json
{
  "analyzer": "ik_max_word", // 我，是，中，国，人
  "text": ["我是中国人"]
}
```



##### `ik`分词器自定义词库

1. 自己写个项目，ik分词器给我们自己的项目发送请求，返回生成的词
2. 利用nginx，给nginx发送请求，返回生成的词