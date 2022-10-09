# TKE监控指标说明文档

## 集群组件监控

### 数据源名称(measurement)

* k8s_component
  
### 维度集合(labels)

* tke_cluster_instance_id 属于哪个集群
* node 哪个节点上的组件
* node_role 节点的角色 Node; Master; Etcd; Master,Etcd
* unInstanceId 所在node的unInstanceId
* appid 属于哪个用户
* region 属于哪个地域
  
### 指标集合(metrics)

**k8s_component_apiserver_ready**
* kube-apiserver是否正常

```
max(label_replace(up{job="apiserver"}, "node","$1", "instance","(.*):60002")) by (node) * on(node) group_left(node_role,unInstanceId) kube_node_labels
```
**k8s_component_etcd_ready**
* etcd是否正常

```
max(label_replace(up{job="etcd"}, "node","$1", "instance","(.*):2379")) by (node) * on(node) group_left(node_role,unInstanceId) kube_node_labels
```

**k8s_component_scheduler_ready**
* kube-scheduler是否正常

```
max(label_replace(up{job="scheduler"}, "node","$1", "instance","(.*):10251")) by (node) * on(node) group_left(node_role,unInstanceId) kube_node_labels
```

**k8s_component_controller_manager_ready**
* kube-controller-manager是否正常

```
max(label_replace(up{job="controller-manager"}, "node","$1", "instance","(.*):10252")) by (node) * on(node) group_left(node_role,unInstanceId) kube_node_labels
```

## 容器级别

### 数据源名称(measurement)

* k8s_container

### 维度集合(labels)

* container_name 容器的名字
* pod_name 属于哪个pod
* namespace 属于哪个namespace
* node 属于哪个node
* unInstanceId 所在node的unInstanceId
* node_role 节点的角色 Node; Master; Etcd; Master,Etcd
* workload_kind 所属的工作负载类型
* workload_name 属于哪个工作负载
* tke_cluster_instance_id 属于哪个集群
* appid 属于哪个用户
* region 属于哪个地域


### 指标集合(metrics)

**container_cpu_usage_seconds_total**
* 含义：单个容器实例每秒使用的cpu时间累加值
：kubelet
* 注意：这个指标是原始指标，不会上报
  
**k8s_container_cpu_core_used**
* 含义：单个容器2分钟内的平均cpu核数

```
rate(container_cpu_usage_seconds_total[5m]) * on(namespace, pod_name) group_left(workload_kind ,workload_name ,node,node_role,unInstanceId) __pod_info2
```

**k8s_container_rate_cpu_core_used_request**
* 含义：container使用的核数占request的比例

```
k8s_container_cpu_core_used * 100 / on (pod_name,namespace,container_name) group_left kube_pod_container_resource_requests{resource="cpu"}
```

**k8s_container_rate_cpu_core_used_limit**
* 含义：container使用的核数占limit的比例

```
k8s_container_cpu_core_used * 100 / on (pod_name,namespace,container_name) group_left kube_pod_container_resource_limits{resource="cpu"}
```

**k8s_container_rate_cpu_core_used_node**
* 含义：container使用的核数占node的比例

```
k8s_container_cpu_core_used * 100 / on(node) group_left kube_node_status_capacity_cpu_cores
```

**container_memory_usage_bytes**
* 含义：单个容器的内存使用量
：kubelet
* 注意：这个指标是原始指标，不会上报
  
**k8s_container_mem_usage_bytes**
* 含义：单个容器的内存使用量

```
container_memory_usage_bytes * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_mem_no_cache_bytes**
* 含义：不包含cache的内存使用量

```
(container_memory_usage_bytes - container_memory_cache) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_rate_mem_usage_request**
* 含义：container内存使用占request的比例

```
k8s_container_mem_usage_bytes * 100 / on (pod_name,namespace,container_name) group_left kube_pod_container_resource_requests{resource="memory"}
```

**k8s_container_rate_mem_no_cache_request**
* 含义：container内存使用占request的比例，不含cache

```
k8s_container_mem_no_cache_bytes * 100 / on (pod_name,namespace,container_name) group_left kube_pod_container_resource_requests{resource="memory"}
```

**k8s_container_rate_mem_usage_limit**
* 含义：container内存使用占limit的

```
k8s_container_mem_usage_bytes * 100 / on (pod_name,namespace,container_name) group_left kube_pod_container_resource_limits{resource="memory"}
```

**k8s_container_rate_mem_no_cache_limit**
* 含义：container内存使用占limit的比例，不含cache

```
k8s_container_mem_no_cache_bytes * 100 / on (pod_name,namespace,container_name) group_left kube_pod_container_resource_limits{resource="memory"}
```

**k8s_container_rate_mem_usage_node**
* 含义：container内存使用占node的比例

```
k8s_container_mem_usage_bytes * 100 / on(node) group_left kube_node_status_capacity_memory_bytes
```

**k8s_container_rate_mem_no_cache_node**
* 含义：container内存使用占node的比例,不含cache

```
k8s_container_mem_no_cache_bytes * 100 / on(node) group_left kube_node_status_capacity_memory_bytes
```

