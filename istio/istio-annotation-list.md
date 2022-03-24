# istio注解列表

注解加在workload的spec.template.metadata.annotations字段下

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      annotations:
        sidecar.istio.io/status: xxxxx
```

Istio支持控制其行为的各种资源注释列表如下


|Annotation Name|	Resource Types|	Description|
|---|---|---|
|kubernetes.io/ingress.class|	[Ingress]	|Ingress 资源上的注释，表示负责它的控制器类。|
|networking.istio.io/exportTo|	[Service]|	指定此服务应导出到的命名空间。 '*' 值表示它可以在网格 '.' 内到达。表示它在其命名空间内是可访问的。|
|policy.istio.io/check|	[pod]	|确定无法连接到 Mixer 时的行为策略。如果未设置，则设置 FAIL_CLOSE，拒绝请求。|
|policy.istio.io/checkBaseRetryWaitTime|	[pod]	|重试之间等待的基本时间，将通过退避和抖动进行调整。以持续时间格式。如果未设置，则为 80 毫秒。|
|policy.istio.io/checkMaxRetryWaitTime|	[pod]	|重试 Mixer 之间等待的最长时间。以持续时间格式。如果未设置，则为 1000 毫秒。 |
|policy.istio.io/checkRetries	|[pod]	|传输错误到 Mixer 的最大重试次数。如果未设置，则为 0，表示不重试。|
|policy.istio.io/lang	|[pod]	|为 Mixer 选择属性表达式语言运行时。|
|readiness.status.sidecar.istio.io/applicationPorts|	[pod]	|指定应用容器暴露的端口列表。由 Envoy sidecar 就绪探测器使用，以确定 Envoy 已配置并准备好接收流量。|
|readiness.status.sidecar.istio.io/failureThreshold	|[pod]	|指定 Envoy sidecar 就绪探测的失败阈值。|
|readiness.status.sidecar.istio.io/initialDelaySeconds	|[pod]	|指定 Envoy sidecar 就绪探测的初始延迟（以秒为单位）。|
|readiness.status.sidecar.istio.io/periodSeconds	|[pod]	|指定 Envoy sidecar 就绪探测的周期（以秒为单位）。|
|sidecar.istio.io/bootstrapOverride|	[pod]	|指定一个替代的 Envoy 引导程序配置文件。|
|sidecar.istio.io/componentLogLevel|	[pod]	|指定 Envoy 的组件日志级别。|
|sidecar.istio.io/controlPlaneAuthPolicy|	[pod]	|指定 Istio 控制平面使用的身份验证策略。如果为 NONE，则不会加密流量。如果是 MUTUAL_TLS，Envoy sidecar 之间的流量将被包装到相互的 TLS 连接中。|
|sidecar.istio.io/discoveryAddress|	[pod]	|指定 Envoy sidecar 使用的 XDS 发现地址。|
|sidecar.istio.io/inject	|[pod]	|指定 Envoy sidecar 是否应该自动注入到工作负载中。|
|sidecar.istio.io/interceptionMode	|[pod]|	指定用于将入站连接重定向到 Envoy（REDIRECT 或 TPROXY）的模式。|
|sidecar.istio.io/logLevel|	[pod]|	指定 Envoy 的日志级别。|
|sidecar.istio.io/proxyCPU|	[pod]	|为 Envoy sidecar 指定请求的 CPU 设置。|
|sidecar.istio.io/proxyCPULimit|	[pod]	|为 Envoy sidecar 指定limit的 CPU 设置。|
|sidecar.istio.io/proxyImage|	[pod]	|指定 Envoy sidecar 使用的 Docker 镜像。|
|sidecar.istio.io/proxyMemory	|[pod]	|指定 Envoy sidecar 请求的内存设置。|
|sidecar.istio.io/proxyMemoryLimit	|[pod]	|指定 Envoy sidecar limit的内存设置。|
|sidecar.istio.io/rewriteAppHTTPProbers	|[pod]|	重写 HTTP 准备和活动探测器以重定向到 Envoy sidecar。|
|sidecar.istio.io/statsInclusionPrefixes|	[pod]	|指定由 Envoy 发出的统计信息前缀的逗号分隔列表。|
|sidecar.istio.io/statsInclusionRegexps|	[pod]|	指定要由 Envoy 发出的统计信息应匹配的逗号分隔的正则表达式列表。|
|sidecar.istio.io/statsInclusionSuffixes|	[pod]	|指定由 Envoy 发出的统计数据后缀的逗号分隔列表。|
|sidecar.istio.io/status	|[pod]	|由 Envoy sidecar 注入生成，指示操作的状态。包括执行模板的版本哈希，以及注入资源的名称。|
|sidecar.istio.io/userVolume|	[pod]	|指定要添加到 Envoy sidecar 的一个或多个用户卷（作为 JSON 数组）。|
|sidecar.istio.io/userVolumeMount|	[pod]	|指定要添加到 Envoy sidecar 的一个或多个用户卷挂载（作为 JSON 数组）。|
|status.sidecar.istio.io/port	|[pod]	|指定 Envoy sidecar 的 HTTP 状态端口。如果为零，sidecar 将不提供状态。|
|traffic.sidecar.istio.io/excludeInboundPorts	|[pod]	|一个逗号分隔的入站端口列表，要从重定向到 Envoy 中排除。仅当所有入站流量（即“*”）被重定向时才适用。|
|traffic.sidecar.istio.io/excludeOutboundIPRanges	|[pod]	|以逗号分隔的 CIDR 格式的 IP 范围列表，要从重定向中排除。仅当所有出站流量（即“*”）都被重定向时才适用。|
|traffic.sidecar.istio.io/excludeOutboundPorts|	[pod]	|一个逗号分隔的出站端口列表，要从重定向到 Envoy 中排除。|
|traffic.sidecar.istio.io/includeInboundPorts	|[pod]|	以逗号分隔的入站端口列表，其流量将被重定向到 Envoy。通配符“*”可用于为所有端口配置重定向。空列表将禁用所有入站重定向。|
|traffic.sidecar.istio.io/includeOutboundIPRanges	|[pod]	|以逗号分隔的 CIDR 形式的 IP 范围列表，用于重定向到 Envoy（可选）。通配符“*”可用于重定向所有出站流量。空列表将禁用所有出站重定向。|
|traffic.sidecar.istio.io/kubevirtInterfaces|	[pod]|	以逗号分隔的虚拟接口列表，其入站流量（来自 VM）将被视为出站流量。 |