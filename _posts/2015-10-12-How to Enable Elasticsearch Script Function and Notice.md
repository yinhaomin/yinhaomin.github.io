---
layout: post
title: How to Enable Elasticsearch Script Function and Notice
comments: true
keywords: Elasticsearch,脚本功能开启,注意事项
---
Elasticsearch支持更新可以通过使用简单的脚本来进行。但是实际操作中会遇到问题。org.elasticsearch.script.ScriptException: scripts of type [inline], operation [update] and lang [groovy] are disabled

这个例子使用一个脚本将age加5：
```
	curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
        {
          "script" : "ctx._source.age += 5"
        }'
```

es在更改其配置后不生效，一直以为自己配置错了，查了很多资料，发现没有配置错误，最后ps -ef，发现存在很多的elasticsearch的进程，kill掉所有以后，启动就生效了。
多次启动es，不报错，sh elasticsearch stop命令并不能将其关闭

以下是解决脚本功能未打开的配置

```
	script.engine.groovy.file.aggs: on
	script.engine.groovy.file.mapping: on
	script.engine.groovy.file.search: on
	script.engine.groovy.file.update: on
	script.engine.groovy.file.plugin: on
	script.engine.groovy.indexed.aggs: onecetest
	script.engine.groovy.indexed.mapping: on
	script.engine.groovy.indexed.search: on
	script.engine.groovy.indexed.update: on
	script.engine.groovy.indexed.plugin: on
	script.engine.groovy.inline.aggs: on
	script.engine.groovy.inline.mapping: on
	script.engine.groovy.inline.search: on
	script.engine.groovy.inline.update: on
	script.engine.groovy.inline.plugin: on
```
