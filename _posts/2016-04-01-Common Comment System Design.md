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

### <a id="problems-solution"></a>2.1 问题和思路

采用服务化的思想，考虑以下的问题：
服务的高可用性：
将试着将各个服务进行服务化，使用navi-rpc框架和zookeeper实现服务的高可用性。
各个服务方将自己的服务注册到zookeeper上，基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，集群整体保证高可用性。

服务的效率：
使用多个Redis集群和一致性哈希算法实现缓存的高可用性，将数据全部写入到Redis中去。

服务的能力：
对数据库分库分表，保证数据库足够的容纳能力

### <a id="related-db-table"></a>2.2 相关的DB table

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

### <a id="related-service"></a>2.3 相关服务

#### <a id="related-service-write"></a>2.3.1 数据写入服务

[数据写入示意图]

![gras](/images/postsImages/1-评论写入流程.png)

2.3.1.1 数据写入MQ 中

由内容控制服务对数据进行订阅处理。

2.3.1.2  数据过滤服务

由内容控制服务调用判断数据的合法性，验证数据是否有安全性的问题，验证数据是否有各种XSS，DB攻击风险和是否为违禁词。

2.3.1.3 Redis 服务

调用id生成的服务，和一致性哈希服务，提供数据的查询和写入服务。

2.3.1.4 唯一性ID生成服务

可以使用redis自增，也可以使用DB自增。

2.3.1.5 一致性哈希算法服务

将Redis机器的信息和数据信息一起哈希，我们将热门信息散列到临近的机器上，调用该服务时，当一台机器挂了，将选择放到临近的下一台机器。

2.3.1.6 数据重建

选择从Redis中对MySQL的数据进行重建。在数据写入到Redis中的时候，同时写入到MQ中去，数据重建服务对MQ中的数据进行订阅，在MySQL中的数据进行重建。

2.3.1.7 MySQL分库分表

对于用户表user，按照user_id 分库，8个数据库，每个库1个表，每个表上限2000W的数据，大约可以存储1.6亿的数据量。

对于评论关系表user_comment表，按照回复者的维度分表，暂时使用8个数据库，每个库8个表，每个表上限2000W的数据，大约可以存储12.8亿的数据量。

对于评论回复关系表user_reply表，按照被回复者的维度进行分表，8个数据库，每个库1个表，每个表上限2000W的数据，大约可以存储1.6亿的数据量。
 
对于评论表comment，按照article_id分表，每个库8张表，总共64张表。每个表上限2000W的数据，大约可以存储12.8亿的数据量，将article_id 和user_id存储到其中。

#### <a id="data-read-service"></a>2.3.2 数据读取服务

2.3.2.1读取文章数据服务

实现分页

在Redis 中以zset的形式存储文章的id，以下为add命令: 

zadd article:id `timestamp` `commentId`

获取分页数据: ZREVRANGE article:id 0 19 获取前20条数据

根据获取的数据的id集合，从Redis 中 batch 获取到文章评论的内容。

支持直线式的回复(微博，人人，知乎)

在article:id zset 中，顺序的存储所有的回复的id，获取数据时，按照存入的数据timestamp排序取出来。

添加数据: 

| |key|data|
|:-------|:-------|:-------|
|数据zset|article:id|`timestamp` `commentId`|

读取数据:

| |comment|
|:-------|:-------|
|数据zset|ZREVRANGE article:id 0 19|

支持缩进式回复

在article:id zset中，仅仅存储一级评论id 相关的数据，将其他的评论数据以一级评论id形成主键，存储其回复数据id的信息。

添加评论: 

| |key|data|
|:-------|:-------|:-------|
|一级回复数据zset|article:id|`timestamp` `commentId`|
|非一级回复数据zset|article:id:parentCommetId|`timestamp` `commentId`|
|非一级回复数据zset|article:id:layer:commentId|commentLayer(评论层级)|

读取数据:

