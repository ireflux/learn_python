---
title: "Elasticsearch 入门 | 一" 
date: 2020-03-23
lastmod: 2020-03-23
draft: false
categories: ["Clavicula Salomonis"]
tags: ["Elasticsearch"]
author: "sherry"
---
本文是我开始学习 Elasticsearch 系列的第一篇笔记，计划先以我初学者的状态对 Elasticsearch 的看法以及目前学到的知识做个概览性的总结，之后的系列再单独对其细节专门去学习和记录，留备后期翻阅查看。

注：截至本文发表前，Elasticsearch 的版本号为 7.6.1，本文的内容也建立于此之上。

## Elasticsearch 是什么？

根据官方文档和 Wikipedia 的说法：

Elasticsearch是一个基于Apache Lucene构建的分布式的开源搜索和分析引擎，可以处理所有类型的数据，包括文本，数字，地理空间，结构化和非结构化的数据。

<!--more-->

## Elasticsearch 有什么用？

- 应用程序搜索
- 网站搜索
- 企业搜索
- 记录和日志分析
- 基础架构指标和容器监控
- 应用程序性能监控
- 地理位置分析和可视化
- 安全分析
- 商业分析

了解到此就可以了，想要更深入的了解可以去翻阅[官方文档](https://www.elastic.co/what-is/elasticsearch)

## Elasticsearch 的配置

在此跳过了 ES 的安装，是因为在我的安装体验里，并没有遇到什么阻力，因此无需记录。关于下载安装，可以查阅[此页面](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html)

ES 的配置主要是修改 `elasticsearch.yml`，打开这个文件，文件中已经列出了大多数重要的设置，划分了几个区域，分别是 Cluster，Node，Paths，Memory，Network，Discorey，Gateway，Various

### 关于路径的设置

在生产中一定要设置其日志和数据的路径。如果使用默认路径，在升级 ES 的时候很可能会被删除掉。

### 关于集群名称的设置

不要在不同的环境中使用相同的名称，否则新节点可能会加入错误的集群中。

### 关于网络的配置

默认情况下，Elasticsearch 绑定的是环回地址，例如：127.0.0.1，[::1]，这在单台服务器下是没问题的。但如果要和其他服务器上的 ES 组成集群，需修改 `network.host`。例如：

```
network.host: x.x.x.x
```

注：一旦为 `network.host` 设置了自定义配置，ES 就会认为你从开发模式转移到了生产模式，会自动将系统启动检查从警告升级为异常。见 [Development mode vs production mode](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html#dev-vs-prod)

### 关于 Discovery 的配置

为使集群中各个 ES 之间可见，应设置 `discovery.seed_hosts`：

以下引用官网的例子：

```
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 
   - seeds.mydomain.com 
   - [0:0:0:0:0:ffff:c0a8:10c]:9301 
```

注: 如果不加端口号，则默认是9300。

当首次启动 ES 集群时，会有一个引导步骤，它确定了首次选举中来对其票数进行计数的有主要资格节点的集合。这个“计数权”在开发模式中是由节点自动确定的，但却是不安全的。若在生产模式中，应当指定 `cluster.initial_master_nodes`，具体配置可详见[文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-settings.html#initial_master_nodes)

## ES 的启动与关闭

一般而言，ES 可使用 `./bin/elasticsearch` 来启动，但我们往往需要它作为一个守护进程来运行。除了使用 `nohup` 之外，ES 本身有自带的方法：`./bin/elasticsearch -d -p pid`

注：-p 选项将进程记录到文件中，因此此处的 pid 应为文件路径。

一旦启动后，可以通过 `curl localhost:9200` 来获取 ES 的相关信息。

若开启了集群，可以通过 `curl -X GET "localhost:9200/_cat/health?v&pretty"` 来查看节点信息。可以看到，多节点副本可用的情况下 status 为 green，若只有单节点的情况下执行，则 status 为 yellow，但也是正常的，只是没有副本而已。

关闭时，只需 `pkill -F pid` 即可。或者使用更通用的方法 `ps -ef | grep Elasticsearch` 或者 `jps | grep Elasticsearch` 来获取进程号。

## 参考资料

[Elasticsearch Wikipedia](https://en.wikipedia.org/wiki/Elasticsearch)   
[Elasticsearch](https://www.elastic.co/)