**k8s_container_network_receive_bytes_bw**
* 含义：网络入带宽，单位：bps

```
sum(rate(container_network_receive_bytes_total[5m])) without(interface) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_network_transmit_bytes_bw**
* 含义：网络出带宽，单位：bps

```
sum(rate(container_network_transmit_bytes_total[5m])) without(interface) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_network_receive_packets**
* 含义：网络入包量 (个/s)

```
sum(rate(container_network_receive_packets_total[5m])) without(interface) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_network_transmit_packets**
* 含义：网络出包量 (个/s)

```
sum(rate(container_network_transmit_packets_total[5m])) without(interface) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_network_receive_bytes**
* 含义：网络入流量，单位：byte

```
sum(idelta(container_network_receive_bytes_total[5m])) without(interface) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_network_transmit_bytes**
* 含义：网络出流量，单位：byte

```
sum(idelta(container_network_transmit_bytes_total[5m])) without(interface) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_fs_read_times**
* 含义：块设备读取次数(个/s)

```
sum(rate(container_fs_reads_total[5m])) without(device) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_fs_write_times**
* 含义：块设备写次数(个/s)

```
sum(rate(container_fs_writes_total[5m])) without(device) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_fs_read_bytes**
* 含义：块设备读取带宽(B/s)

```
sum(rate(container_fs_reads_bytes_total[5m])) without(device) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_fs_write_bytes**
* 含义：块设备写入带宽(B/s)

```
sum(rate(container_fs_writes_bytes_total[5m])) without(device) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_container_gpu_used**
* 含义：单个容器gpu使用值,单位：CUDA Core， 一张gpu卡=100单位CUDA Core

```
container_gpu_utilization{gpu="total"} * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role) __pod_info2
```

**k8s_container_rate_gpu_used_request**
* 含义：container使用的gpu占request的比例，单位：百分比，示例：0.46表示46%

```
k8s_container_gpu_used / on (pod_name,namespace,container_name) group_left container_request_gpu_utilization
```

**k8s_container_rate_gpu_used_node**
* 含义：container使用的gpu占node的比例，单位：百分比，示例：0.46表示46%

```
k8s_container_gpu_used * 100 / on(node) group_left kube_node_status_capacity_gpu
```

**k8s_container_gpu_memory_used_bytes**
* 含义：单个pod gpu memory使用量，单位：byte

```
container_gpu_memory_total{gpu_memory="total"} * 1024 * 1024 * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role) __pod_info2
```

**k8s_container_rate_gpu_memory_used_request**
* 含义：container使用的gpu memory占request的比例，单位：百分比，示例：0.46表示46%

```
k8s_container_gpu_memory_used_bytes * 100 / on (pod_name,namespace,container_name) group_left() (container_request_gpu_memory * 1024 * 1024)
```

**k8s_container_rate_gpu_memory_used_node**
* 含义：container使用的gpu memory占node的比例，单位：百分比，示例：0.46表示46%

```
k8s_container_gpu_memory_used_bytes * 100 / on(node) group_left() kube_node_status_capacity_gpu_memory_bytes
```

## pod级别

### 数据源名称(measurement)

* k8s_pod

### 维度集合(labels)

* pod_name 属于哪个pod
* node 属于哪个node
* namespace 属于哪个namespace
* unInstanceId 所在node的unInstanceId
* node_role 节点的角色 Node; Master; Etcd; Master,Etcd
* workload_kind 所属的工作负载类型
* workload_kind 所属的工作负载类型
* workload_name 属于哪个工作负载
* tke_cluster_instance_id 属于哪个集群
* appid 属于哪个用户
* region 属于哪个地域
* 
### 指标集合(metrics)

**__pod_info1**
* 含义：用于补tag的指标，不会上报

```
kube_pod_info* on(node) group_left(node_role,unInstanceId) kube_node_labels
```

**__pod_info2**
* 含义：用于补tag的指标，不会上报

```
label_replace(label_replace(__pod_info1{workload_kind="ReplicaSet"} * on (workload_name,namespace) group_left(owner_name, owner_kind) label_replace(kube_replicaset_owner,"workload_name","$1","replicaset","(.*)"),"workload_name","$1","owner_name","(.*)"),"workload_kind","$1","owner_kind","(.*)") or on(pod_name,namesapce) __pod_info1{workload_kind != "ReplicaSet"}
```

**k8s_pod_cpu_core_used**
* 含义：单个pod 5分钟内的平均cpu核数，单位：core
  
```
sum(k8s_container_cpu_core_used) without (container_name,container_id)
```

**k8s_pod_rate_cpu_core_used_request**
* 含义：pod使用的核数占request的比例，单位：百分比

```
sum(k8s_container_cpu_core_used + on (container_name, pod_name, namespace) group_left kube_pod_container_resource_requests{resource="cpu"} * 0) without(container_name,container_id) * 100 / on (pod_name,namespace) group_left sum(kube_pod_container_resource_requests{resource="cpu"}) without(container_name,container_id)
```

**k8s_pod_rate_cpu_core_used_limit**
* 含义：pod使用的核数占limit的比例，单位：百分比

