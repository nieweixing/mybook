# prometheus常用告警promsql

k8s中，告警监控我们都是通过prometheus加上alertmanager来实现的，在alertmanager里面通过写promsql来进行告警的设置，然后发生到不同的告警渠道，下面我们来说说常用告警promsql配置。

## master告警配置

证书过期告警，表示证书将在24小时后到期

```
apiserver_client_certificate_expiration_seconds_count{job="apiserver"} > 0 and on(job) histogram_quantile(0.01, sum by (job, le) (rate(apiserver_client_certificate_expiration_seconds_bucket{job="apiserver"}[5m]))) < 86400
```

apiserver状态异常告警

```
sum(up{job="kube-apiserver"}) == 0
```

kube-scheduler状态异常告警

```
sum(up{job="kube-scheduler"}) == 0
```

kube-controller-manager状态异常告警

```
sum(up{job="kube-controller-manager"}) == 0
```

## 节点告警配置

节点状态异常告警

```
kube_node_status_condition{condition="Ready",status="true"} == 0
```

节点存在MemoryPressure告警

```
kube_node_status_condition{condition="MemoryPressure",status="true"} == 1
```

节点存在DiskPressure告警

```
kube_node_status_condition{condition="DiskPressure",status="true"} == 1
```

节点存在OutOfDisk告警

```
kube_node_status_condition{condition="OutOfDisk",status="true"} == 1
```

节点pod数量超过节点容纳最大pod数量的90%告警

```
sum by (node) ((kube_pod_status_phase{phase="Running"} == 1) + on(uid) group_left(node) (0 * kube_pod_info{pod_template_hash=""})) / sum by (node) (kube_node_status_allocatable{resource="pods"}) * 100 > 90
```

节点的时钟同步异常告警

```	
min_over_time(node_timex_sync_status[5m]) == 0 and node_timex_maxerror_seconds >= 16
```

节点的Conntrack超过上限设置的75%告警

```
(node_nf_conntrack_entries / node_nf_conntrack_entries_limit) > 0.75
```

节点磁盘空间不足告警

```
( node_filesystem_avail_bytes{job="node-exporter",fstype!=""} / node_filesystem_size_bytes{job="node-exporter",fstype!=""} * 100 < 3 and node_filesystem_readonly{job="node-exporter",fstype!=""} == 0 )
```

## pod告警配置

pod的cpu占limit使用率超过80%

```
sum(rate(container_cpu_usage_seconds_total{job="cadvisor", image!="", container!="POD"}[1m])) by (namespace, pod, container) / sum(kube_pod_container_resource_limits_cpu_cores) by (namespace, pod, container) > 0.8
```

pod的cpu占request使用率超过80%

```
sum(rate(container_cpu_usage_seconds_total{job="cadvisor", image!="", container!="POD"}[1m])) by (namespace, pod, container) / sum(kube_pod_container_resource_requests_cpu_cores) by (namespace, pod, container) > 0.8
```

pod的内存占limit使用率超过80%

```
sum(rate(container_memory_working_set_bytes{job="cadvisor", image!="", container!="POD"}[1m])) by (namespace, pod, container) / sum(kube_pod_container_resource_limits_memory_bytes) by (namespace, pod, container) > 0.8
```

pod的内存占request使用率超过80%

```
sum(rate(container_memory_working_set_bytes{job="cadvisor", image!="", container!="POD"}[1m])) by (namespace, pod, container) / sum(kube_pod_container_resource_requests_memory_bytes) by (namespace, pod, container) > 0.8
```

## workload告警配置

deployment的副本数不匹配

```
kube_deployment_spec_replicas != kube_deployment_status_replicas_available
```

StatefulSet和期望实例副本数不匹配

```
kube_statefulset_status_replicas_ready != kube_statefulset_status_replicas
```

负载5分钟内容器重启次数

```
rate(kube_pod_container_status_restarts_total{job="kube-state-metrics"}[5m]) * 60 * 5 > 0
```

job执行失败

```	
kube_job_failed{job="kube-state-metrics"} > 0
```

hpa的副本数和预期不一致告警

```
(kube_hpa_status_desired_replicas{job="kube-state-metrics"} != kube_hpa_status_current_replicas{job="kube-state-metrics"}) and changes(kube_hpa_status_current_replicas[15m]) == 0
```

hpa副本数达到最大值告警

```
kube_hpa_status_current_replicas{job="kube-state-metrics"} == kube_hpa_spec_max_replicas{job="kube-state-metrics"}
```

## 存储告警配置

pv磁盘剩余可用空间小于3%

```
kubelet_volume_stats_available_bytes{job="kubelet"} / kubelet_volume_stats_capacity_bytes{job="kubelet"} < 0.03
```

预测卷的磁盘空间是否4天后用尽

```
( kubelet_volume_stats_available_bytes{job="kubelet"} / kubelet_volume_stats_capacity_bytes{job="kubelet"} ) < 0.15 and predict_linear(kubelet_volume_stats_available_bytes{job="kubelet"}[6h], 4 * 24 * 3600) < 0
```

pv状态异常告警

```
kube_persistentvolume_status_phase{phase=~"Failed|Pending",job="kube-state-metrics"} > 0
```
