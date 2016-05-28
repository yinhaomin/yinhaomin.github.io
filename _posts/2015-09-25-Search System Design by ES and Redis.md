---
layout: post
title: Search System Design by ES and Redis
comments: true
author: "Yin Haomin"
keywords: 检索系统,设计,Search System Design by ES and Redis
---

## 1. 需求

1. 在繁多的信息中，找到真正需要的信息

2. 支持花样查询

3. 速度要快

4. 支持大量的查询 

## 2. 解决方案
在使用like时，一般都用不到索引，除非使用前缀匹配，才能用得上索引。但普通的需求并非前缀匹配。
比如like ‘%化痰冲剂%’就不能把”化痰止咳冲剂“搜索出来。但是普通的用户，需求就是这样
数据库匹配某个关键字的记录可能有好几千，但数据库往往返回用户一些不关心的记录

|需求|Mysql|ES/Redis/Rank server|
|:-------|:-------|:-------|
|	精确	|	like的不能做到完全的模糊匹配	|	ElasticSearch可以使用ik实现中文分词	|
|	排序	|	like无法根据匹配度进行排序	|	ElasticSearch可以被干预使用一些方式排序，也可以使用专门的Rank server排序	|
|	速度	|	使用like搜索效率太低，一般用不到索引	|	使用ElasticSearch 实现检索，高性能的key-value数据库实现数据的读取	|
|	抗压	|	其读的抗压参数，没有去查。但是速度很明显不一样，因为Mysql基于文件系统，而Redis在内存中存储数据。	|	ElasticSearch 实现检索，高性能的key-value数据库Redis实现数据的读取,实现了数据的读写分离	|

### 2.1 整体的架构

![gras](/images/postsImages/3-检索系统设计.png)

### 2.2 详细解决方案

ElasticSearch
是一个基于Lucene的搜索服务器
可以人工干预使用其他方式排序

Elasticsearch搭建一个接近实时的搜索服务
Index实现索引信息的新建和更新
Update
Delete
Get (Domain Specific Language)
Portal演示
ElasticSearch Java API
https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/index.html 

数据的写入
根据输入的参数build出来请求，并发请求得到数据
核心代码

```
  Client client = esFactory.client();
        SearchRequestBuilder builder = client.prepareSearch(config.getItemIndice())
                .setTypes(config.getItemType())
                .setSearchType(SearchType.DFS_QUERY_THEN_FETCH);
        QueryBuilder queryBuilder = condition.query();
        if (queryBuilder != null) {
);
        } builder.setQuery(queryBuilder
        FilterBuilder filterBuilder = condition.filter();
        if (filterBuilder != null) {
            builder.setPostFilter(filterBuilder);
        }
```

根据输入的参数build出来请求，并发请求得到数据

```
  builder.setFrom((page.getPageNo() - 1) * page.getPageSize())
                .setSize(page.getPageSize());
        
        if (StringUtils.isNotEmpty(page.getOrderBy())
                && ())) {
            builder.addSort(page.getOrderBy(), StringUtils.isNotEmpty(page.getOrder
                    SortOrder.valueOf(page.getOrder()));
        } else {
            builder.addSort("updatetime", SortOrder.DESC);
        }
```

根据输入的参数build出来请求，并发请求得到数据

```
 SearchResponse response = builder.addField(RankConstants.FIELDS.ID)
                .addField(RankConstants.FIELDS.UPDATETIME)
                .addField(RankConstants.FIELDS.PUBLISH_TIME)
                .setExplain(true).execute().actionGet();
```

Rank server-提供数据的简单Rank功能

```
public int compareToBig(ItemRankBO o) {
        if (rank > o.rank) {
            return 1;
        }
        if (rank < o.rank) {
            return -1;
        }
        if (publishtime > o.publishtime) {
            return 1;
        }
        if (publishtime < o.publishtime) {
            return -1;
        }
        return 0;
    }
```