```
sum(k8s_container_cpu_core_used + on (container_name, pod_name, namespace) group_left kube_pod_container_resource_limits{resource="cpu"} * 0) without(container_name,container_id) * 100 / on (pod_name,namespace) group_left sum(kube_pod_container_resource_limits{resource="cpu"}) without(container_name,container_id)
```

**k8s_pod_rate_cpu_core_used_node**
* 含义：pod使用的核数占node的比例，单位：百分比

```
k8s_pod_cpu_core_used *100 / on(node) group_left kube_node_status_capacity_cpu_cores
```

**k8s_pod_mem_usage_bytes**
* 含义：pod内存使用量，单位：byte

```
sum(k8s_container_mem_usage_bytes) without (container_name,container_id)
```

**k8s_pod_mem_no_cache_bytes**
* 含义：pod内存使用，不包含cache，单位：byte

```
sum(k8s_container_mem_no_cache_bytes) without (container_name,container_id)
```

**k8s_pod_rate_mem_usage_request**
* 含义：pod内存使用占request的比例，单位：百分比

```
sum(k8s_container_mem_usage_bytes + on (container_name, pod_name, namespace) group_left kube_pod_container_resource_requests{resource="memory"} * 0) without(container_name,container_id) * 100 / on (pod_name,namespace) group_left sum(kube_pod_container_resource_requests{resource="memory"}) without(container_name,container_id)
```

**k8s_pod_rate_mem_no_cache_request**
* 含义：pod内存使用占request的比例，不含cache，单位：百分比

```
sum(k8s_container_mem_no_cache_bytes + on (container_name, pod_name, namespace) group_left kube_pod_container_resource_requests{resource="memory"} * 0) without(container_name,container_id) * 100 / on (pod_name,namespace) group_left sum(kube_pod_container_resource_requests{resource="memory"}) without(container_name,container_id)
```

**k8s_pod_rate_mem_usage_limit**
* 含义：pod内存使用占limit的，单位：百分比

```
sum(k8s_container_mem_usage_bytes + on (container_name, pod_name, namespace) group_left kube_pod_container_resource_limits{resource="memory"} * 0) without(container_name,container_id) * 100 / on (pod_name,namespace) group_left sum(kube_pod_container_resource_limits{resource="memory"}) without(container_name,container_id)
```

**k8s_pod_rate_mem_no_cache_limit**
* 含义：pod内存使用占limit的比例，不含cache，单位：byte

```
sum(k8s_container_mem_no_cache_bytes + on (container_name, pod_name, namespace) group_left kube_pod_container_resource_limits{resource="memory"} * 0) without(container_name,container_id) * 100 / on (pod_name,namespace) group_left sum(kube_pod_container_resource_limits{resource="memory"}) without(container_name,container_id)
```

**k8s_pod_rate_mem_usage_node**
* 含义：pod内存使用占node的比例，单位：百分比

```
k8s_pod_mem_usage_bytes * 100 / on(node) group_left kube_node_status_capacity_memory_bytes
```

**k8s_pod_rate_mem_no_cache_node**
* 含义：pod内存使用占node的比例,不含cache，单位：byte

```
k8s_pod_mem_no_cache_bytes * 100 / on(node) group_left kube_node_status_capacity_memory_bytes
```

**k8s_pod_network_receive_bytes_bw**
* 含义：pod网络入带宽，单位： bps

```
sum(k8s_container_network_receive_bytes_bw) without (container_name,container_id)
```

**k8s_pod_network_transmit_bytes_bw**
* 含义：pod网络出带宽, 单位：bps

```
sum(k8s_container_network_transmit_bytes_bw) without (container_name,container_id)
```

**k8s_pod_network_receive_packets**
* 含义：pod网络入包量 (个/s)

```
sum(k8s_container_network_receive_packets) without (container_name,container_id)
```

**k8s_pod_network_transmit_packets**
* 含义：pod网络出包量 (个/s)

```
sum(k8s_container_network_transmit_packets) without (container_name,container_id)
```

**k8s_pod_network_receive_bytes**
* 含义：pod网络入流量，单位：byte

```
sum(k8s_container_network_receive_bytes) without (container_name,container_id)
```

**k8s_pod_network_transmit_bytes**
* 含义：pod网络出流量，单位：byte

```
sum(k8s_container_network_transmit_bytes) without (container_name,container_id)
```

**k8s_pod_fs_read_times**
* 含义：块设备读取次数(个/s)

```
sum(k8s_container_fs_read_times) without (container_name,container_id)
```

**k8s_pod_fs_write_times**
* 含义：块设备写次数(个/s)

```
sum(k8s_container_fs_write_times) without (container_name,container_id)
```

**k8s_pod_fs_read_bytes**
* 含义：块设备读取带框(B/s)

```
sum(k8s_container_fs_read_bytes) without (container_name,container_id)
```

**k8s_pod_fs_write_bytes**
* 含义：块设备写入带框(B/s)

```
sum(k8s_container_fs_write_bytes) without (container_name,container_id)
```

