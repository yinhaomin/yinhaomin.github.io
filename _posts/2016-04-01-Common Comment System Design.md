---
layout: post
title: Common Comment System Design
comments: true
author: "Yin Haomin"
keywords: Comment,评论系统,系统设计
---

占坑，通用的comment服务，为了解决大规模并发和大规模数据量的评论的写入和读取，用户的读取服务。
能够支持多个APP的读取和写入，以及数据的管理。

## <a id="design-aim"></a>1 设计的目标

### <a id="design-aim-general"></a>1.1 概述

通用的comment服务，为了解决大规模并发和大规模数据量的评论的写入和读取，用户的读取服务。能够支持多个APP的读取和写入，以及数据的管理。

### <a id="design-aim-core"></a>1.2 核心业务

#### <a id="design-aim-core-efficiency"></a>1.2.1 业务的效率要求

|效率要求|说明|
|:-------|:-------|
|数据写入|支持大并发的写入，峰值500条/s的数据写入。一日最大支持约4千万条数据写入请求。|
|数据读取|支持大并发的读取，峰值2000qps，接口能够在500ms内返回。写入数据后，处理并返回展现在页面上，能够在500ms内完成。一日支持最大约1.7亿次数据的读取请求。|


#### <a id="design-aim-core-basic"></a>1.2.2 业务的基本功能要求

|业务要求|说明|
|:-------|:-------|
|文章的回复|支持回复文章，回复别人的回复，@功能。能够支持不同类型的Article|
|文章数据展示|能够在文章详情页支持不同的展示的需求：支持直线式(微博，人人，知乎)，缩进式(QQ， Facebook)，嵌套式的评论(网易).|
|个人中心|能够展示个人中心我的回复，别人对我的回复。能够对别人对我的回复及时提醒|
|后台管理|管理封禁词，管理被举报的评论|

## <a id="design-thoughts"></a>2 设计思路

### <a id="related-service"></a>2.1 问题和思路

采用服务化的思想，考虑以下的问题：
服务的高可用性：
将试着将各个服务进行服务化，使用navi-rpc框架和zookeeper实现服务的高可用性。
各个服务方将自己的服务注册到zookeeper上，基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，集群整体保证高可用性。

服务的效率：
使用多个Redis集群和一致性哈希算法实现缓存的高可用性，将数据全部写入到Redis中去。

服务的能力：
对数据库分库分表，保证数据库足够的容纳能力

#### <a id="related-db-table"></a>2.2 相关的DB table

comment表，记录所有的评论和评论的回复信息

|Name|Comment|
|:-------|:-------|
|id |PK|
|appid|使用方系统分发id|
|user_id|用户Id|
|user_name|用户名|
|portrait|头像|
|user_ip|用户IP|
|article_id|文章ID|
|article_type|文章类型|
|layer|楼层id|
|level|星级|
|parent_id|父评论id|
|content|内容|
|ctime|用户创建时间|
|tags|标签|
|deleted|是否被删除|
|comment_type|评论的类型1-comment,2-reply, 3-@|
|target_user_id|被评论的用户的id|
|target_user_name|被评论的用户的name|
|target_type|被评论的类型|
|target_id|被评论的id|
|reserved1|保留字段|
|reserved2|保留字段|
|reserved3|保留字段|
|addtime|添加时间|
|upddatetime|更新时间|

user表，记录user的信息

|Name|Comment|
|:-------|:-------|
|id|PK|
|appid|使用方系统分发id|
|user_id|用户Id|
|user_name|用户名|
|portrait|头像|
|mobile_no|手机号|
|email|邮箱|
|last_login_time|最后一次登录时间|
|user_status|用户状态|
|frozen_time|冻结时间|
|deleted|是否删除|
|addtime|新增时间|
|updatetime|更新时间|

user_comment表，记录用户和回复的映射

|Name|Comment|
|:-------|:-------|
|id|PK|
|appid|使用方系统分发id|
|user_id|用户Id|
|article_id|文章的id|
|replier_id|回复者的id|
|comment_id|相关的comment的id|
|addtime|新增时间|
|updatetime|更新时间|

user_reply表，记录回复者，comment的reply与user的关系

|Name|Comment|
|:-------|:-------|
|id|PK|
|appid|使用方系统分发id|
|user_id|用户Id|
|article_id|文章的id|
|replier_id|回复者的id|
|addtime|新增时间|
|updatetime|更新时间|

app_info表记录各个产品线的信息

|Name|Comment|
|:-------|:-------|
|id|PK|
|name|app的名称|
|company|app所属公司|
|address|地址|
|type|类型|
|description|详细的描述|
|contact|联系人名|
|phone|电话|
|addtime|新增时间|
|updatetime|更新时间|


filter_word表，记录过滤词

|Name|Comment|
|:-------|:-------|
|id|PK|
|app_id|app的id|
|word|封禁的词|
|word_type|词的类型|
|filter_reason|封禁原因|
|filter_time|封禁的时间|
|addtime|新增时间|
|updatetime|更新时间|

#### <a id="related-service"></a>2.4.1 数据写入服务

[数据写入示意图]

![gras](/images/postsImages/1-评论写入流程.png)

2.4.1.1 数据过滤服务

判断数据的合法性，验证数据是否有安全性的问题，验证数据是否有各种XSS，DB攻击风险和是否为违禁词。

2.4.1.2 Redis 服务

调用id生成的服务，和一致性哈希服务，提供数据的查询和写入服务。

2.4.1.3 唯一性ID生成服务

可以使用redis自增，也可以使用DB自增。

2.4.1.4 一致性哈希算法服务

将Redis机器的信息和数据信息一起哈希，我们将热门信息散列到临近的机器上，调用该服务时，当一台机器挂了，将选择放到临近的下一台机器。

2.4.1.5 MySQL分库分表

对于用户表user，按照user_id 分库，8个数据库，每个库1个表，每个表上限2000W的数据，大约可以存储1.6亿的数据量。

对于评论关系表user_comment表，按照回复者的维度分表，暂时使用8个数据库，每个库8个表，每个表上限2000W的数据，大约可以存储12.8亿的数据量。

对于评论回复关系表user_reply表，按照被回复者的维度进行分表，8个数据库，每个库1个表，每个表上限2000W的数据，大约可以存储1.6亿的数据量。
 
对于评论表comment，按照article_id分表，每个库8张表，总共64张表。每个表上限2000W的数据，大约可以存储12.8亿的数据量，将article_id 和user_id存储到其中。
