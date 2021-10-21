# istio注解列表

|Annotation Name|	Resource Types|	Description|
|---|---|---|
|kubernetes.io/ingress.class|	[Ingress]	|入口资源上的注释表示负责该资源的控制器类别。|
|networking.istio.io/exportTo|	[Service]|	指定应导出此服务的名称空间。"*"值表示可在网格内到达。|
|policy.istio.io/check|	[pod]	|当无法连接到 Mixer 时，确定行为策略。如果不设置，则设置FAIL_CLOSE，拒绝请求。|
|policy.istio.io/checkBaseRetryWaitTime|	[pod]	|在视网膜之间等待的基数时间，将通过回退和抖动来调整。在持续时间格式中。如果不设置，这将是80ms。|
|policy.istio.io/checkMaxRetryWaitTime|	[pod]	|在重述到混合器之间等待的最大时间。在持续时间格式中。如果不设置，这将是1000ms。|
|policy.istio.io/checkRetries	|[pod]	|混合器中运输错误的最大重述次数。如果不设置，这将是 0，表示没有重述。|
|policy.istio.io/lang	|[pod]	|为 Mixer 选择属性表达语言运行时间。|
|readiness.status.sidecar.istio.io/applicationPorts|	[pod]	|指定应用容器暴露的港口列表。由特使sidecar准备调查，以确定特使是配置和准备接收交通。|
|readiness.status.sidecar.istio.io/failureThreshold	|[pod]	|指定特使sidecar准备调查的故障阈值。|
|readiness.status.sidecar.istio.io/initialDelaySeconds	|[pod]	|指定特使sidecar准备调查的初始延迟（秒内）。|
|readiness.status.sidecar.istio.io/periodSeconds	|[pod]	|指定特使sidecar准备调查的期间（秒内）。|
|sidecar.istio.io/bootstrapOverride|	[pod]	|指定替代特使引导配置文件。|
|sidecar.istio.io/componentLogLevel|	[pod]	|指定特使的组件日志级别。|
|sidecar.istio.io/controlPlaneAuthPolicy|	[pod]	|指定 Istio 控制平面使用的 auth 策略。如果没有，流量将无法加密。如果MUTUAL_TLS，特使sidecar之间的交通将包裹成相互的 TLS 连接。|
|sidecar.istio.io/discoveryAddress|	[pod]	|指定特使sidecar使用的 XDS 发现地址。|
|sidecar.istio.io/inject	|[pod]	|具体说明是否应自动将特使sidecar注入工作量。|
|sidecar.istio.io/interceptionMode	|[pod]|	指定用于将入站连接重定向到特使（重定向或 TPROXY）的模式。|
|sidecar.istio.io/logLevel|	[pod]|	指定特使的日志级别。|
|sidecar.istio.io/proxyCPU|	[pod]	|指定特使sidecar的请求 CPU 设置。|
|sidecar.istio.io/proxyImage|	[pod]	|指定特使sidecar使用的码头图像。|
|sidecar.istio.io/proxyMemory	|[pod]	|指定特使sidecar的所要求记忆设置。|
|sidecar.istio.io/rewriteAppHTTPProbers	|[pod]|	重写 HTTP 准备和活性探头，重定向到特使sidecar。|
|sidecar.istio.io/statsInclusionPrefixes|	[pod]	|指定特使将要发出的统计数据的前缀逗号分离列表。|
|sidecar.istio.io/statsInclusionRegexps|	[pod]|	指定由特使发出的统计数据应匹配的逗号分离列表。|
|sidecar.istio.io/statsInclusionSuffixes|	[pod]	|指定特使将发出的统计数据的后缀逗号分离列表。|
|sidecar.istio.io/status	|[pod]	|由特使sidecar注射产生，指示操作状态。包括已执行模板的版本哈希，以及注入资源的名称。|
|sidecar.istio.io/userVolume|	[pod]	|指定一个或多个用户体积（作为 JSON 阵列）添加到特使sidecar。|
|sidecar.istio.io/userVolumeMount|	[pod]	|指定一个或多个用户体积安装（作为 JSON 阵列）添加到特使sidecar。|
|status.sidecar.istio.io/port	|[pod]	|指定特使sidecar的 HTTP 状态端口。如果为零，副车将无法提供状态。|
|traffic.sidecar.istio.io/excludeInboundPorts	|[pod]	|将排除重定向到特使的入境端口的逗号分离列表。仅适用于所有进站流量（即"*"）被重定向时。|
|traffic.sidecar.istio.io/excludeOutboundIPRanges	|[pod]	|CIDR 形式的 IP 范围逗号分离列表，以排除重定向。仅适用于所有出站流量（即"*"）被重定向时。|
|traffic.sidecar.istio.io/excludeOutboundPorts|	[pod]	|将排除重定向到特使的出站端口逗号列表。|
|traffic.sidecar.istio.io/includeInboundPorts	|[pod]|	将流量重定向到特使的入境港口逗号分离列表。通配符字符"*"可用于配置所有端口的重定向。空列表将禁用所有入站重定向。|
|traffic.sidecar.istio.io/includeOutboundIPRanges	|[pod]	|以 CIDR 形式将 IP 范围逗成的逗姆形分离列表，以重定向到特使（可选）。通配符字符"*"可用于重定向所有出站流量。空列表将禁用所有出站重定向。|
|traffic.sidecar.istio.io/kubevirtInterfaces|	[pod]|	其入站流量（来自 VM）将被视为出站的虚拟界面逗号分离列表。|