**k8s_pod_status_ready**
* 含义：pod是否处于ready状态

```
sum(kube_pod_status_ready{condition="true"}) by (namespace,pod_name) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, label_kubernetes_io_node_role_etcd, label_node_role_kubernetes_io_master,unInstanceId) __pod_info2
```

**k8s_pod_restart_total**
* 含义：pod重启次数

```
sum(idelta(kube_pod_container_status_restarts_total [5m])) by (namespace,pod_name) * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role,unInstanceId) __pod_info2
```

**k8s_pod_gpu_used**
* 含义：单个pod gpu使用量，单位：CUDA Core， 一张gpu卡=100单位CUDA Core

```
sum(k8s_container_gpu_used) without (container_name,container_id)
```

**k8s_pod_gpu_request**
* 含义：单个pod gpu申请量，单位：CUDA Core， 一张gpu卡=100单位CUDA Core

```
sum(container_request_gpu_utilization * 100 * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role, unInstanceId) __pod_info2) without(container_name,container_id)
```

**k8s_pod_rate_gpu_used_request**
* 含义：pod使用gpu占request的比例，单位：百分比，示例：0.46表示46%

```
sum(k8s_container_gpu_used + on (container_name, pod_name, namespace) group_left container_request_gpu_utilization * 0) without(container_name,container_id) * 100 / on (pod_name,namespace) group_left k8s_pod_gpu_request
```

**k8s_pod_rate_gpu_used_node**
* 含义：pod使用gpu占node的比例，单位：百分比，示例：0.46表示46%

```
k8s_pod_gpu_used * 100 / on(node) group_left kube_node_status_capacity_gpu
```

**k8s_pod_gpu_memory_used_bytes**
* 含义：单个pod gpu memory使用量，单位：byte

```
sum(k8s_container_gpu_memory_used_bytes) without (container_name,container_id)
```

**k8s_pod_gpu_memory_request_bytes**
* 含义：单个pod gpu memory申请量，单位：byte

```
sum(container_request_gpu_memory * 1024 * 1024 * on(namespace, pod_name) group_left(workload_kind,workload_name,node, node_role, unInstanceId) __pod_info2) without (container_name,container_id)
```

**k8s_pod_rate_gpu_memory_used_request**
* 含义：pod使用gpu memory占request的比例，单位：百分比，示例：0.46表示46%

```
sum(k8s_container_gpu_memory_used_bytes + on (container_name, pod_name, namespace) group_left container_request_gpu_memory * 0) without(container_name,container_id) * 100 / on (pod_name,namespace) group_left k8s_pod_gpu_memory_request_bytes
```

**k8s_pod_rate_gpu_memory_used_node**
* 含义：pod使用gpu memory占node的比例，单位：百分比，示例：0.46表示46%

```
k8s_pod_gpu_memory_used_bytes * 100 / on(node) group_left() kube_node_status_capacity_gpu_memory_bytes
```

## node 级别

### 数据源名称(measurement)

* k8s_node

### 维度集合(labels)

* node 名称
* unInstanceId 所在node的unInstanceId
* node_role
* tke_cluster_instance_id 属于哪个集群
* appid 属于哪个用户
* region 属于哪个地域
  
### 指标集合(metrics)

**kube_node_labels**
* 含义：用于补tag的指标，不会上报
* 来源：从kube-state-metrics采集
  
**k8s_node_status_ready**
* 含义：node是否正常

```
max(kube_node_status_condition{condition="Ready", status="true"} * on (node) group_left(label_kubernetes_io_node_role_etcd ,label_node_role_kubernetes_io_master ,unInstanceId ) kube_node_labels) without(condition, status)
```

**k8s_node_pod_restart_total**
* 含义：node上pod重启次数

```
sum(k8s_pod_restart_total) without(pod_name)
```

**k8s_node_memory_request_bytes_total**
* 含义：node的memory分配量，单位：byte

```
sum(kube_pod_container_resource_requests{resource="memory"} * on(node) group_left(node_role ,unInstanceId) kube_node_labels) without (container_name,namespace,pod_name)
```

**k8s_node_cpu_core_request_total**
* 含义：node的cpu分配量，单位：byte

```
sum(kube_pod_container_resource_requests{resource="cpu"} * on(node) group_left(node_role ,unInstanceId) kube_node_labels) without (container_name,namespace,pod_name)
```

**k8s_node_cpu_usage**
* 含义：node的cpu利用率
* 来源：来至barad
  
**k8s_node_mem_usage**
* 含义：node的内存使用率
* 来源：来至barad


**k8s_node_lan_intraffic**
* 含义：内网入带宽（bps）
* 来源：来至barad


**k8s_node_lan_outtraffic**
* 含义：内网出带宽（bps）
* 来源：来至barad


**k8s_node_wan_intraffic**
* 含义：外网入带宽（bps）
* 来源：来至barad


**k8s_node_wan_outtraffic**
* 含义：外网出带宽（bps）
* 来源：来至barad


**k8s_node_tcp_curr_estab**
* 含义：TCP连接数（个）
* 来源：来至barad


