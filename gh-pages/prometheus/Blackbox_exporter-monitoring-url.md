# Blackbox_exporter监控url

有的时候，我们需要对域名或者一些接口url进行检测，看下是否可以，我们可以部署Blackbox_exporter来采集这些监控数据到prometheus

Blackbox_exporter是Prometheus官方提供的exporter之一，可以提供 http、dns、tcp、icmp的监控数据采集，具体部署可以参考文档<https://cloud.tencent.com/developer/article/1808998>

## blackbox模块配置

下面我们说说怎么配置各种协议的探测，首先需要在blackbox的配置文件设置模块

```
modules:
  http_2xx:
    prober: http
    timeout: 15s
    http:
      prober: http
      timeout: 2s
      http:
        valid_http_versions: ["HTTP/1.1", "HTTP/2"]
        valid_status_codes: [200,301,302]
        method: GET
        preferred_ip_protocol: "ip4"
  tcp_connect:
    prober: tcp
  ssh:
    prober: tcp
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
  icmp:
    prober: icmp
    timeout: 5s
    icmp:
```

上面我们定义了4个模块，下面我们只需要在job中配置下模块的检查内容

## http测试job配置

```
scrape_configs:
- job_name: http-blackbox
  honor_timestamps: true
  params:
    module:
    - http_2xx
  metrics_path: /probe
  scheme: http
  static_configs:
  - targets:
    - https://www.niewx.cn/
    labels:
      domain: www.niewx.cn
      instance: https://www.niewx.cn/
      ip: githubpage
      port: "443"
      project: blog
      service: none
  - targets:
    - https://www.niewx.cn/mybook/
    labels:
      domain: www.niewx.cn
      instance: https://www.niewx.cn/mybook/
      ip: githubpage
      port: "443"
      project: mybook
      service: none
  relabel_configs:
  - source_labels: [__address__]
    separator: ;
    regex: (.*)
    target_label: __param_target
    replacement: $1
    action: replace
  - separator: ;
    regex: (.*)
    target_label: __address__
    replacement: blackbox-exporter.monitor.svc.cluster.local:9115
    action: replace
```

## tcp测试job配置

```
scrape_configs:
- job_name: tcp-blackbox
  honor_timestamps: true
  params:
    module:
    - tcp_connect
  metrics_path: /probe
  static_configs:
  - targets:
    - xxx.xxx.xxx.xxx:80
    labels:
      instance: xxx.xxx.xxx.xxx:80
      ip: xxx.xxx.xxx.xxx
      port: "80"
  - targets:
    - xxx.xxx.xxx.xxx:8080
    labels:
      instance: xxx.xxx.xxx.xxx:8080
      ip: xxx.xxx.xxx.xxx
      port: "8080"
  relabel_configs:
  - source_labels: [__address__]
    separator: ;
    regex: (.*)
    target_label: __param_target
    replacement: $1
    action: replace
  - separator: ;
    regex: (.*)
    target_label: __address__
    replacement: blackbox-exporter.monitor.svc.cluster.local:9115
    action: replace
```

## icmp测试job配置

```
scrape_configs:
- job_name: icmp-blackbox
  honor_timestamps: true
  params:
    module:
    - icmp
  metrics_path: /probe
  static_configs:
  - targets:
    - xx.xx.xx.xx
    labels:
      instance: xx.xx.xx.xx
      ip: xx.xx.xx.xx
  - targets:
    - xxx.xxx.xxx.xxx
    labels:
      instance: xxx.xxx.xxx.xxx
      ip: xxx.xxx.xxx.xxx
  relabel_configs:
  - source_labels: [__address__]
    separator: ;
    regex: (.*)
    target_label: __param_target
    replacement: $1
    action: replace
  - separator: ;
    regex: (.*)
    target_label: __address__
    replacement: blackbox-exporter.monitor.svc.cluster.local:9115
    action: replace
```

配置好之后，数据就可以采集到prometheus了，然后配置下grafana面板查看我们的监控信息即可。

## 问题

问题：当http探测域名的是时候发现面板的连通性是离线，但是http的状态却是200

解决方案：可以将http模块中配置的协议HTTP/2改成HTTP/2.0，然后再重启blackbox即可

