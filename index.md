### ElasticSearch学习笔记

#### 软件下载地址

``` html
ElasticSearch: https://mirrors.huaweicloud.com/elasticsearch/?C=N&O=D
logstash: https://mirrors.huaweicloud.com/logstash/?C=N&O=D
kibana: https://mirrors.huaweicloud.com/kibana/?C=N&O=D

可视化界面elasticsearch-head
https://github.com/mobz/elasticsearch-head   //使用环境需要nodeJs
使用时会有连接不上，跨域问题（跨端口，跨网站等）
修改配置  config/elasticsearch.yml
添加：
1.http.cors.enabled: true
2.http.cors.allow-origin: "*"
可以修改kibana的配置文件
kibana.yml:
	i18n.locale: "zh-CN"
```

#### ES核心概念

> elasticsearch是面向文档;关系行数据库 和elasticsearch 客观的对比!一切都是Json

| Relational DB      | ElasticSearch |
| ------------------ | ------------- |
| 数据库（database） | 索引(indices) |
| 表（tables）       | types         |
| 行(rows)           | document      |
| 字段(columns)      | fields        |

> elastissearch(集群)中可以包含多个索引（数据库），每个索引中可以包含多个类型，每个类型下又包含多个文档（行），每个文档中包含多个字段（列）；
>
> 物理设计：
>
> elasticsearch在后台把每个索引划分成多个分片，每分分片可以在集群中不同服务器间迁移
>
> 逻辑设计：
>
> 一个索引类型中，包含多个文档，比如说文档1，文档2.当我们索引一篇文档时，可以通过这样的一个顺序找到它：索引>类型>文档id,通过这个组合我们就能索引到某个具体的文档。注意：ID不必是整数，实际上它是个字符串。

##### 文档

之前说elasticsearch是面向文档，那么就意味着索引和搜索数据的最小单位是文档，elasticsrearch中,文档有几个重要属性：

​	自我包含，一篇文档同时包含字段和对应的值，也就是同时包含key:value

​    可以是层次型的，一个文档中包含自文档，复杂的逻辑实体就是这么来的！{就是一个json! fastjson进行自动转换}

​    灵活的结构，文档不依赖预先定义的模式，我们知道关系型数据库中，要提前定义字段才能使用，在elasticsearch中，对于字段是非常灵活的，有时候，我们可以忽略该字段，或者动态添加一个新的字段。

尽管我们可以随意的新增或者忽略某个字段，但是每个字段的类型非常重要，比如一个年龄字段类型，可以是字符串也可以是整形。因为elasticsearch会保存字段之间的映射及其他的设置。这种映射具体到每个映射的每种类型，这个也是为什么在elasticsearch中，类型有时候也称为映射类型

##### 类型