**kube_node_status_capacity_gpu**
* 含义：node的gpu容量，单位：CUDA Core， 一张gpu卡=100单位CUDA Core，该指标不上报

```
sum by(node) (kube_node_status_capacity{resource="tencent_com_vcuda_core"})
```

**kube_node_status_capacity_gpu_memory_bytes**
* 含义：node的gpu memory容量，单位：byte，该指标不上报

```
sum by(node) (kube_node_status_capacity{resource="tencent_com_vcuda_memory"} * 256 * 1024 * 1024)
```

**kube_node_status_allocatable_gpu**
* 含义：node的gpu可用core，单位：CUDA Core， 一张gpu卡=100单位CUDA Core，该指标不上报

```
sum by(node) (kube_node_status_allocatable{resource="tencent_com_vcuda_core"})
```

**kube_node_status_allocatable_gpu_memory_bytes**
* 含义：node的gpu可用memory，单位：byte，该指标不上报

```
sum by(node) (kube_node_status_allocatable{resource="tencent_com_vcuda_memory"} * 256 * 1024 * 1024)
```

**k8s_node_gpu_used**
* 含义：node的gpu使用量，单位：CUDA Core， 一张gpu卡=100单位CUDA Core

```
sum(k8s_pod_gpu_used) by (node)
```

**k8s_node_gpu_memory_used_bytes**
* 含义：node的gpu memory使用量，单位：byte

```
sum(k8s_pod_gpu_memory_used_bytes) by (node)
```

**k8s_node_rate_gpu_used**
* 含义：node的gpu使用率，单位：百分比，示例：0.46表示46%

```
sum(k8s_pod_gpu_used) without(namespace,pod_name,workload_kind,workload_name) *100 / on(node) group_left kube_node_status_capacity_gpu
```

**k8s_node_rate_gpu_memory_used**
* 含义：node的gpu memory使用率，单位：百分比，示例：0.46表示46%

```
sum(k8s_pod_gpu_memory_used_bytes) without(namespace,pod_name,workload_kind,workload_name) *100 / on(node) group_left() kube_node_status_capacity_gpu_memory_bytes
```

## workload级别

### 数据源名称(measurement)

* k8s_workload
  
### 维度集合(labels)

* namespace 属于哪个namespace
* workload_kind 负载类型
* workload_name 工作负载名称
* tke_cluster_instance_id 属于哪个集群
* appid 属于哪个用户
* region 属于哪个地域

### 指标集合(metrics)

**k8s_workload_abnormal**
* 含义：workload是否是不正常状态，非0就是不正常

```
max(label_replace(
label_replace(
label_replace(
kube_deployment_status_replicas_unavailable,
"workload_kind","Deployment","","")
,"workload_name","$1","deployment","(.*)"),
"name", "k8s_workload_abnormal", "name","(.*)")
)
by (namespace, workload_name, workload_kind,name)
or on (namespace,workload_name,workload_kind, name)
max(label_replace(
label_replace(
label_replace(
kube_daemonset_status_number_unavailable,
"workload_kind","Daemonset","","")
,"workload_name","$1","daemonset","(.*)"),
"name", "k8s_workload_abnormal", "name","(.*)") ) by (namespace, workload_name, workload_kind,name)
or on (namespace,workload_name,workload_kind, name)
max(label_replace(
label_replace(
label_replace(
(kube_statefulset_replicas - kube_statefulset_status_replicas_ready),
"workload_kind","Statefulset","","")
,"workload_name","$1","statefulset","(.*)"),
"name", "k8s_workload_abnormal", "name","(.*)") ) by (namespace, workload_name, workload_kind,name)
or on (namespace,workload_name,workload_kind, name)
max(label_replace(
label_replace(
label_replace(
(kube_job_status_failed),
"workload_kind","Job","","")
,"workload_name","$1","job_name","(.*)"),
"name", "k8s_workload_abnormal", "name","(.*)") ) by (namespace, workload_name, workload_kind,name)
or on (namespace,workload_name,workload_kind, name)
max(label_replace(
label_replace(
label_replace(
(kube_cronjob_info * 0),
"workload_kind","CronJob","","")
,"workload_name","","cronjob","(.*)"),
"name", "k8s_workload_abnormal", "name","(.*)") ) by (namespace, workload_name, workload_kind,name)
```

**k8s_workload_pod_restart_total**
* 含义：workload上pod重启次数

```
sum(k8s_pod_restart_total) by(namespace,workload_kind,workload_name)
```

**k8s_workload_cpu_core_used**
* 含义：工作负载5分钟内的平均cpu核数

```
sum(k8s_pod_cpu_core_used) by(workload_name, workload_kind, namespace)
```

**k8s_workload_rate_cpu_core_used_cluster**
* 含义：工作负载所使用的核数占集群总核数的比例

```
k8s_workload_cpu_core_used * 100 / scalar(k8s_cluster_cpu_core_total)
```

**k8s_workload_mem_usage_bytes**
* 含义：工作负载所使用的内存量

```
sum(k8s_pod_mem_usage_bytes) by(workload_name, workload_kind, namespace)
```

