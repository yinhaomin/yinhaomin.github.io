---
layout: post
title: Search System Design by ES and Redis
comments: true
keywords: 检索系统,设计,Search System Design by ES and Redis
---

需求
1. 在繁多的信息中，找到真正需要的信息
2. 支持花样查询
3. 速度要快
4. 支持大量的查询 

解决方案
在使用like时，一般都用不到索引，除非使用前缀匹配，才能用得上索引。但普通的需求并非前缀匹配。
比如like ‘%化痰冲剂%’就不能把”化痰止咳冲剂“搜索出来。但是普通的用户，需求就是这样
数据库匹配某个关键字的记录可能有好几千，但数据库往往返回用户一些不关心的记录

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
