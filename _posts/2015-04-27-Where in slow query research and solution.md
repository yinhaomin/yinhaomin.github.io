---
layout: post
title: Where in slow query research and solution
comments: true
author: "Yin Haomin"
tags:
    - SQL query
    - Where in query
---

Queries that use WHERE ID IN (subquery) perform notoriously badly with mysql.
This 'WHERE ID IN (subquery)' query takes extremely long, around 60 seconds. The database is not very busy otherwise, and performs well on other queries.

Here is the reason and solution:
http://www.cnblogs.com/xh831213/archive/2012/05/09/2491272.html


Solution: change the where in query

```
	select a.id,',', a.name ,',',b.weigou_category_id
	from category a
	inner join category_map b
	on a.id = b.bdh_category_id
	where b.weigou_category_id in(
	select c.weigou_category_id from category_map c group by c.weigou_category_id having count(*) >1
	);
```
to be:

```
	select a.id,',', a.name ,',',b.weigou_category_id
	from category a
	inner join category_map b
	on a.id = b.bdh_category_id
	where b.weigou_category_id in(
	select tb.weigou_category_id from(
	select c.weigou_category_id from category_map c group by c.weigou_category_id having count(*) >1) as tb
	);
```
