### 安装elasticsearch

#### docker环境变量调整
- 参考：https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
```
wsl -d docker-desktop
sysctl -w vm.max_map_count=262144
```


- 参考：https://blog.csdn.net/qq_15973399/article/details/105094444
```
docker pull elasticsearch:7.6.2
```

```
docker run --restart=always -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" --name='elasticsearch' --cpuset-cpus="1" -m 2G -d elasticsearch:7.6.2
```

访问地址：http://localhost:9200

#### 安装分词器 
- 参考：https://github.com/medcl/elasticsearch-analysis-ik
```
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.2/elasticsearch-analysis-ik-7.6.2.zip
```

### 安装kibana
```
docker run  --restart=always --link elasticsearch:elasticsearch --name kibana -p 5601:5601 -d kibana:7.6.2
```

#### 进入容器修改配置文件kibana.yml
```
docker exec  -it kibana bash
vi config/kibana.yml
########################
# 指定es的地址
elasticsearch.hosts: ["http://本机ip:9200"]
# 中文化
i18n.locale: "zh-CN"
# 修改外网访问 可选
server.host: "0.0.0.0"
exit
########################
docker restart kibana
```

打开地址：http://localhost:5601



#### kibana开发工具添加index
```
PUT index

POST index/_mapping
{
        "properties": {
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            }
        }

}


POST index/_create/1
{
  "content": "美国留给伊拉克的是个烂摊子吗"
}

POST index/_create/2
{"content":"公安部：各地校车将享最高路权"}

POST index/_create/3
{"content":"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}


GET index/_search
{
    "query" : { "match" : { "content" : "中国" }},
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "content" : {}
        }
    }
}
```


### springboot项目连接
https://github.com/Motianshi/all-search.git