![img](file:///C:/Users/j30007830/AppData/Roaming/eSpace_Desktop/UserData/j30007830/imagefiles/304DE59B-EBC6-48BC-9667-7200543D6FEA.png)

##### 索引

![img](file:///C:/Users/j30007830/AppData/Roaming/eSpace_Desktop/UserData/j30007830/imagefiles/AE467B82-BCD0-4795-A3F1-D523556D64B5.png)

![img](file:///C:/Users/j30007830/AppData/Roaming/eSpace_Desktop/UserData/j30007830/imagefiles/01C91047-F84A-40F0-8F77-AA050C39CC47.png)

##### 倒排索引

![img](file:///C:/Users/j30007830/AppData/Roaming/eSpace_Desktop/UserData/j30007830/imagefiles/58BB05A2-52B8-4E7F-9B6A-C5B3E26DB50E.png)

![img](file:///C:/Users/j30007830/AppData/Roaming/eSpace_Desktop/UserData/j30007830/imagefiles/955354D9-2689-49AA-86D0-630E30A7DB1E.png)

![img](file:///C:/Users/j30007830/AppData/Roaming/eSpace_Desktop/UserData/j30007830/imagefiles/32A2C170-1A20-4609-816E-07ED6D05CA41.png)

![img](file:///C:/Users/j30007830/AppData/Roaming/eSpace_Desktop/UserData/j30007830/imagefiles/956AFD75-BDAC-403E-9373-91E29E2DF207.png)

##### IK分词器

![img](file:///C:/Users/j30007830/AppData/Roaming/eSpace_Desktop/UserData/j30007830/imagefiles/5CC8CB9D-0A84-4D5E-A3AF-389955DAA6F1.png)

> 下载
>
> ```html
> https://github.com/medcl/elasticsearch-analysis-ik
> //添加到plugins目录下
> ```
>
> 可以构建自己的字典
>
>  新增jtf.dic  {蒋廷峰}
>
> 将自己的dic文件添加到配置中
>
> ```xml
> <!-- IK Analyzer 扩展配置 -->
> <!-- 用户可以在这里配置自己的扩展字典 -->
> <entry key="ext_dict">jtf.dic</entry>
> <!-- 用户可以在这里配置远程扩展字典 -->
> <entry key = "ext_stopwords"></entry>
> ```

#### restful

![img](file:///C:/Users/j30007830/AppData/Roaming/eSpace_Desktop/UserData/j30007830/imagefiles/originalImgfiles/6C93C6EC-B5E4-43DB-B385-FE4EFB66E0B5.png)

#### kibanna测试

```json
PUT /test/type1/1
{
    "name": "ceshi",
    "age": 18
}

PUT /test2 
{
    "mappings":{
        "properties": {
            "name": {
                "type": "text"
            },
            "age": {
                "type": "long" 
            },
            "birthday": {
				"type": "date"
            }
        }
    }
}

POST cyx/user/1/_update
{
  "doc": {
    "name": "王五"
  }
}
//简单查询
GET cyx/user/_search?q=name:张三
```



#### java客户端

```java
/*
 * Copyright Notice:
 *  Copyright  2017-2018, Huawei Technologies Co., Ltd.  ALL Rights Reserved.
 *
 *  Warning: This computer software sourcecode is protected by copyright law
 *  and international treaties. Unauthorized reproduction or distribution
 *  of this sourcecode, or any portion of it, may result in severe civil and
 *  criminal penalties, and will be prosecuted to the maximum extent
 *  possible under the law.
 
依赖 
 <dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.6.2</version>
</dependency>

 */
package com.jtf.server.common;

import com.alibaba.fastjson.JSON;
import com.baomidou.mybatisplus.extension.api.R;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexRequest;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.support.master.AcknowledgedResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.action.update.UpdateResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.common.unit.TimeValue;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.index.query.TermQueryBuilder;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

import java.io.IOException;
import java.util.List;
import java.util.concurrent.TimeUnit;

/**
 * @date 2020/10/24
 */
@Configuration
public class ElasticSearchUtil<T> {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    /**
     * @description: 创建索引
     * @date:2020-10-24 17:26:56
     */
    public CreateIndexResponse createIndex(String index) throws IOException {
        CreateIndexRequest createIndexRequest = new CreateIndexRequest(index);
        CreateIndexResponse createIndexResponse = restHighLevelClient.indices().create(createIndexRequest, RequestOptions.DEFAULT);
        return createIndexResponse;
    }

    /**
     * @description: 判断索引是否存在
     * @date:2020-10-24 17:29:43
     */
    public Boolean exist(String index) throws IOException {
        GetIndexRequest getIndexRequest = new GetIndexRequest(index);
        boolean exists = restHighLevelClient.indices().exists(getIndexRequest, RequestOptions.DEFAULT);
        return exists;
    }

    /**
     * @description: 删除索引
     * @date:2020-10-24 17:30:45
     */
    public Boolean delete(String index) throws IOException {
        DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest(index);
        AcknowledgedResponse delete = restHighLevelClient.indices().delete(deleteIndexRequest, RequestOptions.DEFAULT);
        return delete.isAcknowledged();
    }

    /**
     * @description: 文档操作
     * 判断文档是否存在
     * @date:2020-10-24 17:32:15
     * @param index
     * @param id
     */
    public Boolean existDocument(String index, String id) throws IOException {
        GetRequest getIndexRequest = new GetRequest(index, id);
        boolean exists = restHighLevelClient.exists(getIndexRequest, RequestOptions.DEFAULT);
        return exists;
    }

    /**
    * @description: 获取文档
    * @date:2020-10-24 17:35:49
    * */
    public String getDocument(String index,String id) throws IOException {
        GetRequest getRequest = new GetRequest(index, id);
        GetResponse documentFields = restHighLevelClient.get(getRequest, RequestOptions.DEFAULT);
        return documentFields.getSourceAsString();
    }

    /**
    * @description: 更新文档
    * @date:2020-10-24 17:37:33
    * */
    public void updateDocument(String index,String id,T t) throws IOException {
        UpdateRequest updateRequest = new UpdateRequest("zgc_index", "1");
        updateRequest.doc(JSON.toJSONString(t), XContentType.JSON);
        updateRequest.timeout("1s");
        UpdateResponse update = restHighLevelClient.update(updateRequest, RequestOptions.DEFAULT);
        int status = update.status().getStatus();
        if (status != 200) {
            System.out.println("error!");
        }
    }

    /**
    * @description: 删除文档
    * @date:2020-10-24 18:02:23
    * */
    public void delDocument(String index,String id) throws IOException {
        DeleteRequest deleteRequest = new DeleteRequest(index, id);
        deleteRequest.timeout("1s");
        DeleteResponse delete = restHighLevelClient.delete(deleteRequest, RequestOptions.DEFAULT);
        if (delete.status().getStatus() != 200) {
            System.out.println("error!");
        }
    }

    /**
    * @description: 批量查询  可以先进行条件的拼接 拼接条件的时候
    * @date:2020-10-24 18:05:13
     * @param index 索引
     * @param time 查询时间限制大小
    * */
    public Boolean bulkDocument(String index,List<T> t,Long time) throws IOException {
        BulkRequest bulkRequest = new BulkRequest();
        bulkRequest.timeout(TimeValue.timeValueSeconds(time));
        for (int i = 0; i < t.size(); i++) {
            bulkRequest.add(new IndexRequest(index).id("" + (i + 1)).source(JSON.toJSONString(t.get(i)),XContentType.JSON));
        }
        BulkResponse bulk = restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        return !bulk.hasFailures();
    }

    /**
    * @description: 查询  带条件查询
    * @date:2020-10-24 18:23:10
    * */
    public SearchHit[] search(String index,String key,String value) throws IOException {
        SearchRequest searchRequest = new SearchRequest(index);
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
		TermQueryBuilder termQueryBuilder =  QueryBuilders.termQuery(key,value);
		QueryBuilders.boolQuery()
                .should(QueryBuilders.matchQuery("",""))
                .should(QueryBuilders.matchQuery("",""))
                .should(QueryBuilders.matchQuery("",""));
        searchSourceBuilder.query(termQueryBuilder);
        searchSourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));
        searchRequest.source(searchSourceBuilder);
        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
        SearchHit[] hits = search.getHits().getHits();
        return hits;
    }
}
```
<a href="/readme1.html">下一篇--mongdb搭建</a>
