# prometheus进阶学习笔记

## 指标采集方式

指标采集方式无论那种，都需要保证prometheus到被监控对象的网络是通的。

### pull

监控系统周期性得对被监控对象发起采集请求，被监控对象将他想上报的指标的当前值按约定的格式返回给监控系统。

### push 

被监控对象周期性得将指标数据推送到监控系统。

## 指标类型

prometheus一共提供了4种数据类型

### Counter

Counter就是⼀个⾮负递增值，只增不减，适⽤于例如总请求数这类指标，我们在查询的时候，可以通过查询语句，在Counter的数据上计算出增⻓率等

### Gauge

Gauge没啥限制，可增可减，适⽤于例如内存使⽤量这种指标。

### Histogram

Histogram是桶统计类型，它会⾃动将你定时的指标变成3个指标，例如你指定⼀个Histogram叫a，那就会⽣成

* a_bucket{le="某个桶值，例如100"}⽤于统计打点值低于100的次数。
* a_sum表示打点值的和
* a_count表示打点次数

### Summary

Summary是百分位统计类型，会在客户端对于⼀段时间内（默认是10分钟）的每个采样点进⾏统计，并形成分位图。 它也会⽣成3个指标，例如你指定⼀个Summary叫a，那就会⽣成

* a{quantile="0.99"} p99百分位数，这个0.99是定义指标时指定的，可以指定好⼏个。
* a_sum表示打点值的和
* a_count表示打点次数

## 采集配置

全局配置

```
global: # 全局配置，对所有采集⽣效
 scrape_interval: 15s # 默认采集间隔
 external_labels: # 这些label会被加到所有采集到的指标上
 cluster: a
 replica: b
scrape_configs: # 最重要，所有采集配置都在这⾥了
- job_name: "job1" # 采集任务，下⾯介绍
 ....
- job_name: "job2"
 ....
```

单个job的采集配置

```
scrape_configs:
- job_name: "job1"
 kubernetes_sd_configs: # 1. 服务发现，有哪些对象（Target)需要监控，这⾥以K8s为例⼦
 ...
 relabel_configs: # 2. 被发现的对象会⾃带⼀些label，这⼀步可以修改添加label，也可以根据label过滤
掉采集对象。
 ...
 metrics_path: /metrics # 3. 定义发起采集请求的⽅式，例如path
 scheme: https # 3. 定义发起采集请求的协议，例如https
 metric_relabel_configs: # 4. 针对采集过来的指标，进⾏lable的添加修改，也可以根据label过滤掉⼀些
指标
 ...
```

pod的所有k8s label会被转换成前缀是 __meta_kubernetes_pod_label_ 的label。 由于Prometheus的k8s label的key不能包含⼀些特殊字符⽐如 . / 等等，所以这些字符都会变成下划线 _ 例如如果label是 app.kubernetes.io 就会转换成 __meta_kubernetes_pod_label_app_kubernetes_io根据上边的特性，我们如果要过滤出包含 app.kubernetes.io=test 这样的k8s label的pod，就可以⽤以下配置

```
scrape_configs:
- job_name: "job1"
  kubernetes_sd_configs: # 使⽤k8s服务发现
  - role: pod # 发现的是pod，还可以是node，endpoint等
  relabel_configs: # 可以设置多个匹配规则
  - action: keep # keep是保留匹配的，抛弃不匹配的，可写drop, drop就是抛弃满⾜条件的，留下其他的继续往下匹配
    source_labels:
    - __meta_kubernetes_pod_label_app_kubernetes_io # 对应k8s label是app.kubernetes.io
    regex: "test" # ⽬标label的⽬标值，⽀持正则
```

## 修改target的label

添加label

```
- job_name: "test"
  relabel_configs: # 可以设置多个匹配规则
  - action: replace # 从某个label中取值写⼊到另外⼀个label
    source_labels:
    - __meta_kubernetes_pod_name # ⽬标label，由于它是__开头，默认就不会被保留
    regex: (.*) # ⽤正则从⽬标label的值中取值，.* 表示取全部
    target_label: pod_name # 新label的名字，它不是__开头，就会被加到所有指标上
    replacement: ${1} # ${1} 是上边正则中第⼀个括号的值
```

上边例⼦的语义就是从 __meta_kubernetes_pod_name 的值中匹配出 (.*) 并设置到名为 pod_name 的新label中，label的值从正则匹配的值取

修改label

```
- job_name: "test"
  relabel_configs: # 可以设置多个匹配规则
  - action: replace # 从某个label中取值写⼊到另外⼀个label
  source_labels:
  - __meta_kubernetes_pod_ip # ⽬标label
  regex: (.*)
  target_label: "__address__" # 该值会被覆盖掉
  replacement: "${1}:8001" # 现在__address__就变成了 podIP:8001
```