**k8s_workload_mem_no_cache_bytes**
* 含义：工作负载所使用的内存占集群总内存的大小，不包含cache

```
sum(k8s_pod_mem_no_cache_bytes) by(workload_name, workload_kind, namespace)
```

**k8s_workload_rate_mem_usage_bytes_cluster**
* 含义：工作负载所使用的内存占集群总内存的大小

```
k8s_workload_mem_usage_bytes * 100 / scalar(k8s_cluster_memory_total)
```

**k8s_workload_rate_mem_no_cache_cluster**
* 含义：工作负载所使用的内存占集群总内存的大小，不含cache

```
k8s_workload_mem_no_cache_bytes * 100 / scalar(k8s_cluster_memory_total)
```

**k8s_workload_network_receive_bytes_bw**
* 含义：workload网络入带宽，单位：bps

```
sum(k8s_pod_network_receive_bytes_bw) by(workload_name, workload_kind, namespace)
```

**k8s_workload_network_transmit_bytes_bw**
* 含义：workload网络出带宽，单位：bps

```
sum(k8s_pod_network_transmit_bytes_bw) by(workload_name, workload_kind, namespace)
```

**k8s_workload_network_receive_packets**
* 含义：workload网络入包量 (个/s)

```
sum(k8s_pod_network_receive_packets) by(workload_name, workload_kind, namespace)
```

**k8s_workload_network_transmit_packets**
* 含义：workload网络出包量 (个/s)

```
sum(k8s_pod_network_transmit_packets) by(workload_name, workload_kind, namespace)
```

**k8s_workload_network_receive_bytes**
* 含义：workload网络入流量，单位：byte

```
sum(k8s_pod_network_receive_bytes) by(workload_name, workload_kind, namespace)
```

**k8s_workload_network_transmit_bytes**
* 含义：workload网络出流量，单位：byte

```
sum(k8s_pod_network_transmit_bytes) by(workload_name, workload_kind, namespace)
```

**k8s_workload_fs_read_times**
* 含义：块设备读取次数(个/s)

```
sum(k8s_pod_fs_read_times)by (workload_name, workload_kind, namespace)
```

**k8s_workload_fs_write_times**
* 含义：块设备写次数(个/s)

```
sum(k8s_pod_fs_write_times) by (workload_name, workload_kind, namespace)
```

**k8s_workload_fs_read_bytes**
* 含义：块设备读取次数(个/s)

```
sum(k8s_pod_fs_read_bytes)by (workload_name, workload_kind, namespace)
```

**k8s_workload_fs_write_bytes**
* 含义：块设备读取次数(个/s)

```
sum(k8s_pod_fs_write_bytes) by (workload_name, workload_kind, namespace)
```

**k8s_workload_gpu_used**
* 含义：工作负载gpu使用量，单位：CUDA Core， 一张gpu卡=100单位CUDA Core

```
sum(k8s_pod_gpu_used) by(workload_name, workload_kind, namespace)
```

**k8s_workload_rate_gpu_used_cluster**
* 含义：工作负载gpu使用量占集群总量的比例，单位：百分比，示例：0.46表示46%

```
k8s_workload_gpu_used * 100 / scalar(k8s_cluster_gpu_total)
```

**k8s_workload_gpu_memory_used_bytes**
* 含义：工作负载gpu memory使用量，单位：byte

```
sum(k8s_pod_gpu_memory_used_bytes) by(workload_name, workload_kind, namespace)
```

**k8s_workload_rate_gpu_memory_used_cluster**
* 含义：工作负载gpu memory使用量占集群总量的比例，单位：百分比，示例：0.46表示46%

```
k8s_workload_gpu_memory_used_bytes * 100 / scalar(k8s_cluster_gpu_memory_total_bytes)
```

## namespace级别

### 数据源名称(measurement)

* k8s_namespace

### 维度集合(labels)

* namespace 属于哪个namespace
* tke_cluster_instance_id 属于哪个集群
* appid 属于哪个用户
* region 属于哪个地域
  
### 指标集合(metrics)

**k8s_namespace_cpu_core_used**
* 含义：namespace 5分钟内的平均cpu核数

```
sum(k8s_pod_cpu_core_used) by (namespace)
```

**k8s_namespace_rate_cpu_core_used_cluster**
* 含义：命名空间所使用的核数占集群总核数的比例

```
k8s_namespace_cpu_core_used * 100 / scalar(k8s_cluster_cpu_core_total)
```

**k8s_namespace_mem_usage_bytes**
* 含义：namespace内存使用量，单位：byte

```
sum(k8s_pod_mem_usage_bytes) by (namespace)
```

**k8s_namespace_mem_no_cache_bytes**
* 含义：namespace 内存使用量，不包含cache

```
sum(k8s_pod_mem_no_cache_bytes) by (namespace)
```

**k8s_namespace_rate_mem_usage_bytes_cluster**
* 含义：namespace 内存使用量占集群的比例

```
k8s_namespace_mem_usage_bytes * 100 / scalar(k8s_cluster_memory_total)
```

