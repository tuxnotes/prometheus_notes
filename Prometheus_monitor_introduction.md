# Prometheus Monitor

# 学习目标

- 能安装prometheus server
- 能通过安装node_exporter监控远程Linux
- 能通过安装mysqld_exporter监控远程MySQL数据库
- 能安装grafana
- 能在grafana添加prometheus数据源
- 能在grafana添加监控CPU负载的图形
- 能在grafana图形显示MySQL监控数据
- 能通过grafana+onealert实现报警

# 任务背景

某公司是一家电商网站，由于公司的业务快速发展，公司要求对现有机器进行业务监控，责成运维部门来实现这个项目。

# 任务要求

1. 部署监控服务器，实现7X24实时监控
2. 针对公司的业务及研发部门设计监控系统，对监控像和触发器拿出合理意见
3. 做好问题预警机制，对可能出现的问题要及时告警并形成严格的处理机制
4. 做好监控告警系统，要求可以实现告警分级
   - 一级报警   电话通知 onealert支持电话
   - 二级报警  微信通知
   - 三级报警  邮件通知
5. 处理号公司服务器异地集中监控问题

K8S内部使用的监控系统就是prometheus

# 任务分析

为什么要监控？

实时收集数据，通过报警及时发现问题，及时处理。数据为优化也可以提供依据。

监控四要素：

- 监控对象		[主机状态 服务 资源 页面 url]
- 用什么监控            [zabbix-server zabbix-agent]
- 什么时间监控        [7X24   5X8]
- 报警给谁                [管理员]

项目选型：

- mrtg (Multi Router Traffic Grapher)通过SNMP协议得到设备的流量信息，并以包含PNG格式的图形的HTML方式显示给用户
- cacti 用PHP语言实现的一个软件，它的主要功能是用是你们跑服务获取数据，然后用rrdtool存储和更新数据
- ntop https://www.ntop.org
- nagios 能够跨平台，插件多，报警功能强大
- centreon 底层使用的nagios,是一个nagios整合版软件
- ganglia 设计用于测量数以千计的节点，资源消耗非常小
- open-falcon 小米发布的运维监控软件，高效率，高可用。时间较短，用户基数小
- **zabbix** 跨平台，画图，多条件告警，多种API接口，使用技术特别大
- **prometheus** 基于时间序列的数值数据的容器监控解决方案

综合分析：prometheus比较适合公司的监控需求

# 一、Prometheus概述

Prometheus是一套开源的监控&报警&时间序列数据库的组合。适合监控docker容器。因为k8s的流行带动了prometheus的发展。

# 二、时间序列数据

## 1、什么是序列数据

**时间序列数据**(TimeSeries Data): 按时间顺序记录系统、设备状态变化的数据被称为时序数据。

应用场景很多，如：

- 无人驾驶车辆运行中要记录的经度，纬度，速度，方向，旁边物体的距离等。每时每刻都要将数据记录下来做分析。
- 某个地区的个车辆的形式轨迹数据
- 传统证券行业实时交易数据
- 实时运维监控数据等

## 2、时间序列数据特点

- 性能好

  关系型数据库对于大规模数据的处理性能糟糕。 nosql可以比较号的处理大规模数据，但依然比不上时间序列数据库

- 存储成本低

  高效的压缩算法，节省存储空间，有效降低IO。Prometheus有着非常高效的时间序列数据存储方法，每个采样数据仅占用3.5byte左右空间。上百万条时间序列，30秒间隔，保留60天，大概花了200G(官方数据)

## 3、Prometheus主要特征

- 多维度数据模型
- 灵活的查询语言
- 不依赖分布式存储，单个服务器节点是自主的
- 以HTTP方式，通过pull模型拉取时间序列数据
- 也可以通过中间网关支持push模型
- 通过服务发现或静态配置，来发现目标服务对象
- 支持多种多样的图标和界面展示，一般要结合grafana

## 4、Prometheus原理架构图

![](./prometheus_architecture.png)

# 三、实验环境准备

grafana服务器： 10.1.1.15 运维成像=>数据转换为图形

prometheus服务器：10.1.1.13

被监控服务器：10.1.1.14 如LB，web, mysql等

1.静态ip(要求能上外网)

2.主机名：各自配置好主机名

3.时间同步

4.关闭防火墙，selinux

## 1、安装prometheus

从官网下载相应版本安装到服务器上，官网提供的是二进制版，解压就能用不需要编译

直接使用默认配置文件启动：

```bash
# /use/local/prometheus/prometheus --config.file="/usr/local/prometheus/prometheus.yml" &
```

默认端口：9090

```bash
# lsof -i:9090
```

## 2、prometheus界面

通过浏览器访问http://ip:9090就可以访问到prometheus的主界面