| |comment|
|:-------|:-------|
|一级回复数据zset|ZREVRANGE article:id 0 19|
|非一级回复数据zset|zrange article:id:parentCommetId 0 19|
|非一级回复数据zset|get article:id:layer:commentId|

支嵌套式的评论(网易)

添加评论: 

| |key|data|
|:-------|:-------|:-------|
|一级回复数据zset|article:id|`timestamp` `commentId`|
|一级回复数据zset|article:id:upId:commentId|0 (其回复的评论id)|
|非一级回复数据zset|article:id|`timestamp` `commentId`|
|非一级回复数据zset|article:id:upId:commentId|id(其回复的评论id)|

读取数据:

| |comment|
|:-------|:-------|
|一级回复数据zset|ZREVRANGE article:id 0 19|
|非一级回复数据zset|ZREVRANGE article:id 0 19|
|非一级回复数据zset|get article:id:upId:commentId直至其upId 为0| 

以上显示支持用户配置策略

2.3.2.2 个人中心数据读取

我的评论数据以每个user为维度，记录其回复的数据的 commentId

| |Key|
|:-------|:-------|
|我的回复|zset	comment:userId|

分别显示我的回复和别人对我的回复

数据写入

| |key|data|
|:-------|:-------|:-------|
|回复数据zset|comment:userId|`timestamp` `commentId`|

数据读取

| |Comment|
|:-------|:-------|
|我的回复 zset|ZREVRANGE comment:userId 0 19|

根据获取到的id list，batch获取到相应的comment

支持在我的回复中显示被回复的信息

数据写入

| |key|data|
|:-------|:-------|:-------|
|回复数据zset|comment:userId|`timestamp` `commentId`|
|回复我的数据|zset:comment:userId:commentId|`timestamp` `commentId`|
|回复我的数据|comment:userId:comtId:unread|未读id list|

回复我的数据zset包含我在我的评论下面对别人的回复

数据读取

| |Comment|
|:-------|:-------|
|我的回复 zset|ZREVRANGE comment:userId 0 19|
|回复我的数据zset|ZREVRANGE comment:userId:commentId 0 19|

当数据读取出来后，立即将读取出id从未读list删除掉。

2.3.2.3 个人中心未读数据的提醒

使用Comet框架，记录用户未读数据的数量unread:userId，未读数量有变化时，对用户进行提醒。

2.3.2.4 MySQL数据读取

如何获取个人页面的comment

由于user是分库存储的，那么拿到一个user的时候，根据其user_id 在user_comment表获取其所有的comment_id 和article_id信息。

获取个人被回复的数据

根据user_reply表，拿到reply的信息。可以按照用户的不同的需求和在Redis中类似的拼接出来不同的回复表现形式。

如何获取文章的comment

由于按照article维度分表，根据article_id信息，可以在一张表中拿到所有的回复数据。可以按照用户的不同的需求和在Redis中类似的拼接出来不同的回复表现形式。

2.3.2.5 规则配置逻辑

为了实现上述的各种不同的规则配置，其逻辑如下图所示

![gras](/images/postsImages/2-评论规则配置逻辑.png)

2.3.2.6用户特征数据库

可以记录数据的登入信息，30天内有几次登录，或者复杂一点，记录各种行为，为各种行为进行评分。

### <a id="serch-service"></a>2.3.3 搜索服务

为了满足APP管理员的管理Comment的需求，或者可能的APP用户的搜索需求，我们在将数据写入到MySQL的时候，使用CDP->Change-dumper->检索端的逻辑。

## <a id="reference"></a>3 参考资料

[1] 通用评论系统_潘静_0715

[2] [支撑5亿用户、1.5亿活跃用户的Twitter最新架构详解及相关实现](http://blog.csdn.net/wyxhd2008/article/details/24870933)

[3] [构建一个类timeline系统的架构设计](http://www.tuicool.com/articles/iymuamv)

[4] [浅析评论系统设计](http://ratwu.com/2011/11/comment/)

[5] 百度口碑的设计

[6] [分布式 数据库表 sharding 综述](http://www.liaoqiqi.com/post/250)















