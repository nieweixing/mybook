# 问题

当我们的一些服务往istio上迁移的时候，会出现一个问题，就是某些依赖数据库的服务会一直起不了，pod启动失败，这里排查原因是envoy容器还没起来，服务容器就起来了，导致业务流量无法被转发出去，从而连接数据库异常。 


# 解决方案

这里解决问题的方案就是保证envoy这个sidecar容器先于业务容器启动，那么怎么保证sidecar容器先于业务容器启动呢？

## istio1.7之后版本解决方案

istio1.7通过给istio-injector注入逻辑增加一个叫HoldApplicationUntilProxyStarts的开关来解决了该问题

> 添加了 values.global.proxy.holdApplicationUntilProxyStarts config选项，它使sidecar注入器在pod容器列表的开始处注入sidecar，并将其配置为阻止所有其他容器的开始，直到代理就绪为止。默认情况下禁用此选项。(#11130)

也就是说只要开启个这个特性，那么就可以保证pod里的sidecar容器内先于业务容器启动，这样就不会出现pod内容器启动顺序的问题

这里我们可以全局开启这个特性和单独给某个deployment开启这个特性

### 全局开启HoldApplicationUntilProxyStarts

全局开启只需要修改istiod的全局配置即可

```
kubectl edit cm -n istio-system istio-1-8-1
```

在defaultConfig字段下加上holdApplicationUntilProxyStarts: true

```
apiVersion: v1
data:
  mesh: |
    accessLogEncoding: JSON
    accessLogFile: /dev/stdout
    accessLogFormat: ""
    defaultConfig:
      holdApplicationUntilProxyStarts: true
      discoveryAddress: istiod-1-8-1.istio-system.svc:15012
```

### 局部开启HoldApplicationUntilProxyStarts

如果单独给某个deployment开启这个特性，需要在pod的注解加上proxy.istio.io/config，将 holdApplicationUntilProxyStarts 置为 true

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
        proxy.istio.io/config: |
          holdApplicationUntilProxyStarts: true
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: "nginx"
```

## istio1.7版本之前的解决方案

如果是istio1.7之前的版本，是没有这个特性的，那么需要采用另外一种方案来解决这个问题，我们在业务容器启动前判断下envoy服务是否已经启动成功了

```
command: ["/bin/bash", "-c"]
args: ["while [[ \"$(curl -s -o /dev/null -w ''%{http_code}'' localhost:15020/healthz/ready)\" != '200' ]]; do echo Waiting for Sidecar;sleep 1; done; echo Sidecar available; start-app-cmd"]
```

这里直接在deployment加上启动命令，如果探测envoy服务的15020端口返回200，则启动业务进程，这样可以保证sidecar先比业务容器启动