上边例⼦的语义就是从 __meta_kubernetes_pod_ip 的值中匹配出 (.*) ，并将 __address__设置成匹配出的值:8001。

## 修改指标的label

修改指标label⽤ metric_relabel_configs 配置来做这个事，他的配置⽅法和 relabel_configs ⼀模⼀样，唯⼀区别就是metric_relabel_configs是针对指标的，⽽relabel_configs是针对Target的

```
scrape_configs:
- job_name: "job1"
  metrics_path: "/metrics" # 默认就是metrics
  metric_relabel_configs: # 主要和relabel_configs区分
  - action: keep
    source_labels:
    - __name__ # 指标的名字
    regex: "cpu_usage_total|mem_usage_bytes" # 使⽤了正则，只保留cpu_usage_total或mem_usage_bytes
```

## 如何检查target是否正常

可以通过up指标来检查target是否正常采集，1表示成功，0表示失败，up会带有Target经过relabel之后label集合，最常⽤的例如 job , instance 等等

```
up{job="xxx"}
```

如果up无法查询到，这就说明Target被过滤掉了，即采集配置⾥的 relabel_configs 配置不正确，Prometheus提供了⼀个接⼝⽤于获得所有被采集的Targets和被过滤的Targets，如果采集失败，还可以看到上次失败的原因。

被监控的Target会放在 activeTargets ⾥，被过滤掉的会放在 droppedTargets 

```
$ curl http://localhost:9090/api/v1/targets
{
Target信息是没法从Grafana上查看的，如果你能直接登录Prometheus原⽣的web⻚⾯，则可以在这⾥看到。
 "status": "success",
 "data": {
 "activeTargets": [
 {
 "discoveredLabels": {
 "__address__": "127.0.0.1:9090",
 "__metrics_path__": "/metrics",
 "__scheme__": "http",
 "job": "prometheus"
 },
 "labels": {
 "instance": "127.0.0.1:9090",
 "job": "prometheus"
 },
 "scrapePool": "prometheus",
 "scrapeUrl": "http://127.0.0.1:9090/metrics",
 "globalUrl": "http://example-prometheus:9090/metrics",
 "lastError": "", # 上次失败的原因在这⾥
 "lastScrape": "2017-01-17T15:07:44.723715405+01:00",
 "lastScrapeDuration": 0.050688943,
 "health": "up",
 "scrapeInterval": "1m",
 "scrapeTimeout": "10s"
 }
 ],
 "droppedTargets": [
 {
 "discoveredLabels": {
 "__address__": "127.0.0.1:9100",
 "__metrics_path__": "/metrics",
 "__scheme__": "http",
 "__scrape_interval__": "1m",
 "__scrape_timeout__": "10s",
 "job": "node"
 },
 }
 ]
 }
}
```

## 如何编写promql

### 基本概念

Instance vector表示⼀个Series集合，但是每个Series只有最近的⼀个点，⽽不是线。

Range vector表示⼀段时间范围⾥的Series，每个Series包含多个点，Range vector也可以表示成⼀张⼆维表，只不过需要把时间戳也加到表⾥，每⾏表示某个时间点下的某个Series。

Scalar不是⼀个Series，⽽是⼀个数值，例如直接在语句⾥写个100，这就是⼀个Scalar，Prometheus提供了⼀个函数，可以将只有⼀个Series的Instance vector转换成Scalar。

直接在查询输入指标名就是查询到Instant vector

```
container_cpu_usage_seconds_total
```

如果是加上时间范围，查询到的就是Range vector

```
container_cpu_usage_seconds_total{pod="cls-provisioner-66cf4d475-f4887"}[1m]
```

### 聚合查询

聚合就是针对查出来的Instant vector，Range vector做⼀些函数运算，得到新的Instant vector或者Scalar。

sum,avg,max等基本统计，这类函数先根据label将⼊参Instant vector进⾏分组，然后按 sum , avg 等⽅式统计组内Series，每个组输出⼀个新的Series，所有分组输出的Series合在⼀起重新构成⼀个新的Instant vector作为输出

```
sum(container_cpu_usage_seconds_total{})

没有指定⽤于分组的label，则所有Series分为⼀组，即最终输出的Instant vector也只有⼀个Series，即所有container_cpu_usage_seconds_total指标值的合

sum(container_cpu_usage_seconds_total{}) by(namespace)

按namespace这个label来分组，namespace⼀样的分到⼀起，即计算namespace相同的container_cpu_usage_seconds_total指标的合
```