**k8s_namespace_rate_mem_no_cache_cluster**
* 含义：namespace 内存使用量占集群的比例，不包含cache

```
k8s_namespace_mem_no_cache_bytes * 100 / scalar(k8s_cluster_memory_total)
```

**k8s_namespace_network_receive_bytes_bw**
* 含义：workload网络入带宽：单位：bps

```
sum(k8s_pod_network_receive_bytes_bw) by(namespace)
```

**k8s_namespace_network_transmit_bytes_bw**
* 含义：workload网络出带宽：单位：bps

```
sum(k8s_pod_network_transmit_bytes_bw) by(namespace)
```

**k8s_namespace_network_receive_packets**
* 含义：workload网络入包量 (个/s)

```
sum(k8s_pod_network_receive_packets) by(namespace)
```

**k8s_namespace_network_transmit_packets**
* 含义：workload网络出包量 (个/s)

```
sum(k8s_pod_network_transmit_packets) by(namespace)
```

**k8s_namespace_network_receive_bytes**
* 含义：workload网络入流量，单位：byte

```
sum(k8s_pod_network_receive_bytes) by(namespace)
```

**k8s_namespace_network_transmit_bytes**
* 含义：workload网络出流量，单位：byte

```
sum(k8s_pod_network_transmit_bytes) by(namespace)
```

**k8s_namespace_fs_read_times**
* 含义：块设备读取次数(个/s)

```
sum(k8s_pod_fs_read_times) by (namespace)
```

**k8s_namespace_fs_write_times**
* 含义：块设备写次数(个/s)

```
sum(k8s_pod_fs_write_times) by (namespace)
```

**k8s_namespace_fs_read_bytes**
* 含义：块设备读取带宽(B/S)

```
sum(k8s_pod_fs_read_bytes) by (namespace)
```

**k8s_namespace_fs_write_bytes**
* 含义：块设备写入带宽(B/S)

```
sum(k8s_pod_fs_write_bytes) by (namespace)
```

**k8s_namespace_gpu_used**
* 含义：namespace gpu使用量，单位：CUDA Core， 一张gpu卡=100单位CUDA Core

```
sum(k8s_pod_gpu_used) by (namespace)
```

**k8s_namespace_rate_gpu_used_cluster**
* 含义：命名空间所使用的gpu占集群总数的比例，单位：百分比，示例：0.46表示46%

```
k8s_namespace_gpu_used * 100 / scalar(k8s_cluster_gpu_total)
```

**k8s_namespace_gpu_memory_used_bytes**
* 含义：namespace gpu memory使用量，单位：byte

```
sum(k8s_pod_gpu_memory_used_bytes) by (namespace)
```

**k8s_namespace_rate_gpu_memory_used_cluster**
* 含义：命名空间所使用的gpu memory占集群总数的比例，单位：百分比，示例：0.46表示46%

```
k8s_namespace_gpu_memory_used_bytes * 100 / scalar(k8s_cluster_gpu_memory_total_bytes)
```

## cluster级别

### 数据源名称(measurement)

* k8s_cluster

### 维度集合(labels)

* tke_cluster_instance_id 属于哪个集群
* appid 属于哪个用户
* region 属于哪个地域

### 指标集合(metrics)

**k8s_cluster_etcd_db_total_size_bytes**
* 含义：集群etcd的db大小，只有独立集群有

```
avg(etcd_debugging_mvcc_db_total_size_in_bytes)
```

**k8s_cluster_cpu_core_total**
* 含义：集群cpu核数总和

```
sum(kube_node_status_allocatable_cpu_cores * on(node) group_left kube_node_labels {node_role="Node"})
```

**k8s_cluster_memory_total**
* 含义：集群内存总和

```
sum(kube_node_status_allocatable_memory_bytes * on(node) group_left kube_node_labels {node_role="Node"})
```

**k8s_cluster_cpu_core_used**
* 含义：集群cpu核数使用量

```
sum(k8s_pod_cpu_core_used{node_role="Node"})
```

**k8s_cluster_rate_cpu_core_used_cluster**
* 含义：集群cpu核数占集群总核数的比例

```
k8s_cluster_cpu_core_used * 100 / scalar(k8s_cluster_cpu_core_total)
```

**k8s_cluster_rate_cpu_core_request_cluster**
* 含义：集群cpu分配率

```
sum(kube_pod_container_resource_requests{resource="cpu"} * on(node) group_left kube_node_labels {node_role="Node"} * on(pod_name, namespace) group_left sum(kube_pod_status_phase{phase!~"Succeeded|Failed"}==1) by (pod_name, namespace)) * 100 / scalar(k8s_cluster_cpu_core_total)
```

**k8s_cluster_mem_usage_bytes**
* 含义：集群内存使用量

```
sum(k8s_pod_mem_usage_bytes{node_role="Node"})
```

**k8s_cluster_mem_no_cache_bytes**
* 含义：集群内存使用量

```
sum(k8s_pod_mem_no_cache_bytes{node_role="Node"})
```

**k8s_cluster_rate_mem_usage_bytes_cluster**
* 含义：集群内存使用量占集群总资源的比例

