# 如何修改sidecar资源配置

istio的sidecar自动注入是依赖于一个istio-sidecar-injector的mutatingwebhookconfigurations，一般自动注入的sidecar容器配置是哪里设置的呢？其实是在一个istio-sidecar-injector的configmap里面，如何istio-proxy容器资源占用大小的默认配置不符合要求时，可按照实际需求进行修改，下面我们来说说如何配置。


## 修改单个sidecar资源配置

如何你只想调整某个服务的sidecar容器资源配置，可以通过注解的方式来进行配置，单个工作负载注解配置的优先级会大于全局配置。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tke-debug
  namespace: default
  labels:
    app: debug
spec:
  replicas: 1
  selector:
    matchLabels:
      app: debug
  template:
    metadata:
      labels:
        app: debug
      annotations:
        sidecar.istio.io/proxyCPU: 100m
        sidecar.istio.io/proxyCPULimit: 1
        sidecar.istio.io/proxyMemory: 100mi
        sidecar.istio.io/proxyMemoryLimit: 1Gi
    spec:
      containers:
      - name: debug
        image: ccr.ccs.tencentyun.com/nwx_registry/troubleshooting:latest
      tolerations:
      - operator: Exists
      dnsPolicy: ClusterFirst
```

## 修改全局的sidecar资源配置

如果你是希望修改全局的sidecar资源配置设置，需要改下istio-sidecar-injector这个configmap，如果你是用云厂商托管的istio，控制面是接触不到，一般需要提工单让后端修改。如果是自建的istio或者独立的网格，可以自行修改。

```
# kubectl edit configmap istio-sidecar-injector -n istio-system
```

然后在configmap找到这个字段

```
          containers:
          - name: istio-proxy
```

这个容器找下Values.global.proxy.resources这个配置

```
{- else }
  {- if .Values.global.proxy.resources }
    { toYaml .Values.global.proxy.resources | indent 6 }
  {- end }
```

修改方式有2中，可以直接修改value对应的值，修改resources即可

```
values: |-
  {
    "global": {
      "caAddress": "",
      "configValidation": true,
      "defaultNodeSelector": {},
      "imagePullPolicy": "",
      "imagePullSecrets": [],
      "istioNamespace": "istio-system",
      "istiod": {
        "enableAnalysis": false
      },
      "jwtPolicy": "first-party-jwt",
      "priorityClassName": "",
      "proxy": {
        "autoInject": "enabled",
        "clusterDomain": "cluster.local",
        "componentLogLevel": "misc:error",
        "enableCoreDump": false,
        "excludeIPRanges": "",
        "logLevel": "warning",
        "privileged": false,
        "readinessFailureThreshold": 30,
        "readinessInitialDelaySeconds": 1,
        "readinessPeriodSeconds": 2,
        "resources": {
          "limits": {
            "cpu": "2000m",
            "memory": "1024Mi"
          },
          "requests": {
            "cpu": "100m",
            "memory": "128Mi"
          }
        },
```

还有就是直接配置resource配置，不通过变量引用的方式。


```
{- else }
  {- if .Values.global.proxy.resources }
    #{ toYaml .Values.global.proxy.resources | indent 6 }
    requests:
      cpu: 100m
      memory: 12Mi
    limits:
      cpu: 1
      memory: 1Gi
  {- end }
```
