# First steps

Prometheus是一个监控平台，从被监控对象的HTTP endpoints中获取metrics。这里将带领你进行安装，配置prometheus，以及使用prometheus进行我们的第一个
资源监控。下载并安装运行prometheus，你也可以下载安装exporter, exporter是一个工具，用来暴露主机和服务的时间序列数据。我们的第一个exporter是
prometheus自己，它提供了主机层面的metrics，包括内存，垃圾回收等。

## 下载prometheus

下载二进制tar包，解压，进入解压目录，执行二进制文件即可运行
```bash
# tar zxf prometheus-*.tar.gz # * 表示版本
# cd prometheus-*
# ./prometheus --help
```

在运行prometheus前，先设置一下

## 配置prometheus

prometheus的配置文件是一个YAML文件，下载的prometheus包含一个配置文件样本，叫`prometheus.yml`.

去掉配置示例文件中的注释内容，得到下面的内容
```yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```
配置文件由三部分组成：`global`, `rule_files`, `scape_configs`.
- global 部分控制prometheus服务的全局设置。当前我们有两个选项，第一个是`scape_interval`，控制prometheus获取目标数据的频率。可以改变这个设置。
这里设置为每隔i5秒抓取一次。第二设置项是`evaluation_interval`，控制多久prometheus进行一次 evalute rules。prometheus使用rules创建新的时间
序列数据，并生成告警。

- rule_files部分指定了prometheus加载rules文件的位置。我们这里并没有设置rules

- 第三个部分是`scape_configs`，控制prometheus监控什么资源。由于prometheus通过一个HTTP endpoint暴露自己的数据，所以它也监控自身的健康状况。
默认情况下只有一个job称为`prometheus`用于抓取prometheus server暴露的数据。这个job包含了一个单独的静态配置，在localhost的9090端口。默认job
通过URL: http://localhost:9090/metrics获取数据。

## 启动prometheus

```bash
# ./prometheus --config.file=prometheus.yml
```
## Using the expression browser

来看看prometheus收集的关于自身的一些数据。为了使用prometheus内置的`expression browser`, 打开http://localhost:9090/graph选择"Console"中
的"Graph"。如同你可以通过http://localhost:9090/metrics获取一样的数据，prometheus暴露自己的metric称为
`promhttp_metric_handler_requests_total`(对/metrics请求的总量),在表达式输入框中输入`promhttp_metric_handler_requests_total`。
这会返回一些时序数据，metric name均为promhttp_metric_handler_requests_total，但是标签不同，标签表明了不同的请求状态。如果值关心状态码是200的
请求，可以改写表达式为：`promhttp_metric_handler_requests_total{code="200"}`.如果要统计返回的数量可以写为：
`count(promhttp_metric_handler_requests_total)`
详情参考expression language document

## Using the graphing interface

找到http://localhost:9090/graph使用"Graph"面板，下面的表达式将返回每秒HTTP请求率，且状态码为200：
`rate(promhttp_metric_handler_requests_total{code="200"}[1m])`

## 监控其他目标

Monitoring Linux or macOS host using a node exporter guide is a good place to start.
