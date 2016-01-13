---
title: 'ElasticSearch初体验:index的CRUD'
date: 2016-01-13 15:41:57
tags: [ElasticSearch, NoSQL]
categories: ElasticSearch
description: ElasticSearch初体验:index的CRUD
---

最近要使用ElasticSearch, 记录下学习的东西.

## 安装
首先要先安装, 要求JDK7以上, 直接获取安装包

```bash
// 下在安装包
$ curl -L -O https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.4.tar.gz
// 解压
$ tar xvf elasticsearch-1.7.4.tar.gz
$ cd elasticsearch-1.7.4/bin
// 启动, 集群名为 es-test
$ ./elasticsearch --cluster.name es-test
```
<!-- more -->

可以通过http请求的rest API来对集群进行操作, 主要可以做以下操作

  * 检查cluster, node, index的健康度, 状态, 统计信息
  * 管理cluster, node, index的数据和元数据
  * CRUD操作
  * 执行高级搜索操作,如分页,排序,过滤等等.

## 健康检查:

```bash
$ curl 'localhost:9200/_cat/health?v'

// 返回
epoch      timestamp cluster status node.total node.data shards pri relo init unassign pending_tasks
1452669619 15:20:19  es-test green           1         1      0   0    0    0        0             0
```
其中status表示集群的状态, 有3种状态, green表示一切正常, yellow表示数据是可用的, 但有的replica分片没有创建, red表示有部分数据不可用.


## 查询node:

```bash
$ curl 'localhost:9200/_cat/nodes?v'

// 返回
host                  ip          heap.percent ram.percent load node.role master name
localhost.localdomain 127.0.0.1              5          73 0.37 d         *      Whirlwind
```

## 查询index

```bash
$ curl 'localhost:9200/_cat/indices?v'

// 返回
health status index pri rep docs.count docs.deleted store.size pri.store.size
```

## 创建index

创建名为customer的index

```bash
$ curl -XPUT 'localhost:9200/customer?pretty'

// 返回
{
  "acknowledged" : true
}
```

创建完之后再查询

```bash
$ curl 'localhost:9200/_cat/indices?v'

// 返回
health status index    pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer   5   1          0            0       575b           575b
```

默认创建5个primary分片, 1个replica分片,0个documents.

## 创建document

在customer的index上创建type为external, id为1的document

```bash
$ curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
> {
>   "name": "John Doe"
> }'

// 返回
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "created" : true
}
```

## 查询document

查询创建的document

```bash
$ curl -XGET 'localhost:9200/customer/external/1?pretty'

// 返回
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source":
{
  "name": "John Doe"
}
}
```

## 删除index

```bash
$ curl -XDELETE 'localhost:9200/customer?pretty'

// 返回
{
  "acknowledged" : true
}
```

删除之后再查询

```bash
$ curl 'localhost:9200/_cat/indices?v'

// 返回
health status index pri rep docs.count docs.deleted store.size pri.store.size
```