irate, increase等Range统计，这类函数⼊参都是Range vector，即⼀段时间范围的点，他们都是根据统计类型计算每个Series在此段时间范围内所有的点并获得⼀个点，计算完后，每个Series只会剩下⼀个点，作为当前时间戳下的数值，即Range vector计算完后会变成Instant vector

```
irate(container_cpu_usage_seconds_total{}[1m])

还记得Range vector的查询⽅式吗，指标后边带上时间范围即可，由于这类函数都是针对每个Series计算的，不会对Series进⾏分组，所以不⽀持by这类分组关键字
```

多重聚合，所有聚合语句的结果，要吗是Instance vector，要吗是Scalar。所以，聚合的结果还可以作为参数输⼊给⼊参是Instance vector的函数如 sum 这类

```
avg( sum(container_cpu_usage_seconds_total{}) by(namespace) )

计算所有namespace的平均值

avg( irate(container_cpu_usage_seconds_total{}[1m]) )
```

需要注意的是，没法把聚合过的数据传给需要Range vector作为参数的函数，如irate，因为聚合后得到是Instance vector，下面的语句是报错的

```
irate(avg(container_cpu_usage_seconds_total{}))
```

对2个Instant vector进行聚合

Prometheus⽀持如 + , - , * , / , and , or 等

```
A * B
取A中每个Series，在B中找到和它所有Label（不含指标名）的key和value都相等的Series，把他两的值相乘，作为A中那个Series的值，如果没找到，则抛弃这个Series。
```

我们也可以指定只按某些label去匹配

```
A * on(pod,container) B
取A中每个Series，在B中找到和它pod, container这2个label都相等的Series，把他两的值相乘，作为A中那个Series的值，如果没找到，则抛弃这个Series。
```

上面的语句聚合出来的结果，label只有pod, container，其他label全没了，怎么才能保留A中的所有label呢，我们要加下group_left

```
A * on(pod,container) group_left B
取A中每个Series，在B中找到和它pod, container这2个label都相等的Series，把他两的值相乘，作为A中那个Series的值，如果没找到，则抛弃这个Series。结果的label集合采⽤左边A的
```

如果你希望把B中的⼀些label，附加到A原来的label集合⾥，⽤来扩展A原来的label集合，能做到吗？能！

```
A * on(pod,container) group_left(mylabel1, mylabel2) B
取A中每个Series，在B中找到和它pod, container这2个label都相等的Series，把他两的值相乘，作为A中那个Series的值，如果没找到，则抛弃这个Series。结果的label集合采⽤左边A的。不仅如此，还会从匹配的B中Series上复制出mylabel1,mylabel2追加到结果的label集合上。；
```

## label_replace和label_join

label_replace和label_join是PromQL的预置函数，支持将label的value进行截取和拼接，生成新的label。

值得注意的是，它们不改变源label的name及value，仅生成新label。

### label_replace

支持对某个label的值进行正则匹配，截取出某些值，生成新的label。

```
label_replace(v instant-vector, dst_label string, replacement string, src_label string, regex string)
```

示例如下

```
原始指标
node_network_transmit_bytes_total{container="kube-rbac-proxy",device="eth0",endpoint="https",instance="k8s-v22-1",job="node-exporter",namespace="monitoring",pod="node-exporter-nvj7r",service="node-exporter"}    9457895219

通过通过label_replace将device==>iface
label_replace(node_network_transmit_bytes_total{job="node-exporter", device="eth0"}, "iface", "$1", "device", "(.+)")

结果
node_network_transmit_bytes_total{container="kube-rbac-proxy",device="eth0",endpoint="https",iface="eth0",instance="k8s-v22-1",job="node-exporter",namespace="monitoring",pod="node-exporter-nvj7r",service="node-exporter"}    9465702581
```

### label_join

支持将某些label拼接起来，生成新的label。

```
label_join(v instant-vector, dst_label string, separator string, src_label_1 string, src_label_2 string, ...)
```

示例如下

```
原始指标
kubelet_node_name{endpoint="https-metrics",instance="178.104.163.63:10250",job="kubelet",metrics_path="/metrics",namespace="kube-system",node="k8s-v22-1",service="kubelet"}    1

通过label_join将instance和metrics_path拼接起来，生成新label: metrics_url
label_join(kubelet_node_name, "metrics_url", "", "instance", "metrics_path")

结果
kubelet_node_name{endpoint="https-metrics",instance="178.104.163.63:10250",job="kubelet",metrics_path="/metrics",metrics_url="178.104.163.63:10250/metrics",namespace="kube-system",node="k8s-v22-1",service="kubelet"}    1
```