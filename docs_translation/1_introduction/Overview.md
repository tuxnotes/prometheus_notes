# OVERVIEW
### Prometheus是什么
Prometheus是一个开源的监控与告警工具集，最初由SoundCloud公司创建。现今其是一个独立的开源项目，独立于任何公司的维护。为了强调这点，明确项目的管理结构，
于2016年加入了CNCF是继kubernetes之后的第二项目。

## 特性

Prometheus的主要特性如下：
- 基于时间序列的多维度数据模型，数据通过metric name和key/value pairs来标识
- PromQL,一种灵活的查询语言，来获取纬度数据
- 不依赖于分布式存储，每个服务中的节点都是自治的
- 时间序列数据的手机可以通过基于HTTP的pull model
- 通过intermediary gateway支持push模式的数据收集
- 可以通过service discory或静态配置来发现监控目标
- 支持多种模式的图标和dashboard

## 组件
Prometheus生态由多个组件构成，其中一些是可选的：
- Prometheus server用户抓取和存储时序数据
- client libraries，应用代码的测量仪表
- pushing gateway用于支持short-lived jobs
- special-purpose exporters for services like HAProxy, StatsD, Graphite, etc
- alertmanager处理告警
- 各种工具

大部分prometheus组件用go语言编写

## 架构
prometheus架构如下：
![架构图](https://prometheus.io/assets/architecture.png)

Prometheus可直接从instrumented jobs获取metrics，对于short-lived jobs也可以通过an intermediary push gateway。在本地存储所以获取的样本数据，
并在这些数据上进行进行聚合操作，生成新的时序数据，或者是生成告警，Grafana和其他API可以使用这些收集来的数据进行可视化展示。

## 适用场景
Prometheus适用于记录任何纯数字的时间序列数据。它即适用于以机器为中心的数据，也适用于监控动态性较高的service-oriented架构。在微服务的世界里，
它支持多维度的数据收集，查询是其强项。

Prometheus的设计目的是用于可靠性，便于你进行快速的问题诊断。每个prometheus server是独立的，不依赖于网络存储或其他的远程服务。当你基础设施的其他
组件出现问题时，prometheus依然可用，使用它不需要其他额外的基础设施建设。

## 不适用场景
prometheus用于评估可靠性。如果你需要100%的数据准确，如每次请求的费用，这种情况下prometheus并不是一个好的选择，因为prometheus收集的数据不够完全
和详细。这种情况下最好使用其他的收集和分析数据的系统用于billing,剩下的其他监控交给Prometheus.