```
k8s_cluster_mem_usage_bytes * 100 / scalar(k8s_cluster_memory_total)
```

**k8s_cluster_rate_mem_no_cache_bytes_cluster**
* 含义：集群内存使用量占集群总资源的比例，不含cache

```
k8s_cluster_mem_no_cache_bytes * 100 / scalar(k8s_cluster_memory_total)
```

**k8s_cluster_rate_mem_request_bytes_cluster**
* 含义：集群内存使用量占集群总资源的比例

```
sum(kube_pod_container_resource_requests{resource="memory"} * on(node) group_left kube_node_labels {node_role="Node"} * on(pod_name, namespace) group_left sum(kube_pod_status_phase{phase!~"Succeeded|Failed"}==1) by (pod_name, namespace)) * 100 / scalar(k8s_cluster_memory_total)
```

**k8s_cluster_network_receive_bytes_bw**
* 含义：集群网络入带宽，单位：bps

```
sum(k8s_pod_network_receive_bytes_bw{node_role="Node"})
```

**k8s_cluster_network_transmit_bytes_bw**
* 含义：集群网络出带宽，单位：bps

```
sum(k8s_pod_network_transmit_bytes_bw{node_role="Node"})
```

**k8s_cluster_network_receive_packets**
* 含义：集群网络入包量 (个/s)

```
sum(k8s_pod_network_receive_packets{node_role="Node"})
```

**k8s_cluster_network_transmit_packets**
* 含义：集群网络出包量 (个/s)

```
sum(k8s_pod_network_transmit_packets{node_role="Node"})
```

**k8s_cluster_network_receive_bytes**
* 含义：集群网络入流量，单位：byte

```
sum(k8s_pod_network_receive_bytes{node_role="Node"})
```

**k8s_cluster_network_transmit_bytes**
* 含义：集群网络出流量，单位：byte

```
sum(k8s_pod_network_transmit_bytes{node_role="Node"})
```

**k8s_cluster_fs_read_times**
* 含义：块设备读取次数(个/s)

```
sum(k8s_pod_fs_read_times{node_role="Node"})
```

**k8s_cluster_fs_write_times**
* 含义：块设备写次数(个/s)

```
sum(k8s_pod_fs_write_times{node_role="Node"})
```

**k8s_cluster_fs_read_bytes**
* 含义：块设备读取次数(个/s)

```
sum(k8s_pod_fs_read_bytes{node_role="Node"})
```

**k8s_cluster_fs_write_bytes**
* 含义：块设备读取次数(个/s)

```
sum(k8s_pod_fs_write_bytes{node_role="Node"})
```

**k8s_cluster_gpu_total**
* 含义：集群gpu总量，单位：CUDA Core， 一张gpu卡=100单位CUDA Core
  
```
sum(kube_node_status_allocatable_gpu * on(node) group_left kube_node_labels {node_role="Node"})
```

**k8s_cluster_gpu_memory_total_bytes**
* 含义：集群gpu memory总k8s_cluster_gpu_total量，单位：byte

```
sum(kube_node_status_allocatable_gpu_memory_bytes * on(node) group_left kube_node_labels {node_role="Node"})
```

**k8s_cluster_gpu_used**
* 含义：集群gpu使用量，单位：CUDA Core， 一张gpu卡=100单位CUDA Core

```
sum(k8s_pod_gpu_used{node_role="Node"})
```

**k8s_cluster_rate_gpu_used_cluster**
* 含义：集群gpu使用量占集群总数的比例，单位：百分比，示例：0.46表示46%

```
k8s_cluster_gpu_used * 100 / scalar(k8s_cluster_gpu_total)
```

**k8s_cluster_rate_gpu_request_cluster**
* 含义：集群gpu分配率，单位：百分比，示例：0.46表示46%

```
sum(k8s_pod_gpu_request * on(node) group_left kube_node_labels {node_role="Node"} * on(pod_name, namespace) group_left sum(kube_pod_status_phase{phase!~"Succeeded|Failed"}==1) by (pod_name, namespace)) * 100 / scalar(k8s_cluster_gpu_total)
```

**k8s_cluster_gpu_memory_used_bytes**
* 含义：集群gpu memory使用量，单位：byte

```
sum(k8s_pod_gpu_memory_used_bytes{node_role="Node"})
```

**k8s_cluster_rate_gpu_memory_used_cluster**
* 含义：集群gpu memory使用量占集群总数的比例，单位：百分比，示例：0.46表示46%

```
k8s_cluster_gpu_memory_used_bytes * 100 / scalar(k8s_cluster_gpu_memory_total_bytes)
```

**k8s_cluster_rate_gpu_memory_request_cluster**
* 含义：集群gpu memory分配率，单位：百分比，示例：0.46表示46%

```
sum(k8s_pod_gpu_memory_request_bytes * on(node) group_left kube_node_labels {node_role="Node"} * on(pod_name, namespace) group_left sum(kube_pod_status_phase{phase!~"Succeeded|Failed"}==1) by (pod_name, namespace)) * 100 / scalar(k8s_cluster_gpu_memory_total_bytes)
